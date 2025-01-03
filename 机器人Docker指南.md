# Docker拉取Nvidia镜像和ROS镜像并完美建立容器

首先进行ROS_docker安装。

##  配置存储库并更新

```shell
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list \
  && \
    sudo apt-get update
```

## 安装nvidia-docker2

```shell
sudo apt-get install -y nvidia-docker2
```

## 使用nvidia-ctk命令配置container runtime

```shell
sudo nvidia-ctk runtime configure --runtime=docker
```

## 重启docker系统

```shell
sudo systemctl restart docker
```

## 测试并删除一个容器，看看是否能使用显卡

```shell
sudo docker run --rm --gpus all nvidia/cuda:12.4.1-base-ubuntu22.04 nvidia-smi     或者
sudo docker run --rm --gpus all nvidia/cuda:11.0.3-base-ubuntu20.04 nvidia-smi
```

如果成功，则会显示显卡信息

## 使用鱼香ROS一键安装安装docker，并拉取镜像

```shell
wget http://fishros.com/install -O fishros && . fishros
```

第一次会失败，因为目前国内docker镜像全部被限制，所以需要对端口挂上VPN，还是使用上述命令，然后选择端口的配置部分，一键配置，这里使用clash，这样就可以成功拉取镜像。

```shell
wget http://fishros.com/install -O fishros && . fishros
```

## docker将容器打包成镜像

docker commit [-m=&quot;描述信息&quot;] [-a=&quot;创建者&quot;] 容器名称|容器ID 生成的镜像名[:标签名]

​    例如，咱们把名称为noetic1的容器打包成一个叫ltdz的镜像，标签名瞎起一个就行了：

```shell
docker commit noetic1 ltdz:15
```

## 然后查看是否打包成功，列举本地的所有镜像

```shell
docker image ls
```

​	若有你刚才打包的那个，说明打包成功。

## 重建容器

​    最后将这个镜像运行起来，命令与第三节的究极解决方案保持一致，并将镜像名改成你刚才打包成的那个镜像名：

```shell
sudo docker run -dit \
--gpus all \
-e NVIDIA_DRIVER_CAPABILITIES=all \
--name=[your_container_name] \
--privileged  \
-v /dev:/dev \
-v /home/[your_username]:/home/[your_username] \
-v /tmp/.X11-unix:/tmp/.X11-unix  \
-e DISPLAY=unix$DISPLAY \
-w /home/[your_username] \
--net=host \
[你刚才打包出的镜像名，即ltdz:15]
```

## 环境配置及启动脚本

在bashrc文件中加入下面部分

```bash
>>>fishros scripts >>>
export PATH=$PATH:/home/licr/.fishros/bin/ 
#source /opt/ros/melodic/setup.bash
<<< fishros scripts <<<
```

之后在路径下建立文件

```bash
xhost +local: >> /dev/null
echo "请输入指令控制humble2: 重启(r) 进入(e) 启动(s) 关闭(c) 删除(d) 测试(t):"
read choose
case $choose in
s) docker start humble1;;
r) docker restart humble1;;
e) docker exec -it humble1 /bin/bash;;
c) docker stop humble1;;
d) docker stop humble1 && docker rm humble1 && sudo rm -rf /home/licr/.fishros/bin/humble1;;
t) docker exec -it humble1  /bin/bash -c "source /ros_entrypoint.sh && ros2";;
esac
newgrp docker
```

最后，docker，启动！！！

# docker常用命令

## Docker rmi

### `docker rmi` 通过镜像的 ID 删除镜像

要删除镜像，首先需要列出所有镜像以获取镜像的 ID，镜像的名称和其他详细信息。 运行简单的命令 `docker images -a` 或 `docker images`。

之后，明确要删除哪个镜像，然后执行简单命令 `docker rmi <your-image-id>`。然后，列出所有镜像并检查，可以确认镜像是否已删除。

### 一次删除多张镜像

当你要一次删除多张镜像时，可以使用一种方法。首先只需列出镜像即可获取镜像的 ID，然后执行简单的命令：

