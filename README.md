# How to install ovs in docker containers


Hey guys, here we have the step by step process to install OVS over docker containers. You can also use the same commands over any linux-virtual machines. 
If you have any problem, please let me know. 

Check if docker has been installed 
``` python
docker --version
```
If you do not have it, please install dockers. 
``` python
sudo apt install docker.io 
sudo systemctl start docker
sudo systemctl enable docker
sudo systemctl status docker
docker run hello-world
```



sudo docker images 
sudo docker ps -a

docker pull ubuntu


## Remove an imagen 
sudo docker rmi <id-of-image>

## Remove a docker container
sudo docker rm -f <id-of-container>	

sudo docker run -itd --name=sw-ovs10 ubuntu
sudo docker exec -it sw-ovs10  /bin/bash

OVS openvirtualswitch.org
-----------------------------------
apt-get update && apt-get install -y ubuntu-desktop  
apt-get install wget git vim sudo iputils-ping net-tools make nano gedit bridge-utils virt-manager 
apt-get install -y automake autoconf gcc uml-utilities libtool build-essential pkg-config 
apt-get install libssl-dev iproute2 tcpdump linux-headers-4.15.0-46-generic uml-utilities libelf-dev
sudo apt-get install python-twisted-conch python-simplejson python-zope.interface python-qt4
apt install iproute2
apt-get install libsqlite3-dev module-init-tools aptitude  module-assistant auto-install openvswitch-datapath 
apt-get install inetutils-traceroute  traceroute ubuntu-standard
sudo apt-get install qemu-kvm libvirt-bin 
sudo apt-get install autoconf dh-autoreconf  systemd-sysv
pip install ryu
apt-get install python-simplejson python-qt4 python-twisted-conch
apt-get install netstat netcat 

must have
---------------
apt-get update
apt-get install python3-pip git python-pip vim 
apt-get install -y wget git make iputils-ping net-tools  bridge-utils ifupdown
apt-get install -y openvswitch-switch openvswitch-common
apt-get install -y software-properties-common
apt-get install -y automake autoconf gcc uml-utilities libtool build-essential pkg-config 
apt-get install -y nvidia-modprobe
apt-get install -y libsqlite3-dev module-assistant 

## Check your kernel 
uname -r
uname -a
cd  /lib/modules/
ls 

apt-get install -y libssl-dev iproute2 tcpdump linux-headers-`uname -r`  linux-image-`uname -r` uml-utilities libelf-dev
apt-get install -y libssl-dev iproute2 tcpdump linux-headers-5.4.0-81-generic linux-image-5.4.0-81-generic uml-utilities libelf-dev 

------------------
JAVA
apt-get install -y openjdk-8-jre openjdk-8-jdk
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-amd64

## Download the version
wget https://www.openvswitch.org/releases/openvswitch-2.15.0.tar.gz

tar xf openvswitch-2.15.0.tar.gz
cd /home/SDN-LIZ/openvswitch-2.15.0

 modprobe gre
 modprobe nf_conntrack
 modprobe nf_defrag_ipv4
 modprobe nf_nat
 modprobe libcrc32c
 modprobe nf_defrag_ipv6
 modprobe nf_nat_ipv6
 modprobe nf_nat_ipv4 

cd /home/SDN-LIZ/openvswitch-2.15.0
./configure --with-linux=/lib/modules/5.4.0-81-generic/build 
./configure --with-linux=/lib/modules/`uname -r`/build 
./boot.sh
make
make install
modinfo ./datapath/linux/openvswitch.ko
touch /usr/local/etc/ovs-vswitchd.conf
make modules_install
/sbin/modprobe openvswitch
mkdir -p /usr/local/etc/openvswitch
mkdir -p /usr/local/var/run/openvswitch

ovsdb-tool create /usr/local/etc/openvswitch/conf.db ./vswitchd/vswitch.ovsschema

export PATH=$PATH:/usr/local/share/openvswitch/scripts
***ovsdb-server
ovsdb-server --remote=punix:/usr/local/var/run/openvswitch/db.sock \
                     --remote=db:Open_vSwitch,Open_vSwitch,manager_options \
                     --private-key=db:Open_vSwitch,SSL,private_key \
                     --certificate=db:Open_vSwitch,SSL,certificate \
                     --bootstrap-ca-cert=db:Open_vSwitch,SSL,ca_cert \
                     --pidfile --detach

2021-09-14T05:48:54Z|00004|netlink_socket|INFO|netlink: could not enable listening to all nsid (Operation not permitted)

ovs-vsctl --no-wait init  
##start OVS daemon, DBB
ovs-vswitchd --pidfile --detach 
ovs-ctl start
ovs-ctl status
## verify ovs kernel module is loaded
ovs-vsctl --version
ovs-vsctl show
lsmod | grep openvswitch 
ps -ea | grep ovs

https://kb.juniper.net/InfoCenter/index?page=content&id=KB32283&actp=METADATA
check commands
ovs-ofctl show foo
ovs-appctl fdb/show foo
ovs-vsctl list-ports foo


/usr/share/openvswitch/scripts/ovs-ctl start
ovsdb-server& 
ovs-vswitchd& 
ovs-vsctl&
ps -eaf | grep ovsdb-server


  34374 ?        00:00:00 ovsdb-server
  34377 ?        00:00:00 ovs-vswitchd


 ## Restart/Delete previous versions
/etc/init.d/openvswitch-switch status
/etc/init.d/openvswitch-switch restart
rm -rvf /usr/local/etc/openvswitch/conf.db
rm -rvf /etc/openvswitch/conf.db
kill -9 34064


##Configure the interface
ovs-vsctl add-br foo
ovs-vsctl add-port foo eth0
ovs-vsctl set bridge foo protocols=OpenFlow13
ifconfig eth0 0
ifconfig foo up
ifconfig foo 172.17.0.9 netmask 255.255.0.0 up
route add default gw 172.17.0.1 foo
ovs-vsctl set-controller foo tcp:172.17.0.20:6633
ovs-vsctl set bridge br0 stp_enable=true

#Delete bridge 
ovs-vsctl del-port foo eth0
ovs-vsctl del-br foo

#Save image
sudo docker commit sw-ovs10 liz-ovs:1.0.2

sudo docker run -it --name=sw_ovs_20 liz-ovs:1.0.2 /bin/bash
#It allows you to change the bridge
sudo docker run -itd --name=sw_ovs_20  --cap-add NET_ADMIN liz-ovs:1.0.2
 sudo docker exec -it sw_ovs_20 /bin/bash


More information 
https://docs.openvswitch.org/en/latest/intro/install/general/
