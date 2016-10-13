#rbd加入centos7内核
 > 技术研发中心 许新浩 2016/10/10 

* 安装已经编译好的内核
	 
		[root@spark3 ~]# rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
	 	[root@spark3 ~]# rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm
	 	[root@spark3 ~]# yum install -y yum-plugin-fastestmirror
	 	[root@spark3 ~]# yum --enablerepo=elrepo-kernel install kernel-ml kernel-ml-devel
* 查看新内核是否成功支持rbd模块
	 
		[root@spark3 ~]# find / -name rbd.ko
		/usr/lib/modules/3.10.0-229.el7.x86_64/kernel/drivers/block/rbd.ko
		/usr/lib/modules/3.10.0-327.22.2.el7.x86_64/kernel/drivers/block/rbd.ko
		/usr/lib/modules/4.6.4-1.el7.elrepo.x86_64/kernel/drivers/block/rbd.ko
* 修改为以新内核启动
	 	
		[root@spark3 ~]# cat /boot/grub2/grub.cfg | grep menuentry  (以下列表具体根据自己的环境)
		if [ x"${feature_menuentry_id}" = xy ]; then
		menuentry_id_option="--id"
		menuentry_id_option=""
		export menuentry_id_option
		menuentry 'CentOS Linux (4.6.4-1.el7.elrepo.x86_64) 7 (Core)' 太长省略
		menuentry 'CentOS Linux (3.10.0-327.el7.x86_64) 7 (Core)' 太长省略
		menuentry 'CentOS Linux (0-rescue-95f57df9111143f3ae61b8946aba3bee) 7 (Core)' 太长省略

		[root@spark3 ~]# grub2-set-default 'CentOS Linux (4.6.4-1.el7.elrepo.x86_64) 7 (Core)'
		
		[root@spark3 ~]# grub2-editenv list
		saved_entry=CentOS Linux (4.6.4-1.el7.elrepo.x86_64) 7 (Core)
		
		[root@localhost ~]# grub2-mkconfig -o /boot/grub2/grub.cfg
		Generating grub configuration file ...
		Found linux image: /boot/vmlinuz-3.18.1-1.el7.elrepo.x86_64
		Found initrd image: /boot/initramfs-3.18.1-1.el7.elrepo.x86_64.img
		Found linux image: /boot/vmlinuz-3.10.0-123.el7.x86_64
		Found initrd image: /boot/initramfs-3.10.0-123.el7.x86_64.img
		Found linux image: /boot/vmlinuz-0-rescue-0e18ebb9ae3d4dbfb4b95bbd4cf359ed
		Found initrd image: /boot/initramfs-0-rescue-0e18ebb9ae3d4dbfb4b95bbd4cf359ed.img
		done
* 重启系统, 确认已经可以使用rbd模块.
		
		[root@spark3 ~]# uname -r
		4.6.4-1.el7.elrepo.x86_64
	
		[root@spark3 ~]# modprobe rbd
		
		[root@spark3 ~]# lsmod | grep rbd
		rbd                    69632  0 
		libceph               249856  1 rbd