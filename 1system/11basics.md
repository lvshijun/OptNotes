
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

lvm逻辑卷管理器

```
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



