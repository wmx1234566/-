<img width="416" alt="image" src="https://github.com/wmx1234566/Distributed/assets/87322674/0ede3f0a-47ca-44eb-aa4a-45e6cef1d5cc">集群配置及软件说明文档
一、环境节点及网络规划
序号	节点	Ip
1	client	192.168.0.10
2	node1.ceph.local	192.168.0.11
3	node2ceph.local	192.168.0.12
4	node3ceph.local	192.168.0.13
所有节点系统均为CentOS 8.3 2G+4G

二、搭建Ceph存储集群
以下所有操作均在所有节点上执行
（1）安装软件
yum -y install wget vim
<img width="416" alt="image" src="https://github.com/wmx1234566/Distributed/assets/87322674/630bcebe-3a72-4452-bba2-456d393a8fb2">

（2）关闭防火墙，SELinux
systemctl stop firewalld
systemctl disable firewalld
setenforce 0
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config

（3）修改host文件
vim /etc/hosts
192.168.0.10 deploy.ceph.local
192.168.0.11 node1.ceph.local
192.168.0.12 node2.ceph.local
192.168.0.13 node3.ceph.local

Ping一下每个节点
ping deploy.ceph.local -c 1
ping node1.ceph.local -c 1
ping node2.ceph.local -c 1
ping node3.ceph.local -c 1

（4）配置ssh互信
Delpoy节点：
ssh-keygen

for host in deploy.ceph.local node1.ceph.local node2.ceph.local
node3.ceph.local; do ssh-copy-id -i ~/.ssh/id_rsa.pub $host; done

（5）配置NTP
yum -y install ntp ntpdate ntp-doc

systemctl start ntpd
systemctl status ntpd

（6）安装ceph-deploy
yum -y install ceph-deploy python-setuptools

（7）创建目录
创建一个目录，保存 ceph-deploy 生成的配置文件和密钥对
mkdir cluster
cd cluster/
（8）创建集群
ceph-deploy new deploy.ceph.local

（9）修改配置文件
指定前端和后端网络
vim ceph.conf
public network = 192.168.0.0/24
cluster network = 10.0.0.0/24
[mon]
mon allow pool delete = true

（10）安装Ceph
ceph-deploy install deploy.ceph.local
ceph-deploy install node1.ceph.local
ceph-deploy install node2.ceph.local
ceph-deploy install node3.ceph.local

（长图省略）

安装成功。
（11）初始化MON节点
ceph-deploy mon create-initial

（12）收集密钥
ceph-deploy gatherkeys deploy.ceph.local

（13）擦除节点磁盘
ceph-deploy disk zap node1.ceph.local /dev/sdb
ceph-deploy disk zap node1.ceph.local /dev/sdc
ceph-deploy disk zap node1.ceph.local /dev/sdd
ceph-deploy disk zap node2.ceph.local /dev/sdb
ceph-deploy disk zap node2.ceph.local /dev/sdc
ceph-deploy disk zap node2.ceph.local /dev/sdd
ceph-deploy disk zap node3.ceph.local /dev/sdb
ceph-deploy disk zap node3.ceph.local /dev/sdc
ceph-deploy disk zap node3.ceph.local /dev/sdd 

（14）创建OSD
ceph-deploy osd create node1.ceph.local --data /dev/sdb
ceph-deploy osd create node1.ceph.local --data /dev/sdc
ceph-deploy osd create node1.ceph.local --data /dev/sdd
ceph-deploy osd create node2.ceph.local --data /dev/sdb
ceph-deploy osd create node2.ceph.local --data /dev/sdc
ceph-deploy osd create node2.ceph.local --data /dev/sdd
ceph-deploy osd create node3.ceph.local --data /dev/sdb
ceph-deploy osd create node3.ceph.local --data /dev/sdc
ceph-deploy osd create node3.ceph.local --data /dev/sdd

（15）拷贝配置和密钥
把配置文件和admin密钥拷贝至Ceph节点
ceph-deploy admin node1.ceph.local node2.ceph.local node3.ceph.local

（16）初始化MGR节点
ceph-deploy mgr create node1.ceph.local
ceph-deploy mgr create node2.ceph.local
ceph-deploy mgr create node3.ceph.local

cp ~/cluster/*.keyring /etc/ceph/
（17）查看集群状态
查看Ceph状态
ceph -s

查看OSD目录树
ceph osd tree

至此安装和配置工作完成，可以直接进行测试阶段。
