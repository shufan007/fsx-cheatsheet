## K8s

https://kubernetes.io/zh/docs/home/ 

https://kubernetes.io/zh/docs/tutorials/kubernetes-basics/deploy-app/deploy-interactive/ 

https://kubernetes.io/zh/docs/tutorials/hello-minikube/ 

使用 kubeadm 创建集群
https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/ 

kubectl 备忘单
https://kubernetes.io/zh/docs/reference/kubectl/cheatsheet/ 

摆脱 AI 生产“小作坊”：如何基于 Kubernetes 构建云原生 AI 平台
https://zhuanlan.zhihu.com/p/504047864

~/.kube/config

### k8s集群配置

将k8s集群配置yml文件内容放入 ~/.kube/config

~/.kube/config

brew install kubectl 

kubectl version --client
kubectl cluster-info

kubectl get nodes 查看节点

kubectl get pods                              # 列出当前命名空间下的全部 Pods

brew install helm

Installation Ray Cluster
```commandline
# Navigate to the directory containing the chart
$ cd ray/deploy/charts

# Install a small Ray cluster with the default configuration
# in a new namespace called "ray". Let's name the Helm release "example-cluster."
$ helm -n ray install automl-cluster --create-namespace ./ray
# clusterOnly
# helm -n ray-d install automl-gpu --set clusterOnly=true ./ray-gpu

# You can use helm upgrade to adjust the fields minWorkers and maxWorkers without restarting the Ray cluster.

helm -n ray upgrade automl-cluster ./ray
```
### Cleanup
```commandline
# First, delete the RayCluster custom resource.
$ kubectl -n ray delete raycluster example-cluster
raycluster.cluster.ray.io "example-cluster" deleted

# Delete the Ray release.
$ helm -n ray uninstall example-cluster
release "example-cluster" uninstalled

# Optionally, delete the namespace created for our Ray release.
$ kubectl delete namespace ray
namespace "ray" deleted
```

对于单独部署的 ray-operator 可能是以deployment 部署，删除的时候需要以deployment 删除
```commandline
#查看
kubectl get deployment 

kubectl delete deployments ray-operator

```

### 解决K8S的namespace无法删除问题
```commandline
# 查看命名空间
kubectl get namespaces -A

# 列出你的命名空间下有哪些资源没有删除
kubectl api-resources --verbs=list --namespaced -o name | xargs -n 1 kubectl get --show-kind --ignore-not-found -n <你的命名空间>
# 添加补丁：
kubectl -n <你的命名空间> patch <红框内复制过来> -p '{"metadata":{"finalizers":[]}}' --type='merge'

# 如：
kubectl -n ray-d1 patch raycluster.cluster.ray.io/automl-cluster -p '{"metadata":{"finalizers":[]}}' --type='merge'

kubectl -n ray-d1 patch secret/ray-operator-serviceaccount-token-nnjhf -p '{"metadata":{"finalizers":[]}}' --type='merge'

# 再次查看命名空间

```
#### 删除CRD
```commandline
kubectl get crd |grep ray
kubectl delete crd rayclusters.cluster.ray.io
```
#### 删除 serviceAccount
```commandline
kubectl get serviceAccounts |grep ray
# kubectl get serviceaccounts/ray-operator-serviceaccount -o yaml
kubectl delete serviceaccounts/ray-operator-serviceaccount
```

#### 查看 Deployment
```commandline
kubectl get deployment --all-namespaces

```

#### 查看跑起来的容器有哪些
kubectl get pod --all-namespaces


### Running multiple Ray clusters
install the Operator and two Ray Clusters in three separate Helm releases
#### 注：经实验，ray operator 必须单独安装，并且命名空间只能是默认的default，否则无法成功安装多集群
Install the operator in its own Helm release.

```commandline
# Install the operator in its own Helm release.
$ helm install ray-operator --set operatorOnly=true ./ray

# Install a Ray cluster in a new namespace "ray".
$ helm -n ray install example-cluster --set clusterOnly=true ./ray --create-namespace

# Install a second Ray cluster. Launch the second cluster without any workers.
$ helm -n ray install example-cluster2 \
    --set podTypes.rayWorkerType.minWorkers=0 --set clusterOnly=true ./ray
```

### Running Ray cluster dynamic
```commandline
# 首先安装 operator 并创建命名空间，运行在单独的helm release
helm -n ray install ray-operator --create-namespace --set operatorOnly=true ./ray

# 然后安装集群
helm -n ray install automl-cluster1 --set clusterOnly=true --set podTypes.rayHeadType.memory=16 --set podTypes.rayWorkerType.minWorkers=2 --set podTypes.rayWorkerType.memory=16  --set podTypes.rayWorkerType.CPU=2 ./ray

# 向集群提交job
# 可通过yaml文件提交，或命令行提交
# kubectl create -nray -f automl-ray-d.yaml

kubectl -n ray run automl-cluster1-job1 --image=rayproject/ray:latest --env="RAY_ADDRESS=automl-cluster1-ray-head:8265" -- /bin/bash -c "git clone xxx && cd dml-autotabular && ray job submit --runtime-env-json='{\"working_dir\":\"./\", \"env_vars\": {\"HADOOP_USER_NAME\": \"prod_alita\", \"HADOOP_USER_PASSWORD\": \"xxx\"}}' -- bash ./run.sh --task_id task_123456 --train_data xxx.parquet --test_data xxx.parquet --label_col label --output_path ./test.outputs --time_budget 60 --task classification --metric roc_auc --estimator_list '[\"lgbm\"]' --estimator_kwargs '{\"n_jobs\":4,\"n_concurrent_trials\":1}'  --seed 1"

```


