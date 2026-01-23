# 大模型图像与视频生成评估指标全面技术指南

**图像与视频生成质量评估已从单一的FID指标演变为多维度评估体系**。传统指标如FID虽仍是论文中最常引用的基准，但其高斯分布假设与样本偏差问题日益受到质疑。2024年以来，VBench、CMMD和ImageReward等新范式正在重塑评估标准——**CMMD在Stable Diffusion迭代过程中表现出与人类判断更高的一致性，而VBench的16维度评估框架已成为视频生成的事实标准**。本报告系统梳理所有主流评估指标的数学原理、代码实现与最新应用。

---

## 一、基于分布距离的图像评估指标

### FID：行业标准但存在根本性缺陷

Fréchet Inception Distance由Heusel等人于2017年提出，通过比较真实图像与生成图像在Inception V3特征空间中的分布距离来评估生成质量。其核心假设是两组图像的特征服从多元高斯分布。

**数学公式推导**：设真实图像特征分布为 $\mathcal{N}(\mu_r, \Sigma_r)$，生成图像特征分布为 $\mathcal{N}(\mu_g, \Sigma_g)$，则Fréchet距离（2-Wasserstein距离）为：

$$\text{FID} = ||\mu_r - \mu_g||_2^2 + \text{Tr}\left(\Sigma_r + \Sigma_g - 2(\Sigma_r \Sigma_g)^{1/2}\right)$$

其中 $\mu$ 为**2048维均值向量**（来自Inception V3的pool3层），$\Sigma$ 为协方差矩阵，$\text{Tr}(\cdot)$ 表示矩阵的迹。选择pool3层是因为该层位于分类层之前，既包含高级语义信息又保留了足够的空间特征。

**关键参数与实践要求**：

- 样本数量：**至少10,000-50,000张图像**才能获得稳定估计，2000样本时偏差显著
- 特征维度：默认2048维，也可选择64/192/768维（但与视觉质量相关性下降）
- 图像预处理：需resize至299×299，像素值归一化

**PyTorch实现（pytorch-fid库）**：

```python
# 命令行使用
python -m pytorch_fid path/to/real_images path/to/generated_images --device cuda:0

# torchmetrics代码实现
import torch
from torchmetrics.image.fid import FrechetInceptionDistance

fid = FrechetInceptionDistance(feature=2048)
# 注意：输入需要是uint8格式，范围0-255
real_images = torch.randint(0, 255, (10000, 3, 299, 299), dtype=torch.uint8)
fake_images = torch.randint(0, 255, (10000, 3, 299, 299), dtype=torch.uint8)

fid.update(real_images, real=True)
fid.update(fake_images, real=False)
fid_score = fid.compute()  # 越低越好，0表示完全相同分布
```

**FID的根本性缺陷**（CVPR 2024 CMMD论文揭示）：

|缺陷类型|具体表现|
|---|---|
|高斯假设不成立|Inception激活值约2%为精确零值（ReLU导致），统计检验p值接近零|
|有偏估计|偏差随样本量和模型特性变化，小样本时尤为严重|
|预处理敏感|不同库的bicubic缩放实现可导致**FID差异≥6**|
|域外泛化差|在人脸等非ImageNet类别上敏感度显著降低|

**Clean-FID改进方案**（CVPR 2022）解决了预处理不一致问题：

```python
from cleanfid import fid

# 使用正确的PIL-bicubic resize（带抗锯齿）
score = fid.compute_fid(fdir1, fdir2, mode="clean")

# 使用CLIP特征替代Inception
score = fid.compute_fid(fdir1, fdir2, mode="clean", model_name="clip_vit_b_32")

# 使用预计算的数据集统计
score = fid.compute_fid(gen_path, dataset_name="FFHQ", dataset_res=1024, 
                         dataset_split="trainval70k", mode="clean")
```

### IS：多样性与质量的联合度量

Inception Score由Salimans等人于2016年提出，基于条件概率分布与边缘分布之间的KL散度。

**数学公式**：

