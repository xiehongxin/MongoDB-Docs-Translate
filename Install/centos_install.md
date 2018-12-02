
# Centos

本教程为MongoDB社区版在Centos下的安装教程，

- [Centos](#centos)
    - [概览](#%E6%A6%82%E8%A7%88)
    - [安装包](#%E5%AE%89%E8%A3%85%E5%8C%85)
    - [安装MongoDB](#%E5%AE%89%E8%A3%85mongodb)
        - [使用源安装（推荐安装）](#%E4%BD%BF%E7%94%A8%E6%BA%90%E5%AE%89%E8%A3%85%E6%8E%A8%E8%8D%90%E5%AE%89%E8%A3%85)
        - [下载安装包进行安装](#%E4%B8%8B%E8%BD%BD%E5%AE%89%E8%A3%85%E5%8C%85%E8%BF%9B%E8%A1%8C%E5%AE%89%E8%A3%85)
    - [运行MongoDB](#%E8%BF%90%E8%A1%8Cmongodb)
    - [卸载MongoDB](#%E5%8D%B8%E8%BD%BDmongodb)


## 概览
本安装指南仅支持64位系统。


## 安装包
MongoDB在自己的存储库中提供官方支持的包， 此存储库包含以下包：  
mongodb-org	  
mongodb-org-server  
mongodb-org-mongos   
mongodb-org-shell   
mongodb-org-tools（包含mongoimport bsondump, mongodump, mongoexport, mongofiles, mongorestore等包）  

## 安装MongoDB
想要安装之前的版本请参见[这里](https://docs.mongodb.com/v3.6/tutorial/install-mongodb-on-red-hat/)

### 使用源安装（推荐安装）
```python
# 1.新建源文件
>vim /etc/yum.repos.d/mongodb-org-4.0.repo

# 2.填入以下信息
[mongodb-org-4.0]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/4.0/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-4.0.asc

# 3.安装
>sudo yum install -y mongodb-org

# 如果想要安装特定版本，或者特定的组件（不想要安装全部），可使用以下方式
sudo yum install -y mongodb-org-4.0.4 mongodb-org-server-4.0.4 mongodb-org-shell-4.0.4 mongodb-org-mongos-4.0.4 mongodb-org-tools-4.0.4
```

### 下载安装包进行安装
```python
# 1.安装对应的依赖
>yum install libcurl openssl

# 2.下载包
地址为： https://www.mongodb.com/download-center/community?jmp=docs

# 3.解压
> tar -zxvf mongodb-linux-*-4.0.4.tgz

# 4.添加环境变量
>vim ~/.bashrc
# 加入这行信息，<mongodb-install-directory>是你实际的解压路径
export PATH=<mongodb-install-directory>/bin:$PATH 

```

## 运行MongoDB
1.要运行MongoDB，首先要确保 SELinux 已经关闭，不然会有意外错误（当然配置SELinux也是可以的，但是比较麻烦）

2.启动服务
>sudo service mongod start

3.停止服务
>sudo service mongod stop

4.重启服务
>sudo service mongod restart

5.启动mongo客户端
>mongo

## 卸载MongoDB
1.停止服务
>sudo service mongod stop

2.卸载包
>sudo yum erase $(rpm -qa | grep mongodb-org)

3.删除数据和日志目录
>sudo rm -r /var/log/mongodb
sudo rm -r /var/lib/mongo

补充：
配置默认目录：/etc/mongod.conf
数据默认目录：/var/lib/mongo
日志默认目录：/var/log/mongodb/

更多详情可参考 https://docs.mongodb.com/manual/tutorial/install-mongodb-on-red-hat/