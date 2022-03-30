### 一、MAVEN配置安装
> 下载的spark为source版本，需要进行build，此过程需要用到maven。

#### 1、maven安装
> 从官网下载maven-3.3.9即以上版本，否则spark不兼容（Java7+），解压后在`/etc/profile`中添加如下编码配置；并执行：
> `source /etc/profile`

```shell
export MAVEN_HOME=/software/maven
export PATH=$PATH:$MAVEN_HOME/bin:$PATH
```
### 二、spark源码编译
#### 1、从官网下载spark
> 下载链接：[https://spark.apache.org/downloads.html](https://spark.apache.org/downloads.html)
> 解压到指定文件夹后，在`/etc/profile`中添加配置编码，并执行：`source /etc/profile `使其生效。

```shell
#!spark
export SPARK_HOME=/software/spark-2.4.0-bin-2.7.0
export PATH=$PATH:$SPARK_HOME/bin:$PATH
```
![image.png](https://cdn.nlark.com/yuque/0/2022/png/25797561/1644742391035-f0e90b33-4b95-4ce8-9977-64666a674bc7.png#clientId=u5717e03a-e9ef-4&from=paste&id=ucba3661d&margin=%5Bobject%20Object%5D&name=image.png&originHeight=219&originWidth=1011&originalType=binary&ratio=1&size=34605&status=done&style=none&taskId=u637cbbba-12b9-4440-8702-448428af374)
#### 2、前置要求
> 1. maven版本和Java版本要求：Building Spark using Maven requires Maven 3.3.9 or newer and Java 7+。
> 1. 执行：`export MAVEN_OPTS="-Xmx2g -XX:ReservedCodeCacheSize=512m`

#### 3、mvn编译方法
> 编译命令：（在spark目录下）
> `./build/mvn -Pyarn -Phadoop-2.4 -Dhadoop.version=2.4.0 -DskipTests clean package`
> 黄色部分为版本号，修改成相应版本即可。

```shell
example：
./build/mvn -Pyarn -Phadoop-2.7 -Phive -Phive-thriftserver -Dhadoop.version=2.7.0 -DskipTests clean package
```
#### 4、生成部署包（推荐使用）
> 在源码包下找到脚本`make-distribution.sh`执行(spark目录下),时间约为半小时左右.
> 最终生成和官网下载编译好的包一样，可以直接部署。

```shell
./dev/make-distribution.sh --name 2.7.0 --tgz  -Pyarn -Phadoop-2.7 -Phive -Phive-thriftserver -Dhadoop.version=2.7.0
```
#### 5、spark源码编译的坑

1. maven环境内存要符合条件。如果用maven进行编译需要先设置 maven内存：
> `export MAVEN_OPTS="-Xmx2g -XX:ReservedCodeCacheSize=512m`

2. 如果scala 版本为2.10 ,需要先执行：` ./dev/ change-scala-version.sh 2.10`。
### 
 三、spake操作
#### 1、spark local模式搭建
> 解压上面所生成的压缩包，并进入该目录。
> 进入spark目录中，执行：
> `./bin/spark-shell --master local[2]`
> 其中2表示进程数目。

#### ![image.png](https://cdn.nlark.com/yuque/0/2022/png/25797561/1644766127413-40aa4eca-26cb-4702-bec0-0848468ea192.png#clientId=ub363a3e4-62d9-4&from=paste&id=u33b25ea4&margin=%5Bobject%20Object%5D&name=image.png&originHeight=415&originWidth=962&originalType=binary&ratio=1&size=43664&status=done&style=none&taskId=uc7776f0e-ecdc-4f80-ba04-6e0c2528c6e)2、spark standalone集群部署模式搭建
> 1.配置文件`/conf/spark-env.sh` 
> 打开文件后，添加如下编码配置

```shell
SPARK_MASTER_HOST=master	主机名
SPARK_WORKER_CORES=2			spark应用程序可以使用的core的个数
SPARK_WORKER_MEMORY=2g		spark application可以使用的内存
SPARK_WORKER_INSTANCES=1	启动实例个数
```
> 2.进入sbin目录，执行代码：
> `./start-all.sh`
> PS：注意，如果启动不成功，报错：`JAVA_HOME is not set`，则需要修改`spark-config.sh`，添加：
> `export JAVA_HOME=/software/java`。
> 启动成功后会显示一个端口，打开master:端口，则可以通过浏览器查看具体细节。
> 端口号可以再日志中查看。

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25797561/1644916535003-7607b533-fb67-409c-b556-493ca0e1c99a.png#clientId=ub66d6c76-764f-4&from=paste&id=ua6de30d3&margin=%5Bobject%20Object%5D&name=image.png&originHeight=125&originWidth=783&originalType=binary&ratio=1&size=26180&status=done&style=none&taskId=ud4420a1d-3af6-47d5-83eb-1521f786665)![image.png](https://cdn.nlark.com/yuque/0/2022/png/25797561/1644916616774-8a96cee9-0883-499d-ba8a-076ddc2b8374.png#clientId=ub66d6c76-764f-4&from=paste&id=ub5e944a3&margin=%5Bobject%20Object%5D&name=image.png&originHeight=347&originWidth=770&originalType=binary&ratio=1&size=48028&status=done&style=none&taskId=u4178ddcb-c149-4e18-a79a-c53db2fb2ce)
