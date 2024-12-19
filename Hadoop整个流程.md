（一）集群搭建

## 1.1新建虚拟机

1. 点击文件 -> 新建虚拟机 -> 稍后安装 -> Linux系统（CentOS7版本）-> 虚拟机名称位置

![image-20241218181150830](C:\Users\Yan\AppData\Roaming\Typora\typora-user-images\image-20241218181150830.png)

![image-20241218181220361](C:\Users\Yan\AppData\Roaming\Typora\typora-user-images\image-20241218181220361.png)

2. 编辑虚拟机位置 -> DVD使用ISO映像文件

![image-20241218181507247](C:\Users\Yan\AppData\Roaming\Typora\typora-user-images\image-20241218181507247.png)

3. 开启虚拟机，选择 -> install CentOS
   * 选择中文
   *  确认安装位置
   * 网络主机名
   * 文件选择
   * 设置root密码
   * 等待下载完成后重启
   * 接受许可证

![image-20241218182017174](C:\Users\Yan\AppData\Roaming\Typora\typora-user-images\image-20241218182017174.png)

![image-20241218182145782](C:\Users\Yan\AppData\Roaming\Typora\typora-user-images\image-20241218182145782.png)

![image-20241218182307949](C:\Users\Yan\AppData\Roaming\Typora\typora-user-images\image-20241218182307949.png)

## 1.2模板虚拟机环境准备

1. 查看本地电脑的VMent8地址，将虚拟机 IP 设为静态，并进行配置

   ```bash
   ipconfig
   ```

   ```
   vim /etc/sysconfig/network-scripts/ifcfg-ens33
   ```

![image-20241218183803425](C:\Users\Yan\AppData\Roaming\Typora\typora-user-images\image-20241218183803425.png)

![image-20241218192205846](C:\Users\Yan\AppData\Roaming\Typora\typora-user-images\image-20241218192205846.png)

2. 修改模板虚拟机名称

```
vim /etc/hostname 
```

* 修改为“node100”

3. 配置模板虚拟机主机名称映射 hosts 文件

```
vim /etc/hosts
```

![image-20241218184251932](C:\Users\Yan\AppData\Roaming\Typora\typora-user-images\image-20241218184251932.png)

4. 重启虚拟机

```
reboot
```

5. 在本地Windows中打开C:\Windows\System32\drivers\etc

![image-20241218184619499](C:\Users\Yan\AppData\Roaming\Typora\typora-user-images\image-20241218184619499.png)

6. 使用SSH工具连接虚拟机

```
[root@node100 ~]# yum install -y epel-release
```

7. 安装 epel-release

* 配置镜像源

```
vi /etc/yum.repos.d/CentOS-Base.repo
```

```
# CentOS-Base.repo
#
# The mirror system uses the connecting IP address of the client and the
# update status of each mirror to pick mirrors that are updated to and
# geographically close to the client.  You should use this for CentOS updates
# unless you are manually picking other mirrors.
#
# If the mirrorlist= does not work for you, as a fall back you can try the 
# remarked out baseurl= line instead.
#
#
 
[base]
name=CentOS-$releasever - Base - mirrors.aliyun.com
failovermethod=priority
baseurl=https://mirrors.aliyun.com/centos-vault/7.9.2009/os/$basearch/
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/centos-vault/RPM-GPG-KEY-CentOS-7
 
#released updates 
[updates]
name=CentOS-$releasever - Updates - mirrors.aliyun.com
failovermethod=priority
baseurl=https://mirrors.aliyun.com/centos-vault/7.9.2009/updates/$basearch/
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/centos-vault/RPM-GPG-KEY-CentOS-7
 
#additional packages that may be useful
[extras]
name=CentOS-$releasever - Extras - mirrors.aliyun.com
failovermethod=priority
baseurl=https://mirrors.aliyun.com/centos-vault/7.9.2009/extras/$basearch/
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/centos-vault/RPM-GPG-KEY-CentOS-7
 
#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-$releasever - Plus - mirrors.aliyun.com
failovermethod=priority
baseurl=https://mirrors.aliyun.com/centos-vault/7.9.2009/centosplus/$basearch/
gpgcheck=1
enabled=0
gpgkey=https://mirrors.aliyun.com/centos-vault/RPM-GPG-KEY-CentOS-7
 
#contrib - packages by Centos Users
[contrib]
name=CentOS-$releasever - Contrib - mirrors.aliyun.com
failovermethod=priority
baseurl=https://mirrors.aliyun.com/centos-vault/7.9.2009/contrib/$basearch/
gpgcheck=1
enabled=0
gpgkey=https://mirrors.aliyun.com/centos-vault/RPM-GPG-KEY-CentOS-7
```

