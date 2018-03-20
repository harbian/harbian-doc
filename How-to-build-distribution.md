# 创建基于Debian的发行版harbian的方法 

## 总述 

所有发行版大体都遵循这个流程：源码版本控制-> 打包制作安装包 -> 归档入库 -> 制作安装介质.创建一个GNU/Linux发行版，核心关注点个人理解就是三个，安装包，仓库，安装介质：

* 安装包：这项的关注点是如何从源码生成安装包，保证安装包间的依赖关系正确.
* 仓库：这项的关注点是如何把安装包导入仓库，并保证仓库中索引和数据一致正确.
* 安装介质：这项的关注点是从仓库同步获取最新的软件包，制作成可用可引导介质，比如安装光盘，安装U盘.


## 准备工作 
由于debian上游仓库至少有130G的大小，故对于存储空间的分配则必须等于130+100G的大小；

## 如何同步上游仓库

创建参考配置，如下所示：

* 执行命令 `gpg --gen-key`创建签名密钥对，导入当前管理仓库所在的机器，若已经有密钥对，可以使用命令gpg --list-signatures进行查看。 
* 在repo目录 创建`reprepro`需要的配置:  

* conf/distributions

```
Origin: harbian
Label: harbian Linux Server Main Repo
Codename: harbian
Suite: stable
Architectures: i386 amd64 source
Components: main non-free contrib
UDebComponents: main
Contents: udebs percomponent allcomponents
Description: harbian Linux Server 
SignWith: 35AB332DCEEDF90A9EAE1D717A087DAA168064B5
Log: harbian.log
Update: upstream-main
```

* conf/updates

```
Name: upstream-main
Method: http://mirrors.163.com/debian/
Suite: stretch
Components: main contrib non-free
Architectures: i386 amd64 source
GetInRelease: no
FilterSrcList: install filterlist/debian-stretch-src
VerifyRelease: blindtrust
```

* conf/incoming

```
Name: default
IncomingDir: incoming/
TempDir: temp/
MorgueDir: morgue/
LogDir: incoming-logs/
Allow: harbian stretch>harbian
Permit: unused_files older_version
Cleanup: unused_files on_deny on_error
```

* 最后执行命令 
```
harbian@debian:~/harbian-repo$ reprepro -V update  > ~/log 2>&1
```

查看执行结果：
```
harbian@debian:~/harbian-repo$ tailf ~/log 
Reading filelist for pool/main/l/linux/xfs-modules-4.9.0-6-amd64-di_4.9.82-1+deb9u3_amd64.udeb
Reading filelist for pool/main/x/xfsprogs/xfsprogs-udeb_4.9.0+nmu1_amd64.udeb
Reading filelist for pool/main/x/xorg-server/xserver-xorg-core-udeb_1.19.2-1+deb9u2_amd64.udeb
Reading filelist for pool/main/x/xserver-xorg-input-evdev/xserver-xorg-input-evdev-udeb_2.10.5-1_amd64.udeb
Reading filelist for pool/main/x/xserver-xorg-input-libinput/xserver-xorg-input-libinput-udeb_0.23.0-2_amd64.udeb
Reading filelist for pool/main/x/xserver-xorg-video-fbdev/xserver-xorg-video-fbdev-udeb_0.4.4-1+b5_amd64.udeb
Reading filelist for pool/main/z/zlib/zlib1g-udeb_1.2.8.dfsg-5_amd64.udeb
 generating uContents-amd64...
Successfully created './dists/harbian/Release.gpg.new'
Successfully created './dists/harbian/InRelease.new'
```


## 如何打包

### 准备工作  
为保证完整的软件包 (重) 构建能顺利进行,你必须保证系统中已经安装： 

* build-essential 软件包;  
* Build-Depends域的软件包;  
* Build-Depends-indep域的软件包  

然后在源代码目录中执行以下命令:
```
$ dpkg-buildpackage -us -uc
``` 
会自动完成所有从源代码包构建二进制包的工作.

### 全新的包的方法 

### 已有的包的方法 

## 制作安装介质 


## Reference  

https://www.debian.org/mirror/list  
https://github.com/panhaitao/TheRoadToLinuxDistributions  

