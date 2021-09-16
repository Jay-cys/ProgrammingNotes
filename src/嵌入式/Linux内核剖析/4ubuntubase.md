Ubuntu 的移植非常简单，不需要我们编译任何东西，因为 Ubuntu 官方已经将根文件系统制作好了！我们**只需要简单配置一下Ubuntu 官方提供的 base 根文件系统**，使其在我们的开发板上跑起来即可。  在[https://mirrors.ustc.edu.cn/ubuntu-cdimage/](https://mirrors.ustc.edu.cn/ubuntu-cdimage/)中下载ubuntu base，下载arm架构的即可，比如`ubuntu-base-20.04.1-base-armhf.tar.gz`。

# 挂载ubuntu base
基本需要在linux主机上挂载ubuntu base并向操作普通ubuntu系统一样进行简单的配置，之后重新打包变为可用的根文件系统。挂载命令为：
```bash
sudo apt install qemu-user-static
sudo cp /usr/bin/qemu-arm-static ubuntu_base/usr/bin/

sudo mount -t proc /proc ubuntu_base/proc
sudo mount -t sysfs /sys ubuntu_base/sys
sudo mount -o bind /dev ubuntu_base/dev
sudo mount -o bind /dev/pts ubuntu_base/dev/pts
sudo chroot ubuntu_base
# 之后就进入了ubuntu base系统
```
对应的卸载脚本为：
```
sudo umount ubuntu_base/proc
sudo umount ubuntu_base/sys
sudo umount ubuntu_base/dev
sudo umount ubuntu_base/dev/pts
```
