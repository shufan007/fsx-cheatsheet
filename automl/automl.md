# AutoML:

AutoML框架概览：https://zhuanlan.zhihu.com/p/378722852 
automl 汇总列表 https://zhuanlan.zhihu.com/c_1165403884117299200 

https://www.automl.org

## AutoML 理论
走马观花AutoML
https://zhuanlan.zhihu.com/p/212512984 

HPO目前应该是AutoML领域应用最多的一个场景了，前面提到了很多方法，包括随机搜索，贝叶斯优化，CASH，Multi-Fidelity优化等。在实际应用中如何选择这些方法呢？这里给出一个大致的guideline：

当可以应用Multi-Fidelity来做模型评估时，选择BOHB，这是目前综合表现最好且效率最高的方法。
当无法应用Multi-Fidelity时：
连续型的参数组合，模型评估开销较大：使用基于GP的贝叶斯优化。
维度较高，且较为复杂的参数空间：使用基于随机森林或者TPE的SMAC方法。
连续型的参数空间，且模型评估开销低：使用CMA-ES方法。

超参数优化算法-SuccessiveHalving, BH与BOHB
https://blog.csdn.net/zwqjoy/article/details/111151367

automl-超参数优化（HPO）-综述
https://blog.csdn.net/xys430381_1/article/details/104856902

optuna hyperopt

算法工程师技术路线图
https://zhuanlan.zhihu.com/p/192633890 


automl-超参数优化（HPO）-综述
https://blog.csdn.net/xys430381_1/article/details/104856902

如何通俗易懂地介绍 Gaussian Process？
https://www.zhihu.com/question/46631426

贝叶斯优化/Bayesian Optimization
https://zhuanlan.zhihu.com/p/76269142

BO是一个sequential decision-making problem
其实BO不是只能用来调超参的，因为他是一个非常general的gradient-free global optimization的方法，所以他的适用场景一般有两个特点：（1）需要优化的function计算起来非常费时费力，比如上面提到的神经网络的超参问题，每一次训练神经网络都是燃烧好多GPU的；（2）你要优化的function没有导数信息。所以如果你遇到的问题有以上两个特点的话直接闭着眼睛用BO就行了。
BOHB
https://www.automl.org/blog_bohb/

超参数自动优化方法PBT（Population Based Training）
https://blog.csdn.net/jinzhuojun/article/details/100047416

Ray Tune中的超参数调整算法 Hyperband/ASHA/PBT/PB2
https://blog.csdn.net/cansaizheng/article/details/116198497

ASHA就是ASynchronous的SuccessiveHalving算法。异步后SHA就更容易并行。Hyperband的内循环就是SHA，外循环之间没有依赖可以简单的并行/异步化，所以Hyperband也可以对应的异步化为AsyncHyperBand。

## AutoML 应用

解读模型压缩3：高效模型设计的自动机器学习流水线
https://zhuanlan.zhihu.com/p/299422728 

## FLAML

微软AutoML框架之FLAML
https://zhuanlan.zhihu.com/p/410362009
https://www.kdnuggets.com/2021/09/fast-automl-flaml-ray-tune.html
https://zhuanlan.zhihu.com/p/405177840

https://zhuanlan.zhihu.com/p/405015450

机器学习实战 | AutoML自动化机器学习建模
https://blog.csdn.net/ShowMeAI/article/details/123650630
http://www.showmeai.tech/article-detail/210

## AutoML/ML 平台
Vertex AI: 
https://cloud.google.com/vertex-ai 

EasyDL:
https://ai.baidu.com/easydl/

百度EasyDL平台最全讲解：
http://www.360doc.com/content/22/0606/14/7274547_1034774006.shtml

OPEN MLSYS
https://openmlsys.github.io/chapter_distributed_training/pipeline.html

百度BML
https://ai.baidu.com/ai-doc/BML/2l2svt3we

华为云 ModelArts
https://support.huaweicloud.com/develop-modelarts/develop-modelarts-0008.html
订阅算法/内置算法，使用预置框架

