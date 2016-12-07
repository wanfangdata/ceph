#centos7 下用 ceph-deploy 部署 ceph jewel
>技术研发中心许新浩 2016/10/9 

##一、 环境准备

1. 在各个节点上创建普通用户，这里禁止使用ceph的用户名：
       
		useradd -d /home/yourname -m yourname
        passwd yourname
        ssh yourname@ceph-server.ip

2. 确保各 Ceph 节点上新创建的用户都有 sudo 权限(root执行)：
        
		echo "yourname ALL = (root) NOPASSWD:ALL" | tee /etc/sudoers.d/yourname
        chmod 0440 /etc/sudoers.d/yourname

3. 在管理节点上为创建的用户配置免密码登陆：
        
		ssh-keygen
        ssh-copy-id yourname@ip

4. 关闭防火墙，并设置开机不启动        
       
	    systemctl stop firewalld.service 
        systemctl disable firewalld.service

5. 在各节点上执行：

		sudo visudo 
并找到 Defaults requiretty 选项，把它注释掉或者改为:

		Defaults:ceph !requiretty 

6. 把 seLinux 设置为 permissive 或者完全禁用( disabled ) ：
 
		sudo vi /etc/selinux/config 或者 sudo sed -i 's/SELINUX=.*/SELINUX=disabled/' /etc/selinux/config

7. 安装epel源（所有节点）：
        
        sudo yum -y install epel-release 
            
      把base源和epel源配置为阿里源。
            
       修改里面的系统版本为7.2.1511,当前用的centos的版本的的yum源可能已经清空了

        sed -i '/aliyuncs/d' /etc/yum.repos.d/CentOS-Base.repo

        sed -i '/aliyuncs/d' /etc/yum.repos.d/epel.repo

        sed -i 's/$releasever/7.2.1511/g' /etc/yum.repos.d/CentOS-Base.repo

8. 设置环境变量，让ceph-deploy使用国内源（这里使用的163源）：

		export CEPH_DEPLOY_REPO_URL=http://mirrors.aliyun.com/ceph/rpm-jewel/el7
		export CEPH_DEPLOY_GPG_URL=http://mirrors.aliyun.com/ceph/keys/release.asc

9. 安装ntp服务：

        sudo yum install ntp -y
        sudo ntpdate s2m.time.edu.cn

10. 安装yum优先级插件yum-plugin-priorities（所有节点）
    
    	sudo yum -y install yum-plugin-priorities

    执行完后确认存在 /etc/yum/pluginconf.d/priorities.conf 文件，同时确认其已开启，文件内容应为：

	    [main]
	    enabled = 1

    添加key:
    
    	sudo rpm --import 'http://mirrors.aliyun.com/ceph/keys/release.asc'

    
11. 配置内网网卡（如果没有可以不配）：

     查看所有网卡：

        nmcli con show
       为未绑定ip的网卡绑定ip: 

		sudo vi /etc/sysconfig/network-scripts/ifcfg-xxx
	>     DEVICE=xxx
		TYPE=Ethernet
		ONBOOT=yes
		BOOTPROTO=static
		IPADDR=your addr
		NETMASK=255.255.255.0

      查看路由表是否绑定对应网卡：
	
		route -n
        route add -host your addr dev xxx
       
	 重启网络服务：

        sudo service network restart

12. 在管理节点安装ceph-deploy：
       
        sudo yum install ceph-deploy -y


##二、集群部署

1. 先在管理节点上创建一个目录，用于保存 ceph-deploy 生成的配置文件和密钥对。
      
     	mkdir myceph
        cd myceph

2. 创建集群监控节点

        ceph-deploy new your_monitor_node

3. 编辑ceph.conf（具体需要根据自己的配置来）
        
		[global]
		fsid = 36f8fe26-3a47-4d13-aa5f-494357eb1856（）
		mon_initial_members = spark3, spark4, spark5
		mon_host = 168.160.184.211,168.160.184.213,168.160.184.215
		auth_cluster_required = cephx
		auth_service_required = cephx
		auth_client_required = cephx
		public network = 168.160.184.1/24
		cluster_network = 10.0.0.0/24 
		osd pool default size = 3
		osd_pool_default_min_size = 2 
		rgw_override_bucket_index_max_shards = 2
		[mon]
		mon clock drift allowed = 2
		mon clock drift warn backoff = 30
		[client.rgw.spark3]
		rgw_frontends = "civetweb port=80"
		rgw dns name = cephtest.wanfangdata.com.cn
        
