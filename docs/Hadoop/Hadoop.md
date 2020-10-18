# Hadoop



# 概述

* 主要由HDFS,MR,YARN,COMMON组成,1.0中Yarn的功能由MR实现,2.0中独立出来
* HDFS:Hadoop Distributed File System,高可靠,高吞吐量的分布式文件系统
* MR:MapReduce,分布式离线并行计算框架
* YARN:作业调度与集群资源管理的框架
* COMMON:支持其他模块的工具模块,如RPC,序列化,日志等
* 高可靠性:维护多个工作副本,集群部署,在出现故障时可以对失败的节点重新分布处理
* 高扩展性:在集群间分配任务数据,可方便的扩展数以千计的节点
* 高效性:在MR的思想下,hadoop是并行工作的
* 高容错性:自动保存多份副本数据,并且能够自动将失败的任务重新分配



# 应用场景

* 物流仓储:大数据分析系统助力商家精细化运营,提升销量,节约成本
* 零售:分析用户消费习惯,为用户购买商品提供方便,从而提升商品销量
* 旅游:深度结合大数据能力与旅游行业需求,共建旅游产业智慧管理,智慧服务和智慧营销的未来
* 商品广告推荐:给用户推荐可能喜欢的商品
* 保险:海量数据挖掘以及风险预测,助力保险行业精准营销,提升精细化定价能力
* 金融:多维度体现用户特征,帮助金融机构推荐优质客户,防范欺诈风险
* 房产:大数据全面助力房地产行业,打造精准投策和营销,选出合适的地,建合适的楼,卖合适的人
* 人工智能:大数据+算法



# MapReduce

* 是一个分布式运算程序的编程框架,核心功能是将用户编写的业务逻辑代码和自带默认组件整合成一个完整的分布式运算程序,并发运行在一个hadoop集群上
* MR易于编程,简单的实现一些接口就可以完成一个分布式程序
* 良好的扩展性
* 高容错性,即使集群中的一台机器挂掉了,它可以把上面的计算任务转移到另外一个节点上运行
* 适合PB级别以上海量数据的离线处理
* MR不适合做实时计算,流式计算,DAG(有向图)计算
  * 实时计算:因为MR是读取磁盘上的文件,做处理比较慢
  * 流式计算:流式计算的输入数据是动态的,而MR的输入数据是静态的,这取决于MR的设计特点
  * DAG:多个应用程序存在依赖关系,后一个应用程序的输入为前一个的输出.在这种情况下,MR作业的结果会写入到磁盘中,会产生大量的磁盘IO,导致性能很低
* MR的工作流程:
  * 将文件中的内容按行读取,之后按照空格进行切分
  * 再开辟空间进行分区排序,排序按字典排序,将结果放在一个类似map的集合中
  * 排序之后再将相同的项进行合并



# 生态

* HDFS:解决存储问题
* MapReduce:解决计算问题
* Yarn:解决资源调度问题
* Zookeeper:分布式应用协调服务
* Flume:日志收集
* Hive:基于Hadoop的数仓工具,离线计算
* HBase:分布式,面向列的开源数据源,近实时数据查询
* Sqoop:数据传递工具
* Scala:多范式编程语言,面向对象和函数式编程的特性
* Spark:目前企业常用的批处理离线/实时计算引擎
* Flink:目前最火的流处理框架,既支持流处理,也支持批处理
* Elasticsearch:大数据分布式实时弹性搜索引擎



# 核心

* HDFS是一个主从(Master/Slave)结构框架,由一个NameNode和多个DataNode构成
* 存储模型:
  * 文件线性按字节切割成块(block),具有offset,id
  * 文件与文件的block大小可以不一样
  * 一个文件除最后一个block,其他block大小一致
  * block的大小依据硬件的IO特性调整,如果性能好就可以加大该值
  * block被分散存放在集群的节点汇总,具有location
  * block具有副本(replication),没有主从概念,副本不能出现在同一个节点,是满足可靠性和性能的关键
  * 文件上传可以指定block大小和副本数,上传后只能修改副本数
  * 一次写入多次读取,不支持修改,因为修改会改变block的大小,进而改变offset,这需要消耗大量的计算机资源
  * 支持追加数据,只能在block末尾追加
  * hdfs删除时只能删除文件,不能删除block,和修改是同样的原因
* Block的放置策略:
  * 第一个副本:放置在上传文件的DN上.如果是集群外提交,则随机挑选一台磁盘不太满,cpu不太忙的节点
  * 第二个副本:放置在于第一个副本不同的机架节点上
  * 第三个副本:与第二个副本相同机架的节点
  * 更多副本:随机节点