自定义作业，自动搜索作业，使用预置框架

## Ray

理论，论文：
https://arxiv.org/abs/2005.01571 
https://arxiv.org/pdf/1911.04706.pdf 

Blendsearch-on-Ray-benchmark

Ray Datasets: Distributed Data Loading and Compute
https://docs.ray.io/en/latest/data/dataset.html

Ray/Ray Tune：
https://docs.ray.io/en/latest/index.html

https://docs.ray.io/en/master/tune/index.html 

https://howardlau.me/programming/distributed-hyperparameter-tuning-with-ray-tune.html

Large Scale Training at BAIR with Ray Tune
https://bair.berkeley.edu/blog/2020/01/16/tune/ 

Ray.tune
https://zhuanlan.zhihu.com/p/93643769 

分布式机器学习框架Ray的文档
https://zhuanlan.zhihu.com/p/559844289

Ray配置
https://docs.ray.io/en/latest/ray-core/configure.html#configuring-ray 

Deploying on Kubernetes
https://docs.ray.io/en/master/cluster/kubernetes.html#ray-k8s-deploy 

强化学习框架 Ray 在 K8s 上的自动伸缩设计与实现
https://cloud.tencent.com/developer/news/463783 

Ray.tune：把超参放心交给它
https://zhuanlan.zhihu.com/p/93643769 

Ray基本使用方法(draft)
https://zhuanlan.zhihu.com/p/102557115 

Ray Tune FAQ
https://docs.ray.io/en/latest/tune/faq.html

分布式框架Ray及RLlib简易理解
https://zhuanlan.zhihu.com/p/61818897

Ray Tune Hyperparameter Optimization Framework
https://blog.csdn.net/u011254180/article/details/81178651 

Ray 多机分布式加载数据
Ray Datasets: Distributed Data Loading and Compute
https://docs.ray.io/en/latest/data/dataset.html

超参优化工具总结(4)——Ray.tune
https://zhuanlan.zhihu.com/p/93584289 

PyTorch + Ray Tune 调参
https://blog.csdn.net/tszupup/article/details/112059788 

Pytorch 分布式训练
https://zhuanlan.zhihu.com/p/76638962 

Pytorch多机多卡分布式训练
https://zhuanlan.zhihu.com/p/68717029 

Ray Cluster Launcher: a utility for managing resource provisioning and cluster configurations across AWS, GCP, and Kubernetes.

Frameworks: Ray Train is built to abstract away the coordination/configuration setup of distributed deep learning frameworks such as Pytorch Distributed and Tensorflow Distributed, allowing users to only focus on implementing training logic.

从MR到Spark再到Ray，谈分布式编程的发展
https://zhuanlan.zhihu.com/p/98033020 

Ray的系统层是以Task为抽象粒度的，用户可以在代码里任意生成和组合task，比如拆分成多个Stage,每个Task执行什么逻辑，每个task需要多少资源,非常自由，对资源把控力很强。RDD则是以数据作为抽象对象的，你关心的应该是数据如何处理，而不是去如何拆解任务，关心资源如何被分配，这其中涉及的概念比如Job,Stage,task你最好都不要管，RDD自己来决定。虽然抽象不一样，但是最终都能以此来构建解决各种问题的应用框架，比如RDD也能完成流，批，机器学习相关的工作，Ray也是如此。所以从这个角度来看，他们其实并没有太过本质的区别。从交互语言而言，双方目前都支持使用python/java来写任务。另外，就易用性而言，双方差距很小，各有优势。

现在Ray来了，Ray吸取了Spark的诸多营养，走的更远了，首先，既然Python作为世界上当前最优秀的人机交互通用语言，我直接就让Python成为头等公民，安装部署啥的全部采用Python生态。比如你pip 安装了ray,接着就可以用ray这个命令部署集群跑在K8s/Yarn或者多个主机上，也可以直接使用python引入ray API进行编程。易用性再次提高一大截。其次，作为分布式应用，他除了能够把python函数发到任意N台机器上执行，还提供了Actor(类)的机制，让你的类也可以发到任意N台机器上执行。

