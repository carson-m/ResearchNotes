# DeWave: Discrete EEG Waves Encoding for Brain Dynamics to Text Translation

## Core

EEG Wave Encoding

EEG-text alignment

## VQ-VAE 变分量化自动编码器

Variational Quantized Variational Autoencoder

生成模型，通常用于处理高维数据，如图像和音频

### Autoencoder 自编码器

无监督的神经网络模型。可以学习到输入数据的隐含特征，这称为编码(encoding)，同时用学习到的新特征可以重构出原始输入数据，称之为解码(decoding)。

<img src=".\img\v2-f4444b7343ef311fd04f0dc0dc1db3b7_720w.webp" alt="v2-f4444b7343ef311fd04f0dc0dc1db3b7_720w" style="zoom:50%;" />

自编码器模型主要由编码器（Encoder）和解码器（Decoder）组成，其主要目的是将输入$x$转换成中间变量$y$，然后再将$y$转换成 $\tilde x$，然后对比输入$x$和输出$\tilde x$使得他们两个无限接近。

用途：特征降维、特征提取

特征提取：先压缩编码，再解压解码。信息还原地越好，越说明编码学习到了原信号的主要特征。**压缩**使得模型必须提取主要特征。训练好的自编码中间这一部分就是能总结原数据的精髓

### VAE

变分自动编码器（Variational Autoencoder，简称 VAE）是一种生成模型，它通过学习数据的潜在分布来生成新的数据样本。VAE的核心思想是将数据编码成潜在空间中的分布，并从该分布中采样以生成新的样本。这使得 VAE 能够在生成新数据时具有一定的随机性，因此非常适合生成任务。

### VQ-VAE

#### 量化方法（Quantization）

量化方法是一种将连续数据映射到离散数据的技术。在深度学习中，通常使用 K-means 等聚类算法来执行量化。通过引入离散性，我们可以减少数据表示的复杂性，从而降低模型的计算和存储成本。

#### Encoder

VQ-VAE 的编码器部分将输入数据编码成潜在表示。这与标准的自动编码器类似，但编码器的输出不是直接的潜在向量，而是一个表示符号（codebook index）。编码器的任务是找到最接近输入的表示符号，即最接近的聚类中心。

#### Quantizer

量化器接受编码器的输出，将其映射到离散表示。这是通过查找最接近的聚类中心来完成的，然后输出该聚类中心的索引。这个步骤引入了离散性，减小了表示的维度，降低了复杂性。

#### Decoder

解码器部分接受来自量化器的离散表示，并尝试生成与原始输入相匹配的数据。这一过程与标准自动编码器的解码器类似，但在 VQ-VAE 中，解码器的任务更加困难，因为它必须将离散表示映射回连续数据。

#### Loss Function

VQ-VAE 使用了多个损失函数来训练模型，其中包括重建损失（reconstruction loss）和潜在损失（codebook loss）。重建损失用于确保解码器能够生成接近原始输入的数据，而潜在损失则用于推动编码器生成有效的潜在表示。

## Previous Problems

Current methods, require **eye-tracking fixations or event markers to segment** brain dynamics into word-level features, which can restrict the practical application of these systems.

## Objective

## Discrete Codex

### Advantages

1. Rectifies order mismatches between raw wave sequences and text without eye-tracking markers. *Realizes translation on raw waves without marker by introducing text-EEG contrastive alignment training.*
2. Addresses significant distribution variances in EEG waves across individuals. *Alleviates the interference caused by individual differences in EEG waves through an invariant discrete codex with or without markers.*

## Implementation

### Structure

<img src=".\img\Screenshot 2024-01-28 175317.png" alt="Screenshot 2024-01-28 175317" style="zoom:70%;" />

<img src=".\img\Screenshot 2024-01-28 181014.png" alt="Screenshot 2024-01-28 181014" style="zoom:70%;" />

##### Word-Level

1. The EEG waves are first sliced into fragments according to the eye-tracking fixation of word sequences given in the annotation.
2. Calculate the statistical result of four frequency band filters.
3. A multi-head transformer layer is applied to project the embedding into feature sequences with latent size 512.

##### Raw EEG Wave

1. Encoder transforms raw EEG signals into a **sequence** $\chi$ **of embeddings** $x$.
2. $\chi \ni x\rarr$Transformer encoder$\rarr z_c(x)$. $z_c$(Latent Variable)
3. $z_q(x)=\arg \min_j||z_c(x)-c_j||_2 \rarr z_q(\chi)$. $z_q$(Discrete Latent Variable)
4. Different from the original VQ-VAE which decodes the original input, Dewave directly decodes the translation output given the representation $z_q(\chi)$. Given a pre-trained language model, the decoder predicts text output with $P(W|z_q(\chi))$.

### Learning of Codex

learn the discrete codex by the combination of the loss functions in three parts.

<img src=".\img\Screenshot 2024-01-28 200303.png" alt="Screenshot 2024-01-28 200303" style="zoom:67%;" />

The loss maximize the log-likelihood of language outputs log(P(W|zq(X)) and minimize the distance between latent variable z and the codex value z.

### Reconstruction Process

For the reconstruction process, a decoder transformer and transpose convolution structure then convert these discrete embeddings back into raw waves. Given the reconstruction process as $\tilde X = \phi(z_q(X))$, the **self-supervised loss** could be modified into:

<img src=".\img\Screenshot 2024-01-28 201412.png" alt="Screenshot 2024-01-28 201412" style="zoom:67%;" />

#### $L_{contrast}$

To obtain a semantically coherent code.

Operates within a single EEG-text pair. Contrast the EEG-tokenized codex embeddings sequence $z_q$ with the text embeddings sequence $z_t$.

Assuming the raw wave feature extractors can produce a token sequence in an **organized chronological order**, we treat the **diagonal** EEG codex and text word2vec encoding pairs as **positive pairs** within the sequences. All other pairs are considered negative.

The model is trained to minimize the distance between embeddings of positive pairs and maximize that of the negative pairs.

For a given EEG-text pair $(i,j)$

<img src=".\img\Screenshot 2024-01-28 203609.png" alt="Screenshot 2024-01-28 203609" style="zoom:61%;" />

$\tau$ is the temperature parameter, N is the total number of EEG-text pairs in a batch, and the sum in the denominator is over all N EEG-text pairs and N text-EEG pairs.

#### Total Loss

The total loss then becomes a combination of the original Wave2Vec loss and the contrastive loss as $L_{total} = L_{wave} + \alpha L_{contrast}$.

By this means, the model not only learns to reconstruct the EEG signal but also learns a robust representation of the signal that aligns with the corresponding text embeddings. This cross-modal learning can potentially improve the translation system by **bridging the gap between EEG signals and the semantic content of the text**.

## Leveraging Language Model

利用现成的语言模型，可以绕开EEG-text Translation数据过少的问题。语言模型负责text semantic部分，本实验训练的Discrete Encoding 作为从EEG到语言模型的桥梁，这是实验样本量下能实现的。

## Dataset

Normal Reading(NR)

Task Specific Reading(TSR)