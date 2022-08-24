# 声纹识别教程

## Syllabus

* The history of Voice Identity Techniques
* Fundamentals of Audio Processing
* Acoustic Feature Extraction
* Fundamentals of Speaker Recognition
* Early Speaker Recognition Approaches
* Deep Learning Basics（前馈、卷积、循环神经网络；注意力机制）
* Speaker Recognition with Deep Learning
* Data Processing in Speaker Recognition（数据增强、数据融合、常用数据集）



## 1. The history of Voice Identity Techniques

> 声纹技术算法：从简单到复杂（基于DL）的演进过程

### 1.1 What is voice identity？

可以从指纹开始理解。指纹是手指触摸物体后留在其表面的纹路。从古代就开始有应用：

* Sign Written contrast（ancient Egypt，Greek，China）
* Criminal investigation
* Biometric screening for a visa
* unlock your phone

为什么指纹有广泛的应用？因为其唯一性，标志性，每个人指纹不相同。常见特征如下：

* ridge bifurcation/endings
* Arches
* Loops
* Whorls

区别在于：Number、Shape、Position

声纹同样是独一无二的，每个人的声纹都不一样，基本事实是：

* Voices are also different from person to person

导致声纹差异的原因：

* Size of shape of the speech organs
    * Vocal folds
    * Vocal tract

举例：

* Gender
    * male：deeper
    * female：higher
* Age
    * child：tender
    * senior：wavery



### 1.2 Biometrics

| Physiological characteristics | Behavioral characteristics |
| ----------------------------- | -------------------------- |
| 1. Fingerprint                | 1. Voice                   |
| 2. DNA                        | 2. Signature               |
| 3. Face                       | 3. Gait                    |
| 4. Iris                       |                            |



### 1.3 The Earliest voice-id techniques

* 1962，Voiceprint identification，use spectrogram，human reading not algorithm

* Earliest Algorithm：**Pattern Matching**

    收到人工阅读法的启发，但不再采用人工比对时域频谱图，而是**模式匹配算法**。

    > 可以将两个声纹信息的时域频谱图看作两个矩阵A和B（因为横纵坐标分别是时间和频率）。
    >
    > 核心矛盾是得到两个矩阵的差异。
    >
    > 方法一：
    >
    > 对两个矩阵求差值（依然得到一个矩阵）进行**求模**运算，即：$\begin{Vmatrix}{A-B}\end{Vmatrix}_2$
    >
    > 方法二：
    >
    > 求两个矩阵的**相关性**，即：$corr(A,B)$。相关性越接近于1，说明两个矩阵越相似，反之越接近0越不相似。
    >
    > 模式匹配算法的前提条件是**文本相同**，这样才能保证矩阵是同等规模的，才能进行运算。



### 1.4 From waveform(oscillography) to spectrogram

对声纹研究的手段，从示波器的波形图演变为时域频谱图。

* 阶段一：示波器波形图
* 阶段二：时域频谱图（人工但非算法）



## 2. Fundamentals of Audio Processing

### 2.1 Audacity

音频分析、处理、可视化工具——Audacity，该工具具有如下几个优势：

* 开源、免费
* 跨平台

