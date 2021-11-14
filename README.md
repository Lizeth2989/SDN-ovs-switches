# How to install ovs in docker containers (SDN-OVS switches)


Hey guys, here we have the step by step process to install OVS over docker containers. You can also use the same commands over any linux-virtual machines. 
If you have any problem, please let me know.

Here the youtube tutorial: 

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

# Basic Docker commands: to check images and docker containers lists
``` python
sudo docker images 
sudo docker ps -a
```

To pull images from Docker hub. Let's pull a basic Ubuntu
``` python
docker pull ubuntu
```

In case of you want to delete an imagen 
``` python
sudo docker rmi <id-of-image>
```
To delete a docker container
``` python
sudo docker rm -f <id-of-container>	
```

# To run and execute a container with the image
To run a docker with an specific image
``` python
sudo docker run -itd --name=sw-ovs10 ubuntu
```

To exec the container image 
``` python
sudo docker exec -it sw-ovs10  /bin/bash
```
## Install libraries - openvirtualswitch.org

``` python
apt-get update
apt-get install python3-pip git python-pip vim 
apt-get install -y wget git make iputils-ping net-tools  bridge-utils ifupdown
apt-get install -y openvswitch-switch openvswitch-common
apt-get install -y software-properties-common
apt-get install -y automake autoconf gcc uml-utilities libtool build-essential pkg-config 
apt-get install -y nvidia-modprobe
apt-get install -y libsqlite3-dev module-assistant 
```
## Check the kernel 
``` python
uname -r
uname -a
cd  /lib/modules/
ls 
```

Install the linux headers and image of your kernel 
``` python
apt-get install -y libssl-dev iproute2 tcpdump linux-headers-`uname -r`  linux-image-`uname -r` uml-utilities libelf-dev
apt-get install -y libssl-dev iproute2 tcpdump linux-headers-5.4.0-81-generic linux-image-5.4.0-81-generic uml-utilities libelf-dev 
```

Installing Java
``` python
apt-get install -y openjdk-8-jre openjdk-8-jdk
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-amd64
```

## Installing SDN OVS 
``` python
wget https://www.openvswitch.org/releases/openvswitch-2.15.0.tar.gz
tar xf openvswitch-2.15.0.tar.gz
cd /home/SDN-LIZ/openvswitch-2.15.0
```

## Additional libraries 
``` python
 modprobe gre
 modprobe nf_conntrack
 modprobe nf_defrag_ipv4
 modprobe nf_nat
 modprobe libcrc32c
 modprobe nf_defrag_ipv6
 modprobe nf_nat_ipv6
 modprobe nf_nat_ipv4 
 ```
 Make sure you run the kernel you want
``` python
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
```
Ovsdb-server
``` python
ovsdb-server --remote=punix:/usr/local/var/run/openvswitch/db.sock \
                     --remote=db:Open_vSwitch,Open_vSwitch,manager_options \
                     --private-key=db:Open_vSwitch,SSL,private_key \
                     --certificate=db:Open_vSwitch,SSL,certificate \
                     --bootstrap-ca-cert=db:Open_vSwitch,SSL,ca_cert \
                     --pidfile --detach
```
Starting OVS daemon and DBB
``` python
ovs-vsctl --no-wait init  
ovs-vswitchd --pidfile --detach 
ovs-ctl start
ovs-ctl status
```
# To verify ovs kernel module is loaded
``` python
ovs-vsctl --version
ovs-vsctl show
lsmod | grep openvswitch 
ps -ea | grep ovs
ps -eaf | grep ovsdb-server
```
Short-commands version
``` python
/usr/share/openvswitch/scripts/ovs-ctl start
ovsdb-server& 
ovs-vswitchd& 
ovs-vsctl&

```
Make sure you obtain these two services up, the ovsdb-server andd ovs-vswitchd
``` python
  34374 ?        00:00:00 ovsdb-server
  34377 ?        00:00:00 ovs-vswitchd
```

 ## Restart/Delete previous versions
 To restart, please delete conf.db file and dill the ports 
 ``` python
ps -ea | grep ovs
/etc/init.d/openvswitch-switch status
/etc/init.d/openvswitch-switch restart
rm -rvf /usr/local/etc/openvswitch/conf.db
rm -rvf /etc/openvswitch/conf.db
kill -9 34064
```

## Configure the interface
``` python
ovs-vsctl add-br foo
ovs-vsctl add-port foo eth0
ovs-vsctl set bridge foo protocols=OpenFlow13
ifconfig eth0 0
ifconfig foo up
ifconfig foo 172.17.0.9 netmask 255.255.0.0 up
route add default gw 172.17.0.1 foo
ovs-vsctl set-controller foo tcp:172.17.0.20:6633
ovs-vsctl set bridge br0 stp_enable=true
```

## Delete bridge 
``` python
ovs-vsctl del-port foo eth0
ovs-vsctl del-br foo
```
#Save image
``` python
sudo docker commit sw-ovs10 liz-ovs:1.0.2
```

#It allows you to change the bridge
``` python
sudo docker run -itd --name=sw_ovs_20  --cap-add NET_ADMIN liz-ovs:1.0.2
sudo docker exec -it sw_ovs_20 /bin/bash
```
## How to download/pull our docker images from docker hub
Docker container Image 1.0.1 (Openvswitch-2.15.0, Ubuntu kernel 5.4.0-81-generic)
 ``` python
docker pull liz2906/liz_ovs:1.0.1
```
Docker container Image 1.0.2 (Openvswitch-2.15.0, Ubuntu kernel 5.4.0-81-generic, supervisord)
 ``` python
docker pull liz2906/liz_ovs_supervisord:1.0.2
```
Docker container Image 1.0.3 (Openvswitch-2.15.0, Ubuntu kernel 5.4.0-84-generic, supervisord)
 ``` python
docker pull liz2906/liz_ovs_supervisord:1.0.3
```

If you have any problem, let me know @EcuatorianaEnChina
 
More information 
https://docs.openvswitch.org/en/latest/intro/install/general/
 
