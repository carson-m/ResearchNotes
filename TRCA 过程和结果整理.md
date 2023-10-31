# TRCA 的理解、过程和结果整理

按照我的理解，CCA,TRCA方法都是基于相关性的分析方法，核心原理就是
$$
\rho_n=\sum_{m=1}^{N_m}a(m)\bullet{(r_n^{(m)})}^2
$$
用$\rho_n$表示给定响应与第n个激励的响应的相关性，取最大的那个作为识别结果

这一类方法将所有可能激励对应的响应先喂给机器，并且告诉激励和响应的对应关系，而后给一个响应，让机器反推对应激励的方法。具体的反推方法就是将给定响应和已有各个激励的响应计算相关系数，从而得出给定响应和哪个已知激励的响应最相似，从而给出结果。训练过程中用较多的训练数据计算出一个平均响应作为已知响应$\chi$(i.e.参考响应)，这里假定了同一个激励作用于不同实验对象或不同次作用于一个实验对象得到的脑电响应是相似的，较少出现明显的个体差异(只考虑task-related的信号，不考虑无关噪声的影响)，以及不同激励的脑电信号响应满足或近似满足线性性。

这篇研究认为对任意一个通道，其脑电信号响应$x_j(t)=a_js(t)+b_jn(t)$，其中s(t)是"task-related"的，或是说是由激励引起的，n(t)则是其他无关的有着近似随机分布的噪声。如果能够找到一个合适的$\vec{w}=(w_1,w_2,...,w_{N_c})$向量，使得$\sum_{j=1}^{N_c}w_ja_j=1$并且$\sum_{j=1}^{N_c}w_jb_j=0$，就可以尽量压制噪声的影响，提升识别的准确率，这也就是TRCA的核心。寻找合适$w$的方法是多次对同一激励采集脑电响应，寻找$w$向量使得不同次测试之间的协方差之和(或者均值)最大，这样就相当于尽可能消除了随机噪声的影响。这个$w$可以由$Q^{-1}S$最大特征值对应的特征向量得到，其中$Q=Cov(X(t),X(t))$，$S=\sum_{h_1,h_2=1\&h_1\neq h_2}^{N_t}Cov(X^{(h_1)},X^{(h_2)})$。TRCA的 $r_n^{(m)}=\rho({(X^{(m)})}^Tw_n^{(m)},{(\bar{\chi}_n^{(m)})}^Tw_n^{(m)})$，其中$\bar\chi$为训练得到的平均响应。最后通过比较$\rho_n$大小便能得出识别结果。因为本实验使用了filterbank分频段检测，又因为对每个目标$w$会有差异，所以对每个目标n和频段m，有$w_n^{(m)}$。

本实验认为在所涉及到的频段内对不同激励n的响应$x_n(t)$，$w_n^{(m)}$之间有着相似性，所以可以将所有$w_n^{(m)}$并成一个矩阵$W^{(m)}=(w_1^{(m)},......,w_{N_f}^{(m)})$，新的 $r_n^{(m)}=\rho({(X^{(m)})}^TW^{(m)},{(\bar{\chi}_n^{(m)})}^TW^{(m)})$。在计算$X^{(m)}(t)$与$\bar{\chi}_n^{(m)}$之间的相关性时，可综合考虑其他$w^{(m)}$的影响，进一步提高准确率。这给出了第三种方法：Ensemble TRCA

## 过程与结果

在理解这些思路后，我参考https://github.com/mnakanishi/TRCA-SSVEP的代码写了python版本的TRCA和Ensemble TRCA实现。

评价性能的依据是识别的准确率(Accuracy)和信息传输速率(ITR)

$Accuracy = count(result==lable)*100\%$

$ITR=\{\log_2{n}+p*\log_2{p}+(1-p)*\log_2{(\frac{1-p}{n-1})}\}*60/t\ (bits/min)$

其中$p$为准确率，$n$为识别出的目标数，$t$为选择一个目标所需时间(s)

使用我的代码实现的识别结果评估：

**TRCA**

Is Ensemble: False
Trial 1: Accuracy = 100.0000%, ITR = 319.3157 bits/min
Trial 2: Accuracy = 97.5000%, ITR = 301.2679 bits/min
Trial 3: Accuracy = 97.5000%, ITR = 301.2679 bits/min
Trial 4: Accuracy = 95.0000%, ITR = 286.2757 bits/min
Trial 5: Accuracy = 95.0000%, ITR = 286.2757 bits/min
Trial 6: Accuracy = 92.5000%, ITR = 272.4727 bits/min
Summary:
Mean accuracy = 96.2500% (95% CI: 91.5587% - 100.9413%)
Mean ITR = 294.4793 bits/min (95% CI: 265.3213 - 323.6372 bits/min)

**Ensemble TRCA**

Is Ensemble: True
Trial 1: Accuracy = 100.0000%, ITR = 319.3157 bits/min
Trial 2: Accuracy = 100.0000%, ITR = 319.3157 bits/min
Trial 3: Accuracy = 100.0000%, ITR = 319.3157 bits/min
Trial 4: Accuracy = 95.0000%, ITR = 286.2757 bits/min
Trial 5: Accuracy = 92.5000%, ITR = 272.4727 bits/min
Trial 6: Accuracy = 100.0000%, ITR = 319.3157 bits/min
Summary:
Mean accuracy = 97.9167% (95% CI: 91.9714% - 103.8620%)
Mean ITR = 306.0018 bits/min (95% CI: 268.2812 - 343.7225 bits/min)