```commandline
kubectl describe pods 
kubectl -n ray get pods
kubectl describe pods ray-operator-5b985c9d77-wdt6d
```

查看pods
```commandline
kubectl describe pods calico-node-9l2b4 -n kube-system

kubectl describe pods -n ray example-cluster-ray-worker-type-s46vm
```

https://hub.docker.com/r/rayproject/ray
```commandline
kubectl logs  $(kubectl get pod -l cluster.ray.io/component=operator -o custom-columns=:metadata.name) | tail -n 100

```

查看pod
```commandline
kubectl -n ray describe pod example-cluster-ray-head-type-w5pmd
```

查看pod日志
```commandline
kubectl -n ray logs -f automl-test-job3-9flqt

kubectl logs -f  etcd-kmaster1 -n kube-system

kubectl logs --tail 200 -f kube-apiserver -n kube-system  #查看最后200行的日志

kubectl logs -l app=frontend # 返回所有标记为 app=frontend 的 pod 的合并日志。

kubectl logs --since=1h nginx#查看名称为nginx这个pod最近一小时的日志

```

### 进入容器
```commandline
kubectl -n ray exec -it example-cluster-ray-head-type-w5pmd -c ray-node -- /bin/bash
# -c:  --container 

# 查看Pod里业务容器
kubectl get pods myapp-pod -o jsonpath={.spec.containers[*].name}

# 如何运行一个不会关机的shard
kubectl run -i --tty ray-test01 --image=rayproject/ray:latest --restart=Always
kubectl -n ray-test run -i --tty test-node1 --image=fanshuangxi/automl:ray-flaml-cpu-0.1.3 --restart=Always

# 自定义启动配置参数
# "restartPolicy": "Always" 退出容器后会重启
kubectl -n ray-test run ssh-node1 --image=fanshuangxi/automl:ray-flaml-cpu-0.1.3 --overrides='{ "spec": { "restartPolicy": "Always", "containers": [ { "name":"ray", "image":"fanshuangxi/automl:ray-flaml-cpu-0.1.3", "imagePullPolicy": "Always", "tty": true, "stdin": true, "env": [{"name": "HADOOP_USER_NAME","value": "prod_alita"}, {"name": "HADOOP_USER_PASSWORD","value": "xxx"}], "command": ["/bin/bash"], "resources": { "requests": { "cpu": 4, "memory": "16Gi" } }, "volumeMounts":[{"mountPath": "/home/ray/automl","name": "automl"}]} ], "volumes":[{"name":"automl","hostPath":{"path":"/Users/fanshuangxi/Projects/AutoML"}}] } }'

# with gpu
kubectl -n ray-test run luban-node02 --image=ml-registry.xiaojukeji.com/luban/fanshuangxi:20221110175829 --overrides='{ "spec": { "restartPolicy": "Always", "containers": [ { "name":"ray", "image":"ml-registry.xiaojukeji.com/luban/fanshuangxi:20221110175829", "imagePullPolicy": "Always", "tty": true, "stdin": true, "env": [{"name": "HADOOP_USER_NAME","value": "prod_alita"}, {"name": "HADOOP_USER_PASSWORD","value": "xxx"}], "command": ["/bin/bash"], "resources": { "requests": { "cpu": 4,  "memory": "16Gi" } ,"limits": { "nvidia.com/gpu": 2 } }, "volumeMounts":[{"mountPath": "/home/ray/automl","name": "automl"}]} ], "volumes":[{"name":"automl","hostPath":{"path":"/Users/fanshuangxi/Projects/AutoML"}}] } }'

# 挂载本地目录
# 挂载mac 本地不生效
kubectl -n ray-test run -it ssh-node1 --image=fanshuangxi/automl:ray-flaml-cpu-0.1.3-ssh --overrides='{ "spec": { "restartPolicy": "Always", "containers": [ { "name":"ray", "image":"fanshuangxi/automl:ray-flaml-cpu-0.1.3-ssh", "imagePullPolicy": "Always", "tty": true, "stdin": true, "env": [{"name": "HADOOP_USER_NAME","value": "prod_alita"}, {"name": "HADOOP_USER_PASSWORD","value": "xxx"}], "command": ["/bin/bash"], "resources": { "requests": { "cpu": 4, "memory": "16Gi" } }, "volumeMounts":[{"mountPath": "/home/ray/automl","name": "automl"}]} ], "volumes":[{"name":"automl","hostPath":{"path":"/Users/fanshuangxi/Projects/AutoML"}}] } }'
    
# 重新进入Running中的容器
# resume using 'kubectl attach test-node2 -c test-node2 -i -t' command when the pod is running
kubectl -n ray-test attach test-node2 -c test-node2 -i -t


# 如何将外部文件复制进入k8s容器内部
kubectl -n ray-test cp ./example-full.yaml test-node1:/home/ray/

#假如当前pod只有一个容器,运行以下命令即可
kubectl exec -it nginx-56b8c64cb4-t97vb -- /bin/bash

假如一个pod里有多个容器,用--container or -c 参数。例如:假如这里有个Pod名为my-pod,这个Pod有两个容器,分别名为main-app 和 helper-app,下面的命令将打开到main-app的shell的容器里。

```
如何在 Kubernetes Pod 和您的机器之间复制文件
```commandline
kubectl cp example-pod:/tmp/example-dir ./example-dir
kubectl cp example-pod:/tmp/example-dir ./example-dir -c second-container

kubectl -n ray-test cp ./dml-autotabular test-node-dbg1:/home/ray/
```