```
[root@node100 ~]# yum install -y epel-release 
```

7. 关闭防火墙，关闭防火墙开机自启 

```
systemctl stop firewalld
systemctl disable firewalld.service
```

8. 使设置的用户具有root权限，且免密登录

```
[root@node100 ~]# vim /etc/sudoers
```

![image-20241218192544997](C:\Users\Yan\AppData\Roaming\Typora\typora-user-images\image-20241218192544997.png)

9. 在/opt/下创建 2 个目录，并修改所有者和所在组

```
[root@node100 ~]# cd /opt/
[root@node100 opt]# mkdir module/ software/ 
[root@node100 opt]# chown jcx:jcx module/ software/
```

10. 卸载虚拟机自带 JDK

```
[root@node100 opt]# rpm -qa | grep -i java | xargs -n1 rpm -e --nodeps
```

11. 重启虚拟机

```
reboot
```

## 1.3克隆虚拟机并配置网络

***以hadoop100为模板虚拟机，克隆三台虚拟机***

1. 配置node101的 ip地址

```
[root@node100 ~]# vim /etc/sysconfig/network-scripts/ifcfg-ens33
```

2. 修改克隆来的虚拟机主机名

```
[root@node100 ~]# vim /etc/hostname
```

3. 重启

reboot

# （二）安装软件

## 2.1 JDK、Hadoop安装

### 2.1.1 JDK

1. 拖动文件到xshell中等待待上传
2. 解压文件

```
tar -zxvf jdk-8u321-linux-x64.tar.gz -C /opt/module/
```

3. 配置JDK环境变量

```
sudo vim /etc/profile.d/dev_env.sh
```

```
#JAVA_HOME 
export JAVA_HOME=/opt/module/jdk1.8.0_321 
export PATH=$PATH:$JAVA_HOME/bin
```

4. 使修改后的环境变量生效

```
source /etc/profile
```

5. 检测JDK是否安装成功

```
java -version
```

### 2.1.2 Hadoop

1. 解压到指定目录

```
tar -zxvf hadoop-3.2.2.tar.gz -C /opt/module/
```

2. 配置环境变量

```
sudo vim /etc/profile.d/dev_env.sh
```

```
#HADOOP_HOME 
export HADOOP_HOME=/opt/module/hadoop-3.2.2 
export PATH=$PATH:$HADOOP_HOME/bin 
export PATH=$PATH:$HADOOP_HOME/sbin
```

3. 使修改后的文件生效（相当于确定）

```
source /etc/profile
```

4. 检测是否安装成功

```
hadoop version
```

# （三）Hadoop完全分布式配置

## 3.1 配置node123的module

1. 在 node101 把 node101 的/opt/module/jdk1.8.0_321 目录拷贝到 node102

```
scp -r /opt/module/jdk1.8.0_321 jiangyan@node102:/opt/module
```

2. 在 node102 把 node101 的/opt/module/hadoop-3.2.2 目录拷贝到 node102

```
scp -r jiangyan@node101:/opt/module/hadoop-3.2.2 /opt/module/
```

