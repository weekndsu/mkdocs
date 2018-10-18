# kubectl指令

#### 对象资源

| Type                       | Short name |
| -------------------------- | ---------- |
| apiservices                |            |
| certificatesigningrequests | csr        |
| clusters                   |            |
| clusterroles               |            |
| componentstatuses          | cs         |
| configmaps                 | cm         |
| controllerrevisions        |            |
| cronjobs                   |            |
| customresourcedefinition   | crd        |
| daemonsets                 | ds         |
| deployments                | deploy     |
| endpoints                  | ep         |
| events                     | ev         |
| horizontalpodautoscalers   | hpa        |
| ingresses                  | ing        |
| jobs                       |            |
| limitranges                | limits     |
| namespaces                 | ns         |
| networkpolicies            | netpol     |
| nodes                      | no         |
| persistentvolumeclaims     | pvc        |
| persistentvolumes          | pv         |
| poddisruptionbudget        | pdb        |
| podpreset                  |            |
| pods                       | po         |
| podsecuritypolicies        | psp        |
| podtemplates               |            |
| replicasets                | rs         |
| replicationcontrollers     | rc         |
| resourcequotas             | quota      |
| rolebindings               |            |
| roles                      |            |
| secrets                    |            |
| serviceaccounts            | sa         |
| services                   | svc        |
| statefulsets               |            |
| storageclasses             |            |

#### 操作指令

##### annotate	

```
# 添加或更新一个或多个资源的注释
kubectl annotate (-f FILENAME \| TYPE NAME \| TYPE/NAME) KEY_1=VAL_1 … KEY_N=VAL_N [–overwrite] [–all] [–resource-version=version] [flags]
```

##### api-versions

```
# 列出可用的API版本
kubectl api-versions [flags]
```

##### apply

```
# 将来自于文件或stdin的配置变更应用到主要对象中
kubectl apply -f FILENAME [flags]	
```

##### attach

```
#连接到正在运行的容器上，以查看输出流或与容器交互（stdin）
kubectl attach POD -c CONTAINER [-i] [-t] [flags]
```

##### autoscale

```
# 自动扩宿容由副本控制器管理的Pod
kubectl autoscale (-f FILENAME \| TYPE NAME \| TYPE/NAME) [–min=MINPODS] –max=MAXPODS [–cpu-percent=CPU] [flags]
```

##### cluster-info

```
# 显示群集中的主节点和服务的的端点信息
kubectl cluster-info [flags]
```

##### config

```
# 修改kubeconfig文件
kubectl config SUBCOMMAND [flags]
```

##### create

```
# 从文件或stdin中创建一个或多个资源对象
kubectl create -f FILENAME [flags]
```

##### delete

```
# 删除资源对象
kubectl delete (-f FILENAME \| TYPE [NAME \| /NAME \| -l label \| –all]) [flags]
```

##### describe

```
# 显示一个或者多个资源对象的详细信息
kubectl describe (-f FILENAME \| TYPE [NAME_PREFIX \| /NAME \| -l label]) [flags]
```

##### edit

```
# 通过默认编辑器编辑和更新服务器上的一个或多个资源对象
kubectl edit (-f FILENAME \| TYPE NAME \| TYPE/NAME) [flags]
```

##### exec

```
# 在Pod的容器中执行一个命令
kubectl exec POD [-c CONTAINER] [-i] [-t] [flags] [– COMMAND [args…]]
```

##### explain

```
# 获取Pod、Node和服务等资源对象的文档
kubectl explain [–include-extended-apis=true] [–recursive=false] [flags]
```

##### expose

```
# 为副本控制器、服务或Pod等暴露一个新的服务
kubectl expose (-f FILENAME \| TYPE NAME \| TYPE/NAME) [–port=port] [–protocol=TCP\|UDP] [–target-port=number-or-name] [–name=name] [—-external-ip=external-ip-of-service] [–type=type] [flags]
```

##### get

```
# 列出一个或多个资源	
kubectl get (-f FILENAME \| TYPE [NAME \| /NAME \| -l label]) [–watch] [–sort-by=FIELD] [[-o \| –output]=OUTPUT_FORMAT] [flags]
```

##### label

```
# 添加或更新一个或者多个资源对象的标签
kubectl label (-f FILENAME \| TYPE NAME \| TYPE/NAME) KEY_1=VAL_1 … KEY_N=VAL_N [–overwrite] [–all] [–resource-version=version] [flags]
```

##### logs

```
# 显示Pod中一个容器的日志
kubectl logs POD [-c CONTAINER] [–follow] [flags]
```

##### patch

```
# 使用策略合并补丁过程更新资源对象中的一个或多个字段
kubectl patch (-f FILENAME \| TYPE NAME \| TYPE/NAME) –patch PATCH [flags]
```

#####　port-forward

```
# 将一个或多个本地端口转发到Pod
kubectl port-forward POD [LOCAL_PORT:]REMOTE_PORT […[LOCAL_PORT_N:]REMOTE_PORT_N] [flags]
```

#####　proxy	

```
# 为kubernetes API服务器运行一个代理
kubectl proxy [–port=PORT] [–www=static-dir] [–www-prefix=prefix] [–api-prefix=prefix] [flags]
```

##### replace

````
# 从文件或stdin中替换资源对象
kubectl replace -f FILENAME
````

##### rolling-update

```
# 通过逐步替换指定的副本控制器和Pod来执行滚动更新
kubectl rolling-update OLD_CONTROLLER_NAME ([NEW_CONTROLLER_NAME] –image=NEW_CONTAINER_IMAGE \| -f NEW_CONTROLLER_SPEC) [flags]
```

##### run

```
# 在集群上运行一个指定的镜像
kubectl run NAME –image=image [–env=”key=value”] [–port=port] [–replicas=replicas] [–dry-run=bool] [–overrides=inline-json] [flags]
```

##### scale

```
# 扩宿容副本集的数量		
kubectl scale (-f FILENAME \| TYPE NAME \| TYPE/NAME) –replicas=COUNT [–resource-version=version] [–current-replicas=count] [flags]
```

#####　version	

```
# 显示运行在客户端和服务器端的Kubernetes版本
kubectl version [–client] [flags]
```

####  输出格式化

| 选项                              | 描述                                 |
| --------------------------------- | ------------------------------------ |
| -o=custom-columns=<spec>          | 使用以逗号分隔的自定义列打印表格     |
| -o=custom-columns-file=<filename> | 使用文件中自定义列打印表格           |
| -o=json                           | 输出JSON格式的API对象                |
| -o=jsonpath=<template>            | 打印在jsonpath表达式中定义的字段     |
| -o=jsonpath-file=<filename>       | 打印文件中以jsonpath表达式定义的字段 |
| -o=name                           | 仅仅输出资源对象的名称。             |
| -o=wide                           | 输出带有附加信息的纯文本格式         |
| -o=yaml                           | 输出YAML格式的API对象                |

