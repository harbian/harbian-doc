Debian official autobuilder is `wanna-build`, but it's too heavy for our purpose. We only need to rebuild package from source. Which download from debian official repositories. And the main reason is `wanna-build` is not well-documented. So we choose `mini-buildd`

### Compile mini-buildd package

#### Add unstable repo

Edit `/etc/apt/source.list` and add following repositories

```
deb http://ftp.cn.debian.org/debian sid main
deb-src http://ftp.cn.debian.org/debian sid main
deb http://ftp.cn.debian.org/debian experimental main
deb-src http://ftp.cn.debian.org/debian experimental main
```

Update apt cache

```
apt update
apt upgrade
```

#### Install dependencies

```
apt install build-essential git debhelper-compat po-debconf python3-sphinx python3-pygraphviz help2man python3-dateutil python3-argcomplete  python3-keyring python3-daemon  python3-pyftpdlib python3-django python3-django-registration python3-bs4 python3-setuptools
apt install -t buster dh-python
apt install -t experimental python3-twisted
```

#### Download source and build packages

```
cd ~
git clone https://salsa.debian.org/debian/mini-buildd.git
cd mini-buildd
git checkout debian/1.1.36
dpkg-buildpackage
```



Install mini-buildd package

```
cd ../
sudo dpkg -i mini-buildd_1.1.36_all.deb  mini-buildd-doc_1.1.36_all.deb  mini-buildd-utils_1.1.36_all.deb  python3-mini-buildd_1.1.36_all.deb
```

Fix broken dependencies and setting mini-buildd password

```
apt --fix-broken install -y
```

Now we can visit `http://<server ip>:8066/` for further configuration

### configurate mini-buildd autobuilder server


Choose `configurate` tab and login

#### Configurate Daemon Section

Click `Daemon` option -> `mini-buildd: Serving 0 repositories, 0 chroots, using 0 remotes`

change `Email address:` to proper value
change `Sbuild jobs:` based on your CPU cores

And save changes

Select the box of `mini-buildd: Serving 0 repositories, 0 chroots, using 0 remotes` and click `Prepare`
Select the box again and Click `Check`
Select the box again and Click `Activate`

Return to the `Configuration homepage`

#### Configurate Sources Section

Click the `local` of `Archives` row. This action will add local repositories inside `/etc/apt/source.list` to mini-buildd
You can Click  `Archives` to check. Or you can add it manually via `+Add` button. 


| ARCHIVE                                                |
| ------------------------------------------------------ |
| http://mirrors.163.com/debian-security/ (ping -1.0 ms) |
| http://mirrors.163.com/debian/ (ping -1.0 ms)          |

Click the `Debian` of `Sources` row. This action will add well-known debian source。
There's number `18` with `RED` background. 

| COLORED STATUS | SOURCE                                      | 1 ORIGIN         | 2 CODEVERSION | 3 CODENAME               |
| -------------- | ------------------------------------------- | ---------------- | ------------- |  ----------------------- | 
| Removed (-)    | Debian 'bullseye'                           | Debian           |               | bullseye                 |
| Removed (-)    | Debian 'bullseye-security'                  | Debian           |	              | bullseye-security        |
| Removed (-)    | Debian 'buster'	                           | Debian           |               | buster                   |
| Removed (-)    | Debian 'buster/updates                      | Debian           |               | buster/updates           |
| Removed (-)    | Debian 'jessie'	                           | Debian	          |               | jessie                   | 
| Removed (-)    | Debian 'jessie/updates'	                   | Debian	          |               | jessie/updates           | 
| Removed (-)    | Debian 'sid'	                               | Debian	          |               | sid                      | 
| Removed (-)    | Debian 'stretch'	                           | Debian	          |               | stretch                  | 
| Removed (-)    | Debian 'stretch/updates'                    | Debian	          |               | stretch/updates          | 
| Removed (-)    | Debian 'wheezy'	                           | Debian	          |               | wheezy                   | 
| Removed (-)    | Debian 'wheezy/updates'                     | Debian	          |               | wheezy/updates           | 
| Removed (-)    | Debian Backports 'buster-backports'	       | Debian Backports |               | buster-backports         | 
| Removed (-)    | Debian Backports 'jessie-backports'         | Debian Backports |               | jessie-backports         | 
| Removed (-)    | Debian Backports 'jessie-backports-sloppy'  | Debian Backports |               | jessie-backports-sloppy  | 
| Removed (-)    | Debian Backports 'stretch-backports'        | Debian Backports |               | stretch-backports        | 
| Removed (-)    | Debian Backports 'stretch-backports-sloppy' | Debian Backports |               | stretch-backports-sloppy | 
| Removed (-)    | Debian Backports 'wheezy-backports'	       | Debian Backports |               | wheezy-backports         | 
| Removed (-)    | Debian Backports 'wheezy-backports-sloppy'  | Debian Backports |               | wheezy-backports-sloppy  | 

