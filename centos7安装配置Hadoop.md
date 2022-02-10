### 一、JDK安装配置
#### 1、检查是否有jdk
> `yum list installed |grep java`

#### 2、卸载自带Java
> `yum -y remove java-1.8.0-openjdk* `
> `yum -y remove tzdata-java*`

#### 3、配置Java环境变量
进入 /etc/profile编辑，添加
```shell
export JAVA_HOME=/usr/lib/jvm/java-1.8.0
export JRE_HOME=$JAVA_HOME/jre  
export PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
```
#### 4、检查是否安装成功
> 执行
> source /etc/profile
> 检查
> java -version

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25797561/1644325552430-b63e1aa7-1af2-4aa4-90a3-7161081871a9.png#clientId=ud27d9a25-6997-4&from=paste&id=u3322c99a&margin=%5Bobject%20Object%5D&name=image.png&originHeight=87&originWidth=625&originalType=binary&ratio=1&size=10060&status=done&style=none&taskId=u6345a77c-b542-4f0e-a235-0ee908b32c7)
### 二、机器参数配置
#### 1、修改机器名：
> vim  /etc/sysconfig/network

```shell
NETWORKING=yes
HOSTNAME=master
```
#### 2、设置ip和hostname的映射关系: 
> vim  /etc/hosts

```shell
192.168.78.128 master
127.0.0.1 localhost
```
#### 3、ssh免密码登陆
```shell
ssh-keygen -t rsa
cp ~/.ssh/id_rsa.pub ~/.ssh/authorized_keys
```
> 使用ssh master进行IP测试
> 使用ssh localhost进行本地测试

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25797561/1644325428873-9ec3d32c-6456-442c-8687-f75d5b730eb3.png#clientId=ud27d9a25-6997-4&from=paste&id=u8f83e59c&margin=%5Bobject%20Object%5D&name=image.png&originHeight=139&originWidth=713&originalType=binary&ratio=1&size=20596&status=done&style=none&taskId=ue3db07e3-b17a-4074-91c9-546a9530d44)![image.png](https://cdn.nlark.com/yuque/0/2022/png/25797561/1644325452700-ab595017-5e98-42bb-9262-8d5512f9257c.png#clientId=ud27d9a25-6997-4&from=paste&id=ub724737c&margin=%5Bobject%20Object%5D&name=image.png&originHeight=133&originWidth=783&originalType=binary&ratio=1&size=22499&status=done&style=none&taskId=ua631df05-288f-405f-a546-e59f7b5dcde)
### 三、Hadoop配置文件修改(hadoop/etc/hadoop)
#### 1、修改hadoop-env.sh
> 将原有的
> export JAVA_HOME=${JAVA_HOME}
> 修改为(Java的目录)
> export JAVA_HOME=/software/java

#### 2、修改core-site.xml文件
```shell
  <property>
        	<name>fs.defaultFS</name>
        	<value>hdfs://master:8020</value>
   </property>	
   <property>
          <name>hadoop.tmp.dir</name>
					//临时目录
        	<value>/software/tmp</value>
    	</property>	
```
#### 3、修改hdfs-site.xml
```shell
//副本数
<property>
	<name>dfs.replication</name>
	<value>1</value>
</property>
```
#### 4、添加配置文件至/etc/profile
> export HADOOP_HOME=/software/hadoop
> export PATH=$PATH:$HADOOP_HOME/bin

#### 5、格式化HDFS
> 注意：这一步操作，只是在第一次时执行，每次如果都格式化的话，那么HDFS上的数据就会被清空
> 进入Hadoop的bin目录执行：
> ./hdfs namenode -format
> 红框内表示临时目录成功创建

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25797561/1644329152985-168a789e-e170-4ed1-bd9f-21cf969c1278.png#clientId=ud27d9a25-6997-4&from=paste&id=u5b9af6cf&margin=%5Bobject%20Object%5D&name=image.png&originHeight=225&originWidth=967&originalType=binary&ratio=1&size=35585&status=done&style=none&taskId=u4e5ae4ab-a8ef-436b-817d-37382844253)
#### 6、启动HDFS
> 进入sbin目录执行：
> ./start-dfs.sh
> master: NameNode
> localhost: DataNode（datanode启动在hadoop/etc/hadoop中的从节点slaves），slaves中为localhost。

验证是否成功启动：

