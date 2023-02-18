# AutoML Dev Issues:


checkpoint目录问题排查，基于ray2.0接口可解决该问题。
Ddp模模式封装trainable 存在dataset大小限制问题，放弃DistributedTrainableCreator
接口封装，使用ray2.0接口tune.Tuner(tune.with_parameters(train_func…))可解决该问题。
Automl预测模型效果一致性测试，测试发现
ETTm2 数据集 seq_len=96 pred_len=192的效果一致性偏差较大，需进一步查找原因。

Spark数据加载问题：
data_loader 读取 Array table时，当数组太长时报错,
执行X = np.array(df.select(features_col[0]).collect())时报错，
经测试排查，当配置.config("spark.driver.host", ip) 时可避免该问题。

File "/nfs/volume-807-1/fanshuangxi/dev/TS/temp/dml-autotabular/common/pyspark_tool.py", line 87, in load_data
234| X = np.array(df.select(features_col[0]).collect())
235| File "/usr/local/spark-current/python/pyspark/sql/dataframe.py", line 533, in collect
236| sock_info = self._jdf.collectToPython()
237| File "/usr/local/spark-current/python/lib/py4j-0.10.7-src.zip/py4j/java_gateway.py", line 1257, in __call__
238| File "/usr/local/spark-current/python/pyspark/sql/utils.py", line 63, in deco
239| return f(*a, **kw)
240| File "/usr/local/spark-current/python/lib/py4j-0.10.7-src.zip/py4j/protocol.py", line 328, in get_return_value
241| py4j.protocol.Py4JJavaError: An error occurred while calling o145.collectToPython.
242| : org.apache.spark.SparkException: Job aborted due to stage failure: Task 0 in stage 0.0 failed 1 times, most recent failure: Lost task 0.0 in stage 0.0 (TID 0, localhost, executor driver): TaskResultLost (result lost from block manager)


DistributedTrainableCreator 封装时报None type 错误，
Callable func  的 args 参数必须以字典类型传递。

Distributed training Device set bug fix


FEDformer 模型中有复数权重（ComplexFloat），pytorch DDP 不支持


———————————

TODO:

添加 resource_attr

Running Tune experiments with BOHB

https://zhuanlan.zhihu.com/p/559844289

再将它转化为分布式的 multi-worker 训练函数。
只需要使用 ray.train.torch.prepare_model 和 ray.train.torch.prepare_data_loader 工具函数，很容易建立模型&数据，进行分布式训练。这个过程会自动使用 DistributedDataParallel 打包你的模型并将模型放置在正确的设备上，并为你的Dataloaders添加DistributeSampler.


精简参数：
参数梳理，进行中
提供默认搜索参数，模型搜索级别，参数范围搜索级别
参数推断：如对模型构建参数从数据中进行推断
时序预测任务类型（多变量—>多变量，多变量—> 单变量， 单变量—>单变量）参数‘features’ 取消，改为内部推断 
模型结构改造

metric 可选，添加metric

修改predict部分代码


支持多文件数据输入
支持ID的table处理，首先对ID聚合、排序，然后逐个ID进行训练，或对全ID进行采样

下一步优化参数：
embed
loader_num_workers.   
moving_avg 自适应设置
loss  丰富

* data_format参数优化， 取消该参数，改为内部推断

改造模型推断相关流程，包括数据加载，model predict函数

数据加载优化, 加载数据统一位置

代码结构设计调整，统一 model trainable base界接口

* 风险&依赖

* 下周工作计划
1. 时序算法集成automl功能完善。接口优化，稳定性测试。


* 考察 automl以计算代价/flops/模型大小 为优化目标的方案
添加Constraints，可用于做小模型搜索
Constraints on the metrics of the ML model tried in AutoML.

flaml.tune.run
    config_constraints: 
    metric_constraints: 
设计并实现功能，并进行测试效果


探索flaml seq task功能
* 'ts_forecast': time series forecasting.
* 'ts_forecast_classification': time series forecasting for classification.
* 'rank': learning to rank.
* 'seq-classification': sequence classification.
* 'seq-regression': sequence regression.

