
# ZHCHEN 开发分支

### 自定义 ovs-ofctl 工具使用

下发 ecn_ip 过滤策略 (用于取代 red AQM)
```
./utilities/ovs-ofctl ecn_ip <filter_interval> <del | nodel>
```
下发我们的 ecn_tcp 过滤策略
```
 ./utilities/ovs-ofctl ecn_tcp <filter_interval> <del | nodel>
   filter_interval: 过滤间隔, 时间越长, 下发删除策略越久, 过滤包越多
   del  下发策略后并删除
   nodel 仅下发策略

例子:
 ./utilities/ovs-ofctl ecn_ip 500000000 del
```


## 基于源代码make开发方法

### 自定义 ovs-ofctl 工具编译开发
```
vi ./utilities/ovs-ofctl.c  # 修改源文件, 增加自定义提示
make  # 编译
./utilities/ovs-ofctl --help    #查看新版本运行效果
```

### openvswitch.ko 内核编译与开发
将相关内核文件复制到 ./linux-header 中, 便于查看

```
# 单次编译
./configure --with-linux='/lib/modules/`uname -r`/build' && make -C datapath/linux

# 反复编译并挂载
 make && cp /opt/coding/ovs/datapath/linux/openvswitch.ko /lib/modules/3.13.0-86-generic/updates/dkms/openvswitch.ko && rmmod openvswitch && modprobe -v openvswitch && modinfo openvswitch

# 重新挂载后需重启交换机, 并增加流表
ovs-rc-vswitchd.sh restart && ovsdb-rc.sh restart
ovs-ofctl -O Openflow13 add-flow s1 "tcp,nw_dst=10.0.0.2, actions=mod_tp_dst:1,resubmit(,1)"

# 触发测试流
mininet> h2 netperf -H h3 -l 120

# 查看内核日志, 并关闭测试流
dmesg && pkill netperf
 ```

### 初始编译
```
sudo -s
#从 git 下拉 ecn240 分支时可不需要新建
#git checkout v2.4.0 -b ecn240 # 新建ecn开发分支  -b v240rev(hailong分支)
git checkout v2.4.0 -b ecn240 # v240rev(hailong)

./boot.sh # 如出现 libtoolize: can not copy `/usr/share/aclocal/lt~obsolete.m4' to `m4/' 错误则运行autoreconf —install (去掉 —force 参数)
#重编译时需要 make clean && make distclean
./configure # 自编译文件默认安装 到 /usr/local/bin下 (—prefx )
# 不要用 make -j 参数多进程编译, 容易报错
make  && make install
# make modules_install # 这样编译的内核可能会有问题 跳过内核安装，用mininet install.sh / fakeroot来编译安装内核

#复制配置文件
cp /etc/openvswitch/conf.db /usr/local/etc/openvswitch/conf.db
ovs-vswitchd & #启动交换机
ovsdb-server --remote=punix:/usr/local/var/run/openvswitch/db.sock & #启动服务器
#ovs-rc.sh start #启动脚本
ovs-vsctl show  #查看交换机状态
#0b8ed0aa-67ac-4405-af13-70249a7e8a96
# ovs_version: "2.4.0"



## TODO 基于 fakeroot编译方法
DEB_BUILD_OPTIONS="parallel=2 nocheck" fakeroot debian/rules binary



Open vSwitch
============

Build Status:
-------------