删除job
```commandline
kubectl get jobs -n ray

kubectl delete job ray-test-job1 -n ray
```
删除pod
```commandline
kubectl delete pod automl

强制删除pod
kubectl delete pod PODNAME --force --grace-period=0

删除namespace
kubectl delete namespace NAMESPACENAME --force --grace-period=0

kubernetes_Namespace无法删除的解决方法
https://blog.csdn.net/Wqr_18390921824/article/details/124208510
http://t.zoukankan.com/zhangzhide-p-14953915.html

通过api来删除

1.查看命名空间下所有资源
kubectl api-resources -o name --verbs=list --namespaced | xargs -n 1 kubectl get --show-kind --ignore-not-found -n test

    先查找到该namespace的api接口地址
    kubectl get ns/automl -o yaml
2. 将 test 命名空间以 json文件的形式导出
kubectl get namespace automl -o json > automl.json

3.将 spec 下的内容删除：这一步骤的目的在于将内容清空后，以空内容的ns覆盖原有ns，告知k8s-api 要删除的ns内容为空，删除的命令也就无法阻塞了
  删除 "spec": 下 "finalizers": 里面的内容
4.将空内容的ns的 namespace ，通过 api-server 接口，覆盖到k8s集群中

调用接口删除该namespace

因为k8s接口默认使用https访问的，所以需要临时开一个HTTP代理端口
kubectl proxy

新开一个终端，执行调用接口命令，注意接口最后加上finalize

curl -k -H "Content-Type: application/json" -X PUT --data-binary @automl.json http://127.0.0.1:8001/api/v1/namespaces/automl/finalize

```

批量删除pod
```commandline
kubectl  -n ray get pods | grep Error | awk '{print$1}'| xargs kubectl delete pods

kubectl  -n ray get pods | grep job1 | awk '{print$1}'| xargs kubectl delete pods
 
kubectl  get pods -n kube-system | grep Evicted | awk '{print$1}'| xargs kubectl delete -n kube-system pods

```

Scaling Applications on Kubernetes with Ray
https://vishnudeva.medium.com/scaling-applications-on-kubernetes-with-ray-23692eb2e6f0  

##### Ray Job Submission

集群外部 设置 port-forward
kubectl -n ray-d port-forward service/automl-cluster1-ray-head 8265:8265

提交任务
```commandline
export RAY_ADDRESS="http://127.0.0.1:8265"

ray job submit --runtime-env-json='{"working_dir":"./"}' -- python myscript.py

ray job submit --runtime-env-json='{"working_dir":"./"}' -- sh ./run.sh --task_id task_123456 --train_data ./data/iris_data.csv --label_col label --output_path ./test.outputs --time_budget 20 --task classification --metric log_loss --estimator_list '["lgbm"]' --seed 1

ray job submit --runtime-env-json='{"working_dir":"./"}' -- sh ./run.sh --task_id task_123456 --train_data ./data/iris_data.csv --label_col label --output_path ./test.outputs --time_budget 60 --task classification --metric accuracy --estimator_list '["lgbm"]' --estimator_kwargs '{"n_jobs":4,"n_concurrent_trials":3}' --seed 1

# run with gpu
ray job submit --runtime-env-json='{"working_dir":"./"}' -- sh ./run.sh --task_id task_123456 --train_data ./data/iris_data.csv --label_col label --output_path ./test.outputs --time_budget 60 --task classification --metric accuracy --estimator_list '["xgboost"]' --estimator_kwargs '{"n_jobs":4,"n_concurrent_trials":3,"gpu_per_trial":1}' --seed 1
ray job submit --runtime-env-json='{"working_dir":"./"}' -- sh ./run.sh --task_id task_123456 --train_data ./data/iris_data.csv --label_col label --output_path ./test.outputs --time_budget 60 --task classification --metric accuracy --estimator_list '["xgboost"]' --estimator_kwargs '{"n_jobs":-1,"n_concurrent_trials":3,"gpu_per_trial":1}' --seed 1

```

### k8s 提交job
```commandline
kubectl run ng2 --image=nginx --namespace=test --overrides='{ "apiVersion": "apps/v1", "spec": { "serviceAccount": "svcacct1" , "serviceAccountName": "svcacct1" }  }' -o yaml
kubectl -n ${ray_namespace} run ${ray_cluster_name}-job3 --image=fanshuangxi/automl:ray-flaml-cpu-0.1.3 --env="RAY_ADDRESS=${ray_cluster_name}-ray-head:8265" --overrides='{ "spec": { "restartPolicy": "Never", "serviceAccountName": "ray-user" }  }' -- /bin/bash -c "kubectl -n  ${ray_namespace} delete raycluster ${ray_cluster_name}  && helm -n ${ray_namespace} uninstall ${ray_cluster_name}" -o yaml

```

