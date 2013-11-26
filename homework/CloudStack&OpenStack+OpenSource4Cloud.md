# CloudStack&OpenStack介绍#



  **OpenSource4Cloud：**
  
                    1. 段孝妍，duanxiaoyan
        
                    2. 刘成全，cloud2014
                    
                    3. 职承立，zhichengli
                    
## 项目介绍 ##
 
**CloudStack** 

–2008年由Cloud.com开发，有商业和开源两个版本。开源版本采用GPL v2 license 

–2011年7月Cloud.com被Citrix(思杰)收购，CloudStack全部开源 

–2012年4月Citrix将CloudStack贡献给Apache基金会，改为Apache 2.0 license 

**OpenStack** 

–Rackspace和NASA合作研发，2010年10月开源，采用Apache 2.0 license 

–2011年Rackspace宣布成立OpenStack基金会，将OpenStack贡献给该基金会

-核心组件有以下几个，Nova，Swift，Glance，Keystone，Horizon，Quantum&Melange，还包括一些社区项目


## 其他比较 ##
####整体比较####


* 比较项  	| CloudStack	| OpenStack

* 服务层次	|IaaS	       |  IaaS

* 授权协议	| Apache 2.0	| Apache 2.0

* 许可证	| 不需要	| 不需要

* 动态资源调配	| 主机Maintainance模式下自动迁移VM	| 无现成功能，需通过Nova-scheduler组件自己实现

* VM模板	| 支持	| 支持

* VM Console	| 支持	| 支持

* 开发语言	| Java	| Python

* 用户界面	|Web Console,功能较完善	| DashBoard，较简单

* 负载均衡	| 软件负载均衡(Virtual Router)、硬件负载均衡	| 软件负载均衡(Nova-network或 OpenStack Load Balance API)、硬件负载均衡

* 虚拟化技术	| XenServer,Oracle VM，vCenter,KVM,Bare Metal	| XenServer,Oracle VM,KVM,QEMU,ESX/ESXi,LXC(Liunx Container)等

* 最小化部署	| 一管理节点，一主机节点	|支持All in one（Nova,Keystone,Glance组件必选）

* 支持数据库	| MySQL	| PostgreSQL,MySQL,SQLite

* 组件	| Console Proxy VM,Second Storage VM,Virtual Router VM,Host Agent,Management Server	| Nova,Glance,Keystone,Horizon,Swift

* 网络形式	| Isolation（VLAN），Share |	VLAN,FLAT,FLATDhcp

* 版本问题	| 版本发布稳定，不存在兼容性问题	| 存在各版本兼容性问题
 
* VLAN	不能VLAN间互访	| 支持VLAN间互访

####存储比较####

**CloudStack：**

   CloudStack把存储分成了主存储与二级存储。根据Hypervisor种类的不同, 主存支持不同的磁盘镜像格式。iSCSI和FC-San存储在Xenserver中被加载为clustered LVM格式，此种格式下，不支持存储的超配。如果存储支持XenServer的thin-provisioning，对于每个磁盘是链式存储，对磁盘的copy等操作，都会基于该磁盘生成一个新的链，新加内容写入新的链中，所以支持存储超配。
   
   _两种存储的功能：_
   
   主存储主要用来存放虚拟机的磁盘镜像，二级存储用来存放template，snapshot和需要下载的volume。二级存储不直接挂载到hyperviser上，需要由management server或ssvm来进行操作

**OpenStack：**

   OpenStack有三个与存储相关的组件：

* Swift——对象存储 （Object Storage），在概念上类似于Amazon S3服务，swift具有很强的扩展性、冗余和持久性，用于永久类型的静态数据的长期存储。在OpenStack中它与Glance结合，为其存储镜像文件。并且因为它具有无限的可扩展性，也适合用于存储日志文件和数据备份仓库。

* Glance——虚机镜像（Image）存储和管理，Glance是一个虚机镜像的存储。向前端nova提供镜像服务，包括存储，查询和检索。这个模块本身不存储大量的数据，需要挂载后台存储（比如swift）来存放实际的镜像数据。

* Cinder——块存储（Block Storage），类似于Amazon的EBS块存储服务，目前仅给虚机挂载使用。Openstack从Folsom开始使用Cinder替换原来的Nova-Volume服务，为Openstack云平台提供块存储服务。
Cinder通过添加不同厂商的指定drivers来为了支持不同类型和型号的存储。目前能支持的商业存储设备有EMC 和IBM的几款，也能通过LVM支持本地存储和NFS协议支持NAS存储。

####网络设计对比####
* CloudStack网络设计
   CloudStack中根据不同的数据流量类型设计了管理，公共，客户及存储网络，可以简称为PMGS ( Public, Management, Guest, Storage) 网络．
   CloudStack中网络模式可以分成基本网络和高级网络两种．其中MGS(Management，Guest，Storage)三种网络对于基本网络及高级网络通用．而P(Public)则只是针对高级网络才存在．
   在同一个资源域内，虚拟机有两种方式进行隔离：安全组和VLAN．
   当用户创建虚拟机实例后，可以对这些虚拟机实例设定一个或多个安全组．
   同一个安全组下的用户虚拟机实例可以相互通信．安全组通过Ingress和Egress来进行流量控制.
   VLAN
   高级网络模式中默认是通过VLAN进行虚拟机实例之间的相互隔离．当一个账户的第一个虚拟机被创建并运行时，一个隔离的网络也同时创建完成． 

