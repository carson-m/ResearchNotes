# Decoding speech perception from non-invasive brain recordings

## Objective

Non-invasive Natural Speech Decoding

## Challenges

EEG/MEG Signals: noisy, variable across individuals

nature and format of language representation largely unknown

## Main Contributions

1. **It shows how pretrained speech models can leverage the decoding of speech in the brain, without exposing volunteers to a tedious repetition of every single word targeted by the decoder.**

   At evaluation, we input the model with left-out sentences and compute the probability of each 3 s speech segment given each brain representation. The resulting decoding can thus be ‘zero shot’ in that the audio snippets predicted by the model need not be present in the training set. This approach is thus more general than standard classification approaches where the decoder can only predict the categories learnt during training.

   这样训练出的是一个将脑信号向vector的映射关系，网络会找到脑信号和speech vector的对应规律。这样即便遇到训练集里没有的speech segment也可以用同样的映射规律推断出脑信号对应的vector，实现zero-shot.

   This approach is thus more general than standard classification approaches where the decoder can only predict the categories learnt during training.

2. **It shows how specific design choices—including contrastive learning and our multi-participant architecture—improve the processing of continuous EEG and MEG recordings.**

   *Contrastive Learning: assumes that similar instances should be closer together in a learned embedding space, while dissimilar instances should be farther apart.*

   The latent representations of speech sounds, however, appear to be best identified with a pretrained
   speech module, that is, by using wav2vec 2.0, a model pretrained with self-supervised learning on speech sounds only, rather than by jointly learning speech and MEG and EEG representations (Table 2). Overall, these results show the importance, for decoding, of targeting deep representations of speech.

   Decoding performance steadily increases as the model is trained with more participants on the two MEG datasets. This result shows that our model effectively learns neural representations that are common across participants, while also accommodating participant-specific representations through the participant layer described in Methods.

3. **Our results suggest that the speech decoder is primarily based on high-level and semantic representations of speech.**

   The results, shown in Fig. 4, show that the part-of-speech (P < 0.004), word embedding (P < 10−8), bag-of-words embedding (P < 10−23) and phrase embedding (P < 10−23) significantly predict the single-trial decoding predictions.

   <img src=".\imgs\Screenshot 2024-01-27 170757.png" alt="Screenshot 2024-01-27 170757" style="zoom:60%;" />

## Method

A single architecture trained across a large cohort of participants and deep representations of speech learned with self-supervised learning on a large quantity of speech data.  

### Introduction of Contrastive Loss

#### Regression Loss(e.g. RMSE)

This direct regression approach appears to be dominated by a *non-distinguishable broadband component* when speech is present.

Reason why regression loss is ineffective: 

1. our objective requires *maximally distinguishing* different speech segments apart, which doesn't match regression objective.

2. Inappropriate constraint: a regression objective stems from the principle that all of the dimensions of the Mel spectrogram are (1) *equally important* and (2) are *scaled appropriately*: the L2 objective inclines the model to predict low and high frequencies equally well, even if (1) some frequencies (for example, very low) may be irrelevant to speech and (2) some frequencies may vary in orders of magnitudes lowers than others.

#### CLIP Loss

Originally designed to match latent representations in two modalities, text and images.

Leads the model to find a combination of features that maximally discriminates samples in the batch.

Model is inclined to focus on the informative dimensions of the Mel spectrograms and to scale them appropriately.

CLIP Loss擅长区分，擅长学习抓主要矛盾，能自己学习缩放.

具体地，首先在dataset中找N个segment：sample $N−1$ negative samples $\bar{Y}_{j ∈\{1,…,N−1\}}$ over our dataset and we add the positive sample as $\bar{Y}_N=Y$。

$\forall j \in \{1,…,N\}, p_j = \bold{P} [\bar Y_j = Y]$。这里$Y_j$是wave2vec矢量的真值。

而我们通过EEG或MEG得到的是$X$，需要找到一个映射关系$f_{clip}$，使得$Z=f_{clip}(X)$。用$Z$与$\bar Y$中每个$Y_j$做内积即可得到向量间的相似程度，过softmax即得概率分布$\{\hat p_j\}$。优化$\{\hat p_j\}$和真值$\{p_j\}$之间的Cross-Entropy Loss。又因为数据集够大，可以认为N个segment互不相同，$p_N=1$。简化Cross-Entropy Loss为$L_{clip}=-\log{\hat p_n}=-\left<Z,Y\right>+\log{\sum_1^N e^{\left<Z,Y_j\right>}}$。

### Brain Module$f_{clip}$

#### Spatial Attention

用二维平面上的Fourier级数拟合对各个输入通道的Attention权重$a_j(x,y)$，其中$(x_i,y_i)$为输入通道$i$的位置。通过Softmax层加权求和得到输出通道$j$的输出结果$SA(X)^{(j)}$。共$D_1=270$个输出通道。

The initial motivation for spatial attention was to allow for a cross-dataset model to be defined in a way that would generalize
across a diverse number location and set of sensors. Interestingly, we observed this layer to introduce an inductive bias that is beneficial to the prediction accuracy.

<img src=".\imgs\Screenshot 2024-01-27 224657.png" alt="Screenshot 2024-01-27 224657" style="zoom:50%;" />

#### Participant Layer

*To leverage inter-individual variability*, we learn a matrix $M_S\in\bold{R}^{D1,D1}$ for each participant $s\in[S]$ and apply it after the spatial attention layer along the channel dimension.

#### Deep Mel vs. Wav2vec2.0

Deep Mel sees only the audio used in the MEG and EEG datasets.

Wav2vec2.0: pretrained on 56,000 hours of speech from 53 different languages. Effectively encodes a wide variety of linguistic features.

## Experiment Setting

Four public datasets, encompassing 175 volunteers recorded with magneto-encephalography or electro-encephalography while they listened to short stories and isolated sentences.

## Experiments

## Findings & Conclusions

The importance of a contrastive objective, pretrained representations of speech and a common convolutional architecture simultaneously trained across multiple participants.

The analysis of the decoder’s predictions suggests that they primarily depend on **lexical and contextual semantic representations**.  

Our analyses suggest that **advanced MEG and EEG preprocessing does not provide a major advantage** in the current decoding task and that **a simple baseline correction followed by a robust scaler and clamping suffices**.