```commandline
kubectl -n ray run automl --image=fanshuangxi/automl:latest --env="HADOOP_USER_NAME=prod_alita" --env="HADOOP_USER_PASSWORD=xxx" -- python /home/submarine/automl_flaml_interface/automl_main.py --task_id test --train_data xxx.parquet --test_data xxx.parquet --label_col label --time_budget 240 --task classification --metric roc_auc --estimator_list "['lgbm','xgboost']"  --seed 1

kubectl -n ray run automl-test-job1 --image=fanshuangxi/automl:flaml-0.1.4 --env="HADOOP_USER_NAME=prod_alita" --env="HADOOP_USER_PASSWORD=xxx" -- bash ./entrypoint.sh --task_id test --train_data xxx.parquet --test_data xxx.parquet --label_col label --time_budget 240 --task classification --metric roc_auc --estimator_list "['lgbm','xgboost']"  --seed 1
kubectl -n ray run automl-test-job2 --image=fanshuangxi/automl:flaml-0.1.4 --env="HADOOP_USER_NAME=prod_alita" --env="HADOOP_USER_PASSWORD=xxx" -- bash ./entrypoint.sh --task_id task_123456 --train_data xxx.parquet --test_data xxx.parquet --label_col label --time_budget 60 --task classification --metric roc_auc --estimator_list '["lgbm"]' --seed 1

# submmit with multiple commands
# bash -c "command"
kubectl -n ray run automl-job1 --image=fanshuangxi/automl:ray-flaml-cpu-0.1.3 --overrides='{ "spec": { "restartPolicy": "Never" }' --env="HADOOP_USER_NAME=prod_alita" --env="HADOOP_USER_PASSWORD=xxx" -- /bin/bash -c "git clone xxx && cd dml-autotabular && bash ./run.sh --task_id task_123456 --train_data xxx.parquet --test_data xxx.parquet --label_col label --output_path ./test.outputs --time_budget 60 --task classification --metric log_loss --estimator_list '[\"lgbm\"]' --estimator_kwargs '{\"n_jobs\":4,\"n_concurrent_trials\":2}' --seed 1"

kubectl -n ray run automl-job2 --image=fanshuangxi/automl:ray-flaml-cpu-0.1.3 --overrides='{ "spec": { "restartPolicy": "Never", "containers": [ { "name":"ray", "image":"fanshuangxi/automl:ray-flaml-cpu-0.1.3", "imagePullPolicy": "Always", "env": [{"name": "HADOOP_USER_NAME","value": "prod_alita"}, {"name": "HADOOP_USER_PASSWORD","value": "xxx"}], "command": ["/bin/bash", "-c", "--"], "args": [ "git clone xxx && cd dml-autotabular && bash ./run.sh --task_id task_123456 --train_data xxxparquet --test_data xxx --label_col label --output_path ./test.outputs --time_budget 60 --task classification --metric log_loss --estimator_list '[\"lgbm\"]' --estimator_kwargs '{\"n_jobs\":4,\"n_concurrent_trials\":2}' --seed 1 " ], "resources": { "requests": { "cpu": 4, "memory": "16Gi" } } } ] } }'
# 转义
kubectl -n ray run automl-job2 --image=fanshuangxi/automl:ray-flaml-cpu-0.1.3 --overrides="{ \"spec\": { \"restartPolicy\": \"Never\", \"containers\": [ { \"name\":\"ray\", \"image\":\"fanshuangxi/automl:ray-flaml-cpu-0.1.3\", \"imagePullPolicy\": \"Always\", \"env\": [{\"name\": \"HADOOP_USER_NAME\",\"value\": \"prod_alita\"}, {\"name\": \"HADOOP_USER_PASSWORD\",\"value\": \"xxx\"}], \"command\": [\"/bin/bash\", \"-c\", \"--\"], \"args\": [ \"git clone xxx && cd dml-autotabular && bash ./run.sh --task_id task_123456 --train_data xxx.parquet --test_data xxx.parquet --label_col label --output_path ./test.outputs --time_budget 60 --task classification --metric log_loss --estimator_list '[\\\"lgbm\\\"]' --estimator_kwargs '{\\\"n_jobs\\\":4,\\\"n_concurrent_trials\\\":2}' --seed 1 \" ], \"resources\": { \"requests\": { \"cpu\": 4, \"memory\": \"16Gi\" } } } ] } }"

kubectl -n ray run automl-job4 --image=fanshuangxi/automl:ray-flaml-cpu-0.1.2 --env="HADOOP_USER_NAME=prod_alita" --env="HADOOP_USER_PASSWORD=xxx" -- /bin/bash -c "git clone xxx && cd dml-autotabular && bash ./run.sh --task_id task_123456 --train_data ./data/iris_data.csv --label_col label --output_path ./test.outputs --time_budget 20 --task classification --metric log_loss --estimator_list '[\"lgbm\"]' --seed 1"

kubectl -n ray run automl-job5 --image=fanshuangxi/automl:ray-flaml-cpu-0.1.1-dev --env="HADOOP_USER_NAME=prod_alita" --env="HADOOP_USER_PASSWORD=xxx" -- /bin/bash -c  "cd dml-autotabular && bash ./run.sh --task_id task_123456 --train_data ./data/iris_data.csv --label_col label --output_path ./test.outputs --time_budget 20 --task classification --metric log_loss --estimator_list '[\"lgbm\"]' --seed 1"
kubectl -n ray run automl-job6 --image=fanshuangxi/automl:ray-flaml-cpu-0.1.1-dev --env="HADOOP_USER_NAME=prod_alita" --env="HADOOP_USER_PASSWORD=xxx" -- /bin/bash -c  "cd dml-autotabular && bash ./run.sh --task_id task_123456 --train_data xxx.parquet --test_data xxx.parquet --label_col label --output_path ./test.outputs --time_budget 60 --task classification --metric log_loss --estimator_list '[\"lgbm\"]' --estimator_kwargs '{\"n_jobs\":4,\"n_concurrent_trials\":1}' --seed 1"

```

