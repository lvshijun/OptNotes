## jdk安装

【jdk安装规范】：  位置  /usr/local/java/jdk\*

```
             修改环境变量文件全局生效修改/etc/profile文件

             单个用户修改/home/username/.bash\_profile
```

### 1、源码包准备：

[http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)

首先到官网下载jdk-8u66-linux-x64.tar.gz，

### 2、解压源码包

* 通过终端在/usr/local目录下新建java文件夹，命令行：

  `sudo mkdir /usr/local/java`

* 然后将下载到压缩包拷贝到java文件夹中，命令行：

  ```
       cp jdk-8u66-linux-x64.tar.gz /usr/local/java
  ```

* 然后进入java目录，命令行：

  ```
      cd /usr/local/java
  ```

* 解压压缩包，命令行：

  ```
      sudo tar zxvf jdk-8u66-linux-x64.tar.gz
  ```

* 然后可以把压缩包删除，命令行：

  ```
      sudo rm jdk-8u66-linux-x64.tar.gz
  ```

### 3、设置jdk环境变量

这里采用全局设置方法，就是修改etc/profile，它是是所有用户的共用的环境变量

* 编辑/etc/profile文件，打开之后在末尾添加如下内容

`vi /etc/profile`

```
JAVA_HOME=/usr/local/java/jdk1.8.0_211
JRE_HOME=/usr/local/java/jdk1.8.0_211/jre
CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
PATH=$JAVA_HOME/bin:$PATH
export PATH JAVA_HOME CLASSPATH
```

* 使环境变量生效

```
    source /etc/profile
```

* 看看自己的配置是否都正确

```
    echo $JAVA_HOME
    echo $CLASSPATH
    echo $PATH
```

### 4、修改默认JDK

如果系统已经安装了其他版本的Java

```
    update-alternatives --install /usr/bin/java java /usr/java/jdk1.8.0_66/bin/java 300
    update-alternatives --install /usr/bin/javac javac /usr/java/jdk1.8.0_66/bin/javac 300
    update-alternatives --config java
    update-alternatives --config javac
```

### 5、检验是否安装成功

在终端执行命令查看版本是否生效

```
    java -version
```

看看是否安装成功，成功则显示如下

```
java version "1.8.0_66"
Java(TM) SE Runtime Environment (build 1.8.0_66-b17)
Java HotSpot(TM) 64-Bit Server VM (build 25.65-b01, mixed mode)


JAVA_HOME=/usr/local/java/jdk1.8.0_211
JRE_HOME=/usr/local/java/jdk1.8.0_211/jre
CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
PATH=$JAVA_HOME/bin:$PATH
export PATH JAVA_HOME CLASSPATH
```