3. 在 node102 把 node101 的/opt/module 目录下的所有目录拷贝到 node103

```
scp -r jiangyan@node101:/opt/module/* jiangyan@node103:/opt/module
```

## 3.2 编写集群分发脚本 xsync

1. 在 node101 的/home/jcx 下新建目录 bin

```
[jcx@node101 ~]$ mkdir bin 
[jcx@node101 ~]$ cd bin/ 
[jcx@node101 bin]$
```

2. 【创建】xsync 脚本

```
touch /home/jcx/bin/xsync
```

```
#!/bin/bash 
#1. 判断参数个数
if [ $# -lt 1 ] 
then 
 echo Not Enough Arguement! 
 exit; 
fi 
#2. 遍历集群所有机器
for host in node101 node102 node103 
do 
 echo -e "\n==================== $host 
====================" 
 #3. 遍历所有目录，逐个发送
 for file in $@ 
 do 
 #4. 判断文件是否存在
 if [ -e $file ] 
 then 
 #5. 获取父目录
 pdir=$(cd -P $(dirname $file); pwd) 
 #6. 获取当前文件的名称
 fname=$(basename $file) 
 ssh $host "mkdir -p $pdir" 
 rsync -av $pdir/$fname $host:$pdir 
 else 
 echo $file does not exist!
 fi 
 done 
done 
echo -e "\n"
```

3. 使脚本 xsync 具有执行权限

```
chmod +x xsync
```

4. 同步分发/home/jcx/bin 至 node102、node103

```
[jcx@node101 bin]$ cd 
[jcx@node101 ~]$ xsync bin/
```

5. 分发环境变量

```
[jcx@node101 ~]$ sudo ./bin/xsync /etc/profile.d/dev_env.sh
```

6. 在 node102、node103 上分别使修改后的文件生效

```
[jcx@node102 ~]$ source /etc/profile 
[jcx@node103 ~]$ source /etc/profile
```

# （四）SSH免密登录配置

1. 在 node101 上生成私钥和公钥

```
[jcx@node101 ~]$ ssh-keygen -t rsa
```

2. 在 node101 上将公钥拷贝到要免密登录的目标机器

```
[jcx@node101 ~]$ ssh-copy-id node101 
[jcx@node101 ~]$ ssh-copy-id node102 
[jcx@node101 ~]$ ssh-copy-id node103
```

3. 在 node102、node103 上同样进行以上步骤

4. ssh [服务器名称] 进行检测

```
ssh node102
```

5. 在 node101 上切换至 root 用户，生成私钥和公钥

```
[jcx@node101 ~]$ su 
密码：
[root@node101 jcx]# cd 
[root@node101 ~]# ssh-keygen -t rsa
```

6. 用 root 用户在 node101 上将公钥拷贝到要免密登录的目标机器

```
[root@node101 ~]# ssh-copy-id node101 
[root@node101 ~]# ssh-copy-id node102 
[root@node101 ~]# ssh-copy-id node103
```

7. 在 node102、node103 上同样进行以上步骤

8. 记得最后切换到普通用户

# （五）集群配置

1. 进入 hadoop 目录删除文件

```
[jcx@node101 ~]$ cd /opt/module/hadoop-3.2.2/etc/hadoop/
```

```
rm core-site.xml yarn-site.xml hdfs-site.xml mapred-site.xml 
```

2. 添加配置文件

```
vi core-site.xml
vi hdfs-site.xml
vi yarn-site.xml
vi mapred-site.xml
vi workers
```

```
<!-- 指定 NameNode 的地址 --> 
 <property> 
 <name>fs.defaultFS</name> 
 <value>hdfs://node101:8020</value> 
 </property> 
 <!-- 指定 hadoop 数据的存储目录 --> 
 <property> 
 <name>hadoop.tmp.dir</name> 
 <value>/opt/module/hadoop-3.2.2/data</value> 
 </property> 
 <!-- 配置 HDFS 网页登录使用的静态用户为 jcx --> 
 <property> 
 <name>hadoop.http.staticuser.user</name> 
 <value>jiangyan</value> 
</property>
```

