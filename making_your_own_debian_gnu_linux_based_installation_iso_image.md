## Making your own Debian GNU/Linux Based Installation iso image



Download network install cd (netinst.iso)

```
wget https://mirrors.163.com/debian-cd/current/amd64/iso-cd/debian-10.3.0-amd64-netinst.iso
```

Install netinst in your iso making machine

#### Get Desired packages

Install AptMove package

```
apt install apt-move debootstrap
```

modify `/etc/apt-move.conf` and choose your `LOCALDIR` and set your `DIST` code name
in my case `DIST=buster`

```
LOCALDIR=/mirrors/debian
DIST=buster
```

Create Directory

```
mkdir -p /mirrors/debian
```
##### Install or Download desired package

Download desired packages using `apt` or `aptitude`

```
apt install <packages>
```

##### Move your packages

```
apt-move update
```

This command will move deb packages in `/var/cache/apt/archives/` to your `LOCALDIR`, for my case `/mirrors/debian`


#### Get DebianInstaller Binaries


Using `lftp` to get DebianInstaller files

```
apt install lftp
#mkdir -p <debian-dir>/dists/buster/{main,contrib,non-free}/debian-installer/binary-amd64/
mkdir -p /mirrors/debian/dists/buster/{main,contrib,non-free}/debian-installer/binary-amd64/
cd /mirrors/debian/dists/buster/main/debian-installer/binary-amd64/
lftp -c mirror http://ftp.debian.org/debian/dists/buster/main/installer-amd64/current/
```


#### Make Mirror Recognizable to Apt

Download files need by apt-ftparchive


Download `/debian/indices/overrides.<dists>.*.gz` for your distro, (e.g. for buster, /debian/indices/overrides.buster.*.gz) into the directory `indices` which is in the same root directory as your `pool` mirror directory (e.g. /mirrors/debian, so /mirrors/indices).

```
mkdir /mirrors/indices -p
cd /mirrors/indices
wget http://ftp.debian.org/debian/indices/override.buster.contrib.gz
wget http://ftp.debian.org/debian/indices/override.buster.contrib.src.gz
wget http://ftp.debian.org/debian/indices/override.buster.main.debian-installer.gz
wget http://ftp.debian.org/debian/indices/override.buster.main.gz
wget http://ftp.debian.org/debian/indices/override.buster.main.src.gz
wget http://ftp.debian.org/debian/indices/override.buster.non-free.debian-installer.gz
wget http://ftp.debian.org/debian/indices/override.buster.non-free.gz
wget http://ftp.debian.org/debian/indices/override.buster.non-free.src.gz
```
unzip the gzip packages

```
gzip -d -k *
```

#### Generate Package lists and Release files for your 'distribution'


create a directory for your scripts

```
#cd <debian-dir>
cd /mirrors/debian/ 
mkdir custom-cd-scripts
```
create an apt.conf such as the one at `DebianCustomCD/PoolAptConf`

```
  APT {
    FTPArchive {
      Release {
         Origin "debian-cd";
         Label "youretch";
         Suite "testing";
          Version "0.1";
              Codename "etch";
              Architectures "amd64";
              Components "main contrib";
              Description "Your Etch CD Set";
          };
        };
     };
```

create a file named customcd-di.conf such as the one at `DebianCustomCD/PoolDebianInstallerPackagesGzConf`

```
     Dir {
      ArchiveDir "/mirrors/debian";
      OverrideDir "indices";
      CacheDir "/tmp";
     };

     TreeDefault {
      Directory "pool/";
     };

     BinDirectory "pool/main" {
      Packages "dists/buster/main/debian-installer/binary-amd64/Packages";
      BinOverride "override.buster.main";
     };

     Default {
      Packages {
          Extensions ".udeb";
          Compress ". gzip";
      };

      Contents {
          Compress "gzip";
      };
     };
```

execute the command in the directory containing the pool and dists directories

```
cd /mirrors/debian
apt-ftparchive -c custom-cd-scripts/apt.conf generate custom-cd-scripts/customcd-di.conf
```

Generate Release file:

execute the following command in the directory containing the pool and dists directories:

```  
apt-ftparchive -c custom-cd-scripts/apt.conf release dists/buster/ > dists/buster/Release
```

Sign the release file (if you don't have a GPG key use: gpg --gen-key):

In my case I'm using `hardbian-test`

```
gpg -u hardbian-test -bao dists/buster/Release.gpg dists/buster/Release
```

Create the InRelease file:

```
gpg --clearsign --digest-algo SHA512 --local-user hardbian-test -o dists/buster/InRelease dists/buster/Release
```


### Create the actual CD set


Download and install the debian-cd package

```
apt install debian-cd
```
cd `/usr/share/debian-cd` and edit `CONF.sh`

```
export INSTALLER_CD=1
export MIRROR=/mirrors/debian
export TDIR=/mirrors/tmp
export OUT=/mirrors/debian-cd-test
export APTTMP=/mirrors/tmp/apt
#export CONTRIB=1
```

Start a new shell (e.g. bash)

Source `CONF.sh`

```
sudo -s
. CONF.sh
```

Some problem with debian-cd
So this may not work.
Solving this problem ASAP