$$\text{IS} = \exp\left(\mathbb{E}_{x \sim p_{\text{gen}}}[D_{KL}(p(y|x) | p(y))]\right)$$

等价的熵形式为：

$$\ln \text{IS} = H[\mathbb{E}_x[p(y|x)]] - \mathbb{E}_x[H[p(y|x)]]$$

其中 $p(y|x)$ 是单张图像通过Inception V3得到的1000类概率分布，$p(y) = \mathbb{E}_x[p(y|x)]$ 是所有图像的平均分布。

**核心解释**：高IS同时意味着（1）单张图像的类别预测高度确定（**高质量**）和（2）整体生成图像覆盖多种类别（**高多样性**）。理论最高分为1000（ImageNet类别数），实践中优秀GAN模型得分约30-50。

**局限性**：不考虑真实数据分布、对模式坍塌不敏感、严重依赖ImageNet类别。

```python
from torchmetrics.image.inception import InceptionScore

inception = InceptionScore()
imgs = torch.randint(0, 255, (50000, 3, 299, 299), dtype=torch.uint8)
inception.update(imgs)
is_mean, is_std = inception.compute()  # 返回均值和标准差
```

### KID：无偏估计的小样本友好指标

Kernel Inception Distance由Bińkowski等人于2018年提出，使用核方法替代高斯假设，基于Maximum Mean Discrepancy（MMD）理论。

**数学原理**：MMD度量两个分布在再生核希尔伯特空间（RKHS）中均值嵌入的距离：

$$\text{MMD}^2(P, Q) = \mathbb{E}[k(x,x')] + \mathbb{E}[k(y,y')] - 2\mathbb{E}[k(x,y)]$$

KID使用多项式核 $k(x,y) = \left(\frac{1}{d}x^Ty + 1\right)^3$，其中 $d=2048$ 为特征维度。

**无偏估计器**（移除对角线项避免自相似偏差）：

$$\widehat{\text{MMD}}^2_u = \frac{1}{n(n-1)}\sum_{i \neq j}k(x_i, x_j) + \frac{1}{m(m-1)}\sum_{i \neq j}k(y_i, y_j) - \frac{2}{nm}\sum_{i,j}k(x_i, y_j)$$

**KID vs FID对比**：

|特性|FID|KID|
|---|---|---|
|分布假设|高斯分布|无参数假设|
|估计偏差|有偏|**无偏**|
|推荐样本量|10,000-50,000|**1,000-5,000**|
|方差估计|困难|简单（子集采样）|

```python
from torchmetrics.image.kid import KernelInceptionDistance

kid = KernelInceptionDistance(feature=2048, subsets=100, subset_size=1000)
kid.update(real_imgs, real=True)
kid.update(fake_imgs, real=False)
kid_mean, kid_std = kid.compute()  # 返回均值和标准差
```

---

## 二、感知与结构相似性指标

### SSIM：基于人类视觉系统的结构评估

Structural Similarity Index由Wang等人于2004年提出，将图像质量分解为亮度、对比度和结构三个分量。

**完整公式推导**：

亮度比较：$l(x,y) = \frac{2\mu_x\mu_y + C_1}{\mu_x^2 + \mu_y^2 + C_1}$

对比度比较：$c(x,y) = \frac{2\sigma_x\sigma_y + C_2}{\sigma_x^2 + \sigma_y^2 + C_2}$

结构比较：$s(x,y) = \frac{\sigma_{xy} + C_3}{\sigma_x\sigma_y + C_3}$

**综合SSIM公式**（当 $\alpha=\beta=\gamma=1$，$C_3=C_2/2$）：

$$\text{SSIM}(x,y) = \frac{(2\mu_x\mu_y + C_1)(2\sigma_{xy} + C_2)}{(\mu_x^2 + \mu_y^2 + C_1)(\sigma_x^2 + \sigma_y^2 + C_2)}$$