### 通过k8s向Ray cluster 提交任务
向ray cluster 提交任务：
可通过ymal文件提交
kubectl port-forward 通过端口转发映射本地端口到指定的应用端口，从而访问集群中的应用程序(Pod).

kubectl run 直接提交ray cluster 任务报错：
Error from server (Forbidden): services "automl-cluster1-ray-head" is forbidden: User "system:serviceaccount:ray:default" cannot get resource "services" in API group "" in the namespace "ray"

#### k8s集群内部不需要设置port-forward
service/automl-cluster1-ray-head 8265 设置会报错
kubectl -n ray port-forward service/automl-cluster1-ray-head 8265:8265

```commandline
# 注意 k8s集群内部不需要设置port-forward   
kubectl -n ray run automl-cluster1-job2 --image=rayproject/ray:latest --env="RAY_ADDRESS=automl-cluster1-ray-head:8265" -- /bin/bash -c "git clone xxx && cd dml-autotabular && ray job submit --runtime-env-json='{\"working_dir\":\"./\", \"env_vars\": {\"HADOOP_USER_NAME\": \"prod_alita\", \"HADOOP_USER_PASSWORD\": \"xxx\"}}' -- bash ./run.sh --task_id task_123456 --train_data xxx.parquet --test_data xxx.parquet --label_col label --output_path ./test.outputs --time_budget 60 --task classification --metric roc_auc --estimator_list '[\"lgbm\"]' --estimator_kwargs '{\"n_jobs\":4,\"n_concurrent_trials\":1}'  --seed 1"

ray job submit --runtime-env-json='{\"working_dir\":\"./\", \"env_vars\": {\"RAY_ADDRESS\":\"automl-test02-ray-head:8265\", \"HADOOP_USER_NAME\": \"xxx\", \"HADOOP_USER_PASSWORD\": \"xxx\"}}' -- python3 ./main.py --task_id task_123456 --train_data xxx.parquet --test_data xxx.parquet --label_col label --output_path ./test.outputs --time_budget 60 --task classification --metric roc_auc --estimator_list '[\"lgbm\"]' --estimator_kwargs '{\"n_jobs\":4,\"n_concurrent_trials\":1}'  --seed 1

使用非转义
ray job submit --runtime-env-json='{"working_dir":"./", "env_vars": {"HADOOP_USER_NAME": "prod_alita", "HADOOP_USER_PASSWORD": "xxx"}}' -- python3 ./main.py --task_id task_123456 --train_data xxx.parquet --test_data xxx.parquet --label_col label --output_path ./test.outputs --time_budget 60 --task classification --metric roc_auc --estimator_list '["lgbm"]' --estimator_kwargs '{"n_jobs":4,"n_concurrent_trials":1}'  --seed 1

debug

ray job submit --runtime-env-json='{"working_dir":"./", "env_vars": {"HADOOP_USER_NAME": "prod_alita", "HADOOP_USER_PASSWORD": "xxx"}}' -- python3 ./main.py --task_id task_123456 --train_data ./data/iris_data.csv  --label_col label --output_path ./test.outputs --time_budget 60 --task classification --metric accuracy --estimator_list '["lgbm"]' --estimator_kwargs '{"n_jobs":4,"n_concurrent_trials":1}'  --seed 1

kubectl -n ray-d run automl-job5 --image=fanshuangxi/automl:ray-flaml-cpu-0.1.3 --env="HADOOP_USER_NAME=prod_alita" --env="HADOOP_USER_PASSWORD=xxx" -- /bin/bash -c  " git clone xxx && cd dml-autotabular && bash ./run.sh --task_id task_123456 --train_data xxx.parquet --test_data xxx.parquet --label_col label --output_path ./test.outputs --time_budget 60 --task classification --metric log_loss --estimator_list '[\"lgbm\"]' --estimator_kwargs '{\"n_jobs\":4,\"n_concurrent_trials\":1}' --seed 1"

```

通过 yml文件创建任务
```commandline
kubectl -nray create -f https://k8s.io/examples/application/shell-demo.yaml

```
git 添加下载权限
User Settings 添加Access Tokens
```commandline
# user  didi-automl:xxx
git clone http://didi-automl:xxx

```

imagePullPolicy, 镜像的拉取策略
```commandline
Always       总是拉取镜像
IfNotPresent 本地有则使用本地镜像,不拉取
Never        只使用本地镜像，从不拉取，即使本地没有
如果省略imagePullPolicy  策略为always 
```

### 容器中环境变量不生效问题
```commandline
经验证： 
~/.bashrc 中设置的环境变量，进入容器中不生效，source ~/.bashrc 无效
/etc/profile 中设置的环境变量,进入容器中生效
```

另外，
可在dockerfile中设置环境变量
将CLASSPATH 的设置放入run.sh 脚本中
export CLASSPATH=`$HADOOP_HOME/bin/hdfs classpath --glob`

参考 
https://stackoverflow.com/questions/46834331/set-environment-variables-in-docker 

### 服务账户
### Role, ClusterRole, RoleBinding
https://blog.z0ukun.com/?p=312 

