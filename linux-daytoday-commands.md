# change password
```
> passwd
```

# du => View size of directory
```
du -sh ./
du -d 1 -h ./
```
# df
```
Filesystem     1K-blocks      Used Available Use% Mounted on
udev             8088972         0   8088972   0% /dev
tmpfs            1623428      3384   1620044   1% /run
/dev/sda1       97905224  10576480  82312412  12% /
tmpfs            8117132    214208   7902924   3% /dev/shm
tmpfs               5120         4      5116   1% /run/lock
```

# df -h
```
Filesystem      Size  Used Avail Use% Mounted on
udev            7.8G     0  7.8G   0% /dev
tmpfs           1.6G  3.4M  1.6G   1% /run
/dev/sda1        94G   11G   79G  12% /
tmpfs           7.8G  205M  7.6G   3% /dev/shm
tmpfs           5.0M  4.0K  5.0M   1% /run/lock
```

# scp
```
scp ahz_common root@192.168.1.164:/ahezime/bin
scp username@remote:/file/to/send /where/to/put
scp username@remote_1:/file/to/send username@remote_2:/where/to/put

scp ahezime@46.12.97.215:/root/dbbackups/ahezime_workflow.sql ./ahezime_workflow.sql

<> copy a remote folder to local directory
root@ahezime:~# scp -r root@1.2.3.4:/root/file ./file

<> copy local folder to remote directory
> scp -r ./dist ahezime@1.2.3.4:/root/abc/web/platform/
```

# rsync
```
rsync -zahP root@10.42.12.92:/data/sdb/lotus-user-1/.lotus ./
```

# Symbolic link (soft link)
```
ray@rayvm01:/usr/local/jdk1.8.0_271/bin$ sudo ln -s /usr/local/jdk1.8.0_271/bin/java /usr/local/bin
```

# systemctl 
Enable service to auto start
```
systemctl enable v2ray
```

Disable service to not to auto start
```
systemctl disable v2ray 
```

View enabled system services
```
systemctl list-unit-files --state=enabled
```