* NameNode(NN):文件元数据节点
  * 存储文件元数据,包括文件名,目录结构,属性,以及每个文件的block列表和block所在的datanode
  * 完全基于内存运行
  * 需要持久化方案保证数据可靠性
  * 提供副本放置策略
* 元数据持久化
  * 任何对文件爱你系统元数据产生修改的操作,NameNode都会使用一种称为EditLog的事务日志记录下来
  * 使用FsImage存储内存所有的元数据状态
  * 使用本地磁盘保存EditLog和FsImage
  * EditLog具有完整性,数据丢失少,但恢复速度慢,并有体积膨胀风险
  * FsImage具有恢复速度快,体积与内存数据相当,但不能实时保存,数据丢失多
  * NN使用了FsImage+EdieLog整合的方案:滚动将增量的EditLog更新到FsImage,保证更近实时的FsImage和更小的EditLog体积
* 安全模式
  * HDFS搭建时会先格式化,此时会产生一个空的FsImage
  * 当NN启动时,它从硬盘中读取EditLog和FsImage
  * 将所有EditLog中是事务作用在内存中的FsImage中,并将新的FsImage从内存中保存到本地磁盘
  * 删除旧的EditLog,因为这个旧的EditLog已经作用在FsImage上了
  * NN启动后会进入安全模式状态,该状态下不进行数据块的复制,而是接收DN的心跳和块状态报告
  * 每当NN检测确认某个数据块的副本数目达到必须的最小值,则该数据块就会被认为是副本安全的
  * 在一定百分比(可配置)的数据块被NN检测确认是安全后(再加一个额外的30S等待),NN将退出安全模式状态
  * 接下来NN会确定还有哪些数据块的副本没有达到指定数目,并将这些数据库块制到其他DN上
* DataNode(DN):数据节点
  * 在本地文件系统存储文件块数据(block),以及提供块数据的校验,读写
  * DataNode和NameNode维持心跳,并汇报自己持有的block信息
* 客户端和NameNode交互文件的元数据,和DataNode交互文件爱你block信息
* Secondary NameNode(SNN):监控hdfs状态的辅助后台程序,每隔一段时间获得hdfs元数据的快照
  * 在非HA模式下,SNN一般是独立的节点,周期完成对NN的EditLog向FsImage合并,减少EditLog大小
  * 根据配置文件爱你设置的时间间隔:fs.checkpoint.period,默认3600S
  * 根据配置文件设置edits log大小:fs.checkpoint.size,规定edits文件的最大值默认是64M
* ResourceManager(rm):资源管理,一个集群只有一个RM是活动状态
  * 处理客户端请求
  * 监控NodeManager
  * 启动和监控ApplicationMaster
  * 资源分配与调度
* NodeManager(nm):节点管理,集群中有N个,负责单个节点的资源管理和使用以及task运行状况
  * 单个节点上的资源管理和任务管理
  * 处理来自ResourceManager和ApplicationMaster的命令
  * 定期向RM汇报本节点的资源使用请求和各个Container的运行状态
* ApplicationMaster:每个应用/作业对应一个,并分配给内部任务
  * 数据切分
  * 为应用程序向RM申请资源(Container),并分配给内部任务
  * 与NM通信以启动或停止task,task是运行在Container中的
  * task任务监控和容错
* Container:对任务运行环境的抽象,封装了CPU,内存等多维资源以及环境变量,启动命令等任务信息
* YARN执行流程
  * 用户向YRAN提交作业
  * RM为该作业分配第一个Container(AM)
  * RM会与对应的NM通信,要求NM在这个Container上启动应用程序的AM
  * AM首先向RM注册,然后AM将为各个任务申请资源,并监控运行情况
  * AM采用轮询的方式通过RPC协议向RM申请和领取资源
  * AM申请到资源后,便和对应的NM通信,要求NM启动任务
  * NM启动作业对应的task
* HDFS写流程
  * Client和NN连接创建文件元数据,连接后NN判定元数据是否有效
  * NN触发副本放置策略,返回一个有序的DN列表
  * Client和DN建立Pipeline连接
  * Client将块切分成packet(64KB),并使用chunk(512B)+chucksum(4B)填充
  * Client将packet放入发送列队dataqueue中,并向第一个DN发送
  * 第一个DN接收packet后本地保存并发给第二个DN,第二个DN接收packet后本地保存并发给第三个DN
  * 这个过程中,上游节点同时发送下一个packet,HDFS使用这种传输方式,副本数对于Client是透明的
  * 当block传输完成,DN各自向NN汇报,同时Client继续传输下一个block
  * 所以,Client的传输和block的汇报也是并行的