```
<!-- nn web 端访问地址--> 
 <property> 
 <name>dfs.namenode.http-address</name> 
 <value>node101:9870</value> 
 </property> 
 <!-- 2nn web 端访问地址--> 
 <property> 
 <name>dfs.namenode.secondary.http-address</name> 
 <value>node103:9868</value> 
 </property>
```

```
<!-- 指定 MR 走 shuffle --> 
 <property> 
 <name>yarn.nodemanager.aux-services</name> 
 <value>mapreduce_shuffle</value> 
 </property> 
 <!-- 指定 ResourceManager 的地址-->
 <property> 
 <name>yarn.resourcemanager.hostname</name> 
 <value>node102</value> 
 </property> 
 <!-- 环境变量的继承 --> 
 <property> 
 <name>yarn.nodemanager.env-whitelist</name> 
 
<value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_C
ONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_M
APRED_HOME</value> 
 </property> 
 <!-- 开启日志聚集功能 --> 
 <property> 
 <name>yarn.log-aggregation-enable</name> 
 <value>true</value> 
 </property> 
 <!-- 设置日志聚集服务器地址 --> 
 <property> 
 <name>yarn.log.server.url</name> 
 <value>http://node101:19888/jobhistory/logs</value> 
 </property> 
 <!-- 设置日志保留时间为 14 天 --> 
 <property> 
 <name>yarn.log-aggregation.retain-seconds</name> 
 <value>1209600</value> 
 </property>
```

```
<!-- 指定 MapReduce 程序运行在 Yarn 上 --> 
 <property> 
 <name>mapreduce.framework.name</name> 
 <value>yarn</value> 
 </property> 
 <!-- 历史服务器端地址 --> 
 <property> 
 <name>mapreduce.jobhistory.address</name> 
 <value>node101:10020</value> 
 </property> 
 <!-- 历史服务器 web 端地址 --> 
 <property> 
 <name>mapreduce.jobhistory.webapp.address</name> 
 <value>node101:19888</value> 
</property>
```

```
node101 
node102 
node103
```

3. 分发配置文件

```
[jcx@node101 hadoop]$ cd .. 
[jcx@node101 etc]$ xsync hadoop/
```

# （六）编写Hadoop集群脚本

** 一定要用<configuration></configuration> 包裹所有配置文件 ** ，否则就会一直报错“ illegal to have multiple roots”

```

```

------



```
vi myhadoop.sh
```

```
#!/bin/bash 
if [ $# -lt 1 ] 
then 
 echo "No Args Input!" 
 exit; 
fi 
case $1 in 
"start") 
 echo -e "\n================= 启动 hadoop 集群
=================" 
 echo " ------------------- 启动 hdfs --------------------" 
 ssh node101 "/opt/module/hadoop-3.2.2/sbin/start-dfs.sh" 
 echo " ------------------- 启动 yarn --------------------" 
 ssh node102 "/opt/module/hadoop-3.2.2/sbin/start-yarn.sh" 
 echo " --------------- 启动 historyserver ---------------" 
 ssh node101 "/opt/module/hadoop-3.2.2/bin/mapred --daemon 
start historyserver"
echo -e "\n" 
;; 
"stop") 
 echo -e "\n================= 关闭 hadoop 集群
=================" 
 echo " --------------- 关闭 historyserver ---------------" 
 ssh node101 "/opt/module/hadoop-3.2.2/bin/mapred --daemon 
stop historyserver" 
 echo " ------------------- 关闭 yarn --------------------" 
 ssh node102 "/opt/module/hadoop-3.2.2/sbin/stop-yarn.sh" 
 echo " ------------------- 关闭 hdfs --------------------" 
 ssh node101 "/opt/module/hadoop-3.2.2/sbin/stop-dfs.sh" 
 echo -e "\n" 
;; 
*) 
 echo "Input Args Error!" 
;; 
esac
```

