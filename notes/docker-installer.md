# [centos7] docker install + docker-compose install

## 内容

* docker 安装
* docker-compose 安装

## docker安装

### 文档
https://docs.docker.com/engine/installation/linux/docker-ce/centos

### install
```
sudo yum remove docker docker-common docker-selinux docker-engine
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install -y docker-ce
sudo systemctl start docker
sudo systemctl enable docke
```

### update
```
yum -y upgrade
yum -y instal
# uninstall
sudo yum remove docker-ce
sudo rm -rf /var/lib/docker
```

## docker-compose安装

### 文档
https://docs.docker.com/compose/install

### 安装包下载
docker-compose安装包 - https://github.com/docker/compose/releases

### install docker-compose
```
sudo curl -L https://github.com/docker/compose/releases/download/1.17.1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

### uninstall docker-compose
```
sudo rm /usr/local/bin/docker-compose
```
