# ClearLinux
## 安装Docker
```shell
# 解决 Error: cannot acquire lock file. Another swupd process is already running.
swupd autoupdate --disable

swupd bundle-add containers-basic
systemctl start docker && systemctl enable docker
docker version
```
## 安装yum
```shell
swupd bundle-add os-clr-on-clr
cp /etc/yum.conf /etc/yum.conf.old
rm /etc/yum.conf
cp yum.conf /etc/
yum update
```
### yum.conf
```shell
[main]
pkgpolicy=newest
gpgcheck=1

[clearlinux]
name=clearlinux
baseurl=https://download.clearlinux.org/current/x86_64/os/
gpgcheck=0

##optional yum repos below

#un-comment to enable

#[code]
#name=Visual Studio Code
#baseurl=https://packages.microsoft.com/yumrepos/vscode
#enabled=1
#gpgcheck=1
#gpgkey=https://packages.microsoft.com/keys/microsoft.asc

#[epel7]
#name=epel7
#baseurl=https://mirror.sfo12.us.leaseweb.net/epel/7/x86_64/
#gpgkey=https://dl.fedoraproject.org/pub/epel/RPM-GPG-KEY-EPEL-7
```
### yum -y install git 导致以下错误，系统无法启动
Segmentation fault (core dumped)
