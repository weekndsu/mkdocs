# Harbor installation

## docker installation

```
yum remove docker docker-common docker-selinux docker-engine

yum install https://mirrors.aliyun.com/docker-ce/linux/centos/7/x86_64/stable/Packages/docker-ce-selinux-17.03.2.ce-1.el7.centos.noarch.rpm  -y

yum install https://mirrors.aliyun.com/docker-ce/linux/centos/7/x86_64/stable/Packages/docker-ce-17.03.2.ce-1.el7.centos.x86_64.rpm  -y
```

## docker-compose installation

```
curl -L https://github.com/docker/compose/releases/download/1.17.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose

chmod +x /usr/local/bin/docker-compose

docker-compose --version
```

## harbor installation

- 下载harbor

```
wget https://github.com/vmware/harbor/releases/download/v1.1.2/harbor-online-installer-v1.1.2.tgz
```

- 解压harbor

```
tar xvf harbor-online-installer-v1.1.2.tgz
```

- 修改配置文件

```
cd harbor
[root@cicd harbor]# grep -v "^#" harbor.cfg 
hostname = 172.16.5.110       # 设置为本地ip
ui_url_protocol = http
db_password = root123         # mysql的password
max_job_workers = 3 
customize_crt = on
ssl_cert = /data/cert/server.crt
ssl_cert_key = /data/cert/server.key
secretkey_path = /data
admiral_url = NA
email_identity =                   
email_server = smtp.mydomain.com
email_server_port = 25
email_username = sample_admin@mydomain.com
email_password = abc
email_from = admin <sample_admin@mydomain.com>
email_ssl = false
harbor_admin_password = Harbor12345
auth_mode = db_auth
ldap_url = ldaps://ldap.mydomain.com
ldap_basedn = ou=people,dc=mydomain,dc=com
ldap_uid = uid 
ldap_scope = 3 
ldap_timeout = 5
self_registration = on
token_expiration = 30
project_creation_restriction = everyone
verify_remote_cert = on
```

- 启动docker(报错，不支持overlay2)

```
[root@cicd harbor]# systemctl status docker.service
● docker.service - Docker Application Container Engine
  Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; vendor preset: disabled)
  Active: failed (Result: exit-code) since 日 2018-09-30 17:40:29 CST; 45s ago
    Docs: https://docs.docker.com
  Process: 22642 ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock --registry-mirror=https://ms3cfraz.mirror.aliyuncs.com (code=exited, status=1/FAILURE)
Main PID: 22642 (code=exited, status=1/FAILURE)
  Memory: 4.0K
  CGroup: /system.slice/docker.service

9月 30 17:40:28 cicd systemd[1]: Starting Docker Application Container Engine...
9月 30 17:40:28 cicd dockerd[22642]: time="2018-09-30T17:40:28.711908708+08:00" level=warning msg="[!] DON'T BIND ON ANY IP ADDRESS WITHOUT setting -tlsv...DOING [!]"
9月 30 17:40:28 cicd dockerd[22642]: time="2018-09-30T17:40:28.715338277+08:00" level=info msg="libcontainerd: new containerd process, pid: 22645"
9月 30 17:40:29 cicd dockerd[22642]: time="2018-09-30T17:40:29.732983961+08:00" level=error msg="[graphdriver] prior storage driver overlay2 failed: driv...supported"
9月 30 17:40:29 cicd dockerd[22642]: Error starting daemon: error initializing graphdriver: driver not supported
```

- 修改daemon文件

```
cd /etc/docker
vi daemon.json

{ 
    "storage-driver": "overlay2", 
    "storage-opts": [ 
        "overlay2.override_kernel_check=true" 
] }
```

- 修改docker registry

```
vi /var/lib/systemd/system/docker.service
# 设置为insecure registry
ExecStart=/usr/bin/dockerd  -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock  --insecure-registry 172.16.5.110
```

- 启动docker

```
systemctl start docker
```

- 初始化harbor

```
pwd
/root/harbor

./install.sh 
```

- 查看harbor

```
docker-compose ps
      Name                    Command              State                                Ports                              
------------------------------------------------------------------------------------------------------------------------------
harbor-adminserver  /harbor/harbor_adminserver      Up                                                                      
harbor-db            docker-entrypoint.sh mysqld      Up      3306/tcp                                                        
harbor-jobservice    /harbor/harbor_jobservice        Up                                                                      
harbor-log          /bin/sh -c crond && rm -f  ...  Up      127.0.0.1:1514->514/tcp                                        
harbor-ui            /harbor/harbor_ui                Up                                                                      
nginx                nginx -g daemon off;            Up      0.0.0.0:443->443/tcp, 0.0.0.0:4443->4443/tcp, 0.0.0.0:80->80/tcp
registry            /entrypoint.sh serve /etc/ ...  Up      5000/tcp
```

- 登录harbor web

```
# 如果端口占用，我们可以去修改docker-compose.yml文件中，对应服务的端口映射。
# web地址
http://172.16.5.110:80
# 用户名和密码
admin/Harbor12345
```

- 本地登录harbor

```
docker login -u admin -p Harbor12345 172.16.5.110:5000
```