4. 在各节点安装 Ceph
    
    管理节点上执行
         
		ceph-deploy install {ceph-node} [{ceph-node} ...]
       遇到错误 :
        
		[ceph_deploy][ERROR ] RuntimeError: NoSectionError: No section: 'ceph'
      解决办法 
          
		sudo yum remove ceph-release -y  
	再次执行  

		ceph-deploy install your nodes
      
                
	想从头再来，可以用下列命令清除配置：
                     
		ceph-deploy purgedata {ceph-node} [{ceph-node}] 
        ceph-deploy forgetkeys
                 
	用下列命令可以连 Ceph 安装包一起清除：
                      
		ceph-deploy purge {ceph-node} [{ceph-node}]
        
5. 初始化监控节点并收集所有密钥：
            
		ceph-deploy mon create-initial
    
6. 添加osd：
    
    查看节点上的硬盘： 

		ceph-deploy disk list {node-name [node-name]...}   
	**一定要注意盘符** 
	####方法一(会有奇怪的bug,不建议使用)

      对数据盘和Journal盘进行分区，并格式化数据盘，Journal盘不需要格式化：

		#sh 以下盘符参考自己的环境
		sudo parted -s /dev/sdc mklabel gpt
		sudo parted -s /dev/sdc mkpart primary xfs 0% 100%
		sudo mkfs.xfs /dev/sdc1 -f
		sudo parted -s /dev/sdd mklabel gpt
		sudo parted -s /dev/sdd mkpart primary xfs 0% 100%
		sudo mkfs.xfs /dev/sdd1 -f
		sudo parted -s /dev/sde mklabel gpt
		sudo parted -s /dev/sde mkpart primary xfs 0% 100%
		sudo mkfs.xfs /dev/sde1 -f
		sudo parted -s /dev/sdb mklabel gpt
		sudo parted -s /dev/sdb mkpart primary 0% 33%
		sudo parted -s /dev/sdb mkpart primary 34% 66%
		sudo parted -s /dev/sdb mkpart primary 67% 100%

	创建osd:

		#sh 以下盘符参考自己的环境
		ceph-deploy osd create spark3:/dev/sda1:/dev/sdd1
		ceph-deploy osd create spark3:/dev/sdb1:/dev/sdd2
		ceph-deploy osd create spark3:/dev/sdc1:/dev/sdd3
		ceph-deploy osd create spark4:/dev/sda1:/dev/nvme0n1p1
		ceph-deploy osd create spark4:/dev/sdb1:/dev/nvme0n1p2
		ceph-deploy osd create spark4:/dev/sdc1:/dev/nvme0n1p3
		ceph-deploy osd create spark5:/dev/sdc1:/dev/nvme0n1p1
		ceph-deploy osd create spark5:/dev/sdd1:/dev/nvme0n1p2
		ceph-deploy osd create spark5:/dev/sde1:/dev/nvme0n1p3

	####方法二：

   	删除磁盘分区：

		ceph-deploy disk zap {osd-server-name}:{disk-name}

    创建osd： 

		ceph-deploy osd create {node-name}:{disk}[:{path/to/journal}]   
  
  	若对journal分区，需要进行激活：

        ceph-deploy osd activate {node-name}:{data-disk-partition}[:{journal-disk-partition}]
        
    
7. 用 ceph-deploy 把配置文件和 admin 密钥拷贝到Ceph 节点，这样你每次执行 Ceph 命令行时就无需指定 monitor 地址和 ceph.client.admin.keyring ：
    
        ceph-deploy admin {ceph-nodes}

8. 修改 ceph.client.admin.keyring 的操作权限（所有节点）：
        
          sudo chmod +r /etc/ceph/ceph.client.admin.keyring

9. 检查集群的健康状况：
        
        ceph health

                
       
                    
        


                                        




