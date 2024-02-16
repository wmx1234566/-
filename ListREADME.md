集群配置及软件说明文档
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
<img width="416" alt="image" src="https://github.com/wmx1234566/Distributed/assets/87322674/5733dcb6-a41a-4ea6-a2f8-84bf970dfc8e">

（3）修改host文件
vim /etc/hosts
192.168.0.10 deploy.ceph.local
192.168.0.11 node1.ceph.local
192.168.0.12 node2.ceph.local
192.168.0.13 node3.ceph.local
<img width="406" alt="image" src="https://github.com/wmx1234566/Distributed/assets/87322674/2ab71caf-d268-4d09-990b-7c3127402202">

Ping一下每个节点
ping deploy.ceph.local -c 1
ping node1.ceph.local -c 1
ping node2.ceph.local -c 1
ping node3.ceph.local -c 1
<img width="416" alt="image" src="https://github.com/wmx1234566/Distributed/assets/87322674/a97df5dc-cdca-4c69-9619-08515717a232">

（4）配置ssh互信
Delpoy节点：
ssh-keygen
<img width="371" alt="image" src="https://github.com/wmx1234566/Distributed/assets/87322674/058131be-b8ba-4504-bd4f-f6d17cc7da2f">

for host in deploy.ceph.local node1.ceph.local node2.ceph.local
node3.ceph.local; do ssh-copy-id -i ~/.ssh/id_rsa.pub $host; done
<img width="416" alt="image" src="https://github.com/wmx1234566/Distributed/assets/87322674/537096c2-e43a-4b63-80d1-343cf352e9e9">

（5）配置NTP
yum -y install ntp ntpdate ntp-doc
<img width="416" alt="image" src="https://github.com/wmx1234566/Distributed/assets/87322674/6175a147-27d8-471e-974b-c0eb228ecddd">

systemctl start ntpd
systemctl status ntpd
<img width="416" alt="image" src="https://github.com/wmx1234566/Distributed/assets/87322674/798f844a-7d9c-4f9e-9992-7d8ff97b54af">

（6）安装ceph-deploy
yum -y install ceph-deploy python-setuptools
<img width="416" alt="image" src="https://github.com/wmx1234566/Distributed/assets/87322674/c27f4449-cf62-458f-86da-653545c2ec14">

（7）创建目录
创建一个目录，保存 ceph-deploy 生成的配置文件和密钥对
mkdir cluster
cd cluster/
（8）创建集群
ceph-deploy new deploy.ceph.local
<img width="416" alt="image" src="https://github.com/wmx1234566/Distributed/assets/87322674/fe9bb034-935a-42bd-9c35-8d1b569915d5">

（9）修改配置文件
指定前端和后端网络
vim ceph.conf
public network = 192.168.0.0/24
cluster network = 10.0.0.0/24
[mon]
mon allow pool delete = true
<img width="324" alt="image" src="https://github.com/wmx1234566/Distributed/assets/87322674/73c13da9-f519-4eae-8521-d10dc15a39da">

（10）安装Ceph
ceph-deploy install deploy.ceph.local
ceph-deploy install node1.ceph.local
ceph-deploy install node2.ceph.local
ceph-deploy install node3.ceph.local
<img width="416" alt="image" src="https://github.com/wmx1234566/Distributed/assets/87322674/26ecdd7a-3bed-483b-b380-a30ff2446cec">
<img width="416" alt="image" src="https://github.com/wmx1234566/Distributed/assets/87322674/bf6c6063-bec4-452b-9a3b-9b40dd5f1150">
安装成功。
（11）初始化MON节点
ceph-deploy mon create-initial
<img width="416" alt="image" src="https://github.com/wmx1234566/Distributed/assets/87322674/3a71d540-9ddb-4e16-94e3-8cd884b1ebd9">

（12）收集密钥
ceph-deploy gatherkeys deploy.ceph.local
<img width="416" alt="image" src="https://github.com/wmx1234566/Distributed/assets/87322674/9271dd73-ce93-4835-ad28-2b1458c3f4a8">

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
<img width="416" alt="image" src="https://github.com/wmx1234566/Distributed/assets/87322674/ff5b67dc-b750-419f-920b-e8d5585977a9">

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
<img width="416" alt="image" src="https://github.com/wmx1234566/Distributed/assets/87322674/18ce8bca-8f8a-4f85-b78f-aa6ba4392408">

（15）拷贝配置和密钥
把配置文件和admin密钥拷贝至Ceph节点
ceph-deploy admin node1.ceph.local node2.ceph.local node3.ceph.local
<img width="416" alt="image" src="https://github.com/wmx1234566/Distributed/assets/87322674/1a9cd074-56ed-4634-8aee-4d2051bde1b2">

（16）初始化MGR节点
ceph-deploy mgr create node1.ceph.local
ceph-deploy mgr create node2.ceph.local
ceph-deploy mgr create node3.ceph.local
<img width="416" alt="image" src="https://github.com/wmx1234566/Distributed/assets/87322674/4add1058-2cc6-4572-868c-cad89d9acf1a">

cp ~/cluster/*.keyring /etc/ceph/
（17）查看集群状态
查看Ceph状态
ceph -s
<img width="377" alt="image" src="https://github.com/wmx1234566/Distributed/assets/87322674/ccd90f97-2e84-4c72-a7cf-8583cf864cb4">

查看OSD目录树
ceph osd tree
<img width="347" alt="image" src="https://github.com/wmx1234566/Distributed/assets/87322674/90f3f6e2-81c7-4ee7-89d6-37b7f045093f">

至此安装和配置工作完成，可以直接进行测试阶段。

对于测试，过程如下：

创建test.yaml文件（该文件路径：teuthology-api-main\.github\workflows\test.yaml），定义测试用例的名称、目的、参数、步骤和预期结果,这一部分在软件测试文档里有说明。

使用teuthology-suite命令来运行测试用例，指定yaml文件的路径和其他选项。

查看teuthology的web界面，监测测试用例的执行状态和结果。
