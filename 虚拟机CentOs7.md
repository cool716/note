



<center style="font-size:50px;color:red">虚拟机CentOs7</center>

# 一，准备工作：

下载安装VM-15

![image-20210106133043666](images\image-20210106133043666.png)

下载镜像：![image-20210106132948634](images\image-20210106132948634.png)

下载zookeeper

![image-20210106133306696](images\image-20210106133306696.png)

下载jdk

![image-20210106133357793](images\image-20210106133357793.png)

# 二，安装VM

一路默认即可

# 三，安装虚拟机

![image-20210106133710864](images\image-20210106133710864.png)

![image-20210106133725484](images\image-20210106133725484.png)

选择下载好的centos7镜像

![image-20210106133804804](images\image-20210106133804804.png)

选择本地安装路径

<img src="images\image-20210106133931453.png" alt="image-20210106133931453" style="zoom:80%;" />

默认下一步

<img src="images\image-20210106134409216.png" alt="image-20210106134409216" style="zoom:80%;" />

<img src="images\image-20210106134459767.png" alt="image-20210106134459767" style="zoom:80%;" />

默认设置密码为root

<img src="images\image-20210106134545679.png" alt="image-20210106134545679" style="zoom:80%;" />



<img src="images\image-20210106134610921.png" alt="image-20210106134610921" style="zoom:50%;" />

# 四，配置网络

## 1，配置本机网络

打开网络和共享中心，右键属性

![image-20210106134203997](images\image-20210106134203997.png)



双击int4

![image-20210106134307799](images\image-20210106134307799.png )

配置网络

<img src="images\image-20210106134752134.png" alt="image-20210106134752134" style="zoom:80%;" />



## 2，配置vm以及虚拟机

<img src="images\image-20210106135103775.png" alt="image-20210106135103775" style="zoom:80%;" />

<img src="images\image-20210106135150421.png" alt="image-20210106135150421" style="zoom:80%;" />![image-20210106135301359](images\image-20210106135301359.png)![image-20210106135429510](images\image-20210106135429510.png)

<img src="images\image-20210106135150421.png" alt="image-20210106135150421" style="zoom:80%;" />![image-20210106135301359](images\image-20210106135301359.png)![image-20210106135429510](images\image-20210106135429510.png)

<img src="images\image-20210106135901742.png" alt="image-20210106135901742" />

![image-20210106135948385](images\image-20210106135948385.png)

## 3，设置静态ip

1，输入账号密码，设置的都为root

![image-20210106135649704](images\image-20210106135649704.png)

2，找到 /etc/sysconfig/network-scripts

```
cd  /etc/sysconfig/network-scripts
查看命令  ls
```

<img src="images\image-20210106140225179.png" alt="image-20210106140225179" style="zoom:80%;" />

3，编辑ifcfg-ens33文件（有的版本文件名不一样 eth0等）

```
vi ifcfg-ens33
```

原文件

<img src="images\image-20210106140748838.png" alt="image-20210106140748838" style="zoom:80%;" />

修改为，红色为修改及添加内容

<img src="images\image-20210106141322914.png" alt="image-20210106141322914" style="zoom:80%;" />

```
编辑命令
i 进入编辑
ESC  :q!   不保存退出
esc  :wq   保存退出
```

修改内容含义：

<img src="images\20180512125613253.png" alt="20180512125613253" style="zoom:80%;" />

## 4，在终端输入重启网络服务命令：service network restart或/etc/init.d/network restart

```
service network restart或/etc/init.d/network restart
关闭防火墙
systemctl stop firewalld
查看防火墙状态
systemctl status firewalld
```

## 5、测试，通过ping局域网的IP，看能不能ping通

```
ping  本机ip
ping   百度为百度ip (180.101.49.12)
```

<img src="images\image-20210106142332370.png" alt="image-20210106142332370" style="zoom:80%;" />

搭建完毕

# 五，设置虚拟机共享文件夹

## 1，下载安装VMware  Tools

<img src="images\image-20210106142903108.png" alt="image-20210106142903108" style="zoom:80%;" />

## 2，点击设置，启用共享文件夹

<img src="images\image-20210106143041984.png" alt="image-20210106143041984" style="zoom:80%;" />

## 3，共享文件所在目录

```
cd /mnt/hgfs
ls 查看
```

<img src="images\image-20210106143321537.png" alt="image-20210106143321537" style="zoom:80%;" />

# 六，安装配置jdk

## 1，在共享文件夹找到jdk,并解压

在usr下新建java报，将jdk解压到此目录下

```
cd /mnt/hgfs
ls 查看
tar -zxvf jdk-8u192-linux-x64.tar.gz -C /usr/java
```

## 2.接下来就该配置环境变量了，输入以下指令进行配置：

```
vi  /etc/profile
```

输入完毕并回车，在文件尾部添加如下信息：

```
export JAVA_HOME=/usr/java/jdk1.8.0_192
export CLASSPATH=$:CLASSPATH:$JAVA_HOME/lib/
export PATH=$PATH:$JAVA_HOME/bin
```

## 3.编辑完之后，保存并退出，

然后输入以下指令，刷新环境配置使其生效：

```
source /etc/profile
```

## 4.查看jdk是否安装成功

```
javac
java -version
```

<img src="images\image-20210106145151252.png" alt="image-20210106145151252" style="zoom:80%;" />

安装完毕

# 七，安装zookeeper

所在路径：

```
cd /root/software/zookeeper/apache-zookeeper-3.6.2-bin/bin
```

## 1，安装解压与jdk一样：

我这里解压在root下的software/zookeeper下

```
tar -zxvf apache-zookeeper-3.6.2-bin.tar.gz -C 、root/software/zookeeper
```

## 2.将解压后zookeeper-3.4.6

文件夹下的zoo_sample.cfg文件拷贝一份命名为zoo.cfg==不要删除zoo_sample.cfg==

修改代码（注意文件夹不存在就创建，否则报错）

<img src="images\image-20210106145916009.png" alt="image-20210106145916009" style="zoom:80%;" />

```
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes.
dataDir=/root/software/zookeeper/dataDir  
dataLogDir=/root/software/zookeeper/dataLogDir 
# the port at which the clients will connect
clientPort=2181
server.1=192.168.109.100:3888
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the 
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1

## Metrics Providers
#
# https://prometheus.io Metrics Exporter
#metricsProvider.className=org.apache.zookeeper.metrics.prometheus.PrometheusMetricsProvider
#metricsProvider.httpPort=7000
#metricsProvider.exportJvmInfo=true

```

## 3.启动

　　　　启动zookeeper：进入到zookeeper-3.4.6/bin目录下执行命令：

```
./zkServer.sh start
./zkServer.sh stop   停止
```

　查看zookeeper状态：进入到zookeeper-3.4.6/bin目录下执行命令：

```
./zkServer.sh status
```

<img src="images\image-20210106151215726.png" alt="image-20210106151215726" style="zoom:80%;" />

看到Mode: standalone  为正确状态

启动：

```
./zkCli.sh
```

![image-20210106151633608](images\image-20210106151633608.png)

启动成功

## 4，关闭防火墙

```
关闭防火墙
systemctl stop firewalld
查看防火墙状态
systemctl status firewalld
重启网路
service network restart或/etc/init.d/network restart
```

