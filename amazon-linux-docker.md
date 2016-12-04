## 安装
在amazon aws ec2中安装docker比较简单（笔者使用的是Amazon Linux）
```
[ec2-user ~]$ sudo yum update -y
[ec2-user ~]$ sudo yum install -y docker
[ec2-user ~]$ sudo service docker start
Starting cgconfig service:                                 [  OK  ]
Starting docker:	                                   [  OK  ]
```

[Docker Basics - Installing Docker](http://docs.aws.amazon.com/zh_cn/AmazonECS/latest/developerguide/docker-basics.html#install_docker)


## 获取debian镜像
[Docker Hub ](https://hub.docker.com/explore/) 上有大量的高质量的镜像可以使用。
```
sudo docker pull debian
```
获取了debian镜像。tag为latest。

## 启动容器
```
[ec2-user@ip-172-31-17-3 ~]$ sudo docker run -d -it --name helloDocker debian
73e8cce47e440b619af011c7605c902a0aa1c32608dd92553822c51f68784d21
[ec2-user@ip-172-31-17-3 ~]$ sudo docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
73e8cce47e44        debian              "/bin/bash"         9 seconds ago       Up 9 seconds                                    helloDocker

```
-d 后台运行
--name 标识
-i
-t

现在启动了helloDocker容器，运行的是debian镜像，容器的状态是`up`

## 进入容器
```
[ec2-user@ip-172-31-17-3 ~]$ sudo docker attach helloDocker
root@e740b8cbb2e0:/# uname -a
Linux e740b8cbb2e0 4.4.19-29.55.amzn1.x86_64 #1 SMP Mon Aug 29 23:29:40 UTC 2016 x86_64 GNU/Linux
root@e740b8cbb2e0:/# exit
exit
```
这时已经进入helloDocker容器了，运行`exit`可退出

## 安装jdk
需要从helloDocker容器中退出，再从主机中复制jdk到docker容器中
```
[ec2-user@ip-172-31-17-3 ~]$ sudo docker cp jdk-8u111-linux-x64.tar.gz  helloDocker:/usr/local/src/
```
现在已经将jdk文件复制到helloDocker容器的/usr/local/src/目录下了，重新进入容器安装jdk即可
```
[ec2-user@ip-172-31-17-3 ~]$ sudo docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                       PORTS               NAMES
73e8cce47e44        debian              "/bin/bash"         7 minutes ago       Exited (130) 4 minutes ago                       helloDocker
[ec2-user@ip-172-31-17-3 ~]$ sudo docker start helloDocker
helloDocker
[ec2-user@ip-172-31-17-3 ~]$ sudo docker attach helloDocker
root@73e8cce47e44:/# ls /usr/local/src/
jdk-8u111-linux-x64.tar.gz
```
因为之前已经退出容器，容器状态为`Exited`，需要运行`docker start`再启动容器  
现在可以看到/usr/local/src/已经存在jdk文件了，这里可以正常安装jdk了。

## 删除容器
```
[ec2-user@ip-172-31-17-3 ~]$ sudo docker rm helloDocker
helloDocker
```
`docker rmi`可用于删除镜像

[Docker run 命令的使用方法](http://dockone.io/article/152)


