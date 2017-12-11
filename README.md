### Google-Cloud-HDP-1-host-config

## HDP-1-host-config

> Configuration
- 3 VMs on google compute Engine 
- OS: CentOS 7
- RAM: 15 Go
- CPU: 4
- Boot disk: 200Go

> Cluster model

![MetaStore remote database](https://github.com/gamboabdoulraoufou/hdp-1-host-config/blob/master/img/archi_v2.png)

  
> Log as root on all VM and change root password `_All nodes_`  
```sh 
# log as root
sudo su - root

# change root password
passwd # set password
``` 


> Update repo and install some packages `_All nodes_`  
```sh  
# update
yum -y update

# install other packages
yum -y install openssh-server 
yum -y install openssh-clients
yum -y install curl 
yum -y install wget 
yum -y install tar 
yum -y install unzip 
yum -y install telnet 
yum -y install telnet-server
yum -y install lvm2
yum -y install ksh
yum -y install git
yum -y install libgc cpp gcc
```


> Install Java `_All nodes_`
```sh
# download java
curl -LO -H "Cookie: oraclelicense=accept-securebackup-cookie" \
"http://download.oracle.com/otn-pub/java/jdk/8u152-b16/aa0333dd3019491ca4f6ddbe78cdb6d0/jdk-8u152-linux-x64.rpm"

# change file right
chmod +x jdk-8u152-linux-x64.rpm

# installation
rpm -ivh jdk-8u152-linux-x64.rpm

# check java installation
java -version

# export Java path
export JAVA_HOME=/usr/java/jdk1.8.0_152
export PATH=$JAVA_HOME/bin:$PATH  

``` 


> Reduce swappiness of the system `_All nodes_` 
```sh
# Set vm.swappiness to 10
echo 'vm.swappiness = 10' >> /etc/sysctl.conf

# initialise swap value
swapoff -a
swapon -a
``` 

> Enable NTP on the Cluster `_All nodes_` 

```sh
# install
yum -y install ntp

# check status
systemctl is-enabled ntpd

# set the NTP service to auto-start on boot
systemctl enable ntpd

# start ntp
systemctl start ntpd

# check ntp status
systemctl status ntpd

```

> Configure crontab to start ntp after reboot

```sh
# edit crontab
crontab -e

# add the following lines
#### START ####
# start ntp
@reboot sudo systemctl start ntpd
#### END ####
```


> Configure firewall `_All nodes_` 
My VMs have 1 network interface (eth0)

```sh
# check firewall status (is should be running)
firewall-cmd --state

# if firewall is not running, run this command
systemctl enable firewalld

# list all zones details
firewall-cmd --list-all-zones

# check interface zones
firewall-cmd --get-active-zones

# check active zone
firewall-cmd --get-active-zones

# enable hadoop port
firewall-cmd --permanent --zone=trusted --add-port 1-65535/tcp

# reboot to apply change
systemctl restart firewalld

# list all zones details
firewall-cmd --list-all-zones

```


> Disable IPv6 `_All nodes_` 

```sh
# Put the following entry to disable IPv6 for all adapter
echo 'net.ipv6.conf.all.disable_ipv6 = 1' >> /etc/sysctl.conf

# reflect the changes
sysctl -p
``` 


> Disable SELinux, PackageKit and Check umask Value `_All nodes_` 

```sh
# desable SELinux
sed -i -e 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config 

# disable packageKit is not enabled by default
echo "enabled=0" >> /etc/yum/pluginconf.d/refresh-packagekit.conf

# set UMASK value
umask 0022
```


> Disable transparent Huge Page compaction `_All nodes_`   
```sh
# check status
cat /sys/kernel/mm/transparent_hugepage/defrag

# disable
echo never > /sys/kernel/mm/transparent_hugepage/defrag
``` 


> Set Open File Descriptors to 10000 if the current value is less that 10000 `_All nodes_`  
```sh
ulimit -Sn
ulimit -Hn
ulimit -n 10000
```


> Modify sshd_config file `_All nodes_`
- Set PermitRootLogin to yes
- Set PasswordAuthentication to yes

```sh
# create a copy of sshd_config file
cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak

# change current configuration
sed -i -e 's/PermitRootLogin no/PermitRootLogin yes/g' /etc/ssh/sshd_config
sed -i -e 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config

# Restart SSH daemon
systemctl restart sshd.service

# check SSH daemon status
systemctl status sshd.service
```

> Create SSH key `_Ambari server node (hdp-1)_`

```sh
ssh-keygen
```

> Copy SSH key from ambari server to all cluster nodes `_Ambari server node (hdp-1)_`

```sh
ssh-copy-id -i /root/.ssh/id_rsa.pub root@hdp-dev-1.c.equipe-1314.internal
ssh-copy-id -i /root/.ssh/id_rsa.pub root@hdp-dev-2.c.equipe-1314.internal
ssh-copy-id -i /root/.ssh/id_rsa.pub root@hdp-dev-3.c.equipe-1314.internal
```


> Test ssh connexion  `_Ambari server node (hdp-1)_`
```sh
ssh root@poc-hdp-1.c.equipe-1314.internal
exit
ssh root@poc-hdp-2.c.equipe-1314.internal
exit
ssh root@poc-hdp-3.c.equipe-1314.internal
exit
``` 


> Modify sshd_config file `_All nodes_`
- Set PermitRootLogin to yes
- Set PasswordAuthentication to yes

```sh
# change current configuration
sed -i -e 's/PasswordAuthentication yes/PasswordAuthentication no/g' /etc/ssh/sshd_config

# Restart SSH daemon
systemctl restart sshd.service

# check SSH daemon status
systemctl status sshd.service
```

> Test ssh connexion  `_Ambari server node (hdp-1)_`
```sh
ssh root@hdp-dev-1.c.equipe-1314.internal
exit
ssh root@hdp-dev-2.c.equipe-1314.internal
exit
ssh root@hdp-dev-3.c.equipe-1314.internal
exit
```


> Reboot `_All nodes_`   
```sh  
reboot
```