[![Build Status](https://travis-ci.org/openvswitch/ovs.png)](https://travis-ci.org/openvswitch/ovs)

What is Open vSwitch?
---------------------

Open vSwitch is a multilayer software switch licensed under the open
source Apache 2 license.  Our goal is to implement a production
quality switch platform that supports standard management interfaces
and opens the forwarding functions to programmatic extension and
control.

Open vSwitch is well suited to function as a virtual switch in VM
environments.  In addition to exposing standard control and visibility
interfaces to the virtual networking layer, it was designed to support
distribution across multiple physical servers.  Open vSwitch supports
multiple Linux-based virtualization technologies including
Xen/XenServer, KVM, and VirtualBox.

The bulk of the code is written in platform-independent C and is
easily ported to other environments.  The current release of Open
vSwitch supports the following features:

* Standard 802.1Q VLAN model with trunk and access ports
* NIC bonding with or without LACP on upstream switch
* NetFlow, sFlow(R), and mirroring for increased visibility
* QoS (Quality of Service) configuration, plus policing
* Geneve, GRE, GRE over IPSEC, VXLAN, and LISP tunneling
* 802.1ag connectivity fault management
* OpenFlow 1.0 plus numerous extensions
* Transactional configuration database with C and Python bindings
* High-performance forwarding using a Linux kernel module

The included Linux kernel module supports Linux 2.6.32 and up, with
testing focused on 2.6.32 with Centos and Xen patches.  Open vSwitch
also has special support for Citrix XenServer and Red Hat Enterprise
Linux hosts.

Open vSwitch can also operate, at a cost in performance, entirely in
userspace, without assistance from a kernel module.  This userspace
implementation should be easier to port than the kernel-based switch.
It is considered experimental.

What's here?
------------

The main components of this distribution are:

* ovs-vswitchd, a daemon that implements the switch, along with
  a companion Linux kernel module for flow-based switching.
* ovsdb-server, a lightweight database server that ovs-vswitchd
  queries to obtain its configuration.
* ovs-dpctl, a tool for configuring the switch kernel module.
* Scripts and specs for building RPMs for Citrix XenServer and Red
  Hat Enterprise Linux.  The XenServer RPMs allow Open vSwitch to
  be installed on a Citrix XenServer host as a drop-in replacement
  for its switch, with additional functionality.
* ovs-vsctl, a utility for querying and updating the configuration
  of ovs-vswitchd.
* ovs-appctl, a utility that sends commands to running Open
      vSwitch daemons.

Open vSwitch also provides some tools:

* ovs-ofctl, a utility for querying and controlling OpenFlow
  switches and controllers.
* ovs-pki, a utility for creating and managing the public-key
  infrastructure for OpenFlow switches.
* ovs-testcontroller, a simple OpenFlow controller that may be useful
  for testing (though not for production).
* A patch to tcpdump that enables it to parse OpenFlow messages.

What other documentation is available?
--------------------------------------

To install Open vSwitch on a regular Linux or FreeBSD host, please
read [INSTALL.md]. For specifics around installation on a specific
platform, please see one of these files:

- [INSTALL.Debian.md]
- [INSTALL.Fedora.md]
- [INSTALL.RHEL.md]
- [INSTALL.XenServer.md]

To use Open vSwitch...

- ...with Docker on Linux, read [INSTALL.Docker.md]

- ...with KVM on Linux, read [INSTALL.md], read [INSTALL.KVM.md]

- ...with Libvirt, read [INSTALL.Libvirt.md].

- ...without using a kernel module, read [INSTALL.userspace.md].

For answers to common questions, read [FAQ.md].

To learn how to set up SSL support for Open vSwitch, read [INSTALL.SSL.md].

To learn about some advanced features of the Open vSwitch software
switch, read the [tutorial/Tutorial.md].

Each Open vSwitch userspace program is accompanied by a manpage.  Many
of the manpages are customized to your configuration as part of the
build process, so we recommend building Open vSwitch before reading
the manpages.

Contact
-------

bugs@openvswitch.org

[INSTALL.md]:INSTALL.md
[INSTALL.Debian.md]:INSTALL.Debian.md
[INSTALL.Docker.md]:INSTALL.Docker.md
[INSTALL.Fedora.md]:INSTALL.Fedora.md
[INSTALL.KVM.md]:INSTALL.KVM.md
[INSTALL.Libvirt.md]:INSTALL.Libvirt.md
[INSTALL.RHEL.md]:INSTALL.RHEL.md
[INSTALL.SSL.md]:INSTALL.SSL.md
[INSTALL.userspace.md]:INSTALL.userspace.md
[INSTALL.XenServer.md]:INSTALL.XenServer.md
[FAQ.md]:FAQ.md
[tutorial/Tutorial.md]:tutorial/Tutorial.md