**常数选择**：$C_1 = (0.01 \times L)^2$，$C_2 = (0.03 \times L)^2$，其中 $L=255$（8-bit图像）。使用**11×11高斯窗口**（$\sigma=1.5$）逐像素滑动计算。

**MS-SSIM（多尺度版本）** 在5个尺度上加权融合，权重为 $(0.0448, 0.2856, 0.3001, 0.2363, 0.1333)$，与人类感知相关性更高。

```python
from torchmetrics.image import StructuralSimilarityIndexMeasure, MultiScaleStructuralSimilarityIndexMeasure

ssim = StructuralSimilarityIndexMeasure(data_range=1.0)  # 范围[-1,1]
ms_ssim = MultiScaleStructuralSimilarityIndexMeasure(data_range=1.0)

score = ssim(predicted, target)      # 输出范围[-1, 1]，1表示完全相同
ms_score = ms_ssim(predicted, target)
```

### PSNR：简单但与感知脱节

Peak Signal-to-Noise Ratio是最简单的图像质量指标，基于MSE计算：

$$\text{PSNR} = 10 \cdot \log_{10}\left(\frac{\text{MAX}^2}{\text{MSE}}\right) = 20 \cdot \log_{10}\left(\frac{\text{MAX}}{\sqrt{\text{MSE}}}\right)$$

其中 $\text{MSE} = \frac{1}{mn}\sum_{i,j}[I(i,j) - K(i,j)]^2$，$\text{MAX}=255$（8-bit图像）。

**数值解读**：8-bit图像典型范围30-50 dB，>40 dB通常认为是高质量。**核心缺陷**是像素独立假设完全忽略了空间结构信息，与人类感知相关性差。

### LPIPS：深度特征的感知距离

Learned Perceptual Image Patch Similarity由Zhang等人于2018年CVPR提出，核心发现是**深度网络内部激活作为感知相似度度量表现惊人地好**。

**数学公式**：

$$d(x, x_0) = \sum_l \frac{1}{H_l W_l} \sum_{h,w} |w_l \odot (\hat{y}_{hw}^l - \hat{y}_{0hw}^l)|_2^2$$

其中 $\hat{y}^l$ 是第 $l$ 层归一化后的特征图，$w_l \in \mathbb{R}^{C_l}$ 是每个通道的可学习权重。

**网络选择**：

- **AlexNet**：默认选择，前向度量最佳，5层特征
- **VGG**：适合反向传播优化（如训练生成模型）
- **SqueezeNet**：轻量级（2.8MB）

**性能对比**（2AFC人类判断一致率）：

|指标|传统失真|真实算法|
|---|---|---|
|L2|~55%|~53%|
|SSIM|~60%|~55%|
|**LPIPS (VGG-lin)**|**~75%**|**~67%**|
|人类上限|~84%|~70%|

```python
import lpips

# 创建损失函数
loss_fn = lpips.LPIPS(net='alex')  # 或 'vgg', 'squeeze'

# 输入：RGB图像，必须归一化到 [-1, 1]
img0 = (img0 * 2) - 1  # 从[0,1]转换
img1 = (img1 * 2) - 1
distance = loss_fn(img0, img1)  # 距离越小越相似，范围约[0, 1]

# torchmetrics版本
from torchmetrics.image.lpip import LearnedPerceptualImagePatchSimilarity
lpips_metric = LearnedPerceptualImagePatchSimilarity(net_type='alex')
```

### CLIP Score：文本-图像语义对齐

CLIP Score利用CLIP模型的跨模态对齐能力评估生成图像与文本提示的语义一致性：

$$\text{CLIPScore}(I, C) = \max(100 \times \cos(E_I, E_C), 0)$$

其中 $E_I$、$E_C$ 分别是图像和文本的CLIP嵌入。

**局限性**：存在"词袋"问题（混淆"马吃草"和"草吃马"），对复杂组合推理能力有限，且**对视觉质量完全不敏感**。

```python
from torchmetrics.multimodal.clip_score import CLIPScore

metric = CLIPScore(model_name_or_path="openai/clip-vit-base-patch16")
score = metric(image, "a photo of a cat")  # 范围0-100，越高越好
```