开源史海钩沉系列 Ray：分布式计算框架
https://zhuanlan.zhihu.com/p/104022670

Launching Cloud Clusters:
https://docs.ray.io/en/latest/cluster/cloud.html#cluster-private-setup 

Scaling Applications on Kubernetes with Ray
https://vishnudeva.medium.com/scaling-applications-on-kubernetes-with-ray-23692eb2e6f0 

A Step-by-Step Guide to Scaling Your First Python Application in the Cloud
https://medium.com/distributed-computing-with-ray/a-step-by-step-guide-to-scaling-your-first-python-application-in-the-cloud-8761fe331ef1


Distributed PyTorch
https://docs.ray.io/en/releases-0.8.2/raysgd/raysgd_pytorch.html 

How to use Tune with PyTorch
http://44.228.130.106/tune/tutorials/tune-pytorch-cifar.html#tune-torch-ddp

https://docs.ray.io/en/releases-1.11.0/tune/tutorials/tune-pytorch-cifar.html

https://docs.ray.io/en/releases-1.11.0/tune/examples/ddp_mnist_torch.html


Ray Object Store 介绍
https://zhuanlan.zhihu.com/p/554562026 

Ray Distributed Scheduler 介绍
https://zhuanlan.zhihu.com/p/554572007


### Katib

云原生的自动机器学习系统 Katib 论文解读
https://zhuanlan.zhihu.com/p/157589799 

首先是多租户，Katib 是目前开源的超参数搜索系统里唯一原生支持多租户的。其次是分布式的训练能力，Katib 构建于 Kubeflow 的众多项目之上，能够支持分布式的模型训练与并行的超参数搜索。然后是云原生，Katib 是一个 Kubernetes Native 的系统，所有的功能都依托于 Kubernetes 的扩展性能力实现，是一个云原生的系统。最后，是扩展性，Katib 的架构易于扩展，能够集成不同的超参数搜索算法。

Kubeflow机器学习工具包-概述
https://blog.csdn.net/chenxy02/article/details/123426242 
https://zhuanlan.zhihu.com/p/98889237

———————————

Pip 安装 指定源

 -i https://pypi.tuna.tsinghua.edu.cn/simple

 阿里云 http://mirrors.aliyun.com/pypi/simple/ 
 中国科技大学 https://pypi.mirrors.ustc.edu.cn/simple/ 
 豆瓣(douban) http://pypi.douban.com/simple/ 
 清华大学 https://pypi.tuna.tsinghua.edu.cn/simple/ 
 中国科学技术大学 http://pypi.mirrors.ustc.edu.cn/simple/

——————————————————
AutoML 框架：
AutoML框架概览：https://zhuanlan.zhihu.com/p/378722852 

FLAML： https://zhuanlan.zhihu.com/p/410362009 
Ray Tune 
NNI
Kubeflow/Katib：https://zhuanlan.zhihu.com/p/157589799 
https://zhuanlan.zhihu.com/p/77760872 

FLAML + Ray Tune： https://www.kdnuggets.com/2021/09/fast-automl-flaml-ray-tune.html

AutoX
—————

模型下载：
方案1，将模型保存在挂载目录中，通过notebook下载
2.链接数梦，模型部署，下载

分布式部署方案调研

AutoML 

Fast AutoML with FLAML + Ray Tune

https://www.kdnuggets.com/2021/09/fast-automl-flaml-ray-tune.html 


FLAML is a newly released library containing state-of-the-art hyperparameter optimization algorithms. FLAML leverages the structure of the search space to optimize for both cost and model performance simultaneously. It contains two new methods developed by Microsoft Research:
* Cost-Frugal Optimization (CFO)
* BlendSearch
Cost-Frugal Optimization (CFO) is a method that conducts its search process in a cost-aware fashion. The search method starts from a low-cost initial point and gradually moves towards a higher cost region while optimizing the given objective (like model loss or accuracy).
Blendsearch is an extension of CFO that combines the frugality of CFO and the exploration ability of Bayesian optimization. Like CFO, BlendSearch requires a low-cost initial point as input if such point exists, and starts the search from there. However, unlike CFO, BlendSearch will not wait for the local search to fully converge before trying new start points.

