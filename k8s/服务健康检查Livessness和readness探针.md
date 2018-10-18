# 服务健康检查Livessness和readness探针

## Liveness

- **Liveness**  探针主要用于判断container是否处于正常运行状态，比如当服务发生crash或者死锁等情况，kubelet会kill掉container，然后根据其设置的	restart policy进行相应的操作（有可能在本机重启container，或者因为设置了kubernetes的QoS，在本机没有足够资源的情况下被分发其他的machine上进行restart的操作 ）

## readness

- **readness**  探针主要用于判断服务是否已经正常运行工作，如果服务没有正常加载完成或者工作出现异常。服务所在的Pod的IP address会从服务的endpoints中被移除，也就是说，当服务没有ready时，会将其从服务的load balancer中移除，不会再接受或响应任何请求。 

## 探针支持三种类型处理handler

- **ExecAction**

  Container内部执行某个具体的命令 

- **TCPSocketAction**

  通过container的IP、port执行tcp进行检查 

- **HTTPGetAction**

  通过container的IP、port、path，用HTTP Get请求进行检查 

## 探测结果分为三种类型

- 成功（success）
- 失败（failure）
- 未知（unknow）

| 探针类型        | 说明                                                     | 健康检查标准     |
| --------------- | -------------------------------------------------------- | ---------------- |
| ExecAction      | Container内部执行某个具体的命令                          | shell命令返回0   |
| TCPSocketAction | 通过container的IP、port执行tcp进行检查                   | port是否打开     |
| HTTPGetAction   | 通过container的IP、port、path，用HTTP Get   请求进行检查 | 200<=返回值<=400 |

## 服务可用性和自动恢复

- 如果服务的健康检查（readiness）失败，故障的服务实例从service endpoint中下线，外部请求将不会再转发到该服务上，一定程度上保证正在提供的服务的正确性，如果服务自我恢复了（比如网络问题），会自动重新加入service endpoint对外提供服务。
- 另外，如果设置了Container（liveness）的探针，对故障服务的Container（liveness）的探针同样会失败，container会被kill掉，并根据原设置的container重启策略，系统倾向于在其原所在的机器上重启该container、或其他机器重新创建一个pod。
- 由于上面的机制，整个服务实现了自身可用与自动。

### 使用建议

-  建议对全部服务同时设置服务（readiness）和Container（liveness）的健康检查
- 通过TCP对端口检查形式（TCPSocketAction），仅适用于端口已关闭或进程停止情况。因为即使服务异常，只要端口是打开状态，健康检查仍然是通过的。
- 基于第二点，一般建议用ExecAction自定义健康检查逻辑，或采用HTTP Get请求进行检查（HTTPGetAction）。
- 无论采用哪种类型的探针，一般建议设置检查服务（readiness）的时间短于检查Container（liveness）的时间，也可以将检查服务（readiness）的探针与Container（liveness）的探针设置为一致。目的是故障服务先下线，如果过一段时间还无法自动恢复，那么根据重启策略，重启该container、或其他机器重新创建一个pod恢复故障服务。

