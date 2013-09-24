---
layout: post
title: "用net-snmp监控特点的网卡"
date: 2013-09-24 17:15
comments: true
categories: net-snmp
---

net-snmp的监控网卡的默认配置是监控所有的网卡，所以有些你不用的网卡仍然会发送trap，这会给你造成困扰  
默认配置：   
如果配置文件里是   
  
	linkUpDownNotifications yes

等效于下面的配置	

	notificationEvent  linkUpTrap    linkUp   ifIndex ifAdminStatus ifOperStatus  
	notificationEvent  linkDownTrap  linkDown ifIndex ifAdminStatus ifOperStatus  

	monitor  -u cjslyxz -r 600 -e linkUpTrap   "Generate linkUp" ifOperStatus != 2
	monitor  -u cjslyxz -r 600 -e linkDownTrap "Generate linkDown" ifOperStatus == 2

这个是监控所有的网卡的，即使那个网卡被你用“ifconfig”之类的命令disable掉了，它仍然会发送trap.  
可惜net-snmp不支持"ifOperStatus == 2 And ifAdminStatus ！= 2 ”这样的表达式。  
但是我们仍然有两种解决方案。  
##方案一
修改配置如下

	notificationEvent  eth0_linkUpTrap    linkUp   ifIndex.2 ifAdminStatus.2 ifOperStatus.2 ifName.2 ifDescr.2
	notificationEvent  eth0_linkDownTrap  linkDown ifIndex.2 ifAdminStatus.2 ifOperStatus.2 ifName.2 ifDescr.2

	monitor -u cjslyxz -t -r 600 -e eth0_linkUpTrap   "eth0 linkUp"  -I ifOperStatus.2 != 2
	monitor -u cjslyxz -t -r 600 -e eth0_linkDownTrap "eth0 linkDown"  -I ifOperStatus.2 == 2

我们显式使用 “-I”选项表示OID是精确匹配的，那我们是怎样知道这个OID是关联哪个网卡的呢？  
用命令 
  
	snmpwalk -v 2c localhost ifDescr

	.1.3.6.1.2.1.2.2.1.2.1 = STRING: lo
	.1.3.6.1.2.1.2.2.1.2.2 = STRING: eth0
	.1.3.6.1.2.1.2.2.1.2.4 = STRING: eth1
IfOperStatus和ifDescr都是ifEntry的成员，每个ifEntry对应一个interface(网卡)，所以ifOperStaus.2肯定就是eth0的operrate状态。所以上面的配置就是只关心eth0的状态，只要eth0网线断了就会发送trap.  





##方案二
从驱动中把这个网卡禁用掉  


- 显示你的驱动所能识别的网卡的名字  

		# ls -l /sys/bus/pci/drivers/bnx2 
		total 0
		lrwxrwxrwx 1 root root    0 Sep 24 11:34 0000:02:00.0 -> ../../../../devices/pci0000:00/0000:00:1c.0/0000:02:00.0
		lrwxrwxrwx 1 root root    0 Sep 24 11:34 0000:02:00.1 -> ../../../../devices/pci0000:00/0000:00:1c.0/0000:02:00.1
		# ls -Ll /sys/bus/pci/drivers/bnx2/0000:02:00.0/net
		total 0
		drwxr-xr-x 4 root root 0 Sep 24 11:34 eth0

从上面的输出可以看到”0000:02:00.0“ 是和 “eth0“相对应的。

- 从驱动中禁用这个网卡
		
		echo "0000:02:00.0"" >/sys/bus/pci/drivers/bnx2/unbind
 
如果机器重启的话，这个网卡会被重新识别。如果想不重启再重新用这个网卡，用下面的命令：
	
	echo "0000:02:00.0"" >/sys/bus/pci/drivers/bnx2/bind


其实如果你看net-snmp的代码，operstaus 和adminstatus的值都是从调用函数“ioctl”带着“SIOCGIFFLAGS”参数从驱  
动中获取的。如果设备是状态是“IFF_UP”那就设adminstatus=up, 如果是“IFF_RUNNING” 那就设operstaus=up,mib里  
描述的其他状态其实都没处理。  

代码：  

	if(ifentry->os_flags & IFF_UP) {
            ifentry->admin_status = IFADMINSTATUS_UP;
            if(ifentry->os_flags & IFF_RUNNING)
                ifentry->oper_status = IFOPERSTATUS_UP;
            else
                ifentry->oper_status = IFOPERSTATUS_DOWN;
        }
        else {
            ifentry->admin_status = IFADMINSTATUS_DOWN;
            ifentry->oper_status = IFOPERSTATUS_DOWN;
        }

"ifconfig down"命令只会清除“IFF_UP”标志位  
“ifconfig up”命令会设置“IFF_UP”标志位  

如果插上网线，可以发送接收数据包，则设置“IFF_RUNNING”标志位。
  
所以所有出现在ifconfig里的网卡“ifAdminStatus=up”，没插网线  
的网卡“ifOperStatus=down”.还有个规则就是如果“ifAdminStatus=down”  
则对应网卡的“ifOperStatus=down”。