* HDFS读流程
  * 为降低整体带宽消耗和读取延迟,HDFS会进来让读取程序取离它最近的副本
  * 如果在读取程序的同一个机架上有一个副本,那么就读取该副本
  * 如果一个HDFS集群跨越多个数据中心,那么客户端也将首先读本地数据中心的副本
  * 如下载一个文件
    * Client和NN交互文件元数据获取fileBlockLocation
    * NN会按距离策略排序返回
    * Client尝试下载block并校验数据完整性



# API

* hadoop checknative -a:检查hadoop本地库是否正常,false不正常
* hadoop fs -put file/folder /file:上传linux里的file或folder到hadoop的/file,file可自定义;若是/file存在,则删除hadoop fs -rm -r /file,提示deleted file才表示删除成功
* hadoop fs -rm -r /file:删除hadoop集群中的文件或目录
* hadoop fs -ls /file:查看上传的文件是否成功,成功会列出文件地址,否则报错文件不存在
* hadoop jar XXX.jar xx.xx.xxx.TestMain /input /output:运行jar包,需要指定main所在类,/input表示上传文件所在地址,/output表示文件输出地址,且该地址不能是已经存在的
* hadoop fs -ls /output:查看运行生成的文件,若有success文件代表成功,可以从50070的utilities的browse the file system下面查看,可将最后的结果下载下来查看
* hadoop fs -cat /file:查看hadoop中某个文件的内容



# 配置

* 设置集群block的备份数,hdfs-site.xml文件中dfs.replication的value值改成想要的值,但是要重启hadoop
* 设置集群block的备份数,命令bin/hadoop fs -setrep -R 3 /;不需要重启



# 伪分布式

1. 环境为linux,centos7.6

2. 下载jdk1.8.0.tar.gz,hadoop2.9.1.tar.gz,在linux根目录新建目录app,app下新建目录java,hadoop

4. 解压jdk和hadoop到各自文件夹中,tar -zxvf jdk1.8.0.tar.gz

5. 解压完成之后配置环境变量,编辑 vi /etc/profile,在文件最底下添加

   ```shell
   JAVA_HOME=/app/java/java1.8
   CLASSPATH=$JAVA_HOME/lib/
   PATH=$PATH:$JAVA_HOME/bin
   export PATH JAVA_HOME CLASSPATH
   export HADOOP_HOME=/app/hadoop/hadoop-2.9.1
   export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
   ```

6. 添加完之后命令source /etc/profile,输入java -version,出现版本表示安装成功,输入hadoop出现版本信息安装成功

6. 真集群模式下,要想让其他机器能访问hadoop启动后的页面需要先关闭防火墙

   ```shell
   systemctl stop firewalld.service #停止firewall
   systemctl disable firewalld.service #禁止firewall开机启动
   firewall-cmd --state #查看默认防火墙状态（关闭后显示notrunning，开启后显示running）
   vi /etc/selinux/config # 将SELINUX=enforcing改为SELINUX=disabled
   ```

7. 修改自己的ip地址为静态地址,修改主机名

8. 修改/etc/hosts,配置集群其他主机的ip以及主机名,在伪分布式模式下可不配,真集群需要配置

9. 修改hadoop配置文件,所需配置文件都在/app/hadoop/hadoop-2.9.1/etc/hadoop文件夹下

10. 修改core-site.xml,在configuration标签中添加:

   ```xml
<!-- 指定namenode地址,name为名称,可自定义,value为当前服务器地址或主机名,9000默认端口-->
<property>
    <name>fs.defaultFS</name>
    <value>hdfs://192.168.1.146:9000/</value>
</property>
<!-- 指定hadoop运行时产生文件的存储目录 -->
<property>
    <name>hadoop.tmp.dir</name>
    <value>/app/hadoop/data</value>
</property>
   ```

11. 修改hdfs-site.xml,该文件是namenode和datanode的存放地址,在configuration标签添加:

    ```xml
    <!-- 指定hdfs的副本数量,即备份 -->
    <property>
    	<name>dfs.replication</name>
    	<value>1</value>
    </property>
    <!-- 指定namenode的存储路径 -->
    <property>
    	<name>dfs.name.dir</name>
    	<value>/app/hadoop/hadoop-2.9.1/namenode</value>
    </property>
    <!-- 指定secondary namenode的地址 -->
    <property>
    	<name>dfs.namenode.secondary.http.address</name>
    	<value>http://192.168.1.146:50090</value>
    </property>
    <!-- 指定datanode的存储路径 -->
    <property>
    	<name>dfs.data.dir</name>
    	<value>/app/hadoop/hadoop-2.9.1/datanode</value>
    </property>
    <!-- 关闭权限 -->
    <property>
    	<name>dfs.permissions</name>
    	<value>false</value>
    </property>
    ```

