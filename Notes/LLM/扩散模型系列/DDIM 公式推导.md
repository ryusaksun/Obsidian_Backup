## 去噪扩散隐式模型（DDIM）理论基础与公式推导全景研究报告

## 1. 绪论：生成模型的演进与扩散模型的瓶颈

### 1.1 生成式建模的范式转移

在人工智能与机器学习的广阔版图中，生成式建模（Generative Modeling）始终占据着核心地位，其终极目标是捕捉并复现复杂数据分布 $p_{data}(x)$ 的内在结构。回顾过去十年的发展历程，我们见证了从基于似然的显式模型到隐式模型的深刻演变。生成对抗网络（GANs）曾以其能够生成高保真度图像而主导该领域，但其对抗训练的不稳定性及模式坍塌（Mode Collapse）问题始终是难以逾越的障碍。与此同时，变分自编码器（VAEs）和流模型（Flow-based Models）提供了更稳健的概率框架，但在样本质量上往往难以与GAN匹敌。

2020年，去噪扩散概率模型（Denoising Diffusion Probabilistic Models, DDPMs）的横空出世，标志着生成模型进入了一个全新的时代。Ho等人证明了通过学习一个参数化的马尔可夫链逆过程，可以从纯高斯噪声中逐步还原出高质量的数据样本。DDPM不仅在样本质量上超越了GAN，还具备训练过程稳定、覆盖分布多样性好等理论优势。然而，DDPM的卓越性能伴随着极其高昂的计算代价：其生成过程通常需要模拟数千个时间步（例如 $T=1000$）的马尔可夫链。这意味着生成一张图像需要对神经网络进行上千次的前向推理，这种巨大的延迟使得DDPM在实时应用和大规模部署中面临严峻挑战。

### 1.2 DDIM的提出背景与核心贡献