* OpenStack网络设计
   1、OpenStack中nova-network的作用
       OpenStack平台中有两种类型的物理节点，控制节点和计算节点。控制节点包括网络控制、调度管理、api服务、存储卷管理、数据库管理、身份管理和镜像管理等，计算节点主要提供nova-compute服务。控制节点的服务可以分开在多个节点，我们把提供nova-network服务的节点称为网络控制器。
       OpenStack的网络由nova-network（网络控制器）管理，它会创建虚拟网络，使主机之间以及与外部网络互相访问。
       OpenStack的API服务器通过消息队列分发nova-network提供的命令，这些命令之后会被nova-network处理，主要的操作有：分配ip地址、配置虚拟网络和通信。

   2、OpenStack中network的2种ip、3种管理模式
       Nova有固定IP和浮动IP的概念。Nova支持3种类型的网络，对应3种“网络管理”类型：Flat管理模式、FlatDHCP管理模式、VLAN管理模式。默认使用VLAN摸式。
       这3种类型的网络管理模式，可以在一个ОpenStack部署里面共存，可以在不同节点不一样，可以进行多种配置实现高可用性。
       简要介绍这3种管理模式

       1)	Flat（扁平）： 所有实例桥接到同一个虚拟网络，需要手动设置网桥。

       2)	FlatDHCP： 与Flat（扁平）管理模式类似，这种网络所有实例桥接到同一个虚拟网络，扁平拓扑。不同的是，正如名字的区别，实例的ip提供dhcp获取（nova-network节点提供dhcp服务），而且可以自动帮助建立网桥。

       3)	VLAN： 为每个项目提供受保护的网段（虚拟LAN）。

* CloudStack和OpenStack网络设计比较
   从上面他们各自网络设计的介绍，我们可以看出他们都支持两种网络模式：一是类似AWS的扁平网络模式，CloudStack对应基本网络模式，OpenStack对应Flat和FlatDHCP网络模式；二是VLAN模式。但是，OpenStack所有与外部网络通信都要通过网络控制器，存在SPoF（单故障点）问题，而且如果与外网通信太多，会造成控制节点网络的堵塞或者高负载。CloudStack通过VR与外部通信不存在SPoF（单故障点）问题。CloudStack的VR提供的网络功能包括NAT、静态NAT、DHCP、DNS、Load Balancing、Port Forwording、Firewalls、Site-to-Site VPN等。OpenStack的网络控制器支持NAT、DHCP、DNS、Gateway等网络功能。



## 结合自身需要评估后的结论 ##

   CloudStack有一个在线社区免费提供及时的技术支持。可以在论坛中找到许多CloudStack问题的解决方案。还有一个IRC(互联网中继聊天)频道，欢迎每一个人提出问题。CloudStack有60家合作伙伴，包括NTT，瞻博网络，塔塔通信，博科通讯系统公司、英特尔等。CloudStack有大约1500名社区成员。OpenStack拥有比较大的活跃的社区。社区的成员总是愿意帮助其他人找到出现的任何问题的解决方案。核心项目6个，超过180家企业支持，3000多名社区贡献者。包括数据中心设备厂商思科系统、戴尔、惠普和IBM。
   
   CloudStack已经有了许多商用客户，包括GoDaddy、英国电信、日本电报电话公司、塔塔集团、韩国电信等。OpenStack客户有美国国家航空航天局，加拿大半官方机构CANARIE网络的DAIR（Digital Accelerator for Innovation and Research）项目，向大学与中小型企业提供研究和开发云端运算环境；DAIR用户可以按需要快速建立网络拓扑。
惠普云（使用Ubuntu Linux）MercadoLibre的IT基础设施云，现时以OpenStack管理超过6000 台虚拟机器。AT&T的“Cloud Architect”，将在美国的达拉斯、圣地亚哥和新泽西州对外提供云端服务。

   截至目前OpenStack在市场宣传、影响力方面远胜过CloudStack，支持伙伴、社区开发人数及讨论话题数、活跃程度等也高于CloudStack，但
CloudStack的平台成熟度要优于OpenStack，CloudStack的用户体验及安装容易度也都比OpenStack要好，并已在更具生产实际的环境中得到了充分验证，而OpenStack到目前为止则更像是仍处于研发阶段难以称为“成熟的产品化的IT产品”。

## 组员贡献 ##

职承立：开篇项目介绍，整体比较，CloudStack的资源管理

段孝妍：提交文档；CloudStack和OpenStack存储功能的介绍，OpenStack的nova组件介绍

刘成全：CloudStack和Opnstack网络部分的介绍，结束部分的社区活跃度对比和应用实例。

## 参考资料 ##
存储部分：_《程序员》2012年07期 作者 程辉_

_待组员提交或将自己引用部分补齐_
