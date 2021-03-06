# Spark



# 概述

* 类似于MR,运行在yarn上,在逻辑回归的特定算法上比hadoop快100倍以上
* 由Spark Core,Spark SQL,Spark Streaming,Spark MLLib,Spark GraphX组成其生态图
* Spark Core:是其他Spark框架的核心,而Spark的核心是RDD(弹性分布式数据集)
* Spark SQL:类似Hive,Pig,使用SQL语句操作RDD DataFrame(表)
* Spark Streaming:流式计算,类似Storm的实时计算
* Spark MLLib:Spark机器学习类库
* Spark GraphX:图计算



# Spark和Hadoop

* 批处理:
  * Spark:Spark RDDS,java,Scala,Python
  * Hadoop:MapReduce,Pig,Hive,Java
* SQL查询:
  * Spark:Spark SQL
  * Hadoop:Hive
* 流式计算:
  * Spark:Spark Streaming
  * Hadoop:Storm,Kafka
* 机器学习:
  * Spark:Spark ML Lib
  * Hadoop:Mahout,该框架已经不更新
* 实时监控:
  * Spark:Spark能从NoSQL存储中查询数据
  * Hadoop:NoSQL,Hbase,Cassandra
* Hadoop:分布式计算+分布式数据存储;MR做计算;HDFS;不能迭代运行;批处理引擎;
* Spark:只有分布式计算,不存储数据;通用计算;即可以存磁盘也可以存内存;适合做迭代计算;即可以做批处理,也可以做流式处理,在磁盘上的计算比Hadoop快2-10倍,在内存中的计算快100倍;



# 特性

* 完全兼容Hadoop,能够快速的切换hadoop,hbase等
* 只有数据计算,没有数据存储,是不能取代hadoop的
* 框架多样化,可以和其他框架无缝衔接
  * 离线批处理,MapReduce,Hive,Pig
  * 实时流式处理,Storm,JStorm
  * 交互式计算:Impala



# 核心

![](cluster-overview.png)

* DataSets:
* DataFrames:
* RDD:ResilientDistributedDataset:弹性分布式数据集,是Spark的核心抽象.
  * 只读的对象集合,按照集群主机进行分区
  * 多个RDD作为input进行加载并进行一系列转变成新的RDD
  * 弹性是指Spark可以根据计算的来源方式,通过重新计算后进行的自动重构丢失的分区
  * 加载数据或执行变换并不会触发数据处理,只有生成执行计划,直到action执行时才处理



# 编译

* Spark依赖的Hadoop,Yarn等其他框架版本可能会和生产环境中不一致,故而需要重新编译

* 若忽略版本中的差异,可以直接使用官网压缩包解压即可

* 进入Spark官网后选择需要使用的版本的源码下载

* 将源码直接下载到服务器上,先安装maven,jdk,配置好环境变量

* 在官网的Documentation->More->Building Spark查看编译文档,注意Maven和JDK的版本是否符合Spark版本

* Spark有自带的Maven框架,但是仍然最好使用自己安装的Maven

