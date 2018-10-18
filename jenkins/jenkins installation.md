# jenkins installation

## java installation

- 安装JDK

```
tar xzvf jdk-8u181-linux-x64.tar.gz 
```

- 添加环境变量

```
vi /etc/profile

#Java Env
export JAVA_HOME=/root/jdk1.8.0_181
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin
```

- 生效profile

```
source /etc/profile
```

- 测试

```
java -version

java version "1.8.0_181"
Java(TM) SE Runtime Environment (build 1.8.0_181-b13)
Java HotSpot(TM) 64-Bit Server VM (build 25.181-b13, mixed mode)
```

## jenkins installation

- 配置yum

```
vi /etc/yum.repos.d/jenkins.repo

name=Jenkins
baseurl=http://pkg.jenkins-ci.org/redhat
gpgcheck=1
```

- 导入key

```
rpm --import http://pkg.jenkins-ci.org/redhat/jenkins-ci.org.key
```

- 安装jenkins

```
yum install jenkins
```

- jenkins目录

```
/usr/lib/jenkins/：jenkins安装目录，WAR包会放在这里。
/etc/sysconfig/jenkins：jenkins 配置文件，端口、JENKINS_HOME等都在这里配置。
```

- 启动jenkins

```
service jenkins start

Starting jenkins (via systemctl):  Job for jenkins.service failed because the control process exited with error code. See "systemctl status jenkins.service" and "journalctl -xe" for details.
                                                          [失败]
9月 30 13:53:03 cicd jenkins[19219]: Starting Jenkins bash: /usr/bin/java: 没有那个文件或目录  
```



- 查看配置文件

```
[root@cicd sysconfig]# grep -v "^#" jenkins 
JENKINS_HOME="/var/lib/jenkins"

JENKINS_JAVA_CMD=""

JENKINS_USER="jenkins"

JENKINS_JAVA_OPTIONS="-Djava.awt.headless=true"

JENKINS_PORT="8080"

JENKINS_LISTEN_ADDRESS=""

JENKINS_HTTPS_PORT=""

JENKINS_HTTPS_KEYSTORE=""

JENKINS_HTTPS_KEYSTORE_PASSWORD=""

JENKINS_HTTPS_LISTEN_ADDRESS=""

JENKINS_DEBUG_LEVEL="5"

JENKINS_ENABLE_ACCESS_LOG="no"

JENKINS_HANDLER_MAX="100"

JENKINS_HANDLER_IDLE="20"

JENKINS_ARGS=""
```

- 修改配置文件

```
vi /etc/init.d/jenkins

candidates="
/root/jdk1.8.0_181/bin/java    #加入java的path
/etc/alternatives/java

vi /etc/sysconfig/jenkins

JENKINS_JAVA_CMD="$candidate"
JENKINS_USER="root"         

#jenkins默认我启动用户是jenkins，我这里使用root安装，如果不修改jenkins_user为root，则软件会报错java权限拒绝。这里可以全部采用root角色，也可以创建jenkins用户，并在对应的用户下面修改好java的环境变量和权限分配。
```

- 重启jenkins

```
ystemctl daemon-reload

service jenkins start 

Starting jenkins (via systemctl):                          [  确定  ]
```

- 查看初始化的密码

```
more /var/lib/jenkins/secrets/initialAdminPassword
```

- 登录jenkins web

```
http://ip:8080/
```

 