```
vi jpsall
```

```
#!/bin/bash 
for host in node101 node102 node103 
do 
 echo -e "\n=============== $host ===============" 
 ssh $host jps 
done 
echo -e "\n"
```

2. 给予权限

```
[jcx@node101 bin]$ chmod +x myhadoop.sh jpsall
```

3. 分发脚本

```
[jcx@node101 bin]$ cd .. 
[jcx@node101 ~]$ xsync bin/
```

（七）启动集群

1. 集群第一次启动前，需要在 node101 节点格式化 NameNode  

```
[jcx@node101 ~]$ hdfs namenode -format
```

2. 启动集群（三台虚拟机任意一台都可以运行）

```
[jcx@node101 ~]$ myhadoop.sh start
```

![image-20241219121544114](C:\Users\Yan\AppData\Roaming\Typora\typora-user-images\image-20241219121544114.png)

------

<p color='red'>至此，我有了很多感悟。第一次搭建可能只是自己碰巧没遇到很多问题，我就以为都会了，可事实上，我可能只碰到了其中20%甚至更少的问题。我觉得如果要真正做到熟练掌握，就需要多去思考，多去解决问题，解决更多问题，日后才能更得心应手。</p> 
<p color='blue'>此外，问题只是一个问题，我们要学会觑魅，以及正视问题，意识到问题只是我们目前掌握的知识不足以解决它，需要我们学习更多。而并非意味着自我的否定，如果把注意力放在前面——“自我的否定上”，那么我们注定会越来越怯，之后正确认识问题，我们才能够更好的解决问题，从而不断进步</p>

<span> 至此，感谢专注的自己，感谢自己的坚持与不放弃。每一段痛苦都是一次成长。

# （七）Hive安装与配置

1. 进入/opt/software/目录

```
[ljc1@ljc102 ~]$ cd /opt/software/
```

上传apache-hive-3.1.2-bin.tar.gz

上传mysql-5.7.28-1.el7.x86_64.rpm-bundle.tar

上传mysql-connector-java-5.1.37.jar

2. 解压apache-hive-3.1.2-bin.tar.gz、apache-hive-3.1.2-bin.tar.gz

```
[ljc1@ljc102 software]$ tar -zxvf apache-hive-3.1.2-bin.tar.gz -C /opt/module/
```

3. 修改目录名称

```
[ljc1@ljc102 software]$ mv /opt/module/apache-hive-3.1.2-bin/ /opt/module/hive-3.1.2
```

4. 配置hive环境变量，在末尾添加内容

```
[ljc1@ljc102 software]$ sudo vim /etc/profile.d/my_env.sh
```

```
\#HIVE_HOME

export HIVE_HOME=/opt/module/hive-3.1.2

export PATH=$PATH:$HIVE_HOME/bin
```

5. 使环境变量的修改生效

```
[ljc1@ljc102 software]$ source /etc/profile
```

6. 删除冲突jar包

```
[ljc1@ljc102 software]$ rm /opt/module/hive-3.1.2/lib/log4j-slf4j-impl-2.10.0.jar
```

7. 使用脚本启动hadoop集群

```
[ljc1@ljc102 software]$ myhadoop.sh start
```

8. 初始化元数据库

```
[ljc1@ljc102 software]$ cd ../module/hive-3.1.2/bin/
[ljc1@ljc102 bin]$ schematool -dbType derby -initSchema
```

9. 启动hive

```
[ljc1@ljc102 bin]$ hive
```

10. 退出hive

```
hive> exit;
```

# （八）MySQL安装与配置

1. 查看是否已安装MySQL

```
[ljc1@ljc102 ~]$ rpm -qa|grep mariadb
```

2. 若已安装，需要卸载MySQL

