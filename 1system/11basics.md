
---

**单用户模式**

```bash
    Booting Red Hat Enterprise Linux (2.6.32-71.e16.i686) in 3 seconds...【倒计时3 2 1】
    在系统倒计时界面按方向键↓
    敲入“e”，把光标移动到kernel ...那一行，再敲入“e”，
    在kernel 一行的最后加上空格single，回车敲入“b”，
    启动系统，即进入单用户模式
```

---

**Firewalld**

```bash
 使用firewalld打开关闭防火墙与端口

   1.firewalld的基本使用

        systemctl start   firewalld    启动防火墙 
        systemctl stop    firewalld    关闭防火墙
        systemctl status  firewalld    查看防护墙状态 
        systemctl disable firewalld    开机禁用防火墙
        systemctl enable  firewalld    开机启用防火墙 

    2.systemctl是CentOS7的服务管理工具中主要的工具，它融合之前service和chkconfig的功能于一体。

        systemctl start   firewalld.service    启动一个服务 
        systemctl stop    firewalld.service    关闭一个服务 
        systemctl restart firewalld.service    重启一个服务 
        systemctl status  firewalld.service    显示一个服务的状态
        systemctl enable  firewalld.service    在开机时启用一个服务
        systemctl disable firewalld.service    在开机时禁用一个服务
        systemctl is-enabled firewalld.service    查看服务是否开机启动
        systemctl list-unit-files|grep enabled    查看已启动的服务列表
        systemctl --failed    查看启动失败的服务列表

    3.配置firewalld-cmd

        firewall-cmd --version      查看版本： 
        firewall-cmd --help         查看帮助： 
        firewall-cmd --state        显示状态： 
        firewall-cmd --reload       更新防火墙规则： 
        firewall-cmd --panic-on     拒绝所有包： 
        firewall-cmd --panic-off    取消拒绝状态： 
        firewall-cmd --query-panic  查看是否拒绝： 
        firewall-cmd --get-active-zones            查看区域信息: 
        firewall-cmd --get-zone-of-interface=eth0  查看指定接口所属区域： 
        firewall-cmd --zone=public --list-ports    查看所有打开的端口： 


    4.那怎么开启一个端口呢?

        添加放行端口
            firewall-cmd --zone=public --add-port=80/tcp --permanent    （--permanent永久生效，没有此参数重启后失效）
        重新载入规则
            firewall-cmd --reload
        查看域规则
            firewall-cmd --zone= public --query-port=80/tcp
        删除域中某条规则
            firewall-cmd --zone= public --remove-port=80/tcp --permanent
```

---

**lvm逻辑卷管理器**

```bash
block Device-- physical volumes-- volume group ---logical volumes

指令详解:
    pvs --显示物理卷
    pvcreate --创建物理卷 (pvreate /dev/sda )
    pvdisplay --详细显示物理卷的信息
    vgcreate --创建卷组（vgcreate -s 16M[PE size] vg0[vgname] /dev/sda[磁盘] )
    vgextend --增加新的物理卷至某一卷组（vgextend vg0 /dev/sda2)
    vgs --显示卷组信息
    vgdisplay --显示详细的卷组信息
    vgrename --重命名vg名称
    vgchange --vgchange -an vg0 禁用vg0 [-ay激活]
    lvs --显示逻辑卷
    lvcreate --创建逻辑卷 (lvreate -n lv0[卷组名] -l[PE个数] or -L[大小] vg0[指定来自那个卷组] )
    lvextend --扩展逻辑卷大小（lvextend /dev/vg0/lv0 -l[PE个数] or -L[大小] /dev/vg00/lv0[指定目标]
    --扩展大小时‘+‘表示增加多大，不带’+‘表示扩展至多大 ）
    lvdisplay --详细显示逻辑卷的信息
    resize2fs --同步文件系统 ( resize2fs /dev/vg0/lv0 xfs的文件系统使用 xfs_growfs /dev/vg0/lv0)

空间扩展:
    思路 :先把新增加的硬盘变成物理卷--把新的物理卷增加到卷组中--将空间分配至需要扩展的逻辑卷中--同步文件系统
    #实际工作中一般我们是直接增加空间,这时直接从vgextend开始操作即可
        pvcreate -- vgextend -- lvextend -- resize2fs


注意 :新添加的硬盘如果不重启机器是看不到的,生产环境不允许重启.我们可以通过手动扫描发现设备
    echo "- - -" > /sys/class/scsi_host/host0/scan
    echo "- - -" > /sys/class/scsi_host/host1/scan
    echo "- - -" > /sys/class/scsi_host/host2/scan

lvm> lvextend -L +20G /dev/VolGroup00/LogVol02，给分区/var扩展20G的容量
lvm> lvextend -l +100%FREE /dev/VolGroup00/LogVol02，扩展整块硬盘空间 (centos7 有问题)
resize2fs /dev/vg0/lv0
```

---

**Linux内核参数优化**

