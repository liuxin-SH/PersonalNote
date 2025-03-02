# window上安装完DockerDestop后，内存一直无法释放
由于安装DockerDestop后，默认需要开启Hyper-v。
但是在window上开启hyper-v后，虚拟机一直占着资源，无法释放，需要手动进行关闭。
[win menu]--> [hyper-v Manager] 中进行关闭。

# 安装完docker后，执行docker相关命令，出现 Got permission denied：

`
Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get http://%2Fvar%2Frun%2Fdocker.sock/v1.26/images/json: dial unix /var/run/docker.sock: connect: permission denied
`

原因

摘自docker mannual上的一段话：

`
Manage Docker as a non-root user

The docker daemon binds to a Unix socket instead of a TCP port. By default that Unix socket is owned by the user root and other users can only access it using sudo. The docker daemon always runs as the root user.

If you don’t want to use sudo when you use the docker command, create a Unix group called docker and add users to it. When the docker daemon starts, it makes the ownership of the Unix socket read/writable by the docker group

`

大概的意思就是：docker进程使用Unix Socket而不是TCP端口。而默认情况下，Unix socket属于root用户，需要root权限才能访问。

* 解决方法1:  
    使用sudo获取管理员权限，运行docker命令

* 解决方法2

    docker守护进程启动的时候，会默认赋予名字为docker的用户组读写Unix socket的权限，因此只要创建docker用户组，并将当前用户加入到docker用户组中，那么当前用户就有权限访问Unix socket了，进而也就可以执行docker相关命令
    ```bash
    sudo groupadd docker     #添加docker用户组
    sudo gpasswd -a $USER docker     #将登陆用户加入到docker用户组中
    newgrp docker     #更新用户组
    docker ps    #测试docker命令是否可以使用sudo正常使用
    ```

# JFrog 搭建一个docker自己的源

# Use 'docker scan' to run Snyk tests against images to find vulnerabilities and learn how to fix them

# 解决docker使用sudo的问题
docker命令需要添加`sudo`执行。

去掉`sudo`两种方式：
- 使用root登录。`su`命令
- 当前用户添加到docker group中。

## add user to dokcer group on linux
1. 查看会否有docker group组
```
grep docker /etc/group
```

如果存在会显示：
```
docker:x:999
```

否则说经不存在docker group,需要添加：
```
sudo groupadd docker
```

重启docker服务：
```
sudo systemclt restart docker
```

2. add user_name to docer group using the `usermod`command.
```
usermod -aG docker $user_name

usermode -aG docker liqian
````