查询服务账户以及命名空间中的其他 ServiceAccount 资源
```commandline
kubectl get serviceAccounts -n ray

新建一个ServiceAccount
kubectl create sa ray-user -n kube-system
kubectl create -f ray-user-rbac.yaml
kubectl -n automl create -f ray-user-rbac.yaml

# 查询服务帐户对象的完整信息
kubectl get serviceaccounts/ray-user -o yaml
```
要使用非默认的服务账户，将 Pod 的 spec.serviceAccountName 字段设置为你想用的服务账户名称

创建pod
kubectl --n ray  run k8s-ray-test --image=fanshuangxi/automl:ray-flaml-cpu-0.1.3 --overrides='{ "spec": { "serviceAccount": "ray-user" , "serviceAccountName": "ray-user" }  }' -o yaml
或者 serviceAccount 使用 ray-operator-serviceaccount
kubectl -n ray run -i --tty k8s-ray-test1 --image=fanshuangxi/automl:ray-flaml-cpu-0.1.3 --restart=Always --overrides='{ "spec": { "serviceAccount": "ray-user" , "serviceAccountName": "ray-user" }  }' -- /bin/bash
kubectl -n automl run -i --tty k8s-ray-test --image=fanshuangxi/automl:ray-flaml-cpu-0.1.3 --restart=Always --overrides='{ "spec": { "serviceAccount": "automl-user" , "serviceAccountName": "automl-user" }  }' -- /bin/bash

#### serviceaccount 权限问题
```commandline
#Error: list: failed to list: secrets is forbidden: User "system:serviceaccount:default:default" cannot list resource "secrets" in API group "" in the namespace "default"
#Error: uninstall: Release not loaded: automl-cluster4: query: failed to query with labels: secrets is forbidden: User "system:serviceaccount:ray-d:ray-operator-serviceaccount" cannot list resource "secrets" in API group "" in the namespace "ray-d"
```
https://kubernetes.io/zh/docs/tasks/configure-pod-container/configure-service-account/
https://blog.csdn.net/u013189824/article/details/110232938
https://blog.csdn.net/u013189824/article/details/111047145
https://stackoverflow.com/questions/47973570/kubernetes-log-user-systemserviceaccountdefaultdefault-cannot-get-services 

https://blog.z0ukun.com/?p=312

A Step-by-Step Guide to Scaling Your First Python Application in the Cloud
https://medium.com/distributed-computing-with-ray/a-step-by-step-guide-to-scaling-your-first-python-application-in-the-cloud-8761fe331ef1 

### Ray on premise
https://docs.ray.io/en/latest/cluster/cloud.html 
https://medium.com/distributed-computing-with-ray/a-step-by-step-guide-to-scaling-your-first-python-application-in-the-cloud-8761fe331ef1