---

## 三、视频生成评估指标

### FVD：视频领域的FID扩展

Fréchet Video Distance由Unterthiner等人于2019年提出，是FID向视频领域的自然扩展，使用I3D网络提取时空特征。

**核心区别**：

- **特征提取器**：Kinetics-400预训练的I3D网络（3D卷积），而非ImageNet预训练的Inception V3
- **特征层**：推荐使用logits层（400维），与人类判断相关性最高
- **输入格式**：[batch, frames, height, width, channels]，至少10帧

**预处理要求**：

|参数|要求|
|---|---|
|帧数|≥10帧（I3D时间下采样限制）|
|分辨率|resize至224×224|
|样本量|≥2048个视频以获得稳定估计|

**PyTorch实现**（推荐common_metrics_on_video_quality库）：

```python
from calculate_fvd import calculate_fvd

# 输入格式: [N, T, C, H, W], 像素值在[0,1]
fvd = calculate_fvd(real_videos, gen_videos, device='cuda', 
                     method='styleganv', only_final=True)
```

**FVD的重大缺陷**（CVPR 2024论文揭示）：**存在严重的内容偏差**，偏向于单帧质量而对时序失真不敏感。静态视频（重复同一帧）可能获得比动态视频更低的FVD。解决方案是使用**VideoMAE-v2特征**替代I3D。

### FVMD：专注运动一致性评估

Fréchet Video Motion Distance（ICML 2024 Workshop）针对FVD的时序盲点设计：

1. 使用PIPs++模型追踪关键点轨迹
2. 计算速度场和加速度场
3. 构建运动特征直方图
4. 使用Fréchet距离比较分布

```python
from fvmd import fvmd
fvmd_value = fvmd(log_dir='./logs', gen_path='./gen', gt_path='./gt')
```

### VBench：16维度综合评估框架

VBench（CVPR 2024 Highlight）已成为视频生成评估的事实标准，提供**16个评估维度**：

|类别|评估维度|
|---|---|
|时序质量|主体一致性、背景一致性、时间闪烁、运动平滑性|
|帧质量|美学质量、成像质量|
|语义|物体类别、多物体、颜色、空间关系|
|文本对齐|整体一致性、外观风格、时序风格|

**具体实现方法**：

- 主体一致性：DINO特征跨帧余弦相似度
- 背景一致性：CLIP特征跨帧相似度
- 运动平滑性：AMT帧插值模型重建误差
- 成像质量：MUSIQ图像质量预测器

```bash
pip install vbench
vbench evaluate --videos_path /path/to/videos/ --dimension all
```

**VBench 2.0**（2025年1月发布）新增五大维度：人体保真度、可控性、创造力、物理合理性、常识，从"表面忠实度"转向"内在忠实度"评估。

---

## 四、最新模型的评估实践

### Stable Diffusion与DALL-E的评估方法

**SD系列**使用FID作为主要定量指标，配合CLIP Score评估文本对齐度（使用`openai/clip-vit-large-patch14`与预训练保持一致）。SD3在COCO和ImageNet-1k上报告FID，相比SD1.5有显著提升。

**DALL-E 3**引入**GenEval基准**（得分0.67）进行组合能力评估，涵盖单物体、两物体、计数、颜色、位置、属性绑定六类任务。OpenAI还采用大规模人类偏好评估验证自动指标有效性。

### Sora与Goku的视频评估

**Open-Sora 2.0**在VBench上与OpenAI Sora的差距已从4.52%缩小至0.69%。Sora的内部评估包括：

- 视觉质量、提示词一致性、运动质量三个维度的人类评估
- 针对裸体、欺骗性选举内容、自残、暴力的安全评估

**Goku**（字节跳动）在VBench得分**84.85**（榜单第二），GenEval得分**0.76**超越DALL-E 3。其技术亮点包括Rectified Flow Transformer架构和图像-视频联合VAE。

### 新一代评估指标的崛起