- JPS（出现DataNode、SecondaryNameNode、NameNode）
- [http://master:50070/](http://hadoop001:50070/)（成功打开）
#### ![image.png](https://cdn.nlark.com/yuque/0/2022/png/25797561/1644329373023-392e5548-5bcb-42d9-986b-577502bb89ac.png#clientId=ud27d9a25-6997-4&from=paste&id=uae0ad876&margin=%5Bobject%20Object%5D&name=image.png&originHeight=232&originWidth=977&originalType=binary&ratio=1&size=26174&status=done&style=none&taskId=u011afb4d-dfef-4e83-93e6-b2f1ff15b38)
#### 7、关闭HDFS
> ./stop-dfs.sh

### 四、YARN环境搭建
#### 1、修改配置文件mapred-site.xml
> 执行命令复制模板
> cp mapred-site.xml.template mapred-site.xml
> vim mapred-site.xml添加如下代码

```shell
<property>
  <name>mapreduce.framework.name</name>
  <value>yarn</value>
</property>
```
#### 2、修改配置文件yarn-site.xml
> 执行命令：
> vim yarn-site.xml
> 添加如下代码

```shell
<property>
  <name>yarn.nodemanager.aux-services</name>
  <value>mapreduce_shuffle</value>
</property>
```
#### 3、启动yarn，验证是否启动成功
> 进入sbin目录，启动：
> ./start-yarn.sh

验证方式：

- jps（出现ResourceManager、NodeManager）
- web: [http://master:8088](http://hadoop001:8088)

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25797561/1644414914828-b8be912c-0132-42e1-8969-aed4f4e62337.png#clientId=u3dac92f0-1fb0-4&from=paste&id=u54b2a42c&margin=%5Bobject%20Object%5D&name=image.png&originHeight=154&originWidth=906&originalType=binary&ratio=1&size=17463&status=done&style=none&taskId=uccfc71f9-25e7-406e-9009-ca2a519bff1)
#### 4、停止yarn
> ./stop-yarn.sh

#### 5、提交作业至yarn上运行
```shell
hadoop jar #启动命令
/software/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.3.jar #作业jar包位置 
wordcount  #参数，作业名称
/input/wc/hello.txt #hdfs源文件
/output/wc/  #作业在hdfs上的输出目录
```
> hdfs中已存在的目录不能作为输出目录，否则会报错。

### 五、安装MySQL
#### 1、MySQL安装命令
```shell
yum install mysql
yum install mysql-server
yum install mysql-devel
```
> yum install mysql-server安装失败原因：
> CentOS 7 版本将MySQL数据库软件从默认的程序列表中移除，用**mariadb**代替了。
> 解决办法：

```shell
wget http://dev.mysql.com/get/mysql-community-release-el7-5.noarch.rpm
rpm -ivh mysql-community-release-el7-5.noarch.rpm
yum install mysql-community-server
```
> 然后重启MySQL：
> service mysqld restart

#### 2、设置MySQL密码
> 执行命令：
> mysql -u root 		#首次执行没有密码
> 进入MySQL后设置密码：
> set password for 'root'@'localhost' =password('123456'); 	#'123456'为密码

#### 3、MySQL环境配置
> 进入mysql配置文件/etc/my.cnf
> 添加如下编码配置：

```shell
[mysql]
default-character-set =utf8
```
#### 4、设置远程连接
> 把所有数据库的所有表的所有权限赋值给位于所有IP地址的root用户，
> 然后刷新。

```shell
mysql> grant all privileges on *.* to root@'%'identified by '123456';
mysql>flush privileges;
```
### 六、安装配置hive
#### 1、在配置文件/etc/profile中添加如下编码配置：
```shell
#!hive
export HIVE_HOME=/software/hive
export PATH=$PATH:$HIVE_HOME/bin:$PATH
```
> source /etc/profile执行生效

#### 2、配置文件hive-env.sh
> 创建一个文件，执行代码：
> cp hive-env.sh.template hive-env.sh
> 加入Hadoop路径：
> HADOOP_HOME=/software/hadoop

#### 3、配置文件hive-site.xml
> 没有的话自己创建一个：
> touch hive-site.xml
> 加入如下编码配置：

```shell
<configuration>
#其中sparksql为mysql数据库，createDatabaseIfNotExist为true会自动创建，false则需要自己手动创建。
    <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:mysql://localhost:3306/sparksql?createDatabaseIfNotExist=true</value>
    </property>

    <property>
        <name>javax.jdo.option.ConnectionDriverName</name>
        <value>com.mysql.jdbc.Driver</value>
    </property>

#MySQL账户
    <property>
        <name>javax.jdo.option.ConnectionUserName</name>
        <value>root</value>
    </property>
#MySQL账户密码
    <property>
        <name>javax.jdo.option.ConnectionPassword</name>
        <value>123456</value>
    </property>
</configuration>
```
> 注意，要添加mysql驱动到/hive/lib目录下，否则会报错
> jar包名称：mysql-connector-java-5.1.47-bin.jar

#### 4、启动hive
> 1. 初始化，执行：
> 
./schematool -initSchema -dbType mysql
> 2. 打开Hadoop集群
> 2. 进入hive/bin目录执行代码：
> 
./hive