We can delete the release that we don't need. by click release name , like "wheezy" and click `delete`.
In this example, we only choose 

| COLORED STATUS | SOURCE                                      | 1 ORIGIN         | 2 CODEVERSION | 3 CODENAME               |
| -------------- | ------------------------------------------- | ---------------- | ------------- |  ----------------------- | 
| Removed (-)    | Debian 'buster'	                           | Debian           |               | buster                   |
| Removed (-)    | Debian 'buster/updates                      | Debian           |               | buster/updates           |
| Removed (-)    | Debian 'sid'	                               | Debian	          |               | sid                      | 
| Removed (-)    | Debian Backports 'buster-backports'	       | Debian Backports |               | buster-backports         | 

Then we select all the box, and click `Prepare` and then `Check` and then `Activate` following contents   
   
You can add priority source by click the `Extras` of `Priority sources` row. 
   
Click the `Apt keys` of `Source` section. You can remove the keys that marked `removed`

| COLORED STATUS | APT KEY           |
| -------------- | ----------------- |	
| Removed (-)    | CBF8D6FD518E17E1: |
| Removed (-)    | 9D6D8F6BC857C906: |
| Removed (-)    | 7638D0442B90D010: | 
| Removed (-)    | 6FB2A1C265FFB764: |
| Removed (-)    | 8B48AD6246925553: |

delete it one by one.

#### Configurate Repositories Section

Click the `Defaults` of `Layouts` row to add default layouts
Click the `Layouts` to configurate `Default` layout，Click `Show` of `Version Options`

Change
```
Mandatory version regex: ~%IDENTITY%%CODEVERSION%\+[1-9]
Experimental mandatory version regex: ~%IDENTITY%%CODEVERSION%\+0
```
to 

```
Mandatory version regex: .*
Experimental mandatory version regex: .*
```


Click the `Distributions` of `Layouts` row to add distributions information
Click the `Test` of `Repositories` row to add repositories information

And then click `Repositories`, select `test` and click `Prepare` and then `Check` and then `Activate`

The `parepare` in this action would take about 1 mins.

Click the `Uploaders` of `Repositories` Section.
Click `'admin' may upload to '' with key ': '`, you can put your GPG public key in `Key:`

for example

```
gpg --gen-key

Real name: hardenedlinux Archive Team
Email address: hardenedlinux_archive_team@hardenedlinux.org
```
Note: for convenient, please choose empty passphrase

show the key id and export the public key.

list key
```
gpg -k

/root/.gnupg/pubring.kbx
------------------------
pub   rsa3072 2020-08-01 [SC] [expires: 2022-08-01]
      7D492015C7A54E339866843048B077394958138D
uid           [ultimate] hardenedlinux Archive Team <hardenedlinux_archive_team@hardenedlinux.org>
sub   rsa3072 2020-08-01 [E] [expires: 2022-08-01]
```

export public key

