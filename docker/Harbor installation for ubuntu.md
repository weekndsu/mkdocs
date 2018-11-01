## Harbor installation for ubuntu

### SET UP THE REPOSITORY
- check core version
```
uname -r

-----output------

4.4.0-121-generic
```


- Update the apt package index:
```
apt-get update
```
- Install packages to allow apt to use a repository over HTTPS:
```
apt-get install \
apt-transport-https \
ca-certificates \
curl \
software-properties-common
```       
   
- Add Docker’s official GPG key:
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```       

- Verify that you now have the key with the fingerprint 9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88, by searching for the last 8 characters of the fingerprint.
```
apt-key fingerprint 0EBFCD88

-----output------

pub   4096R/0EBFCD88 2017-02-22
      Key fingerprint = 9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid                  Docker Release (CE deb) <docker@docker.com>
sub   4096R/F273FCD8 2017-02-22
```
- set up the stable repository. 
```
add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```   
   
### install docker ce
```
apt-get update
```



- On production systems, you should install a specific version of Docker CE instead of always using the latest. This output is truncated. List the available versions.

```
apt-cache madison docker-ce

-----output------

docker-ce | 17.12.0~ce-0~ubuntu | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
```

- install a specific version
```
apt-get install docker-ce=<VERSION>

-----example-----

apt-cache install docker-ce=17.03.3~ce-0~ubuntu-xenial
```
### modify storage location
```
vi /lib/systemd/system/docker.service

-----output------

ExecStart=/usr/bin/dockerd -H fd://

-----replace------

ExecStart=/usr/bin/dockerd --graph /docker
```

### restart your docker
```
systemctl daemon-reload
systemctl restart docker
```
### verify the storage location
```
docker info

-----output------

Docker Root Dir: /docker
```
### install docker-compose
- 1. download docker-compose
```
curl -L https://github.com/docker/compose/releases/download/1.17.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
```
- 2. chmod+x
```
    chmod +x /usr/local/bin/docker-compose
```
- 3. verify
```
docker-compose -v
```

### install harbor
- requirements
  docker 1.17&&docker-compose>1.10
- download source code
  wget https://storage.googleapis.com/harbor-releases/release-1.5.0/harbor-offline-installe-v1.5.1.tgz
- update
  cd harbor/
  harbor.cfg

