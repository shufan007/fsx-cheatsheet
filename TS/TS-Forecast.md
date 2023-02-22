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

输出格式
单步
[# samples x 1 ]

多步 Multi-horizon
[# samples x # steps ]

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


数据预处理
时序数据转训练样本
特征提取：
 工具：tsfresh
预处理：
[Categorify, FillMissing, Normalize]

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


算法类别：
seq2seq:优势在于便于在不同序列长度输入（T_in）输出（T_out）之间进行变换，缺点是对于长序列有信息压缩。

transformer：能够规避seq2seq中较长的序列数据所产生的信息丢失问题。对于原生transformer常见的变体有：LogSparse Transformer，TFT（Temporal Fusion Transformer），TST（Time Series Transformer），Informer（PorbSpare self-attention）， FEDformer (ICML 2022 paper)

循环神经网络RNN：
卷积神经网络CNN：
空间卷积网络TCN等。

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

TFT关键点
Multi-horizon: 多视界，即多步预测。
LSTM: seq2seq生成中间的时间序列特征。
Self-Attention: 融合时间序列上的特征。单层、可解释的Multi-head Attention。
Quantile Loss: 分位损失，预测多个分位的结果。
三类输入
static input: 不随时间改变的静态属性，如商品的类别、功效等。
observed input: 在过去时间节点上观察到的信息，如当天浏览量、当天油价、当天天气等。
known input: 在所有时间节点上（过去/将来）可知的信息，如过去/将来的价格，过去/将来的营销计划等。

Transformer
transformer本质上是一个信息顺序传递的模型。主要包含了positional encoding(用于区分出现的先后顺序)，self-attention, attention, 以及feed forward网络四大部分。与RNN不同的是，Transformer利用了attention机制进行信息传递。

highlights：
encoder-decoder框架

embedding
编码部分由三部分组成，TokenEmbedding + PositionalEmbedding + TemporalEmbedding。
TokenEmbedding将输入特征转化为特征向量；
PositionalEmbedding表示位置编码；

TemporalEmbedding表示时间戳编码；optional：如果不输入时间特征则无；也可表示字段的类别编码。

encoder
multiheadAttention + norm + feedforward + norm 

FullAttention
Attention主要分三步：使用矩阵乘法计算query 和 key 的相似度得到权值；将权值进行归一化，得到直接可用的权重；将权重和 value 进行加权求和。

transformer使用fullattention+selfattention作为注意力机制，其中full表示attention的计算区域（如图），即对所有key求权重概率，每个key都有一个对应的权重，是一种全局的计算方式（也可以叫Global Attention；self表示attention的输入数据，这种情况下Q,K,V都是同一个X经过线性变换之后的结果，这样一来输出结果就是跟X一样长的向量序列，并且能够直接捕捉X中任意两个向量的关联，而且易于并行。

feedforward
采用两层一维卷积实现。

decoder
multiheadAttention + norm + crossAttention + norm + feedforward + norm 
maskmultiheadAttention
maskmultiheadAttention使用fullattention+selfattention作为第一层注意力机制。
multiheadAttention
multiheadAttention使用fullattention + crossattention作为第二层注意力机制；中crossattention使用第一层注意力机制的输出作为query，encoder的输出作为key和value。
feedforward
采用两层一维卷积实现。


LogsparseTransformer
highlights：
与transformer不同的是使用了空洞自卷积注意力（即稀疏attention），来解决训练效率问题。

稀疏attention
在图1的标准attention中，每一个元素都跟序列内所有元素有关联。计算时间和显存占用量都是𝒪(n2)级别的（n是序列长度），这就意味着如果序列长度变成原来的2倍，显存占用量就是原来的4倍，计算时间也是原来的4倍。
所以，如果要节省显存，加快计算速度，那么一个基本的思路就是减少关联性的计算，也就是认为每个元素只跟序列内的一部分元素相关，这就是稀疏Attention的基本原理。

空洞attention
如下图2所示，它对相关性进行了约束，强行要求每个元素只跟它相对距离为k,2k,3k,…的元素关联，其中k>1是预先设定的超参数。从下左的注意力矩阵看，就是强行要求相对距离不是k的倍数的注意力为0（白色代表0）：


Informer
highlights：
encoder-decoder框架
informer提出了ProbSparse self-attention层(ProbAttention)，主要也是从attention入手，进行不同于以往的稀疏attention。

ProbAttention
ProbSparse Self-Attention


Autoformer
highlights：
encoder-decoder框架
autoformer提出了突破将序列分解作为预处理的传统方法，提出深度分解架构（Decomposition Architecture），能够从复杂时间模式中分解出可预测性更强的组分。
基于随机过程理论，提出自相关机制（Auto-Correlation Mechanism），代替点向连接的注意力机制，实现序列级（series-wise）连接和复杂度，打破信息利用瓶颈。

decomposition
Autoformer将序列分解作为一个内部单元嵌入到编-解码器中。在预测过程中，模型交替进行预测结果优化和序列分解，即从隐变量中逐步分离趋势项与周期项，实现渐进式分解。
序列分解单元（series decomposition block）基于滑动平均思想，得到平滑周期项、突出趋势项：
其中，X为待分解的隐变量，Xt, Xs分别为趋势项和周期项，这种序列分解单元可以嵌入Autoformer层间。

autocorrelation
Autoformer提出了自相关机制来实现高效的序列连接，从而扩展信息效用。由于不同周期的相似相位之间通常表现出相似的子过程，利用这种序列固有的周期性来设计自相关机制，其中，包含基于周期的依赖发现（Period-based dependencies）和时延信息聚合（Time delay aggregation）。

FEDformer (Frequency Enhanced Decomposed Transformer )
highlights：
encoder-decoder框架
fedformer设计了两个注意模块，分别用傅里叶变换和小波变换处理频域中的注意操作。 通过傅里叶变换中的随机模式部分实现线性复杂度。
提出傅立叶增强块和小波增强块，通过频域映射捕获重要结构，代替自我注意和交叉注意
随机选择固定数量的傅里叶分量，该模型实现了线性计算复杂度和存储成本

MOEDecomp
由于周期模式与数据的趋势成分相结合，使用固定窗口平均池化很难提取趋势（autoformer提取趋势的做法）。我们设计混合专家分解块(MOEDecomp)，包含一组不同大小的平均滤波器，从输入信号提取多个趋势成分，以及相关权值，将它们组合成最终的趋势。

Attention
1、Fourier
FEDformer 中两个最主要的结构单元的设计灵感正是来源于此。Frequency Enchanced Block（FEB）和 Frequency Enhanced Attention（FEA）具有相同的流程：频域投影 -> 采样 -> 学习 -> 频域补全 -> 投影回时域： 

1. 首先将原始时域上的输入序列投影到频域。 

2. 在频域上进行随机采样。传统 Transformer 中采用的 Attention 机制是平方复杂度，而 Frequency Enhanced Attention（FEA）中采用的 Attention 是线性复杂度，这极大提高了计算效率。因为 FEA 在频域上进行了采样操作,采样后得到的小矩阵，是对原矩阵的低秩近似。这样做的好处在于极大地降低了输入向量的长度进而降低了计算复杂度,然而这种采样对输入的信息一定是有损的。作者对低秩近似与信息损失的关系进行了研究，并通过理论证明，在频域随机采样的低秩近似法造成的信息损失不会超过一个明确的上界。最终实验证明这种损失对最终的精度影响不大。因为一般信号在频域上相对时域更加“稀疏”。且在高频部分的大量信息是所谓“噪音”，这些“噪音”在时间序列预测问题上往往是可以舍弃的，因为“噪音”往往代表随机产生的部分因而无法预测。相比之下，在图像领域，高频部分的“噪音”可能代表的是图片细节反而不能忽略。

3. 在学习阶段，FEB 采用一个全联接层 R 作为可学习的参数。而 FEA 则将来自编码器和解码器的信号进行 cross-attention 操作，以达到将两部分信号的内在关系进行学习的目的。 

4. 频域补全过程与第 2 步频域采样相对，为了使得信号能够还原回原始的长度，需要对第 2 步采样未被采到的频率点补零。 

5. 投影回时域，因为第 4 步的补全操作，投影回频域的信号和之前的输入信号维度完全一致。


时空算法及代码：

Conv-LSTM：
https://github.com/ndrplz/ConvLSTM_pytorch
ConvLSTM最早由《Convolutional LSTM Network: A Machine Learning Approach for Precipitation Nowcasting》论文提出，目的是用于解决降水预报问题。降水预报问题通常被看做时序上的问题，因此被考虑使用LSTM来解决，但是单纯的LSTM不能通过图片来利用空间上的数据特征，因此空间特征在这个LSTM方法中利用是很不充分的。根据以上的描述，论文提出一种ConvLSTM结构,不仅能够建立类似LSTM时序关系，而且可以拥有类似CNN的空间特征提取能力。并且作者通过实验证明了ConvLSTM在获取时空关系上比LSTM有更好的效果。而且ConvLSTM不仅可以预测天气，还能够解决其他时空序列的预测问题。比如视频分类，动作识别等。

STGCN：
https://github.com/hazdzz/STGCN
网络输入是 M个时间步的图的特征向量 X以及对应的邻接矩阵 W，经过两个时空卷积块和一个输出层，输出 v来预测第 t个时间步后某个时间步特征。


每个时空卷积块由两个时域卷积块和一个空域卷积块组成，其中时域卷积块如上图最右侧所示，对于输入的每个节点沿着时间维度进行一维卷积，对于空域卷积块则是在每个时间步的图上进行（不在时间步之间进行）。
在每个时域卷积块和空域卷积块中都使用了残差连接
输出层包括一个时域卷积层和一个全连接层

GAT：
https://github.com/PetarV-/GAT


DCRNN：
https://github.com/liyaguang/DCRNN
空间依赖建模
将交通流与扩散过程相关联来对空间依赖性进行建模，这明确地捕捉了交通动力学的随机性质。扩散形式以G上的随机游走来刻画，重启概率为α ∈ [0，1]，状态转移矩阵为DO-1W。这里DO= diag(W1)是外度对角矩阵，1∈RN表示全一向量。经过许多时间步长后，这种马尔可夫过程收敛到一个平稳分布P∈ RN×N，行Pi，：∈RN表示从节点vi∈ V扩散的可能性。下面的引理为平稳分布提供了一个封闭形式的解。

ASTGCN：
https://github.com/Davidham3/ASTGCN
同时采用图卷积和注意力机制去对空间网络结构进行建模
模型结构：

框架主要包含3个部分，分别提取邻近、日、周依赖特征。recent部分包含邻近的T个时段，daily-period 部分包含前一天或多天与预测时段相同的多个时间序列，weekly-period部分前一周或多周与预测时段相同的多个时间序列。该三部分具有相同的网络结构，每部分由多个时空块和一个全连接层组成。在每个时空块中都有时空注意力模块和时空卷积模块。为了优化训练效率，文章采用了残差连接。最后，利用一个参数矩阵对三个分量加权合并，得到最终预测结果。
特征提取
假设采样频率为q次/天。假设当前时间为t0，预测窗口大小为Tp。如图所示，在时间轴上截取长度为Th、Td和Tw的三个时间序列片段，分别作为相邻、日周期和周周期分量的输入，其中Th、Td和Tw都是Tp的整数倍：

STSGCN：
https://github.com/mcdragon/STSGCN
提出了一种新颖的时空图卷积模块，以直接同步地捕获局部时空相关性，而不是分别使用不同类型的神经网络模块。
构造了一个多模块层以捕获远程时空图中的异质性。这个多模块层在每个时间段上部署多个模块，从而使每个模块可以专注于提取每个局部时空图上的时空相关性。（异质性，比如不同时间段住宅和商业区呈现出不


GraphWaveNet：
https://github.com/SGT-LIM/GraphWavenet
基本组件由图卷积网络和时间卷积网络组成
图卷积网络
图卷积神经网络主要用于捕捉空间依赖关系，采用了扩散卷积（diffusion GCN），这种卷积方式将图信号的扩散过程表示为有限的K步转移矩阵。其基本公式表示为：


AGCRN：
https://github.com/LeiBAI/AGCRN
采用动态自适应学习图的邻接矩阵，采用切比雪夫拟合卷积核作为gcn空间卷积模块的核心内容，将空间学习模块作为gru学习单元的门控嵌入单元，实现时空学习。
很多算法去挖图卷积的不同空间聚合方法（矩阵A的形式），而agcrn探究的是参数W，从特征变换矩阵入手，挖掘了序列中不同的模式，并且用了矩阵分解结合节点嵌入方法，给每个节点学习了一个特定的参数空间，减少计算量的同时也增加了解释性。
（1）Node Adaptive Parameter Learning (NAPL)，论文的核心创新点，给每个节点上的序列学习特定的模式，同时考虑到计算量的问题，做了个矩阵分解。
（2）Data Adaptive Graph Generation (DAGG) ，把邻接矩阵当做学习的参数，与Graph WaveNet方法雷同。
（3）把图卷积和GRU结合，提出了其模型Adaptive Graph Convolutional Recurrent Network，适应性图卷积循环网络，简称AGCRN。


STGRAT：https://github.com/LMissher/ST-GRAT

时空算法文献：
Convolutional LSTM Network: A Machine Learning Approach for Precipitation Nowcasting，https://arxiv.org/abs/1506.04214
IJCAI2018，Spatio-Temporal Graph Convolutional Networks: A Deep Learning Framework for Traffic Forecasting：https://arxiv.org/abs/1709.04875v4
Graph Attention Networks，http://arxiv.org/abs/1710.10903v3
Diffusion Convolutional Recurrent Neural Network: Data-Driven Traffic Forecasting，https://arxiv.org/abs/1707.01926
Spatial-Temporal Synchronous Graph Convolutional Networks: A New Framework for Spatial-Temporal Network Data Forecasting，https://ojs.aaai.org/index.php/AAAI/article/view/5438
Attention Based Spatial-Temporal Graph Convolutional Networks for Traffic Flow Forecasting，https://aaai.org/ojs/index.php/AAAI/article/download/3881/3759
Graph WaveNet for Deep Spatial-Temporal Graph Modeling，https://arxiv.org/abs/1906.00121v1
Adaptive Graph Convolutional Recurrent Network for Traffic Forecasting，https://arxiv.org/abs/2007.02842
STGRAT: A Spatio-Temporal Graph Attention Network for Traffic Forecasting， https://arxiv.org/abs/1911.13181v1
AAAi2022，Conditional Local Convolution for Spatio-temporal Meteorological Forecasting：https://arxiv.org/abs/2101.01000
VectorNet: Encoding HD Maps and Agent Dynamics from Vectorized Representation：https://arxiv.org/abs/2005.04259（GNN+transformer）


时空开源代码框架：
阡陌：交通预测领域，支持交通状态预测(流量、速度、需求、起点-终点（OD）矩阵、事故预测）、轨迹下一跳预测、到达时间预测、路网匹配、路网表征学习：https://github.com/LibCity/Bigscity-LibCity



automl 模型接口
  使用automl搜索步骤：
  定义模型
  定义训练流程，训练函数可接收搜索配置参数
  定义参数搜索空间
  模型搜索：
  模型类型，几种模型分别定义objective，搜索空间，模型预留搜索配置参数

automl 模型接口
  使用automl搜索步骤：
指定关于超参数的目标函数 tuning objective 
Related arguments:
evaluation_function: 评估函数.
metric: A string of the metric name to optimize for.
mode: A string in ['min', 'max'] to specify the objective as minimization or maximization.
 
指定超参数搜索空间 search space .
Related arguments:

config: A dictionary to specify the search space.
low_cost_partial_config (optional): A dictionary from a subset of controlled dimensions to the initial low-cost values.
cat_hp_cost (optional): A dictionary from a subset of categorical dimensions to the relative cost of each choice.
指定调优约束 tuning constraints, 包括资源约束，配置的约束，和评价指标的特殊约束。
Related arguments:

time_budget_s: The time budget in seconds.
num_samples: An integer of the number of configs to try.
config_constraints (optional): A list of config constraints to be satisfied.
metric_constraints (optional): A list of metric constraints to be satisfied. e.g., ['precision', '>=', 0.9].
并行调优 Parallel tuning
Related arguments:

use_ray: A boolean of whether to use ray as the backend.
resources_per_trial: A dictionary of the hardware resources to allocate per trial, e.g., {'cpu': 1}. Only valid when using ray backend.
训练策略 Trial scheduling
Related arguments:

scheduler: A scheduler for executing the trials.
resource_attr: A string to specify the resource dimension used by the scheduler.
min_resource: A float of the minimal resource to use for the resource_attr.
max_resource: A float of the maximal resource to use for the resource_attr.
reduction_factor: A float of the reduction factor used for incremental pruning.
 
示例
  定义模型

class Net(nn.Module):
    def __init__(self, l1=120, l2=84):
        super(Net, self).__init__()
        self.conv1 = nn.Conv2d(3, 6, 5)
        self.pool = nn.MaxPool2d(2, 2)
        self.conv2 = nn.Conv2d(6, 16, 5)
        self.fc1 = nn.Linear(16 * 5 * 5, l1)
        self.fc2 = nn.Linear(l1, l2)
        self.fc3 = nn.Linear(l2, 10)

    def forward(self, x):
        x = self.pool(F.relu(self.conv1(x)))
        x = self.pool(F.relu(self.conv2(x)))
        x = x.view(-1, 16 * 5 * 5)
        x = F.relu(self.fc1(x))
        x = F.relu(self.fc2(x))
        x = self.fc3(x)
        return x

定义优化目标，目标可以作为函数定义，或嵌入训练流程中
      # Validation loss
      val_loss = 0.0
      val_steps = 0
      total = 0
      correct = 0
      for i, data in enumerate(valloader, 0):
          with torch.no_grad():
              inputs, labels = data
              inputs, labels = inputs.to(device), labels.to(device)

              outputs = net(inputs)
              _, predicted = torch.max(outputs.data, 1)
              total += labels.size(0)
              correct += (predicted == labels).sum().item()

              loss = criterion(outputs, labels)
              val_loss += loss.cpu().numpy()
                

定义训练流程，训练函数可接收搜索配置参数
from ray import tune

def train_cifar(config, checkpoint_dir=None, data_dir=None):
    if "l1" not in config:
        logger.warning(config)
    net = Net(2**config["l1"], 2**config["l2"])

    device = "cpu"
    if torch.cuda.is_available():
        device = "cuda:0"
        if torch.cuda.device_count() > 1:
            net = nn.DataParallel(net)
    net.to(device)

    criterion = nn.CrossEntropyLoss()
    optimizer = optim.SGD(net.parameters(), lr=config["lr"], momentum=0.9)

    # The `checkpoint_dir` parameter gets passed by Ray Tune when a checkpoint
    # should be restored.
    if checkpoint_dir:
        checkpoint = os.path.join(checkpoint_dir, "checkpoint")
        model_state, optimizer_state = torch.load(checkpoint)
        net.load_state_dict(model_state)
        optimizer.load_state_dict(optimizer_state)

    trainset, testset = load_data(data_dir)

    test_abs = int(len(trainset) * 0.8)
    train_subset, val_subset = random_split(
        trainset, [test_abs, len(trainset) - test_abs])

    trainloader = torch.utils.data.DataLoader(
        train_subset,
        batch_size=int(2**config["batch_size"]),
        shuffle=True,
        num_workers=4)
    valloader = torch.utils.data.DataLoader(
        val_subset,
        batch_size=int(2**config["batch_size"]),
        shuffle=True,
        num_workers=4)

    for epoch in range(int(round(config["num_epochs"]))):  # loop over the dataset multiple times
        running_loss = 0.0
        epoch_steps = 0
        for i, data in enumerate(trainloader, 0):
            # get the inputs; data is a list of [inputs, labels]
            inputs, labels = data
            inputs, labels = inputs.to(device), labels.to(device)

            # zero the parameter gradients
            optimizer.zero_grad()

            # forward + backward + optimize
            outputs = net(inputs)
            loss = criterion(outputs, labels)
            loss.backward()
            optimizer.step()

            # print statistics
            running_loss += loss.item()
            epoch_steps += 1
            if i % 2000 == 1999:  # print every 2000 mini-batches
                print("[%d, %5d] loss: %.3f" % (epoch + 1, i + 1,
                                                running_loss / epoch_steps))
                running_loss = 0.0

        # Validation loss
        val_loss = 0.0
        val_steps = 0
        total = 0
        correct = 0
        for i, data in enumerate(valloader, 0):
            with torch.no_grad():
                inputs, labels = data
                inputs, labels = inputs.to(device), labels.to(device)

                outputs = net(inputs)
                _, predicted = torch.max(outputs.data, 1)
                total += labels.size(0)
                correct += (predicted == labels).sum().item()

                loss = criterion(outputs, labels)
                val_loss += loss.cpu().numpy()
                val_steps += 1

        # Here we save a checkpoint. It is automatically registered with
        # Ray Tune and will potentially be passed as the `checkpoint_dir`
        # parameter in future iterations.
        with tune.checkpoint_dir(step=epoch) as checkpoint_dir:
            path = os.path.join(checkpoint_dir, "checkpoint")
            torch.save(
                (net.state_dict(), optimizer.state_dict()), path)

        tune.report(loss=(val_loss / val_steps), accuracy=correct / total)
    print("Finished Training")
  
  定义参数搜索空间
max_num_epoch = 100
config = {
    "l1": tune.randint(2, 9),   # log transformed with base 2
    "l2": tune.randint(2, 9),   # log transformed with base 2
    "lr": tune.loguniform(1e-4, 1e-1),
    "num_epochs": tune.loguniform(1, max_num_epoch),
    "batch_size": tune.randint(1, 5)    # log transformed with base 2
}

搜索资源限制
time_budget_s = 600     # time budget in seconds
gpus_per_trial = 0.5    # number of gpus for each trial; 0.5 means two training jobs can share one gpu
num_samples = 500       # maximal number of trials

  模型参数搜索：
import time
start_time = time.time()
result = flaml.tune.run(
    tune.with_parameters(train_cifar, data_dir=data_dir),
    config=config,
    metric="loss",
    mode="min",
    low_cost_partial_config={"num_epochs": 1},
    max_resource=max_num_epoch,
    min_resource=1,
    scheduler="asha",  # Use asha scheduler to perform early stopping based on intermediate results reported
    resources_per_trial={"cpu": 1, "gpu": gpus_per_trial},
    local_dir='logs/',
    num_samples=num_samples,
    time_budget_s=time_budget_s,
    use_ray=True)
  
调优结果评估
print(f"#trials={len(result.trials)}")
print(f"time={time.time()-start_time}")
best_trial = result.get_best_trial("loss", "min", "all")
print("Best trial config: {}".format(best_trial.config))
print("Best trial final validation loss: {}".format(
    best_trial.metric_analysis["loss"]["min"]))
print("Best trial final validation accuracy: {}".format(
    best_trial.metric_analysis["accuracy"]["max"]))

best_trained_model = Net(2**best_trial.config["l1"],
                         2**best_trial.config["l2"])
device = "cpu"
if torch.cuda.is_available():
    device = "cuda:0"
    if gpus_per_trial > 1:
        best_trained_model = nn.DataParallel(best_trained_model)
best_trained_model.to(device)

checkpoint_path = os.path.join(best_trial.checkpoint.value, "checkpoint")

model_state, optimizer_state = torch.load(checkpoint_path)
best_trained_model.load_state_dict(model_state)

test_acc = _test_accuracy(best_trained_model, device)
print("Best trial test set accuracy: {}".format(test_acc))

模型类型，几种模型分别定义objective，搜索空间，模型预留搜索配置参数