```
[ljc1@ljc102 ~]$ sudo rpm -e --nodeps mariadb-libs
```

3. 进入/opt/software/目录，解压mysql-5.7.28-1.el7.x86_64.rpm-bundle.ta

```
[ljc1@ljc102 ~]$ cd /opt/software/
[ljc1@ljc102 software]$ tar -xf mysql-5.7.28-1.el7.x86_64.rpm-bundle.tar
[ljc1@ljc102 software]$ sudo yum install -y libaio
如果安装成功，或者显示以下内容，即可继续安装步骤
软件包 libaio-0.3.109-13.el7.x86_64 已安装并且是最新版本
无须任何处理
```

4. 安装，依次输入以下5条命令

```
sudo rpm -ivh mysql-community-common-5.7.28-1.el7.x86_64.rpm
sudo rpm -ivh mysql-community-libs-5.7.28-1.el7.x86_64.rpm
sudo rpm -ivh mysql-community-libs-compat-5.7.28-1.el7.x86_64.rpm
sudo rpm -ivh mysql-community-client-5.7.28-1.el7.x86_64.rpm
sudo rpm -ivh mysql-community-server-5.7.28-1.el7.x86_64.rpm
```

5. 初始化数据库

```
[ljc1@ljc102 software]$ sudo mysqld --initialize --user=mysql
```

6. 查看临时密码

```
[ljc1@ljc102 software]$ sudo cat /var/log/mysqld.log
```

7. 启动MySQL

```
[ljc1@ljc102 software]$ sudo systemctl start mysqld
登录
[ljc1@ljc102 software]$ mysql -uroot -p
输入临时密码
Enter password: 临时密码
登陆成功后，修改密码为000000
mysql> set password = password("000000");
使root允许任意ip连接
mysql> update mysql.user set host='%' where user='root';
mysql> flush privileges;
退出MySQL
mysql> exit;
拷贝MySQL的JDBC驱动至hive-3.1.2下的lib/
[ljc1@ljc102 software]$ cp /opt/software/mysql-connector-java-5.1.37.jar /opt/module/hive-3.1.2/lib/
编写配置文件hive-site.xml
[ljc1@ljc102 software]$ cd /opt/module/hive-3.1.2/conf/
[ljc1@ljc102 conf]$ vim hive-site.xml
```

```
输入以下内容，或直接上传资料包内的hive-site.xml
输入或上传前，注意修改主机名
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <!-- jdbc连接的URL -->
    <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:mysql://ljc102:3306/metastore?useSSL=false</value>
    </property>
    <!-- jdbc连接的Driver -->
    <property>
        <name>javax.jdo.option.ConnectionDriverName</name>
        <value>com.mysql.jdbc.Driver</value>
    </property>
    <!-- jdbc连接的username -->
    <property>
        <name>javax.jdo.option.ConnectionUserName</name>
        <value>root</value>
    </property>
    <!-- jdbc连接的password -->
    <property>
        <name>javax.jdo.option.ConnectionPassword</name>
        <value>000000</value>
    </property>
    <!-- Hive元数据存储版本的验证 -->
    <property>
        <name>hive.metastore.schema.verification</name>
        <value>false</value>
    </property>
    <!-- 元数据存储授权 -->
    <property>
        <name>hive.metastore.event.db.notification.api.auth</name>
        <value>false</value>
        </property>
    <!-- Hive默认在HDFS的工作目录 -->
    <property>
        <name>hive.metastore.warehouse.dir</name>
        <value>/user/hive/warehouse</value>
    </property>
    <!-- 指定存储元数据要连接的地址 -->
    <property>
        <name>hive.metastore.uris</name>
        <value>thrift://ljc102:9083</value>
</property>
    <!-- 指定hiveserver2连接的主机名 -->
    <property>
        <name>hive.server2.thrift.bind.host</name>
        <value>ljc102</value>
    </property>
    <!-- 指定hiveserver2连接的端口号 -->
    <property>
        <name>hive.server2.thrift.port</name>
        <value>10000</value>
</property>
</configuration>
```

