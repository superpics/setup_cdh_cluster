
> 部署一个 CDH 集群，在节点数量特别多的时候，这将是一件工作量巨大的任务，机械且重复的步骤实际上可以有脚本去完成。用 shell 实现比较难以维护、不够优雅，于是便有了这个基于 [ansible](https://baike.baidu.com/item/Ansible/20194655) 的 CDH 集群自动部署脚本，实际上只需要编写 yml 配置文件即可告诉 ansible 我们需要做的事情。代码已经上传到 [github](https://github.com/iconsider/setup_cdh_cluster)，要根据实际集群节点数修改 ansible 主机配置 /etc/ansible/hosts 


### 测试节点

| 角色 | IP | hostname |
| --- | --- | --- |
| master  | 192.168.220.31  | cdh1 |
| slave | 192.168.220.32 | cdh2 |
| slave | 192.168.220.33 | cdh3 |
| slave | 192.168.220.34 | cdh4 |
| slave | 192.168.220.35 | cdh5 |


### 准备工作
自动化部署前，仍有少量操作需要手工完成

#### 配置 hostname
> 配置各个节点 IP 到 hostname 的映射
```bash
# 编辑 host 文件
vi /etc/hosts

# 添加下列 ip 和 hostname 映射
192.168.220.31  cdh1
192.168.220.32  cdh2
192.168.220.33  cdh3
192.168.220.34  cdh4
192.168.220.35  cdh5
```

#### 免密登录
> 配置 cdh1 免密登录到 cdh1~5
``` bash
# cdh1 上安装
yum -y install openssh-clients
# cdh1 上生成密钥
ssh-keygen -t rsa

# cdh1 上复制 cdh1 的~/.ssh/id_rsa.pub 公钥到 cdh1~5 的~/.ssh/authorized_keys 中
ssh-copy-id root@cdh1
ssh-copy-id root@cdh2
ssh-copy-id root@cdh3
ssh-copy-id root@cdh4
ssh-copy-id root@cdh5

# 在节点 cdh1 上测试
# 成功的话应该不用输入任何密码
ssh cdh2          

```

#### 安装 Ansible 2.8
``` bash
# cdh1 节点上安装 ansible 2.8
yum install -y centos-release-ansible-28.noarch

# 查看 ansible 版本
ansible --version
```

#### 上传组件安装包
百度云下载链接（总大小 4.49 GB）：[https://pan.baidu.com/s/1mN8adHylRTNEYBQbv1m5nw](https://pan.baidu.com/s/1mN8adHylRTNEYBQbv1m5nw)  密码：hive

压缩包内容:

**base_file/packages:**

1. cloudera-manager-agent-6.2.0-968826.el7.x86_64.rpm
2. cloudera-manager-daemons-6.2.0-968826.el7.x86_64.rpm
3. cloudera-manager-server-6.2.0-968826.el7.x86_64.rpm
4. cloudera-manager-server-db-2-6.2.0-968826.el7.x86_64.rpm
5. enterprise-debuginfo-6.3.1-1466458.el7.x86_64.rpm
6. get-pip.py
7. jdk-8u261-linux-x64.tar.gz
8. my.cnf
9. mysql-5.7.30-1.el7.x86_64.rpm-bundle.tar
10. mysql-community-client-5.7.30-1.el7.x86_64.rpm
11. mysql-community-common-5.7.30-1.el7.x86_64.rpm
12. mysql-community-devel-5.7.30-1.el7.x86_64.rpm
13. mysql-community-embedded-5.7.30-1.el7.x86_64.rpm
14. mysql-community-embedded-compat-5.7.30-1.el7.x86_64.rpm
15. mysql-community-embedded-devel-5.7.30-1.el7.x86_64.rpm
16. mysql-community-libs-5.7.30-1.el7.x86_64.rpm
17. mysql-community-libs-compat-5.7.30-1.el7.x86_64.rpm
18. mysql-community-server-5.7.30-1.el7.x86_64.rpm
19. mysql-community-test-5.7.30-1.el7.x86_64.rpm
20. mysql-connector-java-5.1.47.jar
21. mysql_init.sql
22. RPM-GPG-KEY-cloudera
23. scala-2.13.0-M4.tgz

**base_file/parcels:**

1. CDH-6.2.0-1.cdh6.2.0.p0.967373-el7.parcel
2. CDH-6.2.0-1.cdh6.2.0.p0.967373-el7.parcel.sha
3. CDH-6.2.0-1.cdh6.2.0.p0.967373-el7.parcel.sha256
4. KAFKA-4.1.0-1.4.1.0.p0.4-el7.parcel
5. KAFKA-4.1.0-1.4.1.0.p0.4-el7.parcel.sha1
6. manifest.json
7. SPARK2-2.4.0.cloudera1-1.cdh5.13.3.p0.1007356-el7.parcel
8. SPARK2-2.4.0.cloudera1-1.cdh5.13.3.p0.1007356-el7.parcel.sha

```bash
# 解压 zip 包上传组件安装包到 cdh1，得到两个目录
/opt/base_file/packages
/opt/base_file/parcels
```


#### 组件安装包下载出处
##### 1. JDK

| CDH 版本 | Oracle JDK 支持版本 | OpenJDK 支持版本 |
| --- | --- | --- |
| 5.3 -5.15 | 1.7, 1.8 | none |
| 5.16 and higher 5.x releases | 1.7, 1.8 | 1.8 |
| 6.0 | 1.8 | none |
| 6.1 | 1.8 | 1.8 |
| 6.2 | 1.8 | 1.8 |
| 6.3 | 1.8 | 1.8, 11.0.3 或更高版本 |



| Oracle JDK 版本 | 说明 |
| --- | --- |
| 1.8u181 或以上版本 | 推荐 |
| 1.8u162 | 推荐 |
| 1.8u141 | 推荐 |
| 1.8u131 | 推荐 |
| 1.8u121 | 推荐 |
| 1.8u111 | 推荐 |
| 1.8u102 | 推荐 |
| 1.8u91 | 推荐 |
| 1.8u74 | 推荐 |
| 1.8u31 | 不能低于该版本 |


| OpenJDK 版本  | 说明 |
| --- | --- |
| 1.8u212 或以上版本  | 推荐  |
| 1.8u181  | 不能低于该版本 |

官方对 JDK 要求说明：[传送门](https://docs.cloudera.com/documentation/enterprise/6/release-notes/topics/rg_java_requirements.html#java_requirements)

Oracle 1.8u261 下载: [传送门](https://download.oracle.com/otn/java/jdk/8u261-b12/a4634525489241b9a9e1aa73d9e118e6/jdk-8u261-linux-x64.tar.gz)

##### 2. Scala 2.13.0
[https://www.scala-lang.org/download/](https://www.scala-lang.org/download/)

##### 3. Mysql driver 5.1.47
[https://mvnrepository.com/artifact/mysql/mysql-connector-java/5.1.47](https://mvnrepository.com/artifact/mysql/mysql-connector-java/5.1.47)

##### 4. Mysql 5.7
[https://dev.mysql.com/downloads/mysql/5.7.html](https://dev.mysql.com/downloads/mysql/5.7.html)
[https://downloads.mysql.com/archives/get/p/23/file/mysql-5.7.30-1.el7.x86_64.rpm-bundle.tar](https://downloads.mysql.com/archives/get/p/23/file/mysql-5.7.30-1.el7.x86_64.rpm-bundle.tar)

##### 5. Cloudera Manger
[https://archive.cloudera.com/cm6/6.2.0/redhat7/yum/RPMS/x86_64/](https://archive.cloudera.com/cm6/6.2.0/redhat7/yum/RPMS/x86_64/)

##### 6. Parcels
[https://archive.cloudera.com/cdh6/6.3.2/parcels/](https://archive.cloudera.com/cdh6/6.3.2/parcels/)


#### 部署
```bash
ansible-playbook ${your_path}/setup_cdh_cluster/ansible/deploy_cdh.yml
```

#### 访问 CDH 管理页面

[http://cdh1:7180](http://cdh1:7180)

#### FAQ
**cloudera-scm-server 不能启动**
 查看日志：/var/log/cloudera-scm-server/cloudera-scm-server.log
 
 #### TODO list
 1. 集群进行水平扩展时，对新节点初始化配置。
 
 
 #### 参考
官方安装步骤:  [https://docs.cloudera.com/documentation/enterprise/6/6.2/topics/installation.html](https://docs.cloudera.com/documentation/enterprise/6/6.2/topics/installation.html)