正是在这种背景下，Song等人于ICLR 2021提出了去噪扩散隐式模型（Denoising Diffusion Implicit Models, DDIMs）。DDIM并非对DDPM的简单修补，而是对其理论基础的一次深刻重构。研究团队敏锐地指出，<font color="#00b0f0">DDPM 的训练目标——即去噪得分匹配（Denoising Score Matching）或噪声预测损失——实际上仅依赖于边缘分布（Marginal Distribution）</font> $\color{#00b0f0}{q(x_t|x_0)}$ <font color="#00b0f0">，而不依赖于特定的马尔可夫联合分布</font> $\color{#00b0f0}{q(x_{1:T}|x_0)}$ <font color="#00b0f0">。</font>

<font color="#00b0f0">这一洞见构成了DDIM的基石：只要我们能构造出一族新的前向过程，使其在任意时间步 $t$ 的边缘分布与DDPM保持一致，那么这些过程就可以共享同一个预训练的去噪模型，而无需重新训练</font>。<font color="#00b0f0">DDIM通过引入非马尔可夫（Non-Markovian）的前向过程</font><font color="#00b0f0">，推导出了确定性的逆向生成路径</font>。这不仅允许在采样阶段大幅压缩步数（例如从1000步减少至50步），实现了10倍至50倍的加速，还揭示了扩散模型与神经常微分方程（Neural ODEs）之间的深层联系。

本报告将以详尽的数学推导为核心，层层剖析DDIM的理论架构。我们将从边缘分布的约束出发，重构非马尔可夫前向过程，推导其逆向生成公式，证明其变分目标的等价性，并深入探讨其与概率流ODE（Probability Flow ODE）的数学同构关系及其在加速采样、语义插值和图像反演中的关键应用。

---

## 2. 理论基石：扩散模型的数学形式化

在深入DDIM的推导细节之前，必须建立严谨的数学符号系统，并回顾DDPM中被DDIM继承的关键性质。这不仅是理解DDIM的前提，也是后续进行公式变形和证明的基础。

### 2.1 扩散过程的定义

给定从数据分布 $x_0 \sim q(x_0)$ 中采样的真实数据，扩散模型定义了一个前向过程（Forward Process），逐步向数据中注入高斯噪声，直至数据完全被噪声淹没。这一过程通常被参数化为一个马尔可夫链。

设时间步 $t \in \{1, \dots, T\}$，$\beta_t \in (0, 1)$ 为预定义的方差调度序列（Variance Schedule）。DDPM的前向转移概率定义为：

$$q(x_t | x_{t-1}) = \mathcal{N}(x_t; \sqrt{1 - \beta_t} x_{t-1}, \beta_t \mathbf{I})$$

为了简化计算，引入辅助变量：

- $\alpha_t = 1 - \beta_t$
    
- $\bar{\alpha}_t = \prod_{s=1}^t \alpha_s$
    

利用高斯分布的再生性（Reparameterization Trick），我们可以直接写出任意时间步 $t$ 的状态 $x_t$ 相对于初始状态 $x_0$ 的边缘分布 $q(x_t|x_0)$。这是一个极其关键的性质，DDIM的整个理论大厦都建立在这个性质之上：

$$q(x_t | x_0) = \mathcal{N}(x_t; \sqrt{\bar{\alpha}_t} x_0, (1 - \bar{\alpha}_t) \mathbf{I})$$

这意味着 $x_t$ 可以显式地表示为：

$$x_t = \sqrt{\bar{\alpha}_t} x_0 + \sqrt{1 - \bar{\alpha}_t} \epsilon$$

其中 $\epsilon \sim \mathcal{N}(0, \mathbf{I})$ 是标准高斯噪声。

### 2.2 变分下界与训练目标的回顾

扩散模型的训练本质上是最大化数据对数似然的变分下界（Evidence Lower Bound, ELBO）。在DDPM中，这一目标被简化为预测添加到图像中的噪声。

损失函数 $L_{simple}$ 定义为：

$$L_{simple}(\theta) = \mathbb{E}_{t, x_0, \epsilon} \left[ \| \epsilon - \epsilon_\theta(x_t, t) \|^2 \right]$$

这里需要特别强调的是：该损失函数的计算仅涉及 $q(x_t|x_0)$（用于生成训练样本 $x_t$）和模型网络 $\epsilon_\theta$。它并没有显式地约束 $x_{t-1}$ 和 $x_t$ 之间的具体转移路径。换言之，任何能够产生相同边缘分布 $q(x_t|x_0)$ 的前向过程，在理论上都适用于同一个训练目标。这正是DDIM得以存在的理论缝隙——它改变了过程的轨迹（Joint Distribution），但保留了端点（Marginals）的统计特性。

---

## 3. 非马尔可夫前向过程的构建与推导

DDPM的局限性在于其严格的马尔可夫假设，即 $x_t$ 仅依赖于 $x_{t-1}$。DDIM打破了这一限制，构建了一个非马尔可夫推断过程。本节将详细推导这一过程的数学形式。

### 3.1 联合分布的重构与推断族

为了引入非马尔可夫性，Song等人定义了一族新的推断分布 $q_\sigma(x_{1:T}|x_0)$。与DDPM通过前向乘积 $q(x_t|x_{t-1})$ 定义联合分布不同，DDIM采用了基于贝叶斯逆向分解的形式：

$$q_\sigma(x_{1:T}|x_0) = q_\sigma(x_T|x_0) \prod_{t=2}^T q_\sigma(x_{t-1}|x_t, x_0)$$

这里，我们将 $x_T$ 定义为满足标准高斯分布（假设 $\bar{\alpha}_T \approx 0$），并且每一个 $x_{t-1}$ 都条件依赖于 $x_t$ 和 $x_0$。这种依赖关系的引入，<font color="#00b0f0">使得我们能够显式地利用</font> $\color{#00b0f0}{x_0}$ <font color="#00b0f0">的信息来指导从</font> $\color{#00b0f0}{x_t}$ <font color="#00b0f0">到</font> $\color{#00b0f0}{x_{t-1}}$ <font color="#00b0f0">的推断，而不必受限于马尔可夫性质</font>。

### 3.2 待定系数法推导后验分布 $q_\sigma(x_{t-1}|x_t, x_0)$

我们的核心任务是找到一个分布 $q_\sigma(x_{t-1}|x_t, x_0)$ 的具体形式，使得对于所有的 $t$，边缘分布 $q(x_t|x_0)$ 始终保持为 $\mathcal{N}(\sqrt{\bar{\alpha}_t}x_0, (1-\bar{\alpha}_t)\mathbf{I})$。这是DDIM与DDPM兼容的充要条件。

假设 $q_\sigma(x_{t-1}|x_t, x_0)$ 是一个高斯分布。考虑到 $x_t$ 和 $x_0$ 的线性关系，我们合理假设 $x_{t-1}$ 的均值是 $x_t$ 和 $x_0$ 的线性组合。我们可以将其形式化为：

$$q_\sigma(x_{t-1}|x_t, x_0) = \mathcal{N}(x_{t-1}; \mu_\theta(x_t, x_0), \sigma_t^2 \mathbf{I})$$

这里的 $\sigma_t$ 是一个待定的超参数，它直接控制了生成过程的随机性（Stochasticity）。为了推导均值 $\mu_\theta(x_t, x_0)$ 的解析式，我们利用以下恒等关系。

根据边缘分布定义：

$$x_t = \sqrt{\bar{\alpha}_t}x_0 + \sqrt{1 - \bar{\alpha}_t}\epsilon_t \implies \epsilon_t = \frac{x_t - \sqrt{\bar{\alpha}_t}x_0}{\sqrt{1 - \bar{\alpha}_t}}$$

$$x_{t-1} = \sqrt{\bar{\alpha}_{t-1}}x_0 + \sqrt{1 - \bar{\alpha}_{t-1}}\epsilon_{t-1}$$

在DDIM的推导逻辑中，我们试图将 $x_{t-1}$ 表达为 $x_0$ 和当前噪声 $\epsilon_t$ 的函数。为了引入随机性 $\sigma_t$，我们将 $x_{t-1}$ 中的噪声项 $\sqrt{1 - \bar{\alpha}_{t-1}}\epsilon_{t-1}$ 分解为两个正交的部分：

1. 相关部分（Dependent Part）：与 $x_t$ 中的噪声 $\epsilon_t$ 直接相关。这部分确保了轨迹的连贯性。
    
2. 独立部分（Independent Part）：完全独立的随机噪声，用于模拟随机过程。
    

具体来说，我们将 $x_{t-1}$ 重写为以下三项之和：

$$x_{t-1} = \underbrace{\sqrt{\bar{\alpha}_{t-1}} x_0}_{\text{信号分量 (来自 } x_0)} + \underbrace{\sqrt{1 - \bar{\alpha}_{t-1} - \sigma_t^2} \cdot \epsilon_t}_{\text{方向分量 (与 } x_t \text{ 共享噪声)}} + \underbrace{\sigma_t \epsilon}_{\text{独立随机噪声}}$$

其中 $\epsilon \sim \mathcal{N}(0, \mathbf{I})$ 是新引入的噪声，与 $\epsilon_t$ 独立。

现在，我们将 $\epsilon_t = \frac{x_t - \sqrt{\bar{\alpha}_t}x_0}{\sqrt{1 - \bar{\alpha}_t}}$ 代入上式，得到 $x_{t-1}$ 的均值表达式：

$$\mu(x_t, x_0) = \sqrt{\bar{\alpha}_{t-1}} x_0 + \sqrt{1 - \bar{\alpha}_{t-1} - \sigma_t^2} \cdot \frac{x_t - \sqrt{\bar{\alpha}_t}x_0}{\sqrt{1 - \bar{\alpha}_t}}$$

这给出了DDIM核心后验分布的完整定义：

$$q_\sigma(x_{t-1}|x_t, x_0) = \mathcal{N}\left( \sqrt{\bar{\alpha}_{t-1}}x_0 + \sqrt{1 - \bar{\alpha}_{t-1} - \sigma_t^2} \cdot \frac{x_t - \sqrt{\bar{\alpha}_t}x_0}{\sqrt{1 - \bar{\alpha}_t}}, \sigma_t^2 \mathbf{I} \right)$$

### 3.3 边缘分布一致性的数学证明（归纳法）

为了严谨地证明上述构造确实满足边缘分布一致性，我们采用数学归纳法。

基础步骤：对于 $t=T$，根据定义 $q_\sigma(x_T|x_0) = \mathcal{N}(\sqrt{\bar{\alpha}_T}x_0, (1-\bar{\alpha}_T)\mathbf{I})$，这与DDPM一致。

归纳步骤：假设在时间步 $t$，边缘分布 $q_\sigma(x_t|x_0) = \mathcal{N}(\sqrt{\bar{\alpha}_t}x_0, (1-\bar{\alpha}_t)\mathbf{I})$ 成立。我们需要证明 $q_\sigma(x_{t-1}|x_0)$ 也满足相应的形式。

利用全概率公式：

$$q_\sigma(x_{t-1}|x_0) = \int q_\sigma(x_{t-1}|x_t, x_0) q_\sigma(x_t|x_0) dx_t$$

由于被积函数均为高斯分布，我们可以通过计算期望和方差来确定结果分布。

令 $x_t = \sqrt{\bar{\alpha}_t}x_0 + \sqrt{1-\bar{\alpha}_t}\epsilon$，其中 $\epsilon \sim \mathcal{N}(0, \mathbf{I})$。将 $x_t$ 代入 $x_{t-1}$ 的定义式：

$$\begin{aligned} x_{t-1} &= \sqrt{\bar{\alpha}_{t-1}}x_0 + \sqrt{1 - \bar{\alpha}_{t-1} - \sigma_t^2} \cdot \frac{(\sqrt{\bar{\alpha}_t}x_0 + \sqrt{1-\bar{\alpha}_t}\epsilon) - \sqrt{\bar{\alpha}_t}x_0}{\sqrt{1 - \bar{\alpha}_t}} + \sigma_t \epsilon' \\ &= \sqrt{\bar{\alpha}_{t-1}}x_0 + \sqrt{1 - \bar{\alpha}_{t-1} - \sigma_t^2} \cdot \frac{\sqrt{1-\bar{\alpha}_t}\epsilon}{\sqrt{1-\bar{\alpha}_t}} + \sigma_t \epsilon' \\ &= \sqrt{\bar{\alpha}_{t-1}}x_0 + \sqrt{1 - \bar{\alpha}_{t-1} - \sigma_t^2} \cdot \epsilon + \sigma_t \epsilon' \end{aligned}$$

其中 $\epsilon$ 和 $\epsilon'$ 均为独立标准高斯噪声。根据高斯分布的可加性，两个独立高斯变量的线性组合仍然是高斯变量，其方差相加：

$$Var[\text{噪声项}] = (\sqrt{1 - \bar{\alpha}_{t-1} - \sigma_t^2})^2 + \sigma_t^2 = 1 - \bar{\alpha}_{t-1} - \sigma_t^2 + \sigma_t^2 = 1 - \bar{\alpha}_{t-1}$$

因此，我们得到：

$$x_{t-1} \sim \mathcal{N}(\sqrt{\bar{\alpha}_{t-1}}x_0, (1 - \bar{\alpha}_{t-1})\mathbf{I})$$

证毕。这证明了无论 $\sigma_t$ 取何值，边缘分布始终保持不变。这一结论是DDIM能够复用DDPM权重的理论保障 6。

---

## 4. 逆向生成过程与采样公式的推导

DDIM的生成过程（Generative Process），即逆向去噪过程 $p_\theta(x_{t-1}|x_t)$，是对上述前向后验 $q_\sigma(x_{t-1}|x_t, x_0)$ 的参数化近似。在实际推理中，我们无法获知真实的 $x_0$，因此必须依赖神经网络的预测。

### 4.1 从后验到生成的桥接

在生成阶段的时间步 $t$，我们持有当前的噪声图像 $x_t$ 以及一个训练好的噪声预测网络 $\epsilon_\theta(x_t, t)$。为了模拟后验分布 $q_\sigma(x_{t-1}|x_t, x_0)$，我们需要用一个估计值 $\hat{x}_0$ 来替代真实的 $x_0$。

根据扩散公式 $x_t = \sqrt{\bar{\alpha}_t}x_0 + \sqrt{1 - \bar{\alpha}_t}\epsilon$，我们可以通过重排公式得到 $x_0$ 的预测值。这一步骤在统计学中与 Tweedie's Formula 密切相关，它利用得分函数（Score Function）来对潜变量进行去噪估计 6：

$$\hat{x}_0(x_t) = \frac{x_t - \sqrt{1 - \bar{\alpha}_t}\epsilon_\theta(x_t, t)}{\sqrt{\bar{\alpha}_t}}$$

这个公式告诉我们，给定当前观测 $x_t$ 和模型预测的噪声 $\epsilon_\theta$，我们对原始图像 $x_0$ 的“最佳猜测”是什么。

### 4.2 DDIM 通用采样公式 (The General Update Rule)

将上述估计值 $\hat{x}_0(x_t)$ 代入到 3.2 节推导的后验均值公式中，并加上方差为 $\sigma_t^2$ 的随机噪声，我们得到了DDIM的通用更新公式。这个公式构成了DDIM算法的核心，也是所有代码实现的基础 9：

$$x_{t-1} = \underbrace{\sqrt{\bar{\alpha}_{t-1}} \left( \frac{x_t - \sqrt{1 - \bar{\alpha}_t}\epsilon_\theta(x_t, t)}{\sqrt{\bar{\alpha}_t}} \right)}_{\text{分量 1: 预测的 } x_0 \text{ 贡献}} + \underbrace{\sqrt{1 - \bar{\alpha}_{t-1} - \sigma_t^2} \cdot \epsilon_\theta(x_t, t)}_{\text{分量 2: 指向 } x_t \text{ 的方向贡献}} + \underbrace{\sigma_t \epsilon_t}_{\text{分量 3: 随机噪声}}$$

其中 $\epsilon_t \sim \mathcal{N}(0, \mathbf{I})$。

### 4.3 关键参数 $\sigma_t$ 与 $\eta$ 的解析

公式中的 $\sigma_t$ 是一个自由度极高的参数，它控制了生成过程的随机性。Song等人建议将 $\sigma_t$ 参数化为依赖于超参数 $\eta \in $ 的形式：

$$\sigma_t(\eta) = \eta \sqrt{\frac{1 - \bar{\alpha}_{t-1}}{1 - \bar{\alpha}_t}} \sqrt{1 - \frac{\bar{\alpha}_t}{\bar{\alpha}_{t-1}}}$$

这一复杂的表达式并非随意构造，而是为了在 $\eta=1$ 时能够精确还原DDPM的方差。

#### 情形一：DDPM 模式 ($\eta=1$)

当 $\eta=1$ 时，$\sigma_t$ 等于前向过程后验分布的真实标准差。此时，生成过程不仅边缘分布与DDPM一致，其条件转移分布 $p_\theta(x_{t-1}|x_t)$ 也与DDPM的逆向马尔可夫链完全吻合。因此，$\eta=1$ 时的DDIM在数学上等价于DDPM。

#### 情形二：DDIM 确定性模式 ($\eta=0$)

这是本报告关注的重点。当 $\eta=0$ 时，$\sigma_t = 0$。此时随机噪声项（分量 3）完全消失。采样过程变为完全确定性（Deterministic）的：

$$x_{t-1} = \sqrt{\bar{\alpha}_{t-1}} \left( \frac{x_t - \sqrt{1 - \bar{\alpha}_t}\epsilon_\theta(x_t, t)}{\sqrt{\bar{\alpha}_t}} \right) + \sqrt{1 - \bar{\alpha}_{t-1}} \cdot \epsilon_\theta(x_t, t)$$

这意味着：

1. 固定轨迹：一旦初始噪声 $x_T$ 给定，生成的图像 $x_0$ 是唯一的。
    
2. 隐式模型：生成过程不再是随机游走，而是一个确定性的映射 $x_T \mapsto x_0$。这使得DDIM被归类为“隐式概率模型”（Implicit Probabilistic Model）。
    
3. 一致性：由于没有随机噪声的干扰，模型倾向于在不同的时间步保持特征的一致性，这对于后续的插值和反演至关重要。
    

下表总结了 $\eta$ 对采样行为的影响：

|参数设置|σt​ 值|过程性质|是否等价 DDPM|重建误差|多样性|
|---|---|---|---|---|---|
|$\eta = 1$|$\tilde{\beta}_t^{1/2}$|马尔可夫、随机|是|较高|高|
|$\eta = 0$|$0$|非马尔可夫、确定性|否 (DDIM)|极低|受限于 $x_T$|
|$0 < \eta < 1$|中间值|混合过程|否|中等|中等|

---

## 5. 变分下界（ELBO）的一致性证明

一个自然的问题是：既然DDIM改变了前向过程 $q(x_{1:T}|x_0)$，那么使用DDPM的目标函数训练出来的模型 $\epsilon_\theta$，对于DDIM是否仍然是最优的？这就需要证明DDIM的变分下界 $J_\sigma$ 与DDPM的训练目标是兼容的。

### 5.1 $J_\sigma$ 的推导

DDIM 的变分目标函数 $J_\sigma$ 定义为：

$$J_\sigma(\epsilon_\theta) = \mathbb{E}_{q_\sigma(x_{0:T})} \left$$

利用我们构造的 $q_\sigma$ 和 $p_\theta$ 的高斯形式，我们可以计算各项的KL散度。对于任意时间步 $t$，$\log p_\theta(x_{t-1}|x_t)$ 与 $\log q_\sigma(x_{t-1}|x_t, x_0)$ 之间的差异主要体现在均值的匹配上（假设方差固定）。

根据 3.2 节的推导，后验均值为：

$$\mu_q = \sqrt{\bar{\alpha}_{t-1}}x_0 + \sqrt{1 - \bar{\alpha}_{t-1} - \sigma_t^2} \cdot \frac{x_t - \sqrt{\bar{\alpha}_t}x_0}{\sqrt{1 - \bar{\alpha}_t}}$$

而模型的预测均值 $\mu_\theta$ 是通过将 $x_0$ 替换为 $\hat{x}_0(x_t)$ 得到的：

$$\mu_\theta = \sqrt{\bar{\alpha}_{t-1}}\hat{x}_0(x_t) + \sqrt{1 - \bar{\alpha}_{t-1} - \sigma_t^2} \cdot \frac{x_t - \sqrt{\bar{\alpha}_t}\hat{x}_0(x_t)}{\sqrt{1 - \bar{\alpha}_t}}$$

经过极其繁琐但直接的代数化简（将 $x_0$ 和 $\hat{x}_0$ 用 $\epsilon$ 和 $\epsilon_\theta$ 表示），可以发现两者的均值之差与噪声预测误差成正比：

$$\| \mu_q - \mu_\theta \|^2 \propto \| \epsilon_t - \epsilon_\theta(x_t, t) \|^2$$

因此，DDIM的变分下界可以写为：

$$J_\sigma \propto \sum_{t=1}^T \gamma_t \mathbb{E}_{x_0, \epsilon} [ \| \epsilon - \epsilon_\theta(\sqrt{\bar{\alpha}_t}x_0 + \sqrt{1-\bar{\alpha}_t}\epsilon, t) \|^2 ] + C$$

其中 $\gamma_t$ 是仅依赖于 $\alpha$ 和 $\sigma$ 的权重系数。

### 5.2 结论与意义

上述推导证明了 $J_\sigma$ 与 DDPM 的简化损失 $L_{simple}$ 在形式上是完全一致的，仅仅是不同时间步的权重不同。而在实践中，DDPM 已经忽略了权重的差异直接优化 $L_{simple}$。

这意味着：

1. 我们不需要为了使用 DDIM 采样而重新设计损失函数或重新训练模型。
    
2. 预训练的 DDPM 模型（在 $\eta=1$ 假设下训练）可以直接无缝迁移到 DDIM ($\eta=0$) 的采样过程中。
    
3. 改变 $\sigma_t$（即改变 $\eta$）只会改变变分下界的权重，而不会改变最优解 $\epsilon_\theta^$ 的位置 6。
    

---

## 6. 加速机制：子序列采样与响应式跳步

DDIM 最引人注目的特性是其能够显著加速采样过程。标准的 DDPM 必须严格遵循 $t, t-1, t-2, \dots, 0$ 的序列，因为其逆向转移 $p(x_{t-1}|x_t)$ 是基于相邻时间步微小变化的假设训练的。然而，DDIM 的推导不仅解除了这一限制，还提供了理论上的保障。

### 6.1 子序列采样的理论依据

回顾 DDIM 的更新公式，我们发现 $x_{t-1}$ 的计算仅依赖于 $x_t$ 和边缘分布参数 $\bar{\alpha}_t, \bar{\alpha}_{t-1}$。公式中并没有任何项要求 $t-1$ 必须是 $t$ 的直接前驱。

只要我们能定义一个时间步子序列 $\tau =$，其中 $S \ll T$（例如 $S=50, T=1000$），且 $\tau_S \approx T, \tau_1 \approx 0$，我们就可以直接应用 DDIM 公式从 $x_{\tau_i}$ 跳跃推断到 $x_{\tau_{i-1}}$：

$$x_{\tau_{i-1}} = \sqrt{\bar{\alpha}_{\tau_{i-1}}} \left( \frac{x_{\tau_i} - \sqrt{1 - \bar{\alpha}_{\tau_i}}\epsilon_\theta(x_{\tau_i})}{\sqrt{\bar{\alpha}_{\tau_i}}} \right) + \sqrt{1 - \bar{\alpha}_{\tau_{i-1}} - \sigma_{\tau_i}^2} \cdot \epsilon_\theta(x_{\tau_i}) + \sigma_{\tau_i} \epsilon$$

这种“响应式跳步”（Responsive Skipping）之所以有效，是因为 $\epsilon_\theta(x_{\tau_i})$ 作为一个去噪函数，在 $\tau_i$ 时刻仍然提供了指向真实数据流形方向的梯度估计。虽然跳步较大可能会引入离散化误差，但由于 DDIM 的确定性结构，这种误差比随机游走的 DDPM 要小得多 1。

### 6.2 调度策略：线性 vs 二次

子序列 $\tau$ 的选择策略对生成质量有显著影响。研究中最常用的两种调度策略是：

1. 线性调度 (Linear Schedule)：时间步均匀分布。
    
    $$\tau_i = \lfloor c \cdot i \rfloor, \quad c = T/S$$
    
    适用于大多数通用场景。
    
2. 二次调度 (Quadratic Schedule)：时间步在接近 $t=0$ 时更密集，在接近 $t=T$ 时更稀疏。
    
    $$\tau_i = \lfloor c \cdot i^2 \rfloor$$
    
    原理：在生成过程的早期（高噪声阶段），模型主要确定图像的整体布局和低频结构，变化较平缓，步长可以较大；在生成过程的后期（低噪声阶段），模型需要精细化纹理和高频细节，此时需要更小的步长来减少去噪误差。实验表明，二次调度在 CIFAR-10 等数据集上能获得更优的 FID 分数 1。
    

下表展示了不同步数和调度下 CIFAR-10 的生成效果对比（数据概与其论文）：

|采样步数 (S)|DDPM (FID)|DDIM (FID, Linear)|DDIM (FID, Quadratic)|加速比|
|---|---|---|---|---|
|1000|3.17|4.04|4.00|1x|
|100|6.42|4.16|3.85|10x|
|50|11.45|4.87|4.63|20x|
|20|32.50|7.90|6.50|50x|

数据清晰地表明：在低步数下，DDIM 具有碾压性的优势；且二次调度进一步提升了性能。

---

## 7. 深入洞察：DDIM 与 概率流 ODE

当 $\eta=0$ 时，DDIM 的迭代公式展现出了极其特殊的性质。若我们将时间步长的极限取为 0，这一离散迭代过程将收敛于一个常微分方程（ODE），这被称为概率流 ODE（Probability Flow ODE）。这一发现将扩散模型与流模型（Flow Models）统一在了一起。

### 7.1 从差分方程到微分方程的极限分析

为了推导这一 ODE，我们需要对变量进行重参数化。

令 $\sigma(t) = \sqrt{\frac{1-\alpha(t)}{\alpha(t)}}$，$\bar{x}(t) = \frac{x(t)}{\sqrt{\alpha(t)}}$。

考察 $\eta=0$ 时的 DDIM 更新公式（忽略高阶小量）：

$$\frac{x_{t-\Delta t}}{\sqrt{\alpha_{t-\Delta t}}} - \frac{x_t}{\sqrt{\alpha_t}} \approx \left( \sqrt{\frac{1 - \alpha_{t-\Delta t}}{\alpha_{t-\Delta t}}} - \sqrt{\frac{1 - \alpha_t}{\alpha_t}} \right) \epsilon_\theta(x_t)$$

代入重参数化变量，我们得到：

$$\bar{x}(t-\Delta t) - \bar{x}(t) \approx (\sigma(t-\Delta t) - \sigma(t)) \epsilon_\theta(x_t)$$

两边同除以 $\Delta t$ 并令 $\Delta t \to 0$，我们得到微分形式：

$$d\bar{x}(t) = \epsilon_\theta(x(t)) d\sigma(t)$$

将其还原回原始变量 $x(t)$，并结合 Score Matching 的关系 $\epsilon_\theta(x, t) = -\sqrt{1-\bar{\alpha}_t} \nabla_x \log p_t(x)$，可以推导出著名的概率流 ODE 形式 8：

$$d x_t = \left[ f(x_t, t) - \frac{1}{2} g(t)^2 \nabla_x \log p_t(x_t) \right] dt$$

### 7.2 Euler-Maruyama 离散化与 DDIM 的关系

这就引出了一个深刻的理论问题：DDIM 到底是什么？

- DDPM 可以被视为随机微分方程（SDE）的 Euler-Maruyama 离散化。它包含显式的布朗运动项 $dw$。
    
- DDIM ($\eta=0$) 则是上述 概率流 ODE 的 Euler 离散化。它去除了扩散项，只保留了漂移项（Drift term）。
    

为何 DDIM 比标准 Euler 方法更好？

普通的 Euler 方法在求解 ODE 时假设导数在步长内是常数，这在扩散过程中误差较大。DDIM 的更新公式实际上利用了半解析解（Semi-Analytic Solution）：它假设的是“预测的 $x_0$”在步长内是常数，而不是假设 $x_t$ 的导数是常数。由于 $x_0$（完全去噪图像）比局部梯度更稳定，DDIM 这种特殊的离散化方式使得它比标准的 Euler 方法具有更小的截断误差，从而支持更大的步长 19。

### 7.3 理论意义

1. 可逆性 (Invertibility)：ODE 轨迹在理论上是唯一且可逆的。这意味着我们可以从任意一张真实图片 $x_0$ 出发，通过逆向积分 ODE 得到其潜在编码 $x_T$（Inversion），然后再正向积分还原图像。这为图像编辑（如在潜空间修改属性）提供了数学基础 15。
    
2. 似然估计：通过变量变换公式（Change of Variable Formula），我们可以利用这个 ODE 来精确计算图像的对数似然 $\log p(x_0)$，这是 DDPM 难以做到的 23。
    

---

## 8. 高级应用：反演、编辑与一致性模型

DDIM 的确定性性质使其成为扩散模型高级应用的首选基础。

### 8.1 图像反演 (DDIM Inversion)

在图像编辑任务中，我们需要先找到一张真实图像对应的初始噪声 $x_T$。对于随机的 DDPM，这是不可能的，因为从 $x_0$ 到 $x_T$ 有无数条随机路径。但对于 DDIM，路径是唯一的。

通过运行 $x_{t+1} \leftarrow \text{DDIM\_Rev}(x_t)$，我们可以将 $x_0$ 编码为 $x_T$。研究表明，在 $\eta=0$ 时，重建误差（Reconstruction Error）可以低至 $10^{-4}$ 量级，几乎实现了完美重建。这使得基于 Diffusion 的图像编辑（如使用 Prompt-to-Prompt 技术）成为可能 6。

### 8.2 语义插值 (Semantic Interpolation)

在 DDIM 的潜在空间 $x_T$ 中进行球面线性插值（Slerp）：

$$x_T^{(\alpha)} = \text{Slerp}(x_T^{(1)}, x_T^{(2)}, \alpha)$$

然后使用 DDIM 解码，可以得到两张图像之间平滑过渡的序列。这证明了 DDIM 学习到了语义连贯的潜在空间表示，类似于 StyleGAN 1。

### 8.3 迈向一致性模型 (Consistency Models)

DDIM 仍然需要几十步迭代。为了进一步加速，OpenAI 等机构提出了 一致性模型 (Consistency Models)。其核心思想是直接蒸馏 DDIM 的 ODE 轨迹，训练一个模型 $f_\theta(x_t, t)$ 直接预测 $x_0$，强制要求轨迹上任意一点映射到同一个起点。可以说，DDIM 及其对应的 ODE 理论是一致性模型得以诞生的“母体” 25。

---

## 9. 结论与展望

DDIM 的提出是扩散模型发展史上的一个里程碑。它并没有改变模型的训练方式，而是通过推广前向过程的定义，揭示了扩散模型背后的自由度。

1. 理论统一：DDIM 成功地将马尔可夫扩散（DDPM）与确定性流（ODE）统一在一个变分框架下，证明了它们只是同一枚硬币（边缘分布）的两面。
    
2. 工程突破：通过响应式跳步采样，DDIM 解决了扩散模型推理速度慢的致命弱点，使其从实验室走向了工业界（如 Stable Diffusion 默认采用 DDIM 或其变体 DPM-Solver）。
    
3. 应用基石：其确定性性质为反演、编辑和视频生成等高级任务提供了必要的稳定性。
    

在未来，随着以 ODE 为基础的高阶采样器（如 DPM-Solver++）和蒸馏技术（如 LCM, Consistency Models）的进一步发展，DDIM 的原始公式可能逐渐被更高效的数值解法取代，但其确立的“非马尔可夫后验”和“边缘分布一致性”的理论范式，将继续照亮生成模型的前行之路。

---

## 附录：关键公式速查表

| 名称              | 公式 / 定义                                                                                                      | 物理含义与备注                                                                 |
| ------------------- | ---------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| 边缘分布            | $q(x_t                                                                                                           | x_0) = \mathcal{N}(\sqrt{\bar{\alpha}_t}x_0, (1-\bar{\alpha}_t)\mathbf{I})$ |
| DDIM 后验         | $q_\sigma(x_{t-1}                                                                                                | x_t, x_0) = \mathcal{N}(\dots, \sigma_t^2 \mathbf{I})$                      |
| DDIM 采样公式       | $x_{t-1} = \sqrt{\bar{\alpha}_{t-1}} (\text{pred } x_0) + \text{dir } x_t + \sigma_t \epsilon_t$                 | 通用生成公式，包含去噪、指向、随机三项                                                         |
| 随机参数 $\sigma_t$ | $\eta \sqrt{\frac{1-\bar{\alpha}_{t-1}}{1-\bar{\alpha}_t}} \sqrt{1 - \frac{\bar{\alpha}_t}{\bar{\alpha}_{t-1}}}$ | 控制随机性。$\eta=0$ 为 DDIM, $\eta=1$ 为 DDPM                                      |
| 预测 $x_0$        | $\hat{x}_0 = (x_t - \sqrt{1-\bar{\alpha}_t}\epsilon_\theta) / \sqrt{\bar{\alpha}_t}$                             | Tweedie 公式，对原始信号的估计                                                         |
| 概率流 ODE         | $dx_t = [f(x,t) - \frac{1}{2}g(t)^2 \nabla \log p_t]dt$                                                          | DDIM ($\eta=0$) 在 $\Delta t \to 0$ 时的连续极限                                   |
---

**<font color="#2ecc71">✅ 已格式化</font>**