**CMMD**（CVPR 2024）用CLIP嵌入替代Inception（训练数据量提升400倍），用MMD替代Fréchet距离（无正态假设），在Stable Diffusion迭代过程中表现出单调改善，优于FID的不一致行为：

```python
# https://github.com/google-research/google-research/tree/master/cmmd
from cmmd import compute_cmmd
score = compute_cmmd(real_images, generated_images)
```

**ImageReward**（NeurIPS 2023）是首个通用文本-图像人类偏好奖励模型，基于**137k专家比较对**训练，性能超越CLIP、Aesthetic和BLIP。可直接用于强化学习优化扩散模型（ReFL），调优后胜率达58.4%：

```python
import ImageReward as RM
model = RM.load("ImageReward-v1.0")
reward = model.score(prompt, image)  # 越高越好
```

**DreamSim**（NeurIPS 2023 Spotlight）弥合低级指标（LPIPS）和高级指标（CLIP）的差距，集成CLIP、OpenCLIP、DINO嵌入，在20k人类判断的合成图像三元组上微调，达到**96.2%人类对齐率**。

---

## 五、完整评估代码实现

以下代码展示了图像生成评估的完整流程：

```python
import torch
from torchmetrics.image import (
    FrechetInceptionDistance, 
    KernelInceptionDistance,
    StructuralSimilarityIndexMeasure,
    PeakSignalNoiseRatio
)
from torchmetrics.image.lpip import LearnedPerceptualImagePatchSimilarity
from torchmetrics.multimodal.clip_score import CLIPScore
from cleanfid import fid
import ImageReward as RM

# ========== 1. 分布距离指标（需要大量样本） ==========
# FID (推荐使用clean-fid)
fid_score = fid.compute_fid(real_path, gen_path, mode="clean")

# KID (小样本友好)
kid = KernelInceptionDistance(feature=2048, subsets=100, subset_size=1000)
kid.update(real_imgs, real=True)
kid.update(fake_imgs, real=False)
kid_mean, kid_std = kid.compute()

# ========== 2. 参考图像质量指标（成对比较） ==========
# PSNR
psnr = PeakSignalNoiseRatio()
psnr_score = psnr(pred, target)

# SSIM
ssim = StructuralSimilarityIndexMeasure(data_range=1.0)
ssim_score = ssim(pred, target)

# LPIPS (注意归一化到[-1,1])
lpips = LearnedPerceptualImagePatchSimilarity(net_type='alex')
lpips_score = lpips(pred * 2 - 1, target * 2 - 1)

# ========== 3. 文本-图像对齐 ==========
clip_metric = CLIPScore(model_name_or_path="openai/clip-vit-base-patch16")
clip_score = clip_metric(image_uint8, "a photo of a cat")

# ========== 4. 人类偏好评分 ==========
ir_model = RM.load("ImageReward-v1.0")
reward = ir_model.score(prompt, generated_image)

# ========== 5. 视频评估（VBench） ==========
# 命令行: vbench evaluate --videos_path ./videos --dimension all
```

---

## 结论与最佳实践

**核心建议**：放弃单独使用FID，采用**多指标组合评估**策略：

- **图像生成**：CMMD/Clean-FID + CLIP Score + ImageReward/HPS
- **视频生成**：VBench多维度 + FVMD（运动）+ 人类评估
- **复杂组合任务**：GenEval + TIFA进行细粒度诊断

**技术要点**：

1. 使用Clean-FID确保预处理一致性，避免库差异导致的6+分偏差
2. FID/KID至少需要**5000-10000样本**，CMMD样本效率更高
3. 视频评估必须关注**时序一致性**，FVD对此不敏感
4. 人类对齐验证不可或缺——自动指标与人类判断相关性通常仅0.5-0.7

**研究趋势**：评估体系正从单一数值（FID）向多维度框架（VBench 16维）演进，从分布统计向人类偏好对齐（ImageReward、HPS）转型，从表面忠实度向内在忠实度（物理、常识）深化。