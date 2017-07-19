# 创建harbian发行版的方法 

## 总述 

所有发行版大体都遵循这个流程：源码版本控制-> 打包制作安装包 -> 归档入库 -> 制作安装介质.创建一个GNU/Linux发行版，核心关注点个人理解就是三个，安装包，仓库，安装介质：

* 安装包：这项的关注点是如何从源码生成安装包，保证安装包间的依赖关系正确.
* 仓库：这项的关注点是如何把安装包导入仓库，并保证仓库中索引和数据一致正确.
* 安装介质：这项的关注点是从仓库同步获取最新的软件包，制作成可用可引导介质，比如安装光盘，安装U盘.


## 准备工作 
由于debian上游仓库至少有130G的大小，故对于存储空间的分配则必须等于130+100G的大小；

## 如何同步上游仓库

创建参考配置，如下所示：
我
* 执行命令 `gpg --gen-key`创建签名密钥对，导入当前管理仓库所在的机器，具体执行步骤略：
* 在repo目录 创建`reprepro`需要的配置:  

* conf/distributions

```
Origin: harbian 
Label:  HardenedLinux Server Main Repo 
Codename: harbian 
Suite: stable 
Architectures: i386 amd64 source 
Components: main non-free contrib 
UDebComponents: main 
Contents: udebs percomponent allcomponents 
Description: HardenedLinux Server  
SignWith: 404F8DCC 
Log: harbian.log 
Update: upstream-main
```

* conf/updates

```
Name: upstream-main
Method: http://ftp.cn.debian.org/debian/
Suite: stretch
Components: main contrib non-free
Architectures: i386 amd64 source
GetInRelease: no
FilterSrcList: install filterlist/debian-stretch-src
VerifyRelease: blindtrust
```

* conf/incoming

```
name: default
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
reprepro -V update
```

## 如何打包

### 全新的包的方法 

### 已有的包的方法 

## 制作安装介质 


## Reference  

https://www.debian.org/mirror/list
https://github.com/panhaitao/TheRoadToLinuxDistributions