```
登录MySQL，输入密码
[ljc1@ljc102 ~]$ mysql -uroot -p
新建hive元数据库
mysql> create database metastore;
mysql> exit;
初始化hive元数据库
[ljc1@ljc102 ~]$ cd /opt/module/hive-3.1.2/bin/
[ljc1@ljc102 ~]$ schematool -initSchema -dbType mysql -verbose
启动hive
[ljc1@ljc102 ~]$ hive
退出hive
hive> exit;
```

```
3. hive服务启动脚本
进入家目录下的bin目录
[ljc1@ljc102 ~]$ cd bin/
上传资料包内hvservice.sh
修改权限
[ljc1@ljc102 bin]$ chmod 777 hvservice.sh
脚本使用方法：
hvservice.sh start或stop或restart（重启）或status（查看状态）
启动后需要等待一段时间才可以查看状态。
```

![image-20241219130213396](C:\Users\Yan\AppData\Roaming\Typora\typora-user-images\image-20241219130213396.png)

# （九）Sqoop和zookeeper安装

1. 下载安装包导致指定目录，并解压

```
[qt001@node102 ~]$ cd /opt/software/ 
[qt001@node102 ~]$ tar -zxvf 
sqoop-1.4.7.bin__hadoop-2.6.0.tar.gz -C /opt/module/ 
[qt001@node102 ~]$ mv sqoop-1.4.7.bin_hadoop-2.6.0/ sqoop-1.4.7
```

2. 配置 Sqoop 环境变量

```
[qt001@node102 ~]$ sudo vim /etc/profile.d/my_env.sh 
添加
```

```
#Sqoop_HOME 
export SQOOP_HOME=/opt/module/sqoop-1.4.7 
export 
PATH=$JAVA_HOME/bin:$HADOOP_HOME/bin:$HIVE_HOME/bin:$SQOOP_HOME/bin:$PATH
```

3. 使修改后的文件生效

```
[qt001@node102 ~]$ source /etc/profile 
修改 Sqoop 配置文件
进入 Sqoop 下面的 conf，重命名 sqoop-env-template.sh 为 sqoop-env.sh 
[qt001@node102 ~]$ cd /opt/module/sqoop-1.4.7/conf/ 
[qt001@node102 conf]$ mv sqoop-env-template.sh sqoop-env.sh 
修改 sqoop-env.sh 的内容
[qt001@node102 conf]$ vim sqoop-env.sh 
```

```
修改内容如下：
#Set path to where bin/hadoop is available 
export HADOOP_COMMON_HOME=/opt/module/hadoop-3.2.2 
#Set path to where hadoop-*-core.jar is available 
export HADOOP_MAPRED_HOME=/opt/module/hadoop-3.2.2 
#set the path to where bin/hbase is available 
#export HBASE_HOME= 
#Set the path to where bin/hive is available 
export HIVE_HOME=/opt/module/hive-3.1.2 
export ZOOKEEPER_HOME=/opt/module/zookeeper-3.5.7/ 
#Set the path for where zookeper config dir is 
export ZOOCFGDIR=/opt/module/zookeeper-3.5.7 
```

```
 将 mysql-connector-java-5.1.37.jar 拷贝到 sqoop 的 lib 目录下
[qt001@node102 conf]$ cp 
/opt/software/mysql-connector-java-5.1.37.jar 
/opt/module/sqoop-1.4.7/lib/ 
验证 Sqoop 是否配置正确
[qt001@node102 conf]$ cd /opt/module/sqoop-1.4.7/bin/ 
[qt001@node102 bin]$ sqoop help
```

```
编写Shell 脚本并保存为 quotes_sqoop_import.sh，实现将 MySQL 中
数据加载到 HDFS 上。import_data 方法执行 sqoop import 命令，将
数据库数据导人到 HDFS 指定的路径中。
```