```
gpg  --armor --export 7D492015C7A54E339866843048B077394958138D
-----BEGIN PGP PUBLIC KEY BLOCK-----

mQGNBF8lGHwBDADxjXXFVrOd5//VcP6OSrNiCYJ+ma2FKYIPnequHdYrggmhf9/g
V0jrQydTPb9U6SLad99nPTr/QqdYgiIcCy3ouuw5TIvZX1C1ND+T8H8ogKfEJlmC
i09jDUlnoq96pvZ1RLpz5wQHbwvGOHQcmu359yeUNL9gMp44xQv/i4BVTNHaYxMk
UD4Tm4FcVliYP4/yyQY9i01cwaKeFKtjgyd09XUTQhxC+OO98sIx85kWEwtWYEWU
Vheak5j6LCQ0Ynoqs54M4mfIsdaK/rbFD0jgtSbBq+7uzDVsQcapbvwC4FsOD4bJ
IgBHgU5lDthhvJGWwewub7/1unyFTiQ2TIjSci5WhZzW39nynLafwtsgRTvu9lhR
IQXf/AZX/7FLR5ocbE8qPOgj6kox0btNneQT7H80PFxPBw+2ADzPACn1XEJDUuHG
rFw7FkC6QrKmQrpYhY9K/zhHBSACAVq6x3iimN3D7j8zM22HeDpqTUEqEl4U877K
l6PnjaAJQdWO6wEAEQEAAbRJaGFyZGVuZWRsaW51eCBBcmNoaXZlIFRlYW0gPGhh
cmRlbmVkbGludXhfYXJjaGl2ZV90ZWFtQGhhcmRlbmVkbGludXgub3JnPokB1AQT
AQoAPhYhBH1JIBXHpU4zmGaEMEiwdzlJWBONBQJfJRh8AhsDBQkDwmcABQsJCAcC
BhUKCQgLAgQWAgMBAh4BAheAAAoJEEiwdzlJWBONw3kMANTiT6LYQMaV0nyPc2ZH
M/EjX2XmyrzYF6hYC4MNT/Qh3FZCcIdOFUpxo/1e1WFHwF0cLV2Wr80eoOh7pCXF
Cw7nuaWUMQy93dW0FU8Kp5YS+Xg7KK8eQy5B4PKQd2zqGlcV7Dz6zosjKwN/adOj
hphYDANlI2PyH0p0fQoIkBLQK8N16Dm+FrY+UQ1xQ9GIn0orrTRgLUwFntQvVI1Y
O2FQZ6tmdUJqkBV8o/iu8ZXmTmf25/2oENp+IQtK0dyO2dXNDqUWqmoldPD/ItgO
s3QKEQEZHVsK8Mh1MsGdIesiKcx3F8llcJE9/eza702xezUYnoiIqY4rOAC+JEmS
Sz+LkJrjP/KMnV5hkxJTT0UABNxoHEpNDOjep1XO+E1ucZi0nYM0yFcf3Z5agFjl
OqCb2a9ZE2TXsz1+c44WYLR7qFrrFTQwWEZGTIeyQfMASn4IZMF3kcWB9cd6UZDJ
R/aGuWNrdAMnB0aJnJDCUp9U4KOJb0kHOxOp0fVAnzh46LkBjQRfJRh8AQwA60cx
7R+TAZsDAimK0gRoSuDeQdHoP9qmOXWTwTKbYu+NYV+abnL7cFkjiuUBfLiq8iY0
0OhgTHzjtscKPWXJD29od4tFvsg61aXo7lXbcNtRPissStTC6qgysbBZpWEg60Fk
MNxR8dkuu2W1ApWozZicPZjS77WFPmt2482qxMnCeXjQD+RTM4mIyhdUkhPHWrAZ
xdBMce+c6vlUgwDbLmg0Tp0dupxEneMDIndKcDucdcXUuwJ+NvwyPDubpxLF/+NM
0cRGOk7Cbae2AVLjOnsG6BItW/cA6OdUY5uIEqOrsf4pXqXOTknnvNmcoev7937M
TpV9Mc+pBw005WezDNDiwBOYTJqxJ3IqfRz2HwXSExKmOqNrL4hyDS31gFI3XC/2
cV2VCuMfoFhEN+G+MGx+41XxWjhHcxvLgvTssKYXOMKwQzZ1X6ih/BxzW4xU671k
ZKQzalztZw0xKZ2CeS7v5Oek7XIuLHinWp5+qeYbckCeKwyEXN5vOugjpGoxABEB
AAGJAbwEGAEKACYWIQR9SSAVx6VOM5hmhDBIsHc5SVgTjQUCXyUYfAIbDAUJA8Jn
AAAKCRBIsHc5SVgTjU6uC/460HpFTqhfZtrELq5347YCrcrBpN+cqwb0W3WauSiB
DiN3Mslei+aZrkIPRhr5PDCLdwC6heBBXeDkB+tbsqUovl4jxq0JhAocvVtEcwcN
YCi2u29EZcNVRXBJzxzztMEkq/vFCfxM74jX/856vmICpxZCxGhpZ/Fbth6L/yP9
OzVi1ntLj07Och4wvfcu7athfjU+U3VFjZvgz1l6vXX0QAR69oSsI+Zl0H6x/VzX
dl7Ps7StulhwBsq0dtvk47XF+aT2IaouZFtXrwNykOfKvqTt/ODl212aQj+AV15p
ECLJeG4OZwi3LahNF+TFc9kwCqrIeNAAgE3rkqhzwc/kbL3pcAqsdaHgJgs5t0MV
dTOOQYV8MAPKJdqry+61nSQBG3hLCMuymlJOcUN9aVNiacJYtT038t/6i9zcRloD
Ce9Xz0X72bMqvFCWs9vZePOkUlS1otw70LH4kIXoITAkhdE7qSkn0N9LXOLvRRv2
Lj14sW+fFJTpUGnfOPPmei4=
=NFNm
-----END PGP PUBLIC KEY BLOCK-----
```