12. 修改mapred-site.xml.template,将该文件改名为mapred-site.xml(mv mapred-site.xml.template mapred-site.xml),在configuration下添加:

    ```xml
    <!-- 指定mapreduce运行在yarn下 -->
    <property>
    	<name>mapreduce.framework.name</name>
    	<value>yarn</value>
    </property>
    <!-- 历史服务器的地址 -->
    <property>
    	<name>mapreduce.jobhistory.address</name>
    	<value>192.168.1.146:10020</value>
    </property>
    <!-- 历史服务器页面的地址 -->
    <property>
    	<name>mapreduce.jobhistory.webapp.address>
    	<value>192.168.1.146:19888</value>
    </property>
    ```

13. 修改yarn-site.xml,在configuration下添加:

    ```xml
    <!-- 指定yarn的老大(resourceManager)的地址 -->
    <property>
    	<name>yarn.resourcemanager.hostname</name>
        <!-- ip地址或主机名,主机名需要在hosts文件中已经配置过 -->
    	<value>192.168.1.146</value>
    </property>
    <!-- reducer获取数据的方式 -->
    <property>
    	<name>yarn.nodemanager.aux-services</name>
    	<value>mapreduce_shuffle</value>
    </property>
    <!-- 日志聚集功能 -->
    <property>
    	<name>yarn.log-aggregation-enable</name>
    	<value>true</value>
    </property>
    <!-- 日志保留时间,单位秒 -->
    <property>
    	<name>yarn.log-aggregation.retain-seconds</name>
    	<value>604800</value>
    </property>
    ```

14. 修改hadoop-env.sh,yarn-env.sh,mapred-env.sh,修改或添加jdk的路径

    ```shell
    #export JAVA_HOME=${JAVA_HOME}
    export JAVA_HOME=/app/java/jdk1.8
    ```

15. 修改slaves文件,加入自己的ip地址或主机名,若是真正的集群模式,需要写其他节点的ip或主机名,每一台主机都要写相同的内容

16. 免密钥登录,必须配置

    1. 进入到/home文件夹,输入ssh-keygen -t rsa,连着回车确认
    2. 完成后会生成会生成两个文件id_rsa(私钥),id_rsa.pub(公钥)
    3. 将公钥复制到要免登录的机器上scp id_rsa.pub 192.168.1.111:/home
    4. 将公钥复制到密钥列表cat ~/id_rsa.pub >> ./authorized_keys
    5. 若没有authorized_keys文件,则自己新建touch authorized_keys,并改权限为600
    6. 验证是否成功:ssh localhost,首页登录需要密码确认,yes即可

17. 首次启动hadoop

    1. hdfs namenode -format
    2. 若是有错误或启动失败,需要先进入namenode和datanode目录,删除里面的current目录,否则会出现namespaceid不一致的问题.若不成功,可以直接删除data目录和log目录,之后再format

18. 启动/停止:start-dfs.sh/stop-dfs.sh,jps显示DataNode,NameNode,SecondaryNameNode则正常

    1. 若出现有些程序已经启动,则先要kill -9 进程号,结束这些进程
    2. hadoop-daemon.sh start/stop namenode/datanode/secondarynamenode:单独启动某一个模块

19. 启动/停止:start-yarn.sh/stop-yarn.sh,jps显示NodeManager和ResouceManager则正常

    1. 伪分布式模式下,都在一台服务器,需要启动
    2. 真正集群模式,yarn配置在那台机器上,就在那台机器上启动,其他机器启动会报错
    3. yarn-daemon.sh start/stop resourcemanager/nodemanager:单独启动一个模块

20. 访问192.168.1.146:8088,192.168.1.146:50070,192.168.1.146:19888,若能出现网站表示成功

21. 启动历史服务器:sbin/mr-jobhistory-daemon.sh start historyserver

22. 其他命令:

    1. mr-jobhistory-daemon.sh start|stop historyserver:启动/停止历史服务器
    2. yarn-daemon.sh start|stop resourcemanager:启动/停止总资源管理器
    3. yarn-daemon.sh start|stop nodemanager:启动/停止节点管理器