对比 flaml build-in estimator 和former系列效果
'transformer': Huggingface transformer models for task "seq-classification", "seq-regression", "multichoice-classification", "token-classification" and "summarization".


问题：

1. 搜索后找不到最优模型
except: [Errno 2] No such file or directory: '/nfs/volume-807-1/fanshuangxi/ts_test.outputs/WrappedDistributedTorchTrainable_2022-08-05_16-09-10/WrappedDistributedTorchTrainable_eb5afcc4_2_d_ff=128,d_model=128,learning_rate=0.00032432,model=Autoformer,n_heads=4_2022-08-05_16-09-25/checkpoint_000001/./checkpoint.pth'

参考：
https://docs.ray.io/en/latest/tune/tutorials/tune-checkpoints.html

If all Ray nodes have access to a shared filesystem, e.g. via NFS, they can all write to this directory. In this case, we don’t need any synchronization at all, as it is implicitly done by the operating system.
For this case, we only need to tell Ray Tune not to do any syncing at all (as syncing is the default):

尽可能使用shared or cloud storage
you should always try to use shared or cloud storage if possible when training on a multi node cluster.

If you do not use the cluster launcher, you should set up a NFS or global file system and disable cross-node syncing:
sync_config = tune.SyncConfig(syncer=None)
tuner = tune.Tuner(func, run_config=air.RunConfig(sync_config=sync_config))
results = tuner.fit()


tuner = tune.Tuner(
    trainable,
    run_config=air.RunConfig(
        name="experiment_name",
        local_dir="/path/to/shared/storage/",
        sync_config=tune.SyncConfig(
            syncer=None  # Disable syncing
        )
    )
)
原来的flaml代码flaml.tune.run接口中并没有开放sync_config参数，近期刚更新的代码已经开放了更多的ray参数，通过**ray_args传递。
需要更新最新flaml代码， 然后校验local_dir，进行配置sync_config

在使用 ray.tune 做超参搜索，在Checkpoints时遇到了下面的问题，
模型训练使用了Distributed模式，对训练器使用ray.tune.integration.torch.DistributedTrainableCreator封装，
用 ray.tune.integration.torch.distributed_checkpoint_dir 管理训练过程中的checkpoints
如下方式生存checkpoint目录，保存模型：
with distributed_checkpoint_dir(step=epoch) as checkpoint_dir:

最后如下方式取最优模型的checkpoint目录
result = tune.run(…)
best_trial = result.get_best_trial(…)
拿到最优模型的路径
model_path = best_trial.checkpoint.value
但这样的到的model_path有时候并不是实际保存checkpoint模型的位置。
比如它给出的目录是.../checkpoint_000000/ ， 实际位置可能是在.../worker_0/checkpoint_000000/ 下面，就是可能多了一个/worker_0/  这一层

我的local_dir 参数使用的是 nfs共享目录
根据下面官方文档 https://docs.ray.io/en/latest/tune/tutorials/tune-checkpoints.html   介绍的方法，设置了sync_config
        sync_config=tune.SyncConfig(
            syncer=None  # Disable syncing
        )

但上面的问题还是存在，


checkpoint_dir 调用方式：必须在调用tune.report之前被调用，并且二者的调用需要对应。

You should call this *before* calling ``tune.report``. The reason is
because we want checkpoints to be correlated with the result
(i.e., be able to retrieve the best checkpoint, etc). Many algorithms
depend on this behavior too.

示例：
for epoch in range(config.get("epochs", 10)):
    optimizer.zero_grad()
    output = model(x)
    loss = loss_fn(output, y)
    loss.backward()
    optimizer.step()

    if epoch % 3 == 0:
        if config.get("enable_checkpoint", True):
            with distributed_checkpoint_dir(step=epoch) as checkpoint_dir:
                path = os.path.join(checkpoint_dir, "checkpoint")
                torch.save((model.state_dict(), optimizer.state_dict()), path)
    tune.report(mean_loss=loss.item())


