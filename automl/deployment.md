
## 模型部署

模型部署简介
https://zhuanlan.zhihu.com/p/477743341


### TorchScript 部署

TorchScript 解读（一）：初识 TorchScript
https://zhuanlan.zhihu.com/p/486914187

模型保存 TorchScript 
问题：在调用 torch.jit.script 时未保存，但执行 torch.jit.save 时报如下错误：
```commandline
RuntimeError: Tensors of type UndefinedTensorImpl do not have strides
```
原因：模型入口 forward(..., x_mark_enc=None) 入参有默认值未None的tensor。
需指定类型：如 x_mark_enc: Optional[torch.Tensor]=None, 内层Module的forward()用到默认值未None的地方都要做相应修改，
否则会报类型不匹配的问题。

问题：用如下方式使用列表存放Tensor时的问题
```commandline
attns = []
...
attns.append(attn)
```
```commandline
aten::append.t(t[](a!) self, t(c -> *) el) -> (t[](a!)):
Expected a value of type 't' for argument 'el' but instead found type 'Optional[Tensor]'.
```
修改：
```commandline
attns = [] 
--> 
attns: List[Optional[torch.Tensor]] = []
```

问题：
```commandline
'__torch__.utils.masking.TriangularCausalMask' is being compiled since it was called from 'FullAttention.forward'
```
原因： Module的forward()中引用 utils.masking.TriangularCausalMask
TriangularCausalMask 的实例化必须在 __init__()中进行，直接在forward()引用会报错

问题：
```commandline
TypeError: cannot create weak reference to 'numpy.ufunc' object
```
怀疑代码位置：
```commandline
U_part = self.factor * np.ceil(np.log(L_K)).astype('int').item()  # c*ln(L_k)
u = self.factor * np.ceil(np.log(L_Q)).astype('int').item()  # c*ln(L_q)
```
原因：
经测试跟"np.ceil(np.log(L_K)).astype('int').item()" 中 np.ceil() 的使用有关

Answer from stack overflow
```commandline
It was due to my model using numpy math operations (which are numpy.ufunc). I fixed the issue by replacing all of numpy ufuncs (i.e. np.add, np.ceil, and +, - etc on ndarrays) with corresponding torch versions (i.e. torch.add, torch.sub etc).
```
解决方案：np.ceil --> torch.ceil, 如：
```commandline
U_part = self.factor * int(torch.ceil(torch.log(torch.tensor([L_K]))).item())  # c*ln(L_k)
u = self.factor * int(torch.ceil(torch.log(torch.tensor([L_Q]))).item())  # c*ln(L_q)
```

问题：函数参数类型问题，如：
```commandline
Arguments for call are not valid.
The following variants are available:
  
  aten::randint(int high, int[] size, *, int? dtype=None, int? layout=None, Device? device=None, bool? pin_memory=None) -> (Tensor):
  Expected a value of type 'int' for argument '<varargs>' but instead found type 'Tuple[int, Tensor]'.
...
'ProbAttention._prob_QK' is being compiled since it was called from 'ProbAttention.forward'
```
注意有些函数用法，TorchScript与python语法差异
```commandline
index_sample = torch.randint(L_K, (L_Q, sample_k))
# L_Q 期望是int 类型，被推断为Tensor类型，需做强制转换
index_sample = torch.randint(L_K, (int(L_Q), sample_k))
```

问题：
```commandline
Can't redefine method: forward on class: __torch__.layers.SelfAttention_Family.ProbAttention (of Python compilation unit at: 0x560f26c06a00)
```
原因未查明。可能是某函数内部仍存在参数类型不对应问题

问题：参数类型推断与实际类型不符
```commandline
_get_initial_context(__torch__.layers.SelfAttention_Family.ProbAttention self, Tensor V, Tensor L_Q) -> (Tensor):
Expected a value of type 'Tensor (inferred)' for argument 'L_Q' but instead found type 'int'.
Inferred 'L_Q' to be of type 'Tensor' because it was not annotated with an explicit type.
```
解决方案：
显式知指定参数类型，如：
```commandline
    def _update_context(self, context_in, V, scores, index, L_Q: int, attn_mask: Optional[torch.Tensor]=None):
```

问题：作为索引的tensor类型限制
```commandline
        index_sample = torch.randint(L_K, (L_Q, sample_k))  # real U = U_part(factor*ln(L_k))*L_q
        index_q = torch.arange(L_Q).unsqueeze(1).long()
        K_sample = K_expand[:, :, index_q, index_sample, :]
                   ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ <--- HERE
        Q_K_sample = torch.matmul(Q.unsqueeze(-2), K_sample.transpose(-2, -1)).squeeze()
    
RuntimeError: tensors used as indices must be long, byte or bool tensors
```
解决方案：对index做类型转换
```commandline
index_sample = torch.randint(L_K, (L_Q, sample_k))
--> 
index_sample = torch.randint(L_K, (L_Q, sample_k)).long()
```