```commandline

# On the head node
ray start --head --port=6379

Next steps
  To connect to this Ray runtime from another node, run
    ray start --address='<ip address>:6379'
  Alternatively, use the following Python code:
    import ray
    ray.init(address='auto')
    
# connect to the cluster with 
Running a Ray program on the Ray cluster
To run a distributed Ray program, you’ll need to execute your program on the same machine as one of the nodes.
    ray.init(address='auto')
    
        
  To connect to this Ray runtime from outside of the cluster, for example to
  connect to a remote cluster from your laptop directly, use the following
  Python code:
    import ray
    ray.init(address='ray://<head_node_ip_address>:10001')

# When you want to stop the Ray processes, run ray stop on each node.
ray stop

# ray stop 之后如何反馈集群终止信息？

# ray start --head --node-ip-address 172.17.126.115 --redis-shard-ports=6379

提交任务
在head node上执行代码

export HADOOP_USER_NAME=prod_alita
export HADOOP_USER_PASSWORD=xxx
./run.sh --task_id task_123456 --train_data xxx.parquet --test_data xxx.parquet --label_col label --output_path ./test.outputs --time_budget 60 --task classification --metric roc_auc --estimator_list '["lgbm"]' --estimator_kwargs '{"n_jobs":-1,"n_concurrent_trials":3}'  --seed 1

python3 ./main.py --task_id task_123456 --node_list '["172.17.126.125","172.17.126.126","172.17.126.124"]' --remote 0 --train_data xxx.parquet --test_data xxx.parquet --label_col label --output_path ./test.outputs --time_budget 60 --task classification --metric roc_auc --estimator_list '["lgbm"]' --estimator_kwargs '{"n_jobs":-1,"n_concurrent_trials":3}'  --seed 1

python3 ./main.py --task_id task_123456 --node_list '["172.17.126.125","172.17.126.126","172.17.126.124"]' --remote 0 --train_data ./data/iris_data.csv  --test_data xxx --label_col label --output_path ./test.outputs --time_budget 60 --task classification --metric roc_auc --estimator_list '["lgbm"]' --estimator_kwargs '{"n_jobs":-1,"n_concurrent_trials":3}'  --seed 1

python3 ./main.py --task_id task_123456 --node_list '["172.17.126.125","172.17.126.126","172.17.126.124"]' --remote 1 --train_data xxx.parquet --test_data xxx.parquet --label_col label --output_path ./test.outputs --time_budget 60 --task classification --metric roc_auc --estimator_list '["lgbm"]' --estimator_kwargs '{"n_jobs":-1,"n_concurrent_trials":3}'  --seed 1

```
在ray cluster 外执行
```commandline
ssh 172.17.126.123 "/home/ray/anaconda3/bin/ray start --head --port=6379 && ssh 172.17.126.124 \"/home/ray/anaconda3/bin/ray start --address='172.17.126.123:6379'\" && cd dml-autotabular && ./run.sh --task_id task_123456 --train_data xxx.parquet --test_data xxx.parquet --label_col label --output_path ./test.outputs --time_budget 60 --task classification --metric roc_auc --estimator_list '[\"lgbm\"]' --estimator_kwargs '{\"n_jobs\":-1,\"n_concurrent_trials\":2}'  --seed 1 && ssh 172.17.126.124 \"/home/ray/anaconda3/bin/ray stop\" && /home/ray/anaconda3/bin/ray stop"

在head上执行
ray start --head --port=6379 && ssh 172.17.126.124 "/home/ray/anaconda3/bin/ray start --address='172.17.126.123:6379' " && cd dml-autotabular && ./run.sh --task_id task_123456 --train_data xxx.parquet --test_data xxx.parquet --label_col label --output_path ./test.outputs --time_budget 60 --task classification --metric roc_auc --estimator_list '["lgbm"]' --estimator_kwargs '{"n_jobs":-1,"n_concurrent_trials":2}' --seed 1  && ssh 172.17.126.124 "/home/ray/anaconda3/bin/ray stop " && /home/ray/anaconda3/bin/ray stop

cd dml-autotabular && ./run.sh --task_id task_123456 --train_data xxx.parquet --test_data xxx.parquet --label_col label --output_path ./test.outputs --time_budget 60 --task classification --metric roc_auc --estimator_list '["lgbm"]' --estimator_kwargs '{"n_jobs":-1,"n_concurrent_trials":2}'  --seed 1
cd ../

#ssh 172.17.126.123 "ssh 172.17.126.124 \"/home/ray/anaconda3/bin/ray stop\" && /home/ray/anaconda3/bin/ray stop"

ray start --head --port=6379 && ssh 172.17.126.116 "/home/ray/anaconda3/bin/ray start --address='xxx:6379' " && cd dml-autotabular && ./run.sh --task_id task_123456 --train_data xxx.parquet --test_data xxx.parquet --label_col label --output_path ./test.outputs --time_budget 60 --task classification --metric roc_auc --estimator_list '["lgbm"]' --estimator_kwargs '{"n_jobs":-1,"n_concurrent_trials":2}' --seed 1  && ssh 172.17.126.116 "/home/ray/anaconda3/bin/ray stop " && /home/ray/anaconda3/bin/ray stop

ray start --head --port=6379 && ssh 172.17.126.126 "/home/ray/anaconda3/bin/ray start --address='xxx:6379' " && cd dml-autotabular && ./run.sh --task_id task_123456 --train_data xxx.parquet --test_data xxx.parquet --label_col label --output_path ./test.outputs --time_budget 60 --task classification --metric roc_auc --estimator_list '["lgbm"]' --estimator_kwargs '{"n_jobs":-1,"n_concurrent_trials":2}' --seed 1  && ssh 172.17.126.126 "/home/ray/anaconda3/bin/ray stop " && /home/ray/anaconda3/bin/ray stop

```

```commandline
# GPU

python3 ./main.py --task_id task_123456 --train_data xxx.parquet --test_data xxx.parquet --label_col label --output_path ./test.outputs --time_budget 60 --task classification --metric roc_auc --estimator_list '["xgboost"]' --estimator_kwargs '{"n_jobs":-1,"n_concurrent_trials":3,"gpu_per_trial":1}'  --seed 1

python3 ./main.py --task_id test --hadoop_config '{"HADOOP_USER_NAME":"prod_alita","HADOOP_USER_PASSWORD":"xxx"}' xxx --label_col order_cnt_7 --time_budget 120 --task regression --metric rmse --estimator_list '["xgboost"]' --seed 1 --estimator_kwargs '{"n_jobs":-1,"n_concurrent_trials":3,"gpu_per_trial":1}'
```

### Local on premise 部署方案：

#### 前提条件：
node 提供者需要以Ray镜像启动Node list，并打通ssh
资源配置也在node 提供时配置

在k8s上测试时，可通过如下方式创建几个不关机的pod，然后打通ssh
```commandline
# 如何运行一个不会关机的shard
kubectl run -i --tty ray-test01 --image=rayproject/ray:latest --restart=Always
kubectl -n ray-test run -i --tty test-node1 --image=fanshuangxi/automl:ray-flaml-cpu-0.1.3 --restart=Always
# "自定义启动配置参数"请参考前述介绍

# 进入容器
kubectl -n ray exec -it podname -c ray-node -- /bin/bash

# 重新进入Running中的容器
kubectl -n ray-test attach test-node2 -c ray -i -t

# 将代码下载或复制进入k8s容器内部
kubectl -n ray-test cp ./src test-node1:/home/ray/

```

集群参数
head_ip: "172.17.126.115"
worker_ips: ["172.17.126.116", "172.17.126.117"]
auth:
    ssh_user: ray

remote_submit=False

任务提交方式：
on the Ray cluster
out of the Ray cluster:remote

通过任务参数判断是否为并行任务。
如果是并行，则创建集群, 初始化ray，否则不初始化ray
需要编写operator管理ray集群的启动，状态监控，删除等工作

创建集群 根据任务提交方式分两种情况:
1 on the Ray cluster
 创建集群：