* Maven的编译命令如下,[官网](http://spark.apache.org/docs/latest/building-spark.html#apache-maven)可查看:

  ```shell
  # 该处使用的是Spark自带的Maven,也可以使用自己的
  # 可以加上其他的如hive
  ./build/mvn -Pyarn -Dhadoop.version=2.8.5 -DskipTests clean package
  # 使用yarn,hive的相关命令,使用什么插件都可以在官网上查找
  ./build/mvn -Pyarn -Phive -Phive-thriftserver -DskipTests clean package
  ```

  * 因为Spark支持的版本不同,可以查看Spark源码中pom.xml中的插件版本
  * -p:pom.xml中的profiles中的profile配置,需要和pom.xml中一致
  * -D:表示指定hadoop的版本

* 编译可能出现的问题:

  * 缺少hadoop-cient.jar,这是因为仓库中没有该jar包,在pom.xml中添加如下仓库

    ```xml
    <repository>
    	<id>cloudera</id>
    	<url>https://repository.cloudera.com/artifactory/cloudera-repos</url>
    </repository>
    ```

  * 编译时的内不够,机器的内存最好是2-4G之间

    ```shell
    export MAVEN_OPTS="-Xmx2g -XX:ReservedCodeCacheSize=1g"
    ```

  * 在编译Scala之前,需要先从官网上查看最低版本的Scala需要的版本,并且先执行如下:

    ```shell
    # Scala根据Spark官网选择
    ./dev/change-scala-version.sh 2.13
    ./build/mvn -Pscala-2.13 compile
    ```

    

# 安装

详见官网[文档](http://spark.apache.org/docs/latest/#),参数[文档](http://spark.apache.org/docs/latest/submitting-applications.html)



## 本地模式

* 编译完成直接进入sbin中,不用修改任何配置,运行相关命令即可

  ```shell
  # master指定运行模式:local表示运行模式为本地,2表示多线程的线程数
  spark-shell --master local[2]
  ```
  
* 本地模式连接到集群时,若集群中的节点中有同名文件,本地模式加载该文件内容就会随机显示



## StandAlone

* 官网[文档](http://spark.apache.org/docs/latest/spark-standalone.html)

* 当不配置任何works时,可以直接启动sbin/start-master.sh

* 配置文件在conf/spark-env.sh.template,改名为spark-env.sh

  ```shell
  export JAVA_HOME=/app/jdk1.8
  export SPARK_MASTER_HOST=localhost # 主机名
  # SPARK_WORKER_CORES=2 # works核心工作数量
  # SPARK_WORKER_MEMORY=2g # 分配给work的总内存
  ```

* 当含有work时,可以使用sbin/start-all.sh启动spark

* web页面可以从启动日志中查看masterui,能够打开网址即部署成功,默认端口是8080



## 集群模式

* 该模式下需要在master节点的conf/slaves中配置所有的work节点的主机名,只配置work的,不配置master
* 启动:sbin/start-all.sh或spark-shell --master spark://master01:7077,master02:7077,该命令会启动master和所有的work
* 若多次启动了master,则后启动的master在默认情况下是分配不到内存的
* 若需要所有的master都可以运行,需要进行相关配置,详见官网文档



# HA



## 基于文件目录

* 用于开发测试环境的单机环境

* 将worker和application状态写入一个目录,如果出现崩溃,从该目录进行恢复

* 在master上面进行配置如下:

  * 创建一个恢复目录:spark/recovery
  * 修改配置文件:spark-env.sh

  ```shell
  export SPARK_DAEMON_JAVA_OPTS="-Dspark.deploy.recoveryMode=FILESYSTEM -Dspark.deploy.recoveryDirectory=/app/spark/recovery"
  ```



## 基于ZooKeeper

* 用于生产环境

* 搭建zk集群,master节点为master01,master02,worker节点为worker01,worker02

* 修改conf/spark-env.sh

  ```shell
  export JAVA_HOME=/opt/modules/jdk1.8.0_11
  # spark.deploy.recoveryMode:设置为zk开启单点恢复功能,默认值为none
  # spark.deploy.zookeeper.url:zk集群地址
  # spark.deploy.zookeeper.dir:spark信息在zk中保存的目录
  export SPARK_DAEMON_JAVA_OPTS="-Dspark.deploy.recoveryMode=ZOOKEEPER -Dspark.deploy.zookeeper.url=master01:2181,master02:2181,master03:2181 -Dspark.deploy.zookeeper.dir=/spark"
  # SPARK_MASTER_PORT=7077
  # SPARK_MASTER_WEBUI_PORT=8088
  ```

* 修改conf/slaves

  ```shell
  worker01
  worker02
  ```

* 启动集群:sbin/start-all.sh

* 需要手动启动master02:sbin/start-master.sh

* spark-shell --master spark://hadoop01:7077:连接到集群



# SparkShell

* bin/spark-shell:进入spark的控制台,可以使用scala操作spark.scala语法见scala文档
* sc:进入spark控制台就存在的上下文环境,可以直接使用,可以自动补全
* sc.textFile("path"):加载linux中的文件,需要赋值给另外的变量
* 



# SparkSQL

官网[文档](http://spark.apache.org/docs/latest/sql-programming-guide.html)



* 在Spark中运行Scala项目,将打包好的Scala的jar包放到指定目录中,如/app/spark/jars

* spark-submit --class jar包中main方法的路径 --master local[2] hadoop的jar包路径 需要被解析的文件路径

  ```shell
  spark-submit --class com.wy.SQLContextApp --master local[2] /app/hadoop/lib/sql-1.0.jar /app/spark/data.json
  ```

* 在Spark中运行Hive源代码,需要额外添加数据库的jar包路径

  ```shell
  spark-submit --class com.wy.HiveContextApp --master local[2] --jars /app/common/jars/mysql-connector-java-5.1.7-bin.jar /app/hadoop/lib/sql-1.0.jar
  ```

  

# 用户行为日志



## 概述

* 记录用户访问行为日志可以得到网站页面的访问量,网站的粘性以及更好的推荐
* 用户行为日志生成渠道:Nginx和Ajax
* 用户日志数据包括:访问者的操作系统,浏览器,点击的url,url跳转(referer),页面上的停留时间,session_id,访问ip



## 数据处理流程

* 数据采集:Flume,web(nginx,tomcat)日志写入到HDFS中
* 数据清洗:脏数据,Spark,Hive,MR或者其他框架,清洗完之后的数据存放在HDFS中
* 数据处理:按照业务逻辑需求进行相应业务统计和分析,Spark,Hive,MR等
* 处理结果入库:结果可以存放到RDBMS,NoSQL
* 数据的可视化:通过图形化展示的方式展现出来,ECharts等