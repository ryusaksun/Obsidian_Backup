WF‑VAE 在论文里把训练讲得比较清楚：它用 **Kinetics‑400** 做训练与验证，并把完整训练分成 **三个阶段**（总计 120 万 step），使用 **8 张 NVIDIA H100** 训练，学习率固定 1e‑5、优化器 AdamW。[[ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/152616778/19898f4b-5479-4923-9bc2-e10d222e0163/WF-VAE.no_watermark.zh-CN.dual.pdf)]​

## 使用的数据集

- 训练与验证：Kinetics‑400。[[ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/152616778/19898f4b-5479-4923-9bc2-e10d222e0163/WF-VAE.no_watermark.zh-CN.dual.pdf)]​
    
- 重建测试：WebVid‑10M、Panda70M。[[ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/152616778/19898f4b-5479-4923-9bc2-e10d222e0163/WF-VAE.no_watermark.zh-CN.dual.pdf)]​
    
- 下游扩散模型训练评估（验证 VAE latent 是否利于训练）：UCF‑101、SkyTime‑lapse，并进行 100,000 steps 的条件/无条件训练评估（作者用于检验“是否便于训练”，不是为了追求最强生成效果）。[[ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/152616778/19898f4b-5479-4923-9bc2-e10d222e0163/WF-VAE.no_watermark.zh-CN.dual.pdf)]​
    

## 训练目标与损失函数（VAE 本体）

作者说明其训练策略参考了两篇工作（文中编号 10、27），并将损失组合为：L1 重建 + 感知损失（LPIPS）+ 对抗损失（GAN）+ KL 正则，再加上他们提出的 **WL loss**（约束能量流结构一致性）。[[ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/152616778/19898f4b-5479-4923-9bc2-e10d222e0163/WF-VAE.no_watermark.zh-CN.dual.pdf)]​  
论文还给出动态对抗权重的做法：用重建损失与对抗损失在 decoder 最后一层梯度幅值的比值去缩放对抗项。[[ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/152616778/19898f4b-5479-4923-9bc2-e10d222e0163/WF-VAE.no_watermark.zh-CN.dual.pdf)]​

## 训练阶段（3-stage schedule）

论文明确写了三阶段训练策略与每阶段目的：[[ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/152616778/19898f4b-5479-4923-9bc2-e10d222e0163/WF-VAE.no_watermark.zh-CN.dual.pdf)]​

- Stage I：25 帧、256×256，总 batch size=8，先训练 800,000 steps（“与文献 6 的设置对齐”）。[[ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/152616778/19898f4b-5479-4923-9bc2-e10d222e0163/WF-VAE.no_watermark.zh-CN.dual.pdf)]​
    
- Stage II：刷新判别器，把帧数升到 49，并把 FPS 减半以增强运动动态，再训练 200,000 steps。[[ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/152616778/19898f4b-5479-4923-9bc2-e10d222e0163/WF-VAE.no_watermark.zh-CN.dual.pdf)]​
    
- Stage III：再次刷新判别器，并把 LPIPS 权重设为 0.1（作者指出较大的 LPIPS 会显著影响视频稳定性），再训练 200,000 steps。[[ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/152616778/19898f4b-5479-4923-9bc2-e10d222e0163/WF-VAE.no_watermark.zh-CN.dual.pdf)]​
    

补充材料的表格也把关键超参按阶段列出（例如 Stage I 的 LPIPS 权重 1.0、WL=0.1、KL 权重 1e‑6、EMA decay 0.999；Stage III 将 LPIPS 权重改为 0.1）。[[ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/152616778/19898f4b-5479-4923-9bc2-e10d222e0163/WF-VAE.no_watermark.zh-CN.dual.pdf)]​

## 训练时间与训练设备

- 设备：训练过程使用 **8× NVIDIA H100 GPU**。[[ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/152616778/19898f4b-5479-4923-9bc2-e10d222e0163/WF-VAE.no_watermark.zh-CN.dual.pdf)]​
    
- 训练时长：论文未给出墙钟时间（训练跑了多少天/小时），只给出了 step 数与硬件规格。[[ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/152616778/19898f4b-5479-4923-9bc2-e10d222e0163/WF-VAE.no_watermark.zh-CN.dual.pdf)]​
    

如果你希望我把“训练细节”整理成一张更工程化的清单（数据预处理/采样、每阶段具体判别器“refresh”怎么做、是否混合精度、梯度累积、并行策略、吞吐等），我可以继续在你这份 PDF 的“Training Details / Appendix”相关段落里把所有可见字段逐条摘出来并结构化。