How CFO and BlendSearch Work
CFO begins with a low-cost initial point (specified through low_cost_init_value in the search space) and performs local updates following its randomized local search strategy. With such a strategy, CFO can quickly move toward the low-loss region, showing a good convergence property. Additionally, CFO tends to avoid exploring the high-cost region until necessary. This search strategy is further grounded with a provable convergence rate and bounded cost in expectation.

BlendSearch further combines this local search strategy used in CFO with global search. It leverages the frugality of CFO and the space exploration capability of global search methods such as Bayesian optimization. Specifically, BlendSearch maintains one global search model, and gradually creates local search threads over time based on the hyperparameter configurations proposed by the global model. It further prioritizes the global search thread and multiple local search threads depending on their real-time performance and cost. It can further improve the efficiency of CFO in tasks with complicated search space, e.g., a search space that contains multiple disjoint, non-continuous subspaces.

How to scale up CFO and BlendSearch with Ray Tune’s distributed tuning
  To speed up hyperparameter optimization, you may want to parallelize your hyperparameter search. For example, BlendSearch is able to work well in a parallel setting: It leverages multiple search threads that can be independently executed without obvious degradation of performance. This desirable property is not always true for existing optimization algorithms such as Bayesian Optimization.
To achieve parallelization, FLAML is integrated with Ray Tune. Ray Tune is a Python library that accelerates hyperparameter tuning by allowing you to leverage cutting edge optimization algorithms at scale. Ray Tune also allows you to scale out hyperparameter search from your laptop to a cluster without changing your code. You can either use Ray Tune in FLAML or run the hyperparameter search methods from FLAML in Ray Tune to parallelize your search. The following code example shows the former usage, which is achieved by simply configuring the n_concurrent_trials argument in FLAML.

Conclusion

FLAML is a newly released library containing state-of-the-art hyperparameter optimization algorithms that leverages the structure of the search space to optimize for both cost and model performance simultaneously. FLAML can also utilize Ray Tune for distributed hyperparameter tuning to scale up these economical AutoML methods across a cluster.

AutoML 理论
走马观花AutoML
https://zhuanlan.zhihu.com/p/212512984 

FLAML 理论
论文：
https://arxiv.org/abs/2005.01571 

Blendsearch-on-Ray-benchmark

————————————————

Ray Tune：
https://docs.ray.io/en/master/tune/index.html 


You can leverage your Kubernetes cluster as a substrate for execution of distributed Ray programs. The Ray Autoscaler spins up and deletes Kubernetes Pods according to the resource demands of the Ray workload. Each Ray node runs in its own Kubernetes Pod.


# image is Ray image to use for the head and workers of this Ray cluster.
# It's recommended to build custom dependencies for your workload into this image,
# taking one of the offical `rayproject/ray` images as base.
image: rayproject/ray:latest


Ray on k8s

方案一：

Ray集群资源固定配置
资源配置：CPU，GPU，memory
先在k8s上拉起Ray集群
task提交到Ray集群，由Ray管理

RayCluster image 使用自定义image，
安装flaml 组件
安装hdfs 链接工具和配置
 
image: rayproject/ray:latest

operatorImage 使用官方镜像 `rayproject/ray`
operatorImage: rayproject/ray:latest

Ray provides a Helm chart to simplify deployment of the Ray Operator and Ray clusters. 

Using Ray Client to connect from within the Kubernetes cluster

提交任务：
使用k8s起一个提交任务的容器，连接Ray Cluster
镜像使用官方镜像: rayproject/ray:latest
提交任务的entrypoint 代码从github下载


当n_concurrent_trials 设置超过可满足n_jobs 的worker数时，实际并行数为满足n_jobs 的worker数。
当不走并行调度（设置n_concurrent_trials=1，或不设置）时，向Ray集群提交任务，默认在head node上执行。
考虑：当head资源不足时会怎么样？是否会调度到worker上？
	head 的调度是否可以关闭？