```
#!/bin/bash 
#数据库名称
db_name=quotes 
tb_name=quote 
#导人数据
import_data(){ 
 /usr/local/software/sqoop-1.4.7/bin/sqoop import \ 
 --connect jdbc:mysql://node102:3306/$db_name \ 
 --username root \ 
 --password root \ 
 --table $tb_name \ 
 --target-dir /origin_data/$db_name/db/$tb_name \ 
 --fields-terminated-by "\t" \ 
 -m 1 \ 
 --delete-target-dir 
} 
import_data 
出 现 ”Exception in thread "main" java.lang.NoClassDefFoundError: 
org/apache/commons/lang/StringUtils”错误
下载替换版本的 commons-lang 的 jar 包
[qt001@node102 software]$ wget --no-check-certificate 
https://dlcdn.apache.org//commons/lang/binaries/commons-lang-2
.6-bin.tar.gz 
将下载的 gz 解压，然后将新版本的 jar 复制到 sqoop 的 lib 目录下
[qt001@node102 software]$ tar -zvxf commons-lang-2.6-bin.tar.gz 
-C /opt/module/ 
[qt001@node102 software]$ cp commons-lang-2.6.jar 
/opt/module/sqoop-1.4.7/lib/ 
```

# （十）zookeeper 安装

1. 上传

```
上传 apache-zookeeper-3.5.7-bin.tar.gz 至 node102 
[qt001@node102 ~]$ cd /opt/software/ 
```

2. 解压

```
[qt001@node102 software]$ tar -zxvf 
apache-zookeeper-3.5.7-bin.tar.gz -C /opt/module/ 
```

```
修改文件夹名称
[qt001@node102 ~]$ cd /opt/module/ 
[qt001@node102 module]$ mv apache-zookeeper-3.5.7-bin/ 
zookeeper-3.5.7 
修改配置
[qt001@node102 module]$ cd zookeeper-3.5.7/conf/ 
[qt001@node102 conf]$ mv zoo_sample.cfg zoo.cfg 
[qt001@node102 conf]$ vim zoo.cfg 
修改 dataDir（数据存储路径）
dataDir=/opt/module/zookeeper-3.5.7/zkData 
在末尾添加
#server.2=node102:2888:3888 
#server.3=node103:2888:3888 
#server.4=node104:2888:3888 
server.2=node102:2888:3888 
server.3=node103:2888:3888 
server.4=node104:2888:3888 
创建文件夹
[qt001@node102 conf]$ cd .. 
[qt001@node102 zookeeper-3.5.7]$ mkdir zkData 
在 zkData 下创建 myid 文件（文件名不可更换）
[qt001@node102 zookeeper-3.5.7]$ cd zkData/ 
[qt001@node102 zkData]$ vim myid 
文件中仅写入以下内容：（不可含有空行和空格，只能有数字 2）
2 
将 zookeeper 分发至 node103、node101 
[qt001@node102 zookeeper-3.5.7]$ xsync 
/opt/module/zookeeper-3.5.7/ 
xsync 同步脚本见 hadoop 集群搭建视频（本视频资料包内也有），没有的可以用以下命令：
[qt001@node102 zookeeper-3.5.7]$ scp -r 
/opt/module/zookeeper-3.5.7/ qt001@node103:/opt/module 
[qt001@node102 zookeeper-3.5.7]$ scp -r 
/opt/module/zookeeper-3.5.7/ qt001@node104:/opt/module 
修改 node103、node104 的 myid 文件内容
[qt001@node103 ~]$ vim /opt/module/zookeeper-3.5.7/zkData/myid 
将数字 2 分别改为 3、1
上传脚本
上传前需要修改文件里的主机名、目录名
[qt001@node102 zookeeper-3.5.7]$ cd ~/bin/ 
上传 zk.sh（zookeeper 启动、关`闭、查看状态）
```