```
内核参数优化
    key=建议优化参数
    /etc/sysctl.conf文件
    #【/etc/sysctl.conf是一个允许你改变正在运行中的Linux系统的接口。
    #它包含一些TCP/IP堆栈和虚拟内存系统的高级选项，可用来控制Linux网络配置，由于/proc/sys/net目录内容的临时性，
    #建议把TCPIP参数的修改添加到/etc/sysctl.conf文件, 然后保存文件，使用命令“/sbin/sysctl –p”使之立即生效。】

net.core.rmem_default = 256960    #默认的TCP数据接收窗口大小（字节）。
net.core.rmem_max = 513920        #最大的TCP数据接收窗口（字节）。
net.core.wmem_default = 256960    #默认的TCP数据发送窗口大小（字节）。
net.core.wmem_max = 513920        #最大的TCP数据发送窗口（字节）。
net.core.netdev_max_backlog = 2000#在每个网络接口接收数据包的速率比内核处理这些包的速率快时，允许送到队列的数据包的最大数目。
net.core.somaxconn = 2048         #定义了系统中每一个端口最大的监听队列的长度，这是个全局的参数。
net.core.optmem_max = 81920       #表示每个套接字所允许的最大缓冲区的大小。
net.ipv4.tcp_mem = 131072  262144  524288
    #确定TCP栈应该如何反映内存使用，每个值的单位都是内存页（通常是4KB）。第一个值是内存使用的下限；
    #第二个值是内存压力模式开始对缓冲区使用应用压力的上限；第三个值是内存使用的上限。
    #在这个层次上可以将报文丢弃，从而减少对内存的使用。对于较大的BDP可以增大这些值（注意，其单位是内存页而不是字节）。
net.ipv4.tcp_rmem = 8760  256960  4088000
    #为自动调优定义socket使用的内存。第一个值是为socket接收缓冲区分配的最少字节数；第二个值是默认值（该值会被rmem_default覆盖），
    #缓冲区在系统负载不重的情况下可以增长到这个值；
    #第三个值是接收缓冲区空间的最大字节数（该值会被rmem_max覆盖）。
net.ipv4.tcp_wmem = 8760  256960  4088000
    #为自动调优定义socket使用的内存。第一个值是为socket发送缓冲区分配的最少字节数；第二个值是默认值（该值会被wmem_default覆盖），
    #缓冲区在系统负载不重的情况下可以增长到这个值；
    #第三个值是发送缓冲区空间的最大字节数（该值会被wmem_max覆盖）。
net.ipv4.tcp_keepalive_time = 1800    #TCP发送keepalive探测消息的间隔时间（秒），用于确认TCP连接是否有效。
net.ipv4.tcp_keepalive_intvl = 30     #探测消息未获得响应时，重发该消息的间隔时间（秒）。
net.ipv4.tcp_keepalive_probes = 3     #在认定TCP连接失效之前，最多发送多少个keepalive探测消息。
net.ipv4.tcp_sack = 1
    #启用有选择的应答（1表示启用），通过有选择地应答乱序接收到的报文来提高性能，让发送者只发送丢失的报文段，
    #（对于广域网通信来说）这个选项应该启用，但是会增加对CPU的占用。
net.ipv4.tcp_fack = 1            #启用转发应答，可以进行有选择应答（SACK）从而减少拥塞情况的发生，这个选项也应该启用。
net.ipv4.tcp_timestamps = 1
    #TCP时间戳（会在TCP包头增加12个字节），以一种比重发超时更精确的方法（参考RFC 1323）来启用对RTT 的计算，
    #为实现更好的性能应该启用这个选项。
net.ipv4.tcp_window_scaling = 1
    #启用RFC 1323定义的window scaling，要支持超过64KB的TCP窗口，必须启用该值（1表示启用），TCP窗口最大至1GB，
    #TCP连接双方都启用时才生效。
net.ipv4.tcp_syncookies = 1
    #表示是否打开TCP同步标签（syncookie），内核必须打开了CONFIG_SYN_COOKIES项进行编译，同步标签可以防止一个
    #套接字在有过多试图连接到达时引起过载。
net.ipv4.tcp_tw_reuse = 1        #表示是否允许将处于TIME-WAIT状态的socket（TIME-WAIT的端口）用于新的TCP连接 。
net.ipv4.tcp_tw_recycle = 1      #能够更快地回收TIME-WAIT套接字。
net.ipv4.tcp_fin_timeout = 30    
    #对于本端断开的socket连接，TCP保持在FIN-WAIT-2状态的时间（秒）。对方可能会断开连接或一直不结束连接或不可预料的进程死亡。
net.ipv4.ip_local_port_range = 1024  65000    #表示TCP/UDP协议允许使用的本地端口号
net.ipv4.tcp_max_syn_backlog = 2048           
    #对于还未获得对方确认的连接请求，可保存在队列中的最大数目。如果服务器经常出现过载，可以尝试增加这个数字。
······························
举例：
    net.ipv4.tcp_syncookies = 1
    net.ipv4.tcp_max_syn_backlog = 16384
    net.core.somaxconn = 16384
```



