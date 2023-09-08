# TRCA Task-Related Component Analysis
多通道EEG信号$\vec{x(t)}\in{R^{N_{c}(Number\ of\ channels)}}$

$x_{i}(t)=a_{i}s_{i}(t)+b_{i}n_{i}(t)$，其中s为任务相关信号，n为任务无关信号

s具有跨试次不变性，n在试次之间会变化$\rarr$对于某一时间窗口，有$0{\lt}cov(s_{i},s_{i})=const,cov(s_{i},n_{i})=0$

## 算法原理与推导

要从$\vec{x(t)}$中提取$\vec{s(t)}$

首先对多通道EEG信号加权求和
$$
y(t)=\sum^{N_{c}}_{i=1}w_{i}*x_{i}(t)=\sum^{N_c}_{i=1}w_i*(a_i*s_i(t)+b_i*n_i(t))
$$
要达到提取$\vec{s(t)}$的效果，就必须有
$$
\sum^{N_c}_{i=1}w_i*a_i=1,\ \sum^{N_c}_{i=1}w_i*b_i=0
$$
关键就在确定$\vec{w}$上

