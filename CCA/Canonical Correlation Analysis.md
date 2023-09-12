# Canonical Correlation Analysis
**CCA 典型相关分析**

用来探索两个多变量（**向量**）之间的关联关系，这两个多变量来自于一个相同的个体

举例：调查家庭特征与家庭消费之间的关系
设X1为每年去餐馆就餐的次数，X2为每年外出看电影的频率
Y1为户主年龄，Y2为家庭年收入，Y3为户主受教育程度
如果直接用协方差矩阵的话为5维(X1,X2,Y1,Y2,Y3)，而且难以提取有用的信息
现做线性变换，将X1,X2线性组合为2维的U，Y1,Y2,Y3组合为2维的V
于是有
$$
Var(U_{i})=A_{\_i}^{T}cov(X,X)A_{\_i}\\
Var(V_{j})=B_{\_j}^{T}cov(Y,Y)B_{\_j}\\
cov(U_{i},V_{j})=A_{\_i}^{T}cov(X,Y)B_{\_j}\\
\rho(U_{i},V_{j})=corr(U_{i},V_{j})=\frac{cov(U_{i},V_{j})}{\sqrt{Var(U_{i})}\sqrt{Var(V_{j})}}
$$

一般有两个典型的目的：
	Data Reduction：用少量的线性组合来解释两组变量之间的相关作用。
	Data Interpretation：寻找特征值，这些特征值对于解释两个变量集合之间的相互作用十分关键。

## Canonical Variates 典型变量
待研究变量
$$
X=(X_{1}, X_{2},......, X_{p})^{T}
$$
$$
Y=(Y_{1}, Y_{2},......, Y_{q})^{T}
$$
$$
(p\le q)
$$
U, V分别为X, Y的线性组合，注意维数变化
$$
U=X\begin{bmatrix}
a_{11} & a_{12} & ....&a_{1p}\\
a_{21}&a_{22}&....&a_{2p}\\
.&.&.&.\\
a_{p1}&a_{p2}&....&a_{pp}
\end{bmatrix}\\
V=Y\begin{bmatrix}
b_{11} & b_{12} & ....&b_{1q}\\
b_{21}&b_{22}&....&b_{2q}\\
.&.&.&.\\
b_{p1}&b_{p2}&....&b_{pq}
\end{bmatrix}
$$
定义
$$
(U_{i}, V_{i})
$$
为第i个**典型变量对(canonical variate pair)**，共p对

## 典型相关 canonical correlation
一种特定的相关，指第i个典型变量对中$U_{i}$与$V_{i}$的相关性
$$
{\rho_{i}}^{*}=\frac{cov(U_{i},V_{i})}{\sqrt{Var(U_{i})}\sqrt{Var(V_{i})}}
$$

## 方法与思想
1. 首先找到$A_{1\_}$与$B_{1\_}$，s.t.$U_{1}$与$V_{1}$的相关性最大

2. 在使$(U_{2},U_{1}),(V_{2},V_{1}),(U_{2},V_{1}),(V_{2},U_{1})$相互不关联(cov=0)的情况下，使$U_{2},V_{2}$的关联性有次大的相关性

3. 以此类推，使$Var(U_{i})=Var(V_{j})=1$，同时除$cov(U_{i},V_{i})$之外的协方差全为0，直到进行到第r部(r<=min(p,q))，两组数据的关联性被提取完毕为止，可得r组变量(r对a和b)

4. 由于$Var(U_{i})=Var(V_{i})=1$，故${\rho_{i}}^{*}=cov(U_{i},V_{i})$

相当于在Var与cov的约束下求$\rho$的最大值，即条件极值问题

## 实操

$$
cov(\begin{bmatrix}x\\y\end{bmatrix})=\begin{bmatrix}cov(x,x)&cov(x,y)\\cov(y,x)&cov(y,y)\end{bmatrix}=\begin{bmatrix}S_{xx}&S_{xy}\\S_{yx}&S_{yy}\end{bmatrix}
$$

若x,y已经标准化，则有
$$
E(X)=[0],E(Y)=[0]
$$
现在要寻找A,B, s.t.
$$
arg\ max_{A_{\_i},B_{\_i}}\ \frac{cov(U_i,V_i)}{\sqrt{Var(U_i)}\sqrt{Var(V_i)}}
$$
即
$$
arg\ max_{A_{\_i},B_{\_i}}\ \frac{A_{\_i}^{T}S_{xy}B_{\_i}}{\sqrt{A_{\_i}^{T}S_{xx}A_{\_i}}\sqrt{B_{\_i}^{T}S_{yy}B_{\_i}}}
$$
**A**和**B**增大相同的倍数最优化的值不改变，所以我们需要应用**svm**的技巧，**固定分母，只优化分子**

原问题变为：在$Var(U_{i})=Var(V_{i})=1$时，寻找$arg\ max_{A_{\_i},B_{\_i}}\ {A_{\_i}^{T}S_{xy}B_{\_i}}$

构造Lagrange等式，得
$$
S_{xx}^{-1}S_{xy}B_{\_i}=\lambda A_{\_i}\\
S_{yy}^{-1}S_{yx}A_{\_i}=\lambda B_{\_i}
$$
整理得
$$
S_{xx}^{-1}S_{xy}S_{yy}^{-1}S_{yx}A_{\_i}=\lambda^2 A_{\_i}
$$
求矩阵
$$
S_{xx}^{-1}S_{xy}S_{yy}^{-1}S_{yx}
$$
的特征值和特征向量即可进一步计算出$\lambda$与**A**，同理可得**B**。