经过对比，Ensemble TRCA确实相比TRCA有着更高的准确率和信息传输速率

## 过程中遇到的问题

1. 将numpy.linalg.eig的返回值顺序搞反了，导致将特征值当特征向量用了

1. scipy.signal.filtfilt有参数padlen，其默认值为padlen=3\*max(len(b),len(a))，MATLAB中使用padlen=3\*(max(len(b),len(a))-1)，导致使用默认参数时结果不一致

## 关于特征向量标准化的探究

### S2

标准化

Is Ensemble: True
Trial 1: Accuracy = 92.5000%, ITR = 272.4727 bits/min
Trial 2: Accuracy = 87.5000%, ITR = 247.0613 bits/min
Trial 3: Accuracy = 85.0000%, ITR = 235.1566 bits/min
Trial 4: Accuracy = 87.5000%, ITR = 247.0613 bits/min
Trial 5: Accuracy = 92.5000%, ITR = 272.4727 bits/min
Trial 6: Accuracy = 92.5000%, ITR = 272.4727 bits/min
Summary:
Mean accuracy = 89.5833% (95% CI: 83.6380% - 95.5286%)
Mean ITR = 257.7829 bits/min (95% CI: 227.9594 - 287.6064 bits/min)

无标准化

Is Ensemble: True
Trial 1: Accuracy = 92.5000%, ITR = 272.4727 bits/min
Trial 2: Accuracy = 87.5000%, ITR = 247.0613 bits/min
Trial 3: Accuracy = 87.5000%, ITR = 247.0613 bits/min
Trial 4: Accuracy = 87.5000%, ITR = 247.0613 bits/min
Trial 5: Accuracy = 95.0000%, ITR = 286.2757 bits/min
Trial 6: Accuracy = 92.5000%, ITR = 272.4727 bits/min
Summary:
Mean accuracy = 90.4167% (95% CI: 84.4714% - 96.3620%)
Mean ITR = 262.0675 bits/min (95% CI: 231.3045 - 292.8305 bits/min)

### S1

无标准化

Is Ensemble: True
Trial 1: Accuracy = 87.5000%, ITR = 247.0613 bits/min
Trial 2: Accuracy = 87.5000%, ITR = 247.0613 bits/min
Trial 3: Accuracy = 97.5000%, ITR = 301.2679 bits/min
Trial 4: Accuracy = 92.5000%, ITR = 272.4727 bits/min
Trial 5: Accuracy = 100.0000%, ITR = 319.3157 bits/min
Trial 6: Accuracy = 92.5000%, ITR = 272.4727 bits/min
Summary:
Mean accuracy = 92.9167% (95% CI: 83.7862% - 102.0471%)
Mean ITR = 276.6086 bits/min (95% CI: 224.6771 - 328.5401 bits/min)

标准化

Is Ensemble: True
Trial 1: Accuracy = 87.5000%, ITR = 247.0613 bits/min
Trial 2: Accuracy = 87.5000%, ITR = 247.0613 bits/min
Trial 3: Accuracy = 97.5000%, ITR = 301.2679 bits/min
Trial 4: Accuracy = 95.0000%, ITR = 286.2757 bits/min
Trial 5: Accuracy = 100.0000%, ITR = 319.3157 bits/min
Trial 6: Accuracy = 92.5000%, ITR = 272.4727 bits/min
Summary:
Mean accuracy = 93.3333% (95% CI: 84.0940% - 102.5727%)
Mean ITR = 278.9091 bits/min (95% CI: 226.7034 - 331.1147 bits/min)

### S3

无标准化

Is Ensemble: True
Trial 1: Accuracy = 97.5000%, ITR = 301.2679 bits/min
Trial 2: Accuracy = 100.0000%, ITR = 319.3157 bits/min
Trial 3: Accuracy = 97.5000%, ITR = 301.2679 bits/min
Trial 4: Accuracy = 97.5000%, ITR = 301.2679 bits/min
Trial 5: Accuracy = 100.0000%, ITR = 319.3157 bits/min
Trial 6: Accuracy = 100.0000%, ITR = 319.3157 bits/min
Summary:
Mean accuracy = 98.7500% (95% CI: 96.3000% - 101.2000%)
Mean ITR = 310.2918 bits/min (95% CI: 292.6053 - 327.9783 bits/min)

标准化

Is Ensemble: True
Trial 1: Accuracy = 97.5000%, ITR = 301.2679 bits/min
Trial 2: Accuracy = 100.0000%, ITR = 319.3157 bits/min
Trial 3: Accuracy = 97.5000%, ITR = 301.2679 bits/min
Trial 4: Accuracy = 97.5000%, ITR = 301.2679 bits/min
Trial 5: Accuracy = 100.0000%, ITR = 319.3157 bits/min
Trial 6: Accuracy = 100.0000%, ITR = 319.3157 bits/min
Summary:
Mean accuracy = 98.7500% (95% CI: 96.3000% - 101.2000%)
Mean ITR = 310.2918 bits/min (95% CI: 292.6053 - 327.9783 bits/min)