任务提交的node 为head node
在head node 上启动ray
    ray start --head --port=6379
然后再ssh到各个worker node上启动ray
    ray start --address='<ip address>:6379'
    该过程异步执行，进程等待，完成后提交任务。
 提交任务：
Ray cluster 创建完成后，在head上提交执行任务
初始化方式：
    import ray
    ray.init(address='auto')

2 outside of the Ray cluster
从当前提交任务的节点ssh到head node上执行启动集群流程，
该过程和在head node上启动流程类似，需注意远程登录的层次。
集群创建完成后，提交任务
一种方式可以直接在head上提交，和 on the Ray cluster 提交方式一致，
需要注意ssh远程登录后执行命令的权限限制。普通用户不允许执行系统二进制（/usr/sbin/）目录下提供的命令。用户需要 root 权限来运行它。
远程执行命令示例：
ssh ray@172.17.126.124 "/home/ray/anaconda3/bin/ray start --address='172.17.126.123:6379'"
第二种方式是从本地进行remote提交，
 初始化方式：
    import ray
    ray.init(address='ray://<head_node_ip_address>:10001')
该方法的问题：创建ray集群和提交任务，删除ray集群 流程如何同步，保持在一个进程中串行？

任务执行完成后删除 Ray cluster
在每个node上执行ray stop，也可只在head上执行。
或者直接回收集群对应的pod
需要对任务执行状态进行监控，能感知到任务完成状态。

该部分可参考ray_operator 接口，或使用配置文件管理

#### Manually Managed

启动集群
先启动head ，然后启动每个worker 连接head
如果在head上执行程序，启动代码中添加
    import ray
    ray.init(address='auto')

如果在ray 集群外部执行程序，启动代码中添加
    import ray
    ray.init(address='ray://<head_node_ip_address>:10001')

在每个pod上查看 cpu，mem使用情况
通过观察，加载数据时，head上的内存上升较多，worker上的内存无变化，
可推断，加载数据只发生在head上，且该过程并未使用ray API所以没有做并行，
后续可考虑当并行执行时是否可使用 ray并行加载数据

资源提供方提供的pod是什么形式，是否带密码？

安装Ray集群
自动方案：
1. [x] 去head上使用ssh 登录worker 起
   需要做ssh免密配置，head上安装ssh server，启动
   检测是否已启动
   ps -e | grep ssh
   看到有ssh字样，说明已启动，如果没有就手动启动
   sudo /etc/init.d/ssh start
   head 上生成公钥 cp到worker上的 authorized_keys里面

$ ssh-keygen -t rsa
$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
2. 同时启动head 和 worker，自己通过重试进行连接。（待验证）
   配置 /etc/hosts 文件
   创建/etc/HOROVOD_HOSTFILE 文件like：
   172.19.116.225  
   172.19.116.226 
   172.19.116.228

执行pdsh命令需设置 
export PDSH_RCMD_TYPE=ssh
或使用
pdsh -R ssh


#### Linux开启ssh服务
首先更新源　　
sudo apt-get update

安装ssh服务
sudo apt-get install openssh-server

检测是否已启动
ps -e | grep ssh
看到有ssh字样，说明已启动，如果没有就手动启动
sudo /etc/init.d/ssh start

配置ssh-server，配置文件位于/etc/ssh/sshd_config，默认端口为22，为了安全，一般自定义为其他端口，然后重启

sudo /etc/init.d/ssh resart


#### linux ssh 远程执行命令
普通用户不允许执行系统二进制（/usr/sbin/）目录下提供的命令。用户需要 root 权限来运行它。
which 命令返回给定命令的完整可执行路径
如：
which ray
/home/ray/anaconda3/bin/ray
远程执行命令

ssh ray@172.17.126.124 "/home/ray/anaconda3/bin/ray start --address='172.17.126.123:6379'"

#### Automatically Managed

setup your ssh keys on the private cluster as follows:

$ ssh-keygen
$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys


To print the logs, run on the cluster’s head node:
```commandline
cat /tmp/ray/session_latest/logs/monitor.*
```

To print a status overview of the cluster:
```commandline
ray status --address <address:port>
```


To see live updates to the status:
```commandline
# 不生效
$ watch -n 1 ray status
```

### 问题：
在head上执行task，训练到最后会卡住不动，原因不明
新创建 pod 执行不会出现该问题
在 cluster外部执行也不会出现该问题

如何把mac本地目录挂载到k8s pod中？
Cluster Launcher Commands
https://docs.ray.io/en/latest/cluster/commands.html#cluster-commands


python coordinator_server.py --ips 172.17.126.115,172.17.126.116,172.17.126.117 --port 6379

### GPU

https://docs.ray.io/en/master/cluster/kubernetes-gpu.html 


python ./autotabular/classification/train.py --train_input alita_dev.criteo_fill --data_format table --output_dir /nfs/volume-1036-1/fanshuangxi/dev/test_logs/test_2022_1107 --label_cols "['label']" --feature_cols "['c1','c2','c3','c4','c5','c6','c7','c8','c9','c10','c11','c12','c13','d1','d2','d3','d4','d5','d6','d7','d8','d9','d10','d11','d12','d13','d14','d15','d16','d17','d18','d19','d20','d21','d22','d23','d24','d25','d26']" --time_budget 600 --metric roc_auc --estimator_list '["lgbm","xgboost"]' --seed 1