TODO：
一个Ray worker node上的多个workers是指什么？跟n_jobs的关系？
内存的问题：验证并行任务时看到head上的内存占用比较高，大数据的并行策略如何？能否数据并行？

当没有任务执行的时候，Ray集群是否一直占用k8s资源？
 看到没有任务时，Ray node上的workers 是0

资源配置的细节问题
并行的原理和效果
GPU如何配置，如何使用


方案二：

对于大任务可以使用该方案，需判断任务大小，根据参数或数据大小判断
需管理任务，校验任务状态

根据提交的任务，配置资源
拉起相应资源的Ray集群
然后把任务提交到Ray集群
任务完成后删除Ray集群，释放资源

方案三：

直接基于k8s 部署automl
需要管理并行训练的任务，可基于kubeflow/katib，但上层应用不太丰富。

对于小任务，可以直接在k8s上提交单机训练任务。

问题：

为什么开启并行trails 可以防止OOM？

Running Ray programs with Ray Jobs Submission

export HADOOP_USER_NAME=prod_alita
export HADOOP_USER_PASSWORD=xxx

ray job submit --runtime-env-json='{"working_dir":"./"}' -- sh ./run.sh --task_id task_123456 --train_data hdfs://DClusterNmg2/user/prod_alita/alita_dev/hive/alita_dev/ml_data_split_13434844_59261508_49e97caa66_1/part-00000-a81f8fff-c57a-42ce-9afc-073cd801e9e6-c000.snappy.parquet --test_data hdfs://DClusterNmg2/user/prod_alita/alita_dev/hive/alita_dev/ml_data_split_13434844_59261508_49e97caa66_2/part-00000-55dfe73a-092f-44f2-92b0-e2801b5ad261-c000.snappy.parquet --label_col label --output_path ./test.outputs --time_budget 240 --task classification --metric roc_auc --estimator_list '["lgbm"]' --estimator_kwargs '{"n_jobs":4,"n_concurrent_trials":5}'  --seed 1
 

TODO ：

### Ray集群
 local on promise 手动部署流程验证，
自动部署流程探索，验证

Ray集群部署，任务提交，集群删除 管理，监控流程方案探索，开发

Flaml 资源配置，任务执行优化

Automl 添加数据拆分功能

模型文件，日志文件挂载问题调研

训练日志由前端/后端负责采集

Customize AutoML.fit()

Constraint

#### Resampling strategy

By default, flaml decides the resampling automatically according to the data size and the time budget. If you would like to enforce a certain resampling strategy, you can set eval_method to be "holdout" or "cv" for holdout or cross-validation.

Warm start

保存automl1.best_config_per_estimator， 下次训练从该处开始，对于大型训练任务更友好。

#### How to set time budget

early_stop=True

#### Tune User Defined Function

#### GPU 的使用

问题：
资源提供方，如何提供资源，运行什么镜像？
是否ssh打通？


NNODES=1 NODE_RANK=0 MASTER_ADDR=10.186.2.241 bash tools/dist_train.sh 1 --data_path=hdfs://difed/tmp/ml_data_split_14862135_48403021_20220418_1 --feature_cols=d1,d2,d3,d4,d5,d6,d7,d8,d9,d10,d11,d12,d13,d14,d15,d16,d17,c10,c9,c11,c3,c5,c7,c8,c6,c4,c2,c12,c13,c1,d26,d25,d24,d23,d22,d21,d20,d19,d18 --sparse_cols=d1,d2,d3,d4,d5,d6,d7,d8,d9,d10,d11,d12,d13,d14,d15,d16,d17,d26,d25,d24,d23,d22,d21,d20,d19,d18 --label_cols=label


./examples/ddt_minist_torch_debug.py --num-workers 1 --num-gpus-per-worker 4 --cluster --train_data /nfs/volume-807-1/fanshuangxi/datasets/cifar-10