2. >> ray.tune 传入过大 dataset 问题

trainable_cls = DistributedTrainableCreator(
            partial(train_func, args=dict(args),
                    train_dataset=train_dataset,
                    val_dataset=val_dataset),
当 dataset 过大时 报错
```commandline
grpc._channel._InactiveRpcError: <_InactiveRpcError of RPC that terminated with:
        status = StatusCode.RESOURCE_EXHAUSTED
        details = "Received message larger than max (258846264 vs. 104857600)"
```

3. create spark session binding address问题
22/09/06 18:36:46 ERROR SparkContext: Error initializing SparkContext.
116| java.net.BindException: Cannot assign requested address: Service 'sparkDriver' failed after 100 retries (on a random free port)! Consider explicitly setting the appropriate binding address for the service 'sparkDriver' (for example spark.driver.bindAddress for SparkDriver) to the correct binding address.

添加配置
.config("spark.driver.bindAddress", "127.0.0.1")


2. timed out问题： timeout_s过小
ray.exceptions.GetTimeoutError: Get timed out: some object(s) not ready.
308| 【2022-08-22 02:18:09】(WrappedDistributedTorchTrainable pid=3166) 2022-08-22 10:18:09,928 ERROR worker.py:449 -- Exception raised in creation task: The actor died because of an error raised in its creation task, ray::_Inner.__init__() (pid=3166, ip=10.83.150.53, repr=<ray.tune.utils.trainable.WrappedDistributedTorchTrainable object at 0x7f38dd7976e0>)

遇到timeout异常时如何退出？
当搜索未完成时而time out 无法正常退出时，需要做异常处理：保存已训练的model

参考： https://docs.ray.io/en/releases-1.11.0/ray-core/fault-tolerance.html


3. 读取 Array table时，当数组太长时报错

py4j.protocol.Py4JJavaError: An error occurred while calling o123.sql.
: java.util.concurrent.ExecutionException: org.apache.hadoop.security.AccessControlException: Permission denied: user=prod_alita, access=READ_EXECUTE, inode="/user/prod_alita/alita_dev/hive/alita_dev/etth1_seq48_pred24_1w":fanshuangxi:alita_dev_0_etth1_seq48_pred24_1w:drwxr-x---
        at org.apache.hadoop.hdfs.server.namenode.FSPermissionChecker.check(FSPermissionChecker.java:399)
        at org.apache.hadoop.hdfs.server.namenode.FSPermissionChecker.checkPermission(FSPermissionChecker.java:261)
        at org.apache.hadoop.hdfs.server.namenode.FSPermissionChecker.checkPermission(FSPermissionChecker.java:193)
        at org.apache.hadoop.hdfs.server.namenode.FSDirectory.checkPermission(FSDirectory.java:1885)
        at org.apache.hadoop.hdfs.server.namenode.FSDirectory.checkPermission(FSDirectory.java:1869)




3. Spark 连接失败问题：连接失败会重试，

22/08/19 15:30:18 WARN TIOStreamTransport: Error closing output stream.
java.net.SocketException: Socket closed
        at java.net.SocketOutputStream.socketWrite(SocketOutputStream.java:116)
        at java.net.SocketOutputStream.write(SocketOutputStream.java:153)
        at java.io.BufferedOutputStream.flushBuffer(BufferedOutputStream.java:82)
        at java.io.BufferedOutputStream.flush(BufferedOutputStream.java:140)


4. 解决 Address family not supported by protocol 的问题

 [W socket.cpp:401] [c10d] The server socket cannot be initialized on [::]:39025 (errno: 97 - Address family not supported by protocol).

[W socket.cpp:558] [c10d] The client socket cannot be initialized to connect to [k8s-deploy-d5indh-1653892003727-79c56ccb75-6pc8h]:39025 (errno: 97 - Address family not supported by protocol).

Application Id: application_1657800494332_18686580, Tracking URL: http://bigdata-nmg-hdprm00.nmg01:8088/proxy/application_1657800494332_18686580/
22/08/16 16:36:53 WARN LzoCompressor: java.lang.UnsatisfiedLinkError: Cannot load liblzo2.so.2 (liblzo2.so.2: cannot open shared object file: No such file or directory)!
22/08/16 16:36:53 ERROR LzoCodec: Failed to load/initialize native-lzo library


Trainables can themselves be distributed

多机多卡，测试 torch.distributed
DistributedDataParallel

代码中添加检测环境变量os.environ['RANK']
测试luban是否已做分布式初始化设置

Ray：从现象看没有做单模型多机分布式调度，需要进一步调查。

模型保存问题：尝试将模型保持在rank0
    if rank == 0:
        # All processes should see same parameters as they all start from same
        # random parameters and gradients are synchronized in backward passes.
        # Therefore, saving it in one process is sufficient.
        torch.save(ddp_model.state_dict(), CHECKPOINT_PATH)

————————
多机多卡DDP方式测试：
lanuch的时候报错：

[c10d] The server socket cannot be initialized on [::]:29500 (errno: 97 - Address family not supported by protocol)

ValueError: DistributedDataParallel device_ids and output_device arguments only work with single-device/multiple-device GPU modules or CPU modules, but got device_ids [0], output_device None, and module parameters {device(type='cpu')}.

———————

TODO：

输出autoTS使用文档

大数据dataset放入trainner 报错问题复现，排查

** Table 类型 支持对ID进行分组，groupby 排序
样本数据存储格式定义

数据类型： 表名， pyspark 读取表   Done
定义输入格式，spark表格式


对于深度学习模型，如果数据过大时，支持分批读取数据，
大表，需要表名转hdfs路径

模型保存checkpoint 增加optimizer保存，支持增量训练


读取数据方案：

1. 在tune.run之前读取数据，好处是数据只需读入一次，弊端是，数据太大的情况下不能送入tunner 
2. 当前方案： 在trainner中读取数据，弊端是，每个trail都要单独读取数据一次，效率低。
问题：
读hive table时可能会报socket异常，距离上次连接过久，已经断开。
 WARN TIOStreamTransport: Error closing output stream.
java.net.SocketException: Socket closed

———

产品优点、创新点，与同类产品相比哪些突出点，价值如何体现。
分布式训练，
调用接口简化


** 后续功能，支持用户自定义trainner
* 考虑支持自定义数据加载方式

考察 automl以计算代价/flops/模型大小 为优化目标的方案
FLOPs

参考 flaml/ nni

metric_constraints=[("score", "<=", 0.4)]

More constraints on the tuning

深度学习模型部署流程
—————

时序预测问题：

原始时序数据 转时序样本数据
提供 SlidingWindow 生成样本数据

—————————
找营销业务人员对接，测试场景表格数据
测试autoformer inference 耗时

各位大佬，我在使用ray.tune.integration.torch.DistributedTrainableCreator 封装一个train_func
trainable_cls = DistributedTrainableCreator(
            partial(train_func, args=dict(args),
                    train_dataset=train_dataset,
                    val_dataset=val_dataset),
使用partial传递训练数据大小被限制在了200M以内，
请问有没有什么方法可以传更大的数据吗？


————————————
Automl. 梳理查看eve当前产品功能，调研市面上产品，补充功能

## Automl Framework issues

ray serialization.py System error: No module named 'trainer'
ERROR serialization.py:354 -- No module named 'trainer'
(train_func pid=14834) ModuleNotFoundError: No module named 'trainer'
(train_func pid=14834) ERROR function_trainable.py:298 -- Runner Thread raised error.

已经加入模块路径，但仍报上面的错误
    package_path = os.path.join(base_path, package_name)
    sys.path.append(package_path)
解决方法：
参考 https://github.com/ray-project/ray/issues/5635 

设置 PYTHONPATH 环境变量，永久设置模块的搜索路径
os.environ['PYTHONPATH'] = package_path

