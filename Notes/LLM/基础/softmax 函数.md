Softmax 函数（归一化指数函数）是一种常用的数学函数，它能将一个包含任意实数的向量“压缩”为概率分布向量，使得每个元素的值都在 (0, 1) 之间，且所有元素的和为 1 。它常被用于神经网络的输出层，特别是在多分类问题中，用于输出各个类别的预测概率 。wikipedia+3​

## 数学定义与公式

Softmax 函数将输入向量 z=[z1,z2,…,zK]\mathbf{z} = [z_1, z_2, \dots, z_K]z=[z1,z2,…,zK] 映射为输出向量 σ(z)\sigma(\mathbf{z})σ(z)。对于向量中的第 iii 个元素，Softmax 的计算公式为：

σ(z)i=ezi∑j=1Kezj\sigma(\mathbf{z})_i = \frac{e^{z_i}}{\sum_{j=1}^K e^{z_j}}σ(z)i=∑j=1Kezjezi

其中：

- ziz_izi 是输入向量的第 iii 个分量（即 Logits）。
    
- ezie^{z_i}ezi 是该分量的指数，保证了结果非负。
    
- ∑j=1Kezj\sum_{j=1}^K e^{z_j}∑j=1Kezj 是所有分量指数之和，作为归一化项（分母），确保输出总和为 1 。cloud.tencent+1​
    

## 核心特性

- **概率分布性质**：输出向量的每个元素都代表一个概率值，且 ∑σ(z)i=1\sum \sigma(\mathbf{z})_i = 1∑σ(z)i=1。这使得它非常适合解释为“属于某一类的概率” 。zhuanlan.zhihu+1​
    
- **放大差异（Soft Max）**：Softmax 的名字来源于它是一种“软”的最大值函数。与硬最大值（Hard Max，只保留最大值为1，其余为0）不同，Softmax 会保留所有信息，但通过指数函数放大了较大值的权重，使大概率项更加显著，同时抑制小概率项 。jiqizhixin+1​
    
- **处处可导**：作为光滑函数，Softmax 在所有点上都有导数，这对于神经网络反向传播算法中的梯度计算至关重要 。[cloud.tencent](https://cloud.tencent.com/developer/article/1676286)​
    

## 应用场景

Softmax 最主要的应用是在 **深度学习** 和 **机器学习** 中：

- **多分类问题的输出层**：在处理像“手写数字识别”（10分类）或“ImageNet图像分类”（1000分类）的任务时，网络的最后一层通常是 Softmax 层，输出该样本属于每个类别的概率 。wikipedia+1​
    
- **注意力机制（Attention Mechanism）**：在 Transformer 模型（如 BERT, GPT）中，Softmax 用于计算注意力权重，决定模型应该关注输入序列的哪些部分。
    

## 代码实现与数值稳定性

在实际编程中（如使用 Python），直接计算 ezie^{z_i}ezi 可能会因为数值过大导致溢出（Overflow）。通常采用“减去最大值”的技巧来提高数值稳定性 。[cloud.tencent](https://cloud.tencent.com/developer/article/1676286)​

python

`import numpy as np def softmax(x):     # 减去最大值，防止指数运算溢出    # shift_x 的最大值为 0，exp(0)=1，保证了数值稳定    shift_x = x - np.max(x)    exp_x = np.exp(shift_x)    return exp_x / np.sum(exp_x) # 示例 logits = np.array([2.0, 1.0, 0.1]) probs = softmax(logits) print(probs)  # 输出: [0.65900114 0.24243297 0.09856589] (总和为 1)`

## Softmax 与 Sigmoid 的关系

Softmax 可以看作是 **Sigmoid 函数** 在多分类情况下的推广。当分类类别数 K=2K=2K=2 时，Softmax 函数退化为 Sigmoid 函数（逻辑回归），两者在数学上是等价的 。csdn+1​

1. [https://zhuanlan.zhihu.com/p/105722023](https://zhuanlan.zhihu.com/p/105722023)
2. [https://zh.wikipedia.org/zh-hans/Softmax%E5%87%BD%E6%95%B0](https://zh.wikipedia.org/zh-hans/Softmax%E5%87%BD%E6%95%B0)
3. [https://blog.csdn.net/xu380393916/article/details/102496419](https://blog.csdn.net/xu380393916/article/details/102496419)
4. [https://zhuanlan.zhihu.com/p/168562182](https://zhuanlan.zhihu.com/p/168562182)
5. [https://www.jiqizhixin.com/graph/technologies/0e9e62e2-9958-4146-b0af-f3d049ff2490](https://www.jiqizhixin.com/graph/technologies/0e9e62e2-9958-4146-b0af-f3d049ff2490)
6. [https://cloud.tencent.com/developer/article/1676286](https://cloud.tencent.com/developer/article/1676286)
7. [https://zh.d2l.ai/chapter_linear-networks/softmax-regression.html](https://zh.d2l.ai/chapter_linear-networks/softmax-regression.html)
8. [https://docs.pytorch.org/docs/stable/generated/torch.nn.Softmax.html](https://docs.pytorch.org/docs/stable/generated/torch.nn.Softmax.html)
9. [https://developer.volcengine.com/articles/7382255487758106675](https://developer.volcengine.com/articles/7382255487758106675)
10. [https://blog.csdn.net/bitcarmanlee/article/details/82320853](https://blog.csdn.net/bitcarmanlee/article/details/82320853)