## View date & time
```
root@ahezime:~# date
Tue Mar 26 16:36:41 CST 2019
```
## Set time zone
```
root@ahezime:~# cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
## View macheine kernel
```
uname -a
```

## Change to root
```
sudo -i
sudo su -
```

## Change password
```
passwd 
```

## Remove everything of the current folder
```
ray@ray-pc:~/go_workspace/pkg/linux_amd64$ sudo rm -rf *
```

## Create a folder and grant permissions to current user
```
chmod -R 777 go
sudo chown -R ray go
```
## Processes related
```
ps -ax | grep nginx
```

## Open nautilus with root
gksudo nautilus

## Delete pattern matched files.
```
find /path/to/directory -type f -name '*[0-9]x[0-9]*[0-9]x[0-9]*.jpg' -delete
find /path/to/directory -type f -name '*[0-9]x[0-9]*[0-9]x[0-9]*.jpg' -exec rm {} +
ray@ray-pc:~/go_workspace/src/wholepro$ find ./ -type f -name 'README_*_*.md' -exec rm {} +
```

## View max socket connections
```
ray@ray-pc:~$ ulimit -n
```

## Restart a service
```
sudo systemctl restart apache2
```

## Restart networking
```
/etc/init.d/networking restart
```

## Restart network-manager
```
sudo service network-manager
```

## show line number in vim
```
:set number or :set nu
:set nonumber or :set nonu
```

## Mout remote server directory by using sshfs
```
sshfs shendu@192.168.1.240:/shendu/bin ~/workspace/trans
sshfs shendu@192.168.1.240:/shendu/bin ~/workspace/trans
```

## Copy ssh public key to remote server.
```
ray@ray:~$ cat ~/.ssh/id_rsa.pub | ssh root@138.197.209.57 'cat >> .ssh/authorized_keys'
```

## Disable ssh public key access and enable password authentication.
```
$> sudo vim /etc/ssh/sshd_config
```
Change PubkeyAuthentication yes to PubkeyAuthentication no
Change PasswordAuthentication no to PasswordAuthentication yes
Restart sshd service
```
$> sudo systemctl restart sshd
```

## Speed up SSH connetion
```
vim /etc/ssh/sshd_config
Change GSSAPIAuthentication yes => GSSAPIAuthentication no
Change GSSAPICleanupCredentials yes => GSSAPICleanupCredentials no
Change UseDNS yes => UseDNS no
```

## Change device name
```
sudo hostname dock-regis-svr
sudo vim /etc/hostname
```

## Count rows of results
```
sudo docker images | tee >(wc -l)
```
### Or
```
sudo docker images | awk '{print} END {print NR}'
```

## Run process in the background by using nohup
```
nohup ./hello &
```

## View directory in tree structure
```
tree dir
```

## View tar.gz file structure
```
tar -tf nsq-1.0.0-compat.linux-amd64.go1.8.tar.gz
```

## netstat
```
sudo netstat -anp | grep :8151
```

## Aliases
```
INFANTGRPC=$GOPATH/src/infant/vendor/github.com/golang/protobuf/protoc-gen-go
#SDGRPC=$GOPATH/src/shendu.com/vendor/github.com/golang/protobuf/protoc-gen-go
SDGRPC=$GOPATH/bin
MICROGRPC=$GOPATH/src/wholepro/vendor/protoc-gen-go
alias iprotoc='PATH=$PATH:$INFANTGRPC /usr/local/bin/protoc -I . --go_out=plugins=grpc:.'
alias sdprotoc='/usr/local/bin/protoc --plugin=$SDGRPC/protoc-gen-go -I . --go_out=plugins=grpc:.'
alias mprotoc='$GOPATH/src/wholepro/vendor/protoc/protoc --plugin=$MICROGRPC/protoc-gen-go -I . --go_out=plugins=micro:.'
```

## supervisor
```
sudo apt-get install supervisor
```
### create supervisord.conf under dir /etc/supervisor
```
cd /etc/sueprvisor
cp echo_suerpvisord_conf > supervisord.conf
```
### Start supervisor service
```
sudo supervisord
```
### restart service
```
service supervisord.service restart
```

## count rows
```
[root@162 ~]# supervisorctl status  | wc -l
203
```

```
[root@162 ~]# supervisorctl status | grep sd_mobi | wc -l
40
```

## ip
### View ip addresses
```
ip address
```

## Create USB bootable disk (Especially for CentOS)
### Download CentOS image
https://www.centos.org/download/
### unetbootin
https://tecadmin.net/how-to-create-bootable-linux-usb-using-ubuntu-or-linuxmint/#
```
$ sudo add-apt-repository ppa:gezakovacs/ppa
$ sudo apt-get update
$ sudo apt-get install unetbootin
```
### dd ddutility (Very good for USB & SD card reader)
https://github.com/thefanclub/dd-utility
https://www.thefanclub.co.za/how-to/dd-utility-write-and-backup-operating-system-img-and-iso-files-memory-card-or-disk

## After installed centos-7 minimal, set up the network to support networking
```
systemctl enable NetworkManager
systemctl start NetworkManager
nmcli conn show
nmcli conn up <name>
```

## Install openssh-server
### If shows error like, check on the topic `https://askubuntu.com/questions/546983/ssh-installation-errors`
```
sudo apt-get install openssh-server
Reading package lists... Done
Building dependency tree       
Reading state information... Done
Some packages could not be installed. This may mean that you have
requested an impossible situation or if you are using the unstable
distribution that some required packages have not yet been created
or been moved out of Incoming.
The following information may help to resolve the situation:

The following packages have unmet dependencies:
 openssh-server : Depends: openssh-client (= 1:7.2p2-4)
                  Depends: openssh-sftp-server but it is not going to be installed
                  Recommends: ssh-import-id but it is not going to be installed
E: Unable to correct problems, you have held broken packages.
```
### Then 
```
sudo apt-get install aptitude
```
### And then, install dependencies
```
sudo aptitude install openssh-client=1:7.2p2-4
```
### Finally, you can install openssh-server
```
sudo apt-get install openssh-server
```
### Check ssh server status
```
service sshd status
```

# Shtudown server
```
ray@ray-mini:~$ sudo shutdown -r now
or
sudo poweroff
```

# Open Files Limit
## Max number of open files limit
```
cat /proc/sys/fs/file-max
```
## 
```
lsof | wc -l
```
## ulimit command
### Limit of per process
```
ulimit -n
```
### Soft limit
```
ulimit -Sn
```
### Hard limit
```
ulimit -Hn
```
### See all limits
> ulimit -Sa
```
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 31242
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 31242
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```
> ulimit -Ha
```
core file size          (blocks, -c) unlimited
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 31242
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1048576
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) unlimited
cpu time               (seconds, -t) unlimited
max user processes              (-u) 31242
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```

# Linux service
## list all serices
```
systemctl list-unit-files --type=service
``` 

# Linux service
## list all serices
```
systemctl list-unit-files --type=service
``` 
# Fix Linux GUI frozen
https://www.fosslinux.com/39434/5-things-to-do-when-your-linux-system-gui-freezes.htm

# clear ubuntu DNS cache
```
sudo systemd-resolve --flush-caches
```
