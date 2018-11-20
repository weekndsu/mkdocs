

### NODE NAME和NODE SELECTOR

#### 1. NodeName

> Pod.spec.nodeName用于强制约束将Pod调度到指定的Node节点上，这里说是“调度”，但其实指定了nodeName的Pod会直接跳过Scheduler的调度逻辑，直接写入PodList列表，该匹配规则是强制匹配。

指定调度节点为m01

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: tomcat
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: tomcat-app
    spec:
      nodeName: m01
      containers:
      - name: tomcat
        image: tomcat
        ports:
        - containerPort: 8080
```

#### 2. NodeSelector

> Pod.spec.nodeSelector是通过kubernetes的label-selector机制进行节点选择，由scheduler调度策略MatchNodeSelector进行label匹配，调度pod到目标节点，该匹配规则是强制约束。启用节点选择器的步骤为
>
> 1. Node添加label
>
>    kubectl label nodes <node-name> <label-key>=<label-value>
>
> 2. 确认 label
>
>    kubectl get nodes <node-name> --show-labels

指定调度节点为带有label标记为：cloudnil.com/role=dev的node节点

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: tomcat
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: tomcat-app
    spec:
      nodeSelector:
        tomcat/role: dev
      containers:
      - name: tomcat
        image: tomcat
        ports:
        - containerPort: 8080
```



