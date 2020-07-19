# setup_cdh_cluster

>[官方安装步骤](https://docs.cloudera.com/documentation/enterprise/6/latest/topics/installation.html)

#### 关闭防火墙
```bash
# RHEL 7 compatible:
sudo systemctl disable firewalld
sudo systemctl stop firewalld
```

#### 配置 hostname
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


#### 下载清单
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

[官方对 JDK 要求说明](https://docs.cloudera.com/documentation/enterprise/6/release-notes/topics/rg_java_requirements.html#java_requirements)

[Oracle 1.8u261 下载](https://download.oracle.com/otn/java/jdk/8u261-b12/a4634525489241b9a9e1aa73d9e118e6/jdk-8u261-linux-x64.tar.gz)

##### 2. Scala
[scala 下载](https://www.scala-lang.org/download/)

##### 2. mysql driver
[mysql driver 5.1.47](https://mvnrepository.com/artifact/mysql/mysql-connector-java/5.1.47)

##### 3. Cloudera Manger
[Cloudera Manger](https://archive.cloudera.com/cdh6/6.3.2/redhat7/yum/)

##### 4. CDH 6.3.2
[CDH 6.3.2](https://archive.cloudera.com/cdh6/6.3.2/parcels/)