after putting PGP Public key Block in `Key:`

In `May upload to:` row, Click `Choose all`. This action will allow user to upload to this autobuilder server, which using this PGP public key to sign `changes` file and `dsc` file.

And then click `Save`

Then we select all the box, and click `Prepare` and then `Check` and then `Activate`


#### Configurate Chroots Section

We can simply click `Default` in `Dir chroots` row to create `dir chroot`, or you could choose other chroot methods

and click `Dir chroots` for further configuration.

Select all box, and click `Prepare` and then `Check` and then `Activate`

The `parepare` in this action would take about 15 to 20 mins.


#### Testing mini-buildd service

visit `http://<server ip>:8066/mini_buildd/` click `Toolbox` and click `keyringpackages`

And Click `Yes, run keyringpackages` to build keyring packages

After built keyring packages, click `Toolbox` and click `testpackages` to build test packages

You can see the building status from `http://<server ip>:8066/mini_buildd/` and see build log from `Events` tab

We can see some of package are mark `FAILED` while some package are mark `INSTALLED`, We can check the details by checking the buildlog

### Using mini-buildd service

We are using dput to upload `changes` file and `dsc` file and source tarball

```
apt install dput
```

visit the mini-buildd homepage to get dput configuration.

visit `http://<server ip>:8066/mini_buildd/` click `Toolbox` and click `getdputconf`

```
[mini-buildd-mini-buildd]
method   = ftp
fqdn     = mini-buildd:8067
login    = anonymous
incoming = /incoming
```

and putting this configuration into `/etc/dput.cf`

Download source code from repository
```
apt source htop
```
get the package directory name
```
PKG_NAME=$(find  -type d -name "htop*" | head -n1 | awk -F / '{print $2}')
```

