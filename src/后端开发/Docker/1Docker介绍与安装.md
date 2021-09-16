
# 1 Docker与虚拟机的对比
- 传统虚拟机技术是虚拟出一套硬件后，在其上运行一个完整操作系统，在该系统上再运行所需应用进程
- **容器内的应用进程直接运行于宿主的内核， 容器内没有自己的内核， 而且也没有进行硬件虚拟**
| 特性 | 容器 | 虚拟机 |
| :--- | :--- | :--- |
| 启动 | 秒级 | 分钟级 |
| 硬盘使用 | 一般为MB | 一般为GB |
| 性能 | 接近原生 | 弱于 |
| 系统支持量 | 单机支持上千个容器 | 一般几十个 |


# 2 Docker基本概念

- 镜像：**一个特殊的文件系统**，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。
- 容器：**镜像的运行实例**（类与对象的关系），容器的实质是进程，但与直接在宿主执行的进程不同，容器进程运行于属于自己的独立的命名空间。 容器可以拥有自己的root文件系统、自己的网络配置、 自己的进程空间，甚至自己的用户ID空间。
- 仓库：一个仓库会包含同一个软件不同版本的镜像，标签就常用于对应该软件的各个版本。可以通过`<仓库名>:<标签>`的格式来指定具体是这个软件哪个版本的镜像。如果不给出标签，将以latest作为默认标签。

# 3 安装Docker
Docker官方为了简化安装流程，提供了一套便捷的安装脚本，通过`--mirror`选项使用国内源进行安装:
```shell
curl -fsSL get.docker.com -o get-docker.sh
sudo sh get-docker.sh --mirror Aliyun

#启动docker服务
sudo systemctl enable docker
sudo systemctl start docker

#建立docker用户组
sudo groupadd docker
sudo usermod -aG docker $USER

#测试是否安装成功
docker run hello-world
```

# 4 docker国内镜像源
新增/etc/docker/daemon.json文件，内容如下：
```cpp
{
  "registry-mirrors": [
    "https://registry.docker-cn.com",
    "https://mirror.baidubce.com"
  ]
}
```

之后重新加载docker：
```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```