```shell
docker rmi <your-image-id> <your-image-id> ...
```

列出镜像的 ID，每个 ID 之间留一个空格。

### 一次删除所有镜像

要删除所有镜像，有一个简单的命令可以做到：`docker rmi $(docker images -q)`。

在上面的命令中，有两个命令，第一个在 `$()` 中执行的命令是 shell 语法，返回以该执行的结果。然后，`-q-` 是一个选项，用于返回唯一的 ID。$() 返回镜像 ID 的结果，然后 `docker rmi` 删除所有这些镜像。

## Docker rm

`docker rm` 根据容器的名称或者 ID 来删除容器。

如果 Docker 容器正在运行，你在删除它们之前需要先停止运行。

- 停止所有容器运行：`docker stop $(docker ps -a -q)`
- 删除所有停止运行的容器：`docker rm $(docker ps -a -q)`

### 删除多个容器

你可以通过向命令传递要删除的容器列表来停止和删除多个容器。shell 语法 `$()` 返回括号中执行的任何结果。因此，你可以在其中创建容器列表，以传递给 `stop` 和 `rm` 命令。

### docker ps -a -q 分解

- `docker ps` 列出容器。
- `-a` 这个选项用于列出所有容器，包括停止运行的。如果没有这个选项，则默认只列出在运行的容器。
- `-q` 这个选项列出容器的数字 ID，而不是容器的所有信息。

### 其他常用指令参考

| **命令**              | **功能**                                         | **示例**                                   |
| --------------------- | ------------------------------------------------ | ------------------------------------------ |
| `docker run`          | 启动一个新的容器并运行命令                       | `docker run -d ubuntu`                     |
| `docker ps`           | 列出当前正在运行的容器                           | `docker ps`                                |
| `docker ps -a`        | 列出所有容器（包括已停止的容器）                 | `docker ps -a`                             |
| `docker build`        | 使用 Dockerfile 构建镜像                         | `docker build -t my-image .`               |
| `docker images`       | 列出本地存储的所有镜像                           | `docker images`                            |
| `docker pull`         | 从 Docker 仓库拉取镜像                           | `docker pull ubuntu`                       |
| `docker push`         | 将镜像推送到 Docker 仓库                         | `docker push my-image`                     |
| `docker exec`         | 在运行的容器中执行命令                           | `docker exec -it container_name bash`      |
| `docker stop`         | 停止一个或多个容器                               | `docker stop container_name`               |
| `docker start`        | 启动已停止的容器                                 | `docker start container_name`              |
| `docker restart`      | 重启一个容器                                     | `docker restart container_name`            |
| `docker rm`           | 删除一个或多个容器                               | `docker rm container_name`                 |
| `docker rmi`          | 删除一个或多个镜像                               | `docker rmi my-image`                      |
| `docker logs`         | 查看容器的日志                                   | `docker logs container_name`               |
| `docker inspect`      | 获取容器或镜像的详细信息                         | `docker inspect container_name`            |
| `docker exec -it`     | 进入容器的交互式终端                             | `docker exec -it container_name /bin/bash` |
| `docker network ls`   | 列出所有 Docker 网络                             | `docker network ls`                        |
| `docker volume ls`    | 列出所有 Docker 卷                               | `docker volume ls`                         |
| `docker-compose up`   | 启动多容器应用（从 `docker-compose.yml` 文件）   | `docker-compose up`                        |
| `docker-compose down` | 停止并删除由 `docker-compose` 启动的容器、网络等 | `docker-compose down`                      |
| `docker info`         | 显示 Docker 系统的详细信息                       | `docker info`                              |
| `docker version`      | 显示 Docker 客户端和守护进程的版本信息           | `docker version`                           |
| `docker stats`        | 显示容器的实时资源使用情况                       | `docker stats`                             |
| `docker login`        | 登录 Docker 仓库                                 | `docker login`                             |
| `docker logout`       | 登出 Docker 仓库                                 | `docker logout`                            |