## 什么是NFS

```
    **nfs是网络文件系统，允许一个节点通过网络访问远程计算机的文件系统，远程文件系统可以被直接挂载到本地，文件操作和本地没有区别，
    如果是局域网的nfs那么io的性能也可以保证，下面就以CentOS7.x为例，配置NFS
```

### 第一步：服务端配置，服务端提供文件系统供客户端来挂载使用，配置过程如下：

```
    首先检查是否缺少基础环境：
        rpm -qa \| grep nfs-utils
        rpm -qa \| grep rpcbind

    如果这两个包存在那么可以直接使用，一般服务器安装的时候都会存在，如果没有的话执行下面命令安装：
        yum -y install nfs-utils
        yum -y install rpcbind

    安装完成之后配置nfs访问目录，配置文件位置/etc/exports，默认是空的这里添加一行：
        /nfs\_test 192.168.1.8\(rw,no\_root\_squash,no\_all\_squash,async\)
        
        【注释】：这个配置表示开放本地存储目录/nfs\_test，只允许192.168.1.8这个主机有访问权限，
                rw表示允许读写；no\_root\_squash表示root用户具有完全的管理权限；
                no\_all\_squash表示保留共享文件的UID和GID，此项是默认不写也可以；
                async表示数据可以先暂时在内存中，不是直接写入磁盘，可以提高性能，
                另外也可以配置sync表示数据直接同步到磁盘；就配置这些就可以，保存退出

    如果想让另外一台主机也可以挂载这个目录，那么直接在后面追加即可，比如：
        /nfs\_test 192.168.1.8\(rw,no\_root\_squash,no\_all\_squash,async\) 192.168.1.9\(rw,no\_root\_squash,no\_all\_squash,async\)
```

\(rw,no\_root\_squash,no\_subtree\_check,async\)

```
        【注释】：多个目录可以每行配置一个，如果想让这个网段的主机都可以访问，假如此时子网掩码是255.255.255.0，
        网关是192.168.1.0，那么ip那里可以写成192.168.1.0/24表示允许地址段的所有主机访问
```

### 第二步： 现在配置完这些配置，启动相关服务：

```
        systemctl start rpcbind.service
        systemctl start nfs.service
        
启动之后可以通过status来查看状态，如果下次修改了配置，可以重启服务来使配置生效，也可以直接执行如下命令刷新配置：

        exportfs -a
        刷新配置即可生效，服务端配置完毕！
```

### 第三步：配置客户端，环境与服务端一样，要保证安装nfs-utils和rpcbind

```
    保证环境没问题和上面一样启动rpcbind服务和nfs服务

    首先创建挂载点： mkdir /mnt/test1
```

```
    然后挂载nfs： mount -t nfs 192.168.1.3:/nfs_test /mnt/test1
```

```
    使用参数: \[hard,intr,vers=3\] 可以使挂载目录按照本机的权限和角色显示。不加则显示为nobody。

    mount -t nfs -o hard,intr,vers=3 10.180.226.128:/nfs_data/fin/essm /home/essm/essmImg/cardHandle
```

```
    挂载成功之后通过 df -h 可以查看挂载的情况，nfs可用空间就是服务端/nfs\_test目录所能使用的最大空间

    现在就可以往nfs写入数据了，服务端往/nfs\_test读写数据和客户端往/mnt/test1读写数据是一样的，这样就实现了文件同步和共享

    卸载nfs和普通文件系统一样，使用： umount /mnt/test1
```

### 第四步：设置开机自动挂载

```
    如果需要设置开机挂载，在/etc/fstab添加一行配置即可：
```

`192.168.1.3:/nfs_test /mnt/test1 nfs rw,tcp,intr 0 1`

然后服务端和客户端都要用enable设置nfs和rpcbind服务开机启动，然后才可以正常挂载

`10.180.226.127:/nfs_data/mobapp/obu_pic /home/mobapp/obu_pic nfs hard,intr,vers=3 0 0`

`mount -t nfs -o hard,intr,vers=3 10.180.226.128:/nfs_data/fin/essm /home/essm/essmImg/cardHandle`

