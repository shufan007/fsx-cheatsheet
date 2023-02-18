## Time Series

2022最新开源时序模型汇总
https://blog.csdn.net/fengdu78/article/details/124743534

https://zhuanlan.zhihu.com/p/435456194

temporal fusion transformer
https://zhuanlan.zhihu.com/p/383036166 

transformer在时间序列预测中的应用
https://zhuanlan.zhihu.com/p/380457276

时序】应用于时间序列的 Transformer 综述论文笔记
https://blog.csdn.net/weixin_39653948/article/details/123934205

Survey
Transformers in Time Series: A Survey
https://zhuanlan.zhihu.com/p/473235278

如何搭建适合时间序列预测的Transformer模型？
https://zhuanlan.zhihu.com/p/517911547

#### Autoformer
基于深度分解架构和自相关机制的长期序列预测模型
https://zhuanlan.zhihu.com/p/385066440

Auto-Correlation Mechanism的核心思路是利用时间序列的自相关系数，寻找时间序列最相关的片段。时间序列的自相关系数计算时间序列和其滑动一个步长后的时间序列的相关系数。
AutoFormer，计算各个滑动步长的自相关系数，并选择相关系数top k的滑动步长。


#### FEDformer
https://zhuanlan.zhihu.com/p/528131016
https://blog.csdn.net/zandaoguang/article/details/125512540
https://zhuanlan.zhihu.com/p/504365735
传统Transformer为平方复杂度，Autoformer (NeurIPS'21)、Informer (AAAI'21 Best paper)、Reformer (ICLR'2020) 等模型能够达到log-线性复杂度，而本文作者所提出的FEDformer因使用了 low-rank approximation 而可以达到线性复杂度，并在精度上大幅超越SOTA（state-of-the-art）结果。

Transformer在CV、NLP等领域取得了很好的效果，但在时间序列预测问题上，情况会更复杂。例如在图片分类问题中，训练集和测试集的图片基本采样自相同的分布。然而在时间序列预测问题中，序列的分布可能随时间轴的推进不断变化，这就需要模型具备更强的外推能力。如下图所示，因为模型输入（input）和真实值（true）的分布差异较大，导致模型的预测值（predict）不准确。（分布差异的大小可以通过Kologrov-Smirnov test来检验）。
为了解决这个问题，作者提出了两种思路：1，通过周期趋势项分解（seasonal-trend decomposition）降低输入输出的分布差异；2，提出了一种在频域应用注意力机制的模型结构，以增加对噪声的鲁棒性。

虽然无法直接理论证明在频域上应用各种神经网络结构能够得到更强的表征能力。但在实验中发现，引入频域信息可以提高模型的效果，这个现象已经得到近期越来越多论文的证实。

FEDformer 中两个最主要的结构单元的设计灵感正是来源于此。Frequency Enchanced Block（FEB）和 Frequency Enhanced Attention（FEA）具有相同的流程：频域投影 -> 采样 -> 学习 -> 频域补全 -> 投影回时域：


https://github.com/thuml/Autoformer

### Time Series ⏰⌛️Tutorial
https://www.kaggle.com/code/saurav9786/time-series-tutorial

### NeuralProphet
https://www.kaggle.com/code/ohseokkim/predicting-future-by-lstm-prophet-neural-prophet

NeuralProphet is developed in a fully modular architecture which makes it scalable to add any additional components in the future.

### Autoformer 
https://github.com/thuml/Autoformer

https://blog.csdn.net/weixin_43332715/article/details/124536853
https://blog.csdn.net/hymn1993/article/details/124746406?spm=1001.2101.3001.6650.16&utm_medium=distribute.pc_relevant.none-task-blog-2~default~OPENSEARCH~default-16-124746406-blog-124536853.pc_relevant_default&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2~default~OPENSEARCH~default-16-124746406-blog-124536853.pc_relevant_default&utm_relevant_index=20
Auto-Correlation 机制基于时间序列的固有周期性，能够提供 series-wise 连接。

https://zhuanlan.zhihu.com/p/472624073

https://zhuanlan.zhihu.com/p/386955393
旨在高效准确的进行长时间时序预测，所谓时序预测就是使用已知的一段历史时间序列预测未来的一段时间序列。长时间时序预测的意思是预测的未来序列长度远大于已知的历史时间序列长度。

Challenge
原始时间序列中的各种趋势信息比较混乱，无法有效提取时间依赖。
Transformer的模型受限于其二次复杂度不能很好的应用于长时间时序预测。
基于Transformer的改进模型主要着力于稀疏注意力特征图 (QK^T) 来降低模型复杂度 ([公式])，然而这些模型仍使用以节点为最小单位进行消息汇聚，这样将造成信息的丢失，这成为了长时间时序预测的性能瓶颈。

作者提出Auto-Correlation机制替代self-attention，其考虑sub-series间的相似度能更好的捕捉到趋势性，不仅保证了[公式]的复杂度，也防止了信息的丢失，做到了又快又好。
结果提升显著，在六个基准上的效果均取得了极大提升。

### Informer 
https://zhuanlan.zhihu.com/p/505466261

### dataset

 
时空数据集：
交通
1、pems系列：pems04、pems07、pems08（可以从官网下载原始数据，或者github上有处理好的tensor数据），记录了美国加州部分高速公路网数据。https://github.com/divanoresia/Traffic，
2、METR-LA
气象预报
1、气象数据及模型：https://github.com/BIRD-TAO/CLCRN
2、中国气象网爬虫，时间粒度一个小时，空间划分以经纬度信息划分。
空气质量预测
1、预处理好数据：https://pan.baidu.com/s/1cln1jpLJYP9BdBH7irrWUg 提取码：057p。以北京为中心的北京（bj）、石家庄（sjz）、太原（ty）、呼和浩特（hhht）、大连（dl）五个城市的数据，以组成城市尺度图（city-scale），加一个区域尺度（region-scale）的数据（mid、midscale）
能源
1、风力发电：2022kdd：https://aistudio.baidu.com/aistudio/competition/detail/152/0/introduction
疫情
1、2019新型冠状病毒（COVID-19）世界范围传播数据：https://www.heywhale.com/mw/dataset/5e437a805f2816002cea0410

时空算法及代码：

Conv-LSTM：
https://github.com/ndrplz/ConvLSTM_pytorch
ConvLSTM最早由《Convolutional LSTM Network: A Machine Learning Approach for Precipitation Nowcasting》论文提出，目的是用于解决降水预报问题。降水预报问题通常被看做时序上的问题，因此被考虑使用LSTM来解决，但是单纯的LSTM不能通过图片来利用空间上的数据特征，因此空间特征在这个LSTM方法中利用是很不充分的。根据以上的描述，论文提出一种ConvLSTM结构,不仅能够建立类似LSTM时序关系，而且可以拥有类似CNN的空间特征提取能力。并且作者通过实验证明了ConvLSTM在获取时空关系上比LSTM有更好的效果。而且ConvLSTM不仅可以预测天气，还能够解决其他时空序列的预测问题。比如视频分类，动作识别等。

STGCN：
https://github.com/hazdzz/STGCN 

GAT：
https://github.com/PetarV-/GAT

DCRNN：
https://github.com/liyaguang/DCRNN

ASTGCN：
https://github.com/Davidham3/ASTGCN
同时采用图卷积和注意力机制去对空间网络结构进行建模

STSGCN：
https://github.com/mcdragon/STSGCN
提出了一种新颖的时空图卷积模块，以直接同步地捕获局部时空相关性，而不是分别使用不同类型的神经网络模块。
构造了一个多模块层以捕获远程时空图中的异质性。这个多模块层在每个时间段上部署多个模块，从而使每个模块可以专注于提取每个局部时空图上的时空相关性。（异质性，比如不同时间段住宅和商业区呈现出不同的模式）

GraphWaveNet：
https://github.com/SGT-LIM/GraphWavenet
基本组件由图卷积网络和时间卷积网络组成

AGCRN：
https://github.com/LeiBAI/AGCRN
采用动态自适应学习图的邻接矩阵，采用切比雪夫拟合卷积核作为gcn空间卷积模块的核心内容，将空间学习模块作为gru学习单元的门控嵌入单元，实现时空学习。

时空开源代码框架：
阡陌：交通预测领域，支持交通状态预测(流量、速度、需求、起点-终点（OD）矩阵、事故预测）、轨迹下一跳预测、到达时间预测、路网匹配、路网表征学习：https://github.com/LibCity/Bigscity-LibCity



#### six benchmarks
six benchmarks, covering five practical applications: energy, traffic, economics, weather and disease.
https://cloud.tsinghua.edu.cn/d/e1ccfff39ad541908bae/

#### ETT
https://www.heywhale.com/mw/notebook/603c552704f2500015a15d37

UCR datasets
https://www.cs.ucr.edu/~eamonn/time_series_data/

Google Brain - Ventilator Pressure Prediction

https://www.kaggle.com/competitions/ventilator-pressure-prediction/data
https://towardsdatascience.com/key-takeaways-from-kaggles-most-recent-time-series-competition-ventilator-pressure-prediction-7a1d2e4e0131
https://zhuanlan.zhihu.com/p/433819132
https://blog.csdn.net/weixin_45794268/article/details/121171605

https://zhuanlan.zhihu.com/p/429649462

NATOPS dataset

### 开源工具
### tsai 
https://github.com/timeseriesAI/tsai 
https://timeseriesai.github.io/tsai/models.TabModel.html
https://timeseriesai.github.io/tsai/data.preparation.html 

State-of-the-art Deep Learning library for Time Series and Sequences.

tsai is an open-source deep learning package built on top of Pytorch & fastai focused on state-of-the-art techniques for time series tasks like classification, regression, forecasting, imputation...

install tsai

pip install tsai, fastcore, fastai, imblearn, pyts
or
pip install tsai[extras]

### pytorch-forecasting(Github 1.9K+)
https://github.com/jdb78/pytorch-forecasting

### tsfresh
https://tsfresh.readthedocs.io/en/latest/

### FOST
https://github.com/microsoft/FOST
FOST(Forecasting open source tool) aims to provide an easy-use tool for spatial-temporal forecasting. The users only need to organize their data into a certain format and then get the prediction results with one command. FOST automatically handles the missing and abnormal values, and captures both spatial and temporal correlations efficiently.
#### Framework of FOST
Preprocessing	Preprocessing module aims at handle varies data situation, currently FOST designed sub-module to handle issues such as missing value, unalignment timestamp and feature selection.
Modeling	FOST contains implements for different mainstream deep learning models such as RNN, MLP and GNN, for better performance on varies custom data. Further model implements such as Transformer, N-beats are in progress.
Fusion	Fusion module aims at automatically select and ensemble model predictions.
Utils	There are many other utils in FOST, such as neural-network trainer and predictor, result plotter and so on.

FEDformer 
https://github.com/DAMO-DI-ML/ICML2022-FEDformer

#### Data Format
1. train.csv
3 columns are required for train.csv:

Node: node name for current data
Date: date or timestamp for current data
TARGET: target for prediction

2. graph.csv (option)
graph.csv should only contains 3 columns:

node_0: node name for fist node, node name should align with node name in train.csv.
node_1: node name for second node, node name should align with node name in train.csv.
weight: weight on connection for node_0 to node_1.

### 模型

#### TFT模型

https://towardsdatascience.com/temporal-fusion-transformer-googles-model-for-interpretable-time-series-forecasting-5aa17beb621

https://zhuanlan.zhihu.com/p/461795429

https://zhuanlan.zhihu.com/p/514287527 

Spatio-Temporal Forecasting
在时空预测中，需要同时考虑时间（temporal）和时空（spatio-temporal）依赖性以进行准确的预测。

Traffic Transformer 设计了一种编码器-解码器结构，使用自注意力模块来捕获时间-时间依赖关系，并使用图神经网络模块来捕获空间依赖关系。

用于交通流量预测的 Spatial-temporal Transformer 更进一步。除了引入时间 Transformer 块来捕获时间依赖关系之外，它还设计了一个空间 Transformer 块，以及一个图卷积网络，以更好地捕捉空间-空间依赖关系。
https://blog.csdn.net/zuiyishihefang/article/details/125775787

Spatio-temporal graph Transformer 设计了一种基于注意力的图卷积机制，能够学习复杂的时空注意力模式来改进行人轨迹预测。


————————————————


问题：

变量选择网络 (VSN) 的工作原理如下：

将 GRN 单独应用于每个特征。
在所有特征的串联上应用 GRN，然后是 softmax 以产生特征权重。
生成单个 GRN 输出的加权总和。

ML-as-a-service 
Michelangelo
https://eng.uber.com/michelangelo-machine-learning-platform/

Orbit
Uber开源Orbit 项目介绍：用于时间序列推断与预测  
Orbit 是 Object-ORiented BayesIan Time Series 的简称（意为“面向对象的贝叶斯时间序列）。Orbit 利用结构贝叶斯时间序列模型进行时间序列推断和预测，并将其应用于实际案例分析中。与其他很多机器学习用例一样，最佳的模型结构主要依赖于特定用例和可用数据。识别合适模型的常用方法是尝试不同的模型类型，进行回溯测试，并判断其性能。具有一致的接口来完成所有这些任务，这大大简化了我们为特定用例确定最佳模型的能力。


### data type

静态变量：（类别型，数值型）
动态变量：（类别型，数值型）
  动态时变变量
  动态时不变变量


输入数据处理方案：

静态变量：
 类别型 -> embedding : stc_emb
 数值型 -> Dense: stc_out
动态变量：（类别型，数值型）
  动态时变变量 -> RNN : d_out
  动态时不变变量

上面变换后的输出全部concat，输入N层全连接层

近24小时时序特征 + 近14天时序特征 + 离散特征 + 连续特征
构建模型，对14天、24小时时序特征使用GRU，离散特征做Emb，concat后做DNN，输出in、out
另外，还对时间排列构建了最大池化输出。



表格数据预处理可以参考下这个project
https://github.com/NVIDIA-Merlin/NVTabular  


### Automl时序预测任务方案

时序预测任务简述

时间序列预测分析就是利用过去一段时间内某事件时间的特征来预测未来一段时间内该事件的特征。 
时间序列模型是依赖于事件发生的先后顺序的。时间序列可以分为平稳序列，即存在某种周期，季节性及趋势的方差和均值不随时间变化的序列，及非平稳序列。

数据类型
单变量
Data Format：
long sequence, or
[# samples x  sequence length]
多变量
Data Format：
[# samples x # variables x sequence length]

多时序
暂不考虑
静态+时序
Data Format：
[# samples x # variables ]
[# samples x # variables x sequence length1]
[# samples x # variables x sequence length2]
......

空间数据/graph
Data Format
1. train data
3 columns are required for train:
Node: node name for current data
Date: date or timestamp for current data
TARGET: target for prediction
 2. graph (option)
graph should only contains 3 columns:
node_0: node name for fist node, node name should align with node name in train.
node_1: node name for second node, node name should align with node name in train.
weight: weight on connection for node_0 to node_1.
 

模型算法
  
统计方法： 
AR,MA,ARMA，ARIMA，Prophet 等。适用于单时间序列问题。
适用场景
适用于具有明显的内在规律的商业行为数据,例如：有如下特征的业务问题：
a.有至少几个月（最好是一年）的每小时、每天或每周观察的历史数据；
b.有多种人类规模级别的较强的季节性趋势：每周的一些天和每年的一些时间；
c.有事先知道的以不定期的间隔发生的重要节假日（比如国庆节）；
d.缺失的历史数据或较大的异常数据的数量在合理范围内；
e.有历史趋势的变化（比如因为产品发布）；
f.对于数据中蕴含的非线性增长的趋势都有一个自然极限或饱和状态。

ARIMA 模型
将自回归模型（AR）、移动平均模型（MA）和差分法结合，就得到了差分自回归移动平均模型 ARIMA（p、d、q），其中 d 是需要对数据进行差分的阶数。
生成 ARIMA 模型的基本步骤：
对序列绘图，进行 ADF 检验，观察序列是否平稳；对于非平稳时间序列要先进行 d 阶差分，转化为平稳时间序列；经过第一步处理，已经得到平稳时间序列。要对平稳时间序列分别求得其自相关系数（ACF）和偏自相关系数（PACF），通过对自相关图和偏自相关图的分析，得到最佳的阶数p、q；由以上得到的d、q、p ，得到 ARIMA 模型。然后开始对得到的模型进行模型检验。

NeuralProphet 全局模型， 可支持多时序

深度学习方法
seq2seq/transformer类，循环神经网络RNN，卷积神经网络CNN，空间卷积网络TCN等。
具体算法：LSTM ， GRU，FCN（Fully Convolutional Network），TCN（Temporal Convolutional Network），TST（Time Series Transformer）等
开源工具：tsai
LSTM +Attention

静态+ 时序 融合（concat）：处理多个时间序列并接受更丰富的特征集。
静态变量：
 类别型 -> embedding : stc_emb
 数值型 -> Dense: stc_out
动态变量：（类别型，数值型）
  动态时变变量 -> RNN : d_out
  动态时不变变量
  
变换后的输出全部concat，输入N层全连接层

TFT（Temporal Fusion Transformer）
为了学习不同尺度的时间关系，TFT 使用循环层进行局部处理，并使用可解释的自注意力层进行长期依赖学习。

automl 模型接口
  使用automl搜索步骤：
  定义模型
  定义训练流程，训练函数可接收搜索配置参数
  定义参数搜索空间
  模型搜索：
  模型类型，几种模型分别定义objective，搜索空间，模型预留搜索配置参数

数据预处理
时序数据转训练样本
特征提取：
 工具：tsfresh
预处理：
[Categorify, FillMissing, Normalize]


