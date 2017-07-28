# 如何对stig-4-debian项目进行打包 

## 安装需要的包 

```
~$ sudo apt-get install build-essential dh-make debhelper lintian git
```

## 配置dh_make所需要的环境变量 

```
$ cat >>~/.bashrc <<EOF
DEBEMAIL="sccxboy@gmail.com"
DEBFULLNAME="Samson W"
export DEBEMAIL DEBFULLNAME
EOF
$ . ~/.bashrc
```

## 下载源代码 

新建一个目录stig-deb-test，将STIG-4-Debian源代码下载到此目录，并进行初始化：

```
~$ cd stig-deb-test
~/stig-deb-test$ git clone https://github.com/hardenedlinux/STIG-4-Debian.git
~/stig-deb-test$ tar zcvf STIG-4-Debian.tar.gz STIG-4-Debian/
~/stig-deb-test$ cd STIG-4-Debian
~/stig-deb-test$ dh_make -f ../STIG-4-Debian.tar.gz
``` 

如上所示进行初始化时，会因为没有遵循Debian Free Software Guidelines的规定而不能够在dh_make进行初始化时成功，根据DFSG规则，对于软件包名的构成只能含有小写字母 (a-z), 数字 (0-9), 加号 (+) 和减号 (-) ,以及点号 (.)。软件包名最短长度两个字符;
它必须以字母开头;它不能与仓库软件包名发生冲突。

故需要对原来的工程名称进行修改，返回第三步tar的操作处先进行修改名称 

```
~/stig-deb-test$ mv STIG-4-Debian stig4debian-0.1.0/
~/stig-deb-test$  tar -czvf stig4debian_0.1.0.tar.gz --exclude=.git stig4debian-0.1.0/
~/stig-deb-test$ cd stig4debian-0.1.0/
~/stig-deb-test/stig4debian-0.1.0$ dh_make -f ../stig4debian_0.1.0.tar.gz
```

## 修改debian目录下的相关文件  

在使用dh_make命令进行了初始化工作后，多了一个debian目录，此目录下有许多文件，这些文件是和定制软件包行为相关的文件。

在此目录下必需的文件主要是control、copyright、changelog、rules这四个文件。根据实际情况进行文件的修改。

### control文件  

```
Source: stig4debian
Section: admin
Priority: optional
Maintainer: Samson W <sccxboy@gmail.com>
Build-Depends: debhelper (>= 9)
Standards-Version: 3.9.8
Homepage: https://github.com/hardenedlinux/STIG-4-Debian
Vcs-Git: https://github.com/hardenedlinux/STIG-4-Debian.git
Vcs-Browser: https://github.com/hardenedlinux/STIG-4-Debian.git

Package: stig4debian
Architecture: all
Depends: ${misc:Depends}
Description: DISA STIG for Debian 9 Porting from DISA RHEL 7 STIG V1 R1.
 DISA STIG(Security Technical Implementation Guides) for Debian 9 Porting from DISA RHEL 7 STIG V1 R1.
```

### rules 

```
#!/usr/bin/make -f
# See debhelper(7) (uncomment to enable)
# output every command that modifies files on the build system.
export DH_VERBOSE = 1


# see FEATURE AREAS in dpkg-buildflags(1)
#export DEB_BUILD_MAINT_OPTIONS = hardening=+all

# see ENVIRONMENT in dpkg-buildflags(1)
# package maintainers to append CFLAGS
#export DEB_CFLAGS_MAINT_APPEND  = -Wall -pedantic
# package maintainers to append LDFLAGS
#export DEB_LDFLAGS_MAINT_APPEND = -Wl,--as-needed


%:
	dh $@


# dh_make generated override targets
# This is example for Cmake (See https://bugs.debian.org/641051 )
#override_dh_auto_configure:
#	dh_auto_configure -- #	-DCMAKE_LIBRARY_PATH=$(DEB_HOST_MULTIARCH)

override_dh_install:
	install -d debian/stig4debian/usr/bin/
	install -g root -o root -m 755 -p stig-4-debian.sh debian/stig4debian/usr/bin/
```

以上的override_dh_install表示忽略掉默认的dh_install的操作，而使用
override_dh_install定义的动作；

##  编译  

```
stig4debian-0.1.0$ dpkg-buildpackage 
```

查看编译出deb包：

```
stig4debian-0.1.0$ ls ../*.deb
../stig4debian_0.1.0-1_all.deb

```