### pyspark 读表数据

若出现报错：
```commandline
ERROR Utils: Failed to create dir in /data/sparktmp/. Ignoring this directory. 
```
则手动创建文件夹，并授权文件夹权限，依次执行：
```commandline
sudo mkdir /data /data/sparktmp/   
sudo chmod  777 /data/sparktmp/
```
是否有更好的解决方案？

报错：
```commandline
ERROR LzoCodec: Failed to load/initialize native-lzo library
```
未解决，不影响读取数据


>>问题：ray.exceptions.GetTimeoutError: Get timed out: some object(s) not ready.

    ***** 此坑被坑过2次 ****** 
    workers_per_host 参数问题，
    在对train_func 进行DistributedTrainableCreator封装时，
    num_workers_per_host 参数只能在ray集群开启当情况下才能设为整数，当不是集群环境时，应设为None

```commandline
    trainable_cls = DistributedTrainableCreator(
        partial(train_func, args=config_env,
                train_dataset=train_dataset,
                val_dataset=val_dataset),
        num_workers=args.num_workers,
        num_gpus_per_worker=args.num_gpus_per_worker,
        num_workers_per_host=args.workers_per_host,
        backend=backend,
        timeout_s=NCCL_TIMEOUT_S,
    )
```
处理方式
```commandline
    if (args.node_list is None) or len(args.node_list) <= 1:
        args.workers_per_host = None
```


>> 问题：checkpoint 问题
通过best_trial.checkpoint.value找不到最优模型路径
```commandline
except: [Errno 2] No such file or directory: '/nfs/volume-807-1/fanshuangxi/ts_test.outputs/WrappedDistributedTorchTrainable_2022-08-05_16-09-10/WrappedDistributedTorchTrainable_eb5afcc4_2_d_ff=128,d_model=128,learning_rate=0.00032432,model=Autoformer,n_heads=4_2022-08-05_16-09-25/checkpoint_000001/./checkpoint.pth'
```

在使用 ray.tune 做超参搜索，在Checkpoints时遇到了下面的问题，
模型训练使用了Distributed模式，对训练器使用ray.tune.integration.torch.DistributedTrainableCreator封装，
用 ray.tune.integration.torch.distributed_checkpoint_dir 管理训练过程中的checkpoints

```commandline
如下方式生存checkpoint目录，保存模型：

with distributed_checkpoint_dir(step=epoch) as checkpoint_dir:

最后如下方式取最优模型的checkpoint目录

result = tune.run(…)
best_trial = result.get_best_trial(…)

拿到最优模型的路径

model_path = best_trial.checkpoint.value
```
但这样的到的model_path有时候并不是实际保存checkpoint模型的位置。
比如它给出的目录是.../checkpoint_000000/ ， 实际位置可能是在.../worker_0/checkpoint_000000/ 下面，就是可能多了一个/worker_0/  这一层

我的local_dir 参数使用的是 nfs共享目录
根据下面官方文档 https://docs.ray.io/en/latest/tune/tutorials/tune-checkpoints.html 介绍的方法，设置了sync_config
        sync_config=tune.SyncConfig(
            syncer=None  # Disable syncing
        )

但上面的问题还是存在。

通过查看"distributed_checkpoint_dir" 和 "tune.checkpoint_dir"代码，结合示例分析
"distributed_checkpoint_dir"和"tune.report"调用方式。
checkpoint_dir 调用方式：必须在调用tune.report之前被调用，并且二者的调用需要一一对应。

You should call this *before* calling ``tune.report``. The reason is
because we want checkpoints to be correlated with the result
(i.e., be able to retrieve the best checkpoint, etc). Many algorithms
depend on this behavior too.

示例：
```commandline
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

```
sync_config 配置部分：
原来的flaml代码flaml.tune.run接口中并没有开放sync_config参数，近期刚更新的代码已经开放了更多的ray参数，通过**ray_args传递。
需要更新最新flaml代码， 然后校验local_dir，进行配置sync_config

参考：
https://docs.ray.io/en/latest/tune/tutorials/tune-checkpoints.html
```commandline
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
```

>> ray.tune 传入过大 dataset 问题

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

>> create spark session binding address 问题

22/09/06 18:36:46 ERROR SparkContext: Error initializing SparkContext.
116| java.net.BindException: Cannot assign requested address: Service 'sparkDriver' failed after 100 retries (on a random free port)! Consider explicitly setting the appropriate binding address for the service 'sparkDriver' (for example spark.driver.bindAddress for SparkDriver) to the correct binding address.

添加配置
.config("spark.driver.bindAddress", "127.0.0.1")


模型和设备不在相同设备
> RuntimeError: Input type (torch.cuda.FloatTensor) and weight type (torch.FloatTensor) should be the same

数据在GPU，模型在CPU
