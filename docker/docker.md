1、获取镜像

可以使用 docker pull 命令来从仓库获取所需要的镜像。

下面的例子将从 Docker Hub 仓库下载一个 Ubuntu 12.04 操作系统的镜像。
```
docker pull ubuntu:18.04
18.04: Pulling from library/ubuntu
6abc03819f3e: Pull complete 
05731e63f211: Pull complete 
0bd67c50d6be: Pull complete 
Digest: sha256:f08638ec7ddc90065187e7eabdfac3c96e5ff0f6b2f1762cf31a4f49b53000a5
Status: Downloaded newer image for ubuntu:18.04
```
完成后，即可随时使用该镜像了，例如创建一个容器，让其中运行 bash 应用。
```
docker run -it --rm ubuntu:18.04 bash
root@fe7fc4bd8fc9:/#
```
-it：这是两个参数，一个是 

-i：交互式操作，一个是 

-t 终端。我们这里打算进入 bash 执行一些命令并查看返回结果，因此我们需要交互式终端。

--rm：这个参数是说容器退出后随之将其删除。默认情况下，为了排障需求，退出的容器并不会立即删除，除非手动 docker rm。我们这里只是随便执行个命令，看看结果，不需要排障和保留结果，因此使用 --rm 可以避免浪费空间。

ubuntu:18.04：这是指用 ubuntu:18.04 镜像为基础来启动容器。

bash：放在镜像名后的是 命令，这里我们希望有个交互式 

Shell，因此用的是 bash。

使用 docker images 显示本地已有的镜像。
```
docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
<none>              <none>              97193b575626        42 minutes ago      292MB
ubuntu              14.04               2c5e00d77a67        13 days ago         188MB
ubuntu              18.04               7698f282e524        13 days ago         69.9MB
```
2、删除本地镜像

如果要删除本地的镜像，可以使用 docker image rm 命令，其格式为：
> $ docker image rm [选项] <镜像1> [<镜像2> ...]

3、利用 commit 理解镜像构成
