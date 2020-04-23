# What is Harbian?

The full name of Harbian is Hardened Debian GNU/Linux. We provides some extra [packages](https://github.com/harbian/harbian_packages) and [configurations](https://github.com/hardenedlinux/hardenedlinux_profiles) for hardening purpose in Harbian. The current version of Harbian is only ship with [harbian-audit](https://github.com/hardenedlinux/harbian-audit) to comply with CIS/STIG.


# Build your own Debian GNU/Linux Base Installation ISO

Give a lady/gentleman an ISO, and you help her/him for a day. Teach a lady/gentleman to build their own distro, and you feed her/him for the whole maintainence life cycle. This is literally what we do on Harbian. To make a Custom Debian GNU/Linux Based Install ISO, you should build your own archive repositories

DAK (Debian Archive Kit) is use to hosting the official Debian repositories

The pros is this is the official solution, but the cons is lack of documentation. 

Inclusion of `.deb` need `.changes` file

In our case, we using `reprepro` to host our own repositories. And to include a `.deb` file, we don't need a '.changes' file.

For make a cd image, we using `simple-cdd`. This is a very simple tool that you can make a cd image with `debian-cd` package

To using simple-cdd, our repositories must have following Components that reprepro can't provide.
We have to manually install it.
Under base directory in repositories

```
doc                                #doc directory
README*                            #README files
extrafiles                         #hash with signature for other file
dists/buster/main/installer-amd64  #Installer for amd64
```

### Configurate Reprepro

Install Reprepro

```
apt install reprepro
```

Untill Apr 9 2020, we need about 130GB storage space for full mirror (only amd64 and source)
If you include `i386` and more architectures you should prepare more disk space.


Making directories for repositories
```
mkdir /data/mirror/debian/conf -p
```
go to the base directory

```
cd /data/mirror/debian/
mkdir -p ./conf/filterlist/
touch ./conf/filterlist/debian-buster-src
```

edit `conf/distributions`

```
Origin: harbian
Label: harbian Linux Server Main Repo
Codename: buster
Suite: stable
Architectures: amd64 source
Components: main 
UDebComponents: main
Contents: udebs percomponent allcomponents
Description: harbian Linux Server 
SignWith: CE7044058CA25835BA2E1ABFA7055AEC9ED4F04C
Log: harbian.log
Update: upstream-main

Origin: harbian
Label: harbian Linux Server Main Repo
Codename: buster-updates
Suite: stable
Architectures: amd64 source
Components: main 
UDebComponents: main
Contents: udebs percomponent allcomponents
Description: harbian Linux Server  updates
SignWith: CE7044058CA25835BA2E1ABFA7055AEC9ED4F04C
Log: harbian.log
Update: upstream-main-updates

Origin: harbian
Label: harbian Linux Server Main Repo
Codename: buster/updates
Suite: stable
Architectures: amd64 source
Components: main 
UDebComponents: main
Contents: udebs percomponent allcomponents
Description: harbian Linux Server  updates
SignWith: CE7044058CA25835BA2E1ABFA7055AEC9ED4F04C
Log: harbian.log
Update: security
```

```
SignWith: <your own public id> 
```
You can get it from `gpg -k`, if you don't have a key yet. you can use`gpg --gen-key` to get one

In my case, I'm using a testing key id: CE7044058CA25835BA2E1ABFA7055AEC9ED4F04C

And we need 3 `Codename` for our repositories work properly with `reprepro`
```
buster         # main repository
buster-updates # normal update
buster/updates # security updates
```
So we have three part above.


edit `conf/updates`

```
Name: upstream-main
Method: http://mirrors.163.com/debian/
Suite: buster
Components: main
Architectures: amd64 source
GetInRelease: no
FilterSrcList: install filterlist/debian-buster-src
VerifyRelease: blindtrust

Name: upstream-main-updates
Method: http://mirrors.163.com/debian/
Suite: buster-updates
Components: main
Architectures: amd64 source
GetInRelease: no
FilterSrcList: install filterlist/debian-buster-src
VerifyRelease: blindtrust

Name: security
Method: http://mirrors.163.com/debian-security
Suite: buster/updates
Components: main
UDebComponents: main
VerifyRelease: blindtrust
```
We using `VerifyRelease: blindtrust` just for test, you should using public key for production.

 

edit `conf/incoming`
```
Name: default
IncomingDir: incoming/
TempDir: temp/
MorgueDir: morgue/
LogDir: incoming-logs/
Allow: buster buster>buster-updates
Permit: unused_files older_version
Cleanup: unused_files on_deny on_error
```


sync the repro
```
reprepro -V update
```


### Manually install missing components


#### Installer-amd64

```
cd /data/mirror/debian/dists/buster/main
lftp -c mirror http://mirrors.163.com/debian/dists/buster/main/installer-amd64
```

#### README*
```
cd /data/mirror/debian
wget http://mirrors.163.com/debian/README
wget http://mirrors.163.com/debian/README.CD-manufacture
wget http://mirrors.163.com/debian/README.html
wget http://mirrors.163.com/debian/README.mirrors.html
wget http://mirrors.163.com/debian/README.mirrors.txt
```

#### Doc

```
cd /data/mirror/debian
lftp -c mirror http://mirrors.163.com/debian/doc
```

you should verify the hash from the upstream repo


#### Generate and sign hash

for installer
```
cd /data/mirror/debian/dists/buster
MD5=$(md5sum main/installer-amd64/current/images/SHA256SUMS | awk  '{print $1}' )
SHA1=$(sha1sum main/installer-amd64/current/images/SHA256SUMS | awk  '{print $1}' )
SHA256=$(sha256sum main/installer-amd64/current/images/SHA256SUMS | awk  '{print $1}' )
SIZE=$(ls -al main/installer-amd64/current/images/SHA256SUMS  | awk '{print $5}')
sed -i "/MD5Sum:/a \ $MD5 $SIZE main/installer-amd64/current/images/SHA256SUMS"  Release
sed -i "/SHA1:/a \ $SHA1 $SIZE main/installer-amd64/current/images/SHA256SUMS" Release
sed -i "/SHA256:/a \ $SHA256 $SIZE main/installer-amd64/current/images/SHA256SUMS" Release
gpg -u harbian-repo-maintainer -bao Release.gpg Release
```

for `extrafiles`(other files)

```
/data/mirror/debian/
rm extrafiles
sha256sum $(find * -type f | egrep -v '(pool|i18n|dep11|source)/|Contents-.*\.(gz|diff)|installer|binary-|(In)?Release(.gpg)?|\.changes' | sort | sed -e "/^conf/d"  -e "/^db/d") > /tmp/extrafile
gpg --no-options --batch --no-tty --armour --personal-digest-preferences=SHA256  --no-options --batch --no-tty --armour --default-key 9ED4F04C --clearsign --output extrafiles /tmp/extrafile
```

### Setting up the http access

Install apache2
```
apt install apache2
```

edit the /etc/apache2/sites-available/000-default.conf
```
<VirtualHost *:80>
	# The ServerName directive sets the request scheme, hostname and port that
	# the server uses to identify itself. This is used when creating
	# redirection URLs. In the context of virtual hosts, the ServerName
	# specifies what hostname must appear in the request's Host: header to
	# match this virtual host. For the default virtual host (this file) this
	# value is not decisive as it is used as a last resort host regardless.
	# However, you must set it for any further virtual host explicitly.
	#ServerName www.example.com

	ServerAdmin webmaster@localhost
	DocumentRoot /data/mirror

	# Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
	# error, crit, alert, emerg.
	# It is also possible to configure the loglevel for particular
	# modules, e.g.
	#LogLevel info ssl:warn

	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined

	# For most configuration files from conf-available/, which are
	# enabled or disabled at a global level, it is possible to
	# include a line for only one particular virtual host. For example the
	# following line enables the CGI configuration for this host only
	# after it has been globally disabled with "a2disconf".
	#Include conf-available/serve-cgi-bin.conf

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
# /etc/apache2/conf.d/repos

<Directory "/data/mirror" >
        # We want the user to be able to browse the directory manually
        Options Indexes FollowSymLinks Multiviews
        Require all granted
</Directory>

# This syntax supports several repositories, e.g. one for Debian, one for Ubuntu.
# Replace * with debian, if you intend to support one distribution only.
<Directory "/data/mirror/debian/db">
        Require all denied
</Directory>

<Directory "/data/mirror/debian/conf/">
        Require all denied
</Directory>

<Directory "/data/mirror/debian/incoming/">
        Require all denied
</Directory>
</VirtualHost>
```

check the config file
```
/usr/sbin/apachectl configtest
```
restart apache2 service  for changes to take effect
```
systemctl restart apache2
```


### Using Simple-CDD to make a cd image

Install simple-cdd packages
```
apt install simple-cdd
```
import your public key to apt-key

```
gpg --armor --output cert.gpg --export harbian-repo-maintainer@hardenedlinux.org
sudo apt-key add cert.gpg
sudo mv /etc/apt/trusted.gpg /etc/apt/trusted.gpg.d/harbian-archive.gpg
```
note: make sure trusted.gpg only have one gpg key under the mail: `harbian-repo-maintainer@hardenedlinux.org`


Build image

```
cd ~/
mkdir my-images
cd my-images
build-simple-cdd --profiles-udeb-dist buster --debian-mirror http://192.168.3.17/debian/ --dist buster --security-mirror http://192.168.3.17/debian --keyring /etc/apt/trusted.gpg.d/harbian-archive.gpg
```
should output

```
/home/debian/my-images/images/debian-10-amd64-CD-1.iso
```

Custom Profiles

###### custom preceed with one shell command
because every build with simple-cdd will using /usr/share/simple-cdd/profiles/default* profile
so we should edit /usr/share/simple-cdd/profiles/default.preseed 

add following content at the end of config file
```
d-i preseed/late_command string \
    in-target /bin/bash -c 'echo "harbian...." > /root/harbian'
```
so we can execute `/bin/bash -c 'echo "harbian...." > /root/harbian'` at the end of installation

###### add a custom deb package

for example using harbianaudit package

```
cd ~/my-images
mkdir custompkg
mkdir profiles
touch profiles/harbian.packages
```
add package name to the `profiles/harbian.packages`
in my case:

```
harbianaudit
```
copy the deb file to local-packages directory

```
cp harbianaudit_0.4.1-1_all.deb ~/my-images/custompkg
```

command example:
```
build-simple-cdd --local-packages /path/to/your/deb/files -p myprofile
```
in my case:
```
build-simple-cdd --profiles-udeb-dist buster --debian-mirror http://192.168.3.17/debian/ --dist buster --security-mirror http://192.168.3.17/debian --keyring /etc/apt/trusted.gpg.d/harbian-archive.gpg --local-packages custompkg/ -p harbian
```
`--local-packages custompkg` specific the local-packages directory is `custompkg`   
`-p harbian` specific the `harbian.*` under the `profiles` directory in your directory.   


###### Custom harbian profile full-step

making necessary directories

```
mkdir ~/harbian/profiles -p
mkdir ~/harbian/custompkg
```

copy harbianaudit package to `custompkg`

```
cp harbianaudit_0.4.1-1_all.deb ~/harbian/custompkg
```

configure `profiles/harbian.packages`
add necessary package

```
less
net-tools
bc
openssh-server
pciutils
network-manager
man-db
harbianaudit
```

according to (debian installer internal)[https://d-i.debian.org/doc/internals/] and debian official install image. We can know the difference betweent `debian official install image` and the image made from `simple-cdd` is simple-cdd's `default.preseed`. So in the `preseed` file, we should modify `profiles-releated`. And leave everything untouched.   

So configure `/usr/share/simple-cdd/profiles/default.preseed`   
and remove all default `preseed` configuration but `anna-install simple-cdd-profiles`   

```
cp /usr/share/simple-cdd/profiles/default.preseed /usr/share/simple-cdd/profiles/default.preseed.bak

echo "d-i preseed/early_command string anna-install simple-cdd-profiles" > /usr/share/simple-cdd/profiles/default.preseed
```

In order to run the script at the end of installation.

configure `profiles/harbian.packages`

```
d-i preseed/late_command string \
    in-target /bin/bash -c '/opt/harbianaudit/bin/harbianaudit.sh' 
```

making custom image

```
build-simple-cdd --profiles-udeb-dist buster --debian-mirror http://192.168.3.17/debian/ --dist buster --security-mirror http://192.168.3.17/debian --keyring /etc/apt/trusted.gpg.d/harbian-archive.gpg --local-packages custompkg/ -p harbian
```
`--local-packages custompkg` specific the local-packages directory is `custompkg`   
`-p harbian` specific the `harbian.*` under the `profiles` directory in your directory.   



##### Reference

http://web.archive.org/web/20140218013924/http://anonscm.debian.org/gitweb/?p=mirrorer/reprepro.git;a=blob_plain;f=docs/short-howto;hb=HEAD   
https://wiki.debian.org/DebianRepository/Setup?action=show&redirect=HowToSetupADebianRepository   
https://computermouth.com/tutorials/custom-debian-distro-simple-cdd/   
https://www.debian.org/releases/buster/amd64/apbs04.en.html   
https://d-i.debian.org/doc/internals/ch02.html   
