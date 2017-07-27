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
~/stig-deb-test$ mv STIG-4-Debian stig-4-debian.0.1.0
~/stig-deb-test$  tar -czvf stig-4-debian.0.1.0.tar.gz --exclude=.git stig-4-debian.0.1.0/
~/stig-deb-test$ cd stig-4-debian.0.1.0/
~/stig-deb-test/stig-4-debian.0.1.0$ dh_make -f ../stig-4-debian.0.1.0.tar.gz 
```


