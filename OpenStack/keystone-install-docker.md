# Keystone容器化
## 安装docker
我使用的操作系统是Ubuntu 16.04 LTS。

    $ uname -a
    Linux ShanahanLi-K42JE 4.4.0-21-generic #37-Ubuntu SMP Mon Apr 18 18:34:49 UTC 2016 i686 i686 i686 GNU/Linux
    
    $ sudo apt-get install docker.io
    正在读取软件包列表... 完成
    正在分析软件包的依赖关系树       
    正在读取状态信息... 完成       
    将会同时安装下列软件：
      bridge-utils cgroupfs-mount containerd runc ubuntu-fan
    建议安装：
      aufs-tools btrfs-tools debootstrap docker-doc rinse zfs-fuse | zfsutils
    下列【新】软件包将被安装：
      bridge-utils cgroupfs-mount containerd docker.io runc ubuntu-fan
    升级了 0 个软件包，新安装了 6 个软件包，要卸载 0 个软件包，有 595 个软件包未被升级。
    需要下载 14.1 MB 的归档。
    解压缩后会消耗 66.8 MB 的额外空间。
    您希望继续执行吗？ [Y/n] y
    
安装完成后，执行docker images，报错：“Cannot connect to the Docker daemon. Is the docker daemon running on this host?”，百度之后，发现是
当前用户没有加入docker group。docker group在安装docker时已经自动创建，现在只需要加入，然后重启操作系统。

    $ sudo usermod -aG docker $USER
    
## 制作镜像
将上一部--[生产环境安装Keystone](keystone-install-production.md)中的keystone-prod拷贝一份到keystone-prod-docker。然后创建一份Dockerfile，来构建镜像。

    $ cp -R keystone-prod keystone-prod-docker
    $ cd keystone-prod-docker
    $ touch Dockerfile
编写Dockerfile，第一次制作docker镜像，照着介绍依葫芦画瓢。首先这份镜像不依赖任何镜像，所有运行过程中使用的库都已经打包在这个目录下。只需要将这个目录拷贝即可，第一次尝试build报错，提示"When using COPY with more than one source file, the destination must be a directory and end with a /"，好在docker的错误提示都很准确，没怎么费劲，我的Dockerfile就可以使用来。
我的Dockerfile：
```docker
    FROM scratch

    COPY * /opt/

```
编译命令如下，镜像构建上下文在当前目录，学着开源软件的通用做法，版本号设置为17.10。

    $ docker build -t keystone-prod:17.10 .
    Sending build context to Docker daemon 164.3 MB
    Step 1 : FROM scratch
     ---> 
    Step 2 : COPY * /opt/
     ---> 7771b6d18cd8
    Removing intermediate container 6187aef1fe8d
    Successfully built 7771b6d18cd8
验证镜像是否成功构建：

    $ docker images
    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    keystone-prod       17.10               7771b6d18cd8        12 seconds ago      154.7 MB
可以看到，keystone-prod在12秒前构建。


### 搭建GO语言环境
参考阿里云容器服务的[博客](https://yq.aliyun.com/articles/110806)进行安装，但是发现安装失败，仔细一看，原来docker不支持32位CPU，没办法，我只好编译docker源码安装了。

    $ sudo apt-get install golang-go
    $ go version
    go version go1.6.2 linux/386
    $ git clone https://github.com/docker/docker-ce.git
    $ cd docker-ce
    
