# Task-related Component Analysis(TRCA)

## Relevant Techniques

$$
\vec{u}=X\ \vec{a}\\
\vec{v}=Y\ \vec{b}\\\\
\begin{align}
Cov(\vec{u},\vec{v})&=E(\vec{u}^T\vec{v})-E(\vec{u})^TE(\vec{v})\\
&=E(\vec{a}^TX^TY\vec{b})-(E(X)\vec{a})^TE(Y)\vec{b}\\
&=\vec{a}^T[E(X^TY)-(EX)^TEY]\vec{b}\\
&=\vec{a}^TCov(X,Y)\vec{b}
\end{align}
$$

## Intro

a data analysis technique

neuroscience & ML

useful dealing with multi dimensional data or signals

extract & analyze components of the data that are specifically related to a given task or stimulus, while filtering out irrelevant or noise-related components.

by comparing the responses of multiple sensors or channels to different task conditions or stimuli and identifying common patterns or components that are task-related.

## Features

- Noise Reduction: One of the advantages of TRCA is its ability to suppress noise and extract task-related information, making it particularly valuable in scenarios where the signal-to-noise ratio is low.
- Discriminative Power: TRCA is designed to enhance the discriminative power of the extracted components, making it easier to distinguish between different task-related conditions or stimuli.
- Multivariate Analysis: TRCA typically involves the simultaneous analysis of multiple data channels or sensors. This multivariate approach allows it to capture complex relationships between different components of the data.