generate `changes` file, and change `Distribution` to `sid-test-unstable`, and make sure change file will include original source by `-sa`

```
cd $PKG_NAME
dpkg-genchanges -S -sa -DDistribution=sid-test-unstable > ../"$PKG_NAME"_source.changes
```

Using `debsign` to sign `changes` and `dsc` file
```
debsign -k 7D492015C7A54E339866843048B077394958138D --re-sign ../"$PKG_NAME"_source.changes
```
Using dput to upload `changes` file, source and dsc file
```
dput mini-buildd-mini-buildd ../"$PKG_NAME"_source.changes
```

### Using mini-buildd repositories service

visit `http://<server ip>:8066/mini_buildd/` click `repositories`

you can find `test: sid buster (Overview)`

we can using `deb+src` to export APT source.list option.

```
deb http://mini-buildd:8066/repositories/test/ sid-test-unstable main contrib non-free
deb http://mini-buildd:8066/repositories/test/ sid-test-testing main contrib non-free
deb http://mini-buildd:8066/repositories/test/ sid-test-stable main contrib non-free
```

please change `mini-buildd:8066` to `<server ip>:8066`
you can find repositories public key by visit `http://<server ip>:8066/mini_buildd/` click `Toolbox` and click `getkey`

you can save it to a file, and using apt-key to trust this public

```
apt-key add mini-buildd.gpg
```

right now, you can update the apt cache

```
apt update
```
Check the htop package by using apt search
```
apt search htop

aha/unstable 0.5-1+b1 amd64
  ANSI color to HTML converter

bashtop/unstable 0.9.25-1 all
  Resource monitor that shows usage and stats

htop/unstable 2.2.0-3 amd64
  interactive processes viewer

htop-dbgsym/sid-test-unstable 2.2.0-3 amd64
  debug symbols for htop

libauthen-oath-perl/stable,unstable 2.0.1-1 all
  Perl module for OATH One Time Passwords

pftools/stable,unstable 3+dfsg-3 amd64
  build and search protein and DNA generalized profiles
```

Install htop's package with our mini-buildd repositories

```
root@mini-buildd:~# apt install htop-dbgsym
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following packages were automatically installed and are no longer required:
  geoip-database libbind9-161 libdns1104 libdns1110 libgeoip1 libicu63 libisc1100 libisc1105 libisccc161 libisccfg163 liblwres161 libmpdec2 libperl5.28 libpython3.7-minimal libpython3.7-stdlib libreadline7 python3-asn1crypto python3.7-minimal
Use 'apt autoremove' to remove them.
The following additional packages will be installed:
  htop
The following NEW packages will be installed:
  htop htop-dbgsym
0 upgraded, 2 newly installed, 0 to remove and 18 not upgraded.
Need to get 286 kB of archives.
After this operation, 462 kB of additional disk space will be used.
Do you want to continue? [Y/n] Y
Get:1 http://mini-buildd:8066/repositories/test sid-test-unstable/main amd64 htop-dbgsym amd64 2.2.0-3 [193 kB]
Get:2 http://mirrors.163.com/debian sid/main amd64 htop amd64 2.2.0-3 [92.9 kB]
Fetched 286 kB in 0s (905 kB/s)    
Selecting previously unselected package htop.
(Reading database ... 71198 files and directories currently installed.)
Preparing to unpack .../htop_2.2.0-3_amd64.deb ...
Unpacking htop (2.2.0-3) ...
Selecting previously unselected package htop-dbgsym.
Preparing to unpack .../htop-dbgsym_2.2.0-3_amd64.deb ...
Unpacking htop-dbgsym (2.2.0-3) ...
Setting up htop (2.2.0-3) ...
Setting up htop-dbgsym (2.2.0-3) ...
Processing triggers for man-db (2.9.3-2) ...
Processing triggers for mime-support (3.64) ...
```


### Reference

https://salsa.debian.org/debian/mini-buildd/    
