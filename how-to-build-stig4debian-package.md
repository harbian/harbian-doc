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
```

## 修改源代码 

为了使在主脚本中调用的脚本存放至/usr/lib目录下面，对原始代码进行添加与修改： 

为了满足FHS（Filesystem Hierarchy Standard，具体可见参考1），需要将执行文件、使用到的库等放置到对应的目录中。
```
stig4debian-0.1.0$ sed -i '/\/usr\/share\/doc\/gnome\/copyright/iSCRIPTS_DIR="/usr/lib/stig4debian/"' stig-4-debian.sh
stig4debian-0.1.0$ sed -i 's/scripts/${SCRIPTS_DIR}\/scripts/g' stig-4-debian.sh 
stig4debian-0.1.0$ sed -i 's/html\//\/usr\/lib\/stig4debian\/html\//g' stig-4-debian.sh 
stig4debian-0.1.0$ sed -i 's/stig-debian-9.txt/\/usr\/lib\/stig4debian\/stig-debian-9.txt/g' stig-4-debian.sh 
stig4debian-0.1.0$ sed -i 's/_LOG=/_LOG=\/var\/log\/stig4debian\//g' stig-4-debian.sh 
stig4debian-0.1.0$ sed -i '$s/STIG-for-Debian-/\/var\/log\/stig4debian\/STIG-for-Debian-/g' stig-4-debian.sh  
stig4debian-0.1.0$ sed -i 's/manual.txt/\/usr\/lib\/stig4debian\/manual.txt/g' stig-4-debian.sh
stig4debian-0.1.0$ mv stig-4-debian.sh stig4debian
```


## 对原始代码打包并进行初始化  

```
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

在此目录下必需的文件主要是control、copyright、changelog、rules这四个文件。根据实际情况进行文件的修改，除changelog、compat、control、copyright、rules等文件及source目录保留外，其它删除掉。

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
	install -g root -o root -m 755 -p stig4debian debian/stig4debian/usr/bin/stig4debian
	install -d debian/stig4debian/usr/lib/stig4debian/scripts/
	install -g root -o root -m 644 -p scripts/* debian/stig4debian/usr/lib/stig4debian/scripts/
	install -d debian/stig4debian/usr/lib/stig4debian/html/
	install -g root -o root -m 644 -p html/*  debian/stig4debian/usr/lib/stig4debian/html/
	install -g root -o root -m 644 -p stig-debian-9.txt  debian/stig4debian/usr/lib/stig4debian/
	install -g root -o root -m 644 -p manual.txt  debian/stig4debian/usr/lib/stig4debian/
	install -d debian/stig4debian/var/log/stig4debian/
	install -d debian/stig4debian/usr/share/man/man1/
	install -g root -o root -m 644 -p README.md debian/stig4debian/usr/share/man/man1/stig4debian.1
```

以上的override_dh_install表示忽略掉默认的dh_install的操作，而使用
override_dh_install定义的动作；


## copyright
```
Format: https://www.debian.org/doc/packaging-manuals/copyright-format/1.0/
Upstream-Name: stig4debian
Source: https://github.com/hardenedlinux/STIG-4-Debian

Files: *
Copyright: 2015-2017 Samson sccxboy@gmail.com
License: GPL-3.0+

Files: debian/*
Copyright: 2017 Samson W <sccxboy@gmail.com>
License: GPL-3.0+

License: GPL-3.0+
 This program is free software: you can redistribute it and/or modify
 it under the terms of the GNU General Public License as published by
 the Free Software Foundation, either version 3 of the License, or
 (at your option) any later version.
 .
 This package is distributed in the hope that it will be useful,
 but WITHOUT ANY WARRANTY; without even the implied warranty of
 MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
 GNU General Public License for more details.
 .
 You should have received a copy of the GNU General Public License
 along with this program. If not, see <https://www.gnu.org/licenses/>.
 .
 On Debian systems, the complete text of the GNU General
 Public License version 3 can be found in "/usr/share/common-licenses/GPL-3".
```

##  编译  

### 编译签名的包 
```
stig4debian-0.1.0$ dpkg-buildpackage 
```

### 编译不进行签名的包 
```
stig4debian-0.1.0$ dpkg-buildpackage -us -uc
```

编译不进行签名的包主要是为了让没有签名密钥的用户安装方便，但是必须进行sha512sum的计算，并提供给用户sha512sum的值的文件，以保证安装包没有被篡改过。

### 生成sha512sum文件   
``` 
$ sha512sum stig4debian_0.1.0-1_all.deb  > stig4debian_0.1.0-1_all.deb.sha512sum
```

查看编译出deb包：

```
stig4debian-0.1.0$ ls ../*.deb
../stig4debian_0.1.0-1_all.deb
```

## 编译环境的清理
```
stig4debian-0.1.0$ dh_clean
```


## 静态分析生成的deb包 

```
stig4debian-0.1.0$ lintian ../stig4debian_0.1.0-1_all.deb 
W: stig4debian: new-package-should-close-itp-bug
E: stig4debian: copyright-contains-dh_make-todo-boilerplate
W: stig4debian: extended-description-line-too-long
W: stig4debian: script-with-language-extension usr/bin/stig4debian
W: stig4debian: manpage-has-bad-whatis-entry usr/share/man/man1/stig4debian.1.gz
W: stig4debian: binary-without-manpage usr/bin/stig4debian
```

## 本地安装包 

若是对于没有签名密钥的用户进行安装未进行签名的包的安装时，首先要进行sha512sum什值的检查，以保证安装包的安全性；  

### sha512sum值的检查 
```
sha512sum -c stig4debian_0.1.0-1_all.deb.sha512sum
stig4debian_0.1.0-1_all.deb: OK
```

### 进行安装 
```
stig4debian-0.1.0# dpkg -i ../stig4debian_0.1.0-1_all.deb 
Selecting previously unselected package stig4debian.
(Reading database ... 41091 files and directories currently installed.)
Preparing to unpack ../stig4debian_0.1.0-1_all.deb ...
Unpacking stig4debian (0.1.0-1) ...
Setting up stig4debian (0.1.0-1) ...
Processing triggers for man-db (2.7.6.1-2) ...
```

## 本地卸载包 

```
stig4debian-0.1.0# dpkg -r stig4debian
(Reading database ... 41096 files and directories currently installed.)
Removing stig4debian (0.1.0-1) ...
Processing triggers for man-db (2.7.6.1-2) ...
```

## 参考

(1) https://www.debian.org/doc/packaging-manuals/fhs/fhs-2.3.html  
(2) https://www.debian.org/doc/manuals/maint-guide/index.en.html  
(3) https://debian-handbook.info/download/stable/debian-handbook.pdf   



