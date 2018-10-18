# ansible安装

- yum安装
```
  yum install -y ansible
```

- 生成公钥（添加到agent认证）

```
ssh-keygen -t rsa
```

- 添加ansible的agent ip

```
[k8s node]
172.16.5.110
```

- 添加agent认证

```
ssh-copy-id -i  .ssh/id_rsa.pub 172.16.5.110
```

- 测试

```
ansible all -m ping
```

