#ceph块设备
>技术研发中心 许新浩 2016/10/09

## 一、基本操作：	
	
* 创建块设备映像：

		rbd create --size {megabytes} {pool-name}/{image-name}

	eg：

		rbd create --size 1024 wfpool/wfimage

* 罗列块设备映像：

		rbd ls [{poolname}]     默认为rbd
			
	eg：

		rbd ls wfpool
			
* 检索映像信息:

		rbd info [{pool-name}/]{image-name} 	（同上默认存储池）
			
	eg：

		rbd info wfpool/wfimage
			
* 调整块设备映像大小：
			
		rbd resize --size 2048 [{pool-name}/]{image-name} (增加)
		rbd resize --size 2048 [{pool-name}/]{image-name} --allow-shrink (减少)
	eg：

		rbd resize --size 2048 wfpool/wfimage 
		rbd resize --size 1024 wfpool/wfimage --allow-shrink
			
* 删除块设备映像：

		rbd rm [{pool-name}/]{image-name}
			
	eg：
		
		rbd rm wfpool/wfimage
		
* 映射块设备为内核模块:

	> 这里使用centos7，内核中是没有rbd模块的。要先编译内核，将rbd模块加入内核。参考:rbd加入centos7内核 文档。
		  		
		sudo rbd map {pool-name}/{image-name} -m {monitor-host} [-k /path/to/ceph.client.admin.keyring]
			
	eg：
	
		sudo rbd map wfpool/wfimage -m mon1 -k /etc/ceph/ceph.client.admin.keyring 出错，参考:块设备映射排错 文档。
			
* 查看已映射块设备：
		
		rbd showmapped
			
* 创建文件系统后就可以使用块设备了:

		sudo mkfs.ext4 -m0 /dev/rbd0


* 挂载此文件系统。

		mkdir ceph-block
		sudo mount /dev/rbd0 ~/ceph-block/
		cd ceph-block
			
			
* 取消块设备映射:
		
		sudo rbd unmap /dev/rbd/{poolname}/{imagename}
			
	eg：

		sudo rbd unmap /dev/rbd/wfpool/wfimage2	
	
##二、快照：
	
* 创建快照：
	
		rbd snap create {pool-name}/{image-name}@{snap-name}
		
	eg：

		rbd snap create wfpool/wfimage2@wfsnap
		
* 罗列快照：
	
		rbd snap ls {pool-name}/{image-name}
		
	eg：

		rbd snap ls wfpool/wfimage2
		
* 回滚快照:
	
		rbd snap rollback {pool-name}/{image-name}@{snap-name}
			
	eg：
	
		rbd snap rollback wfpool/wfimage2@wfsnap
		
* 删除一个快照:
	
		rbd snap rm {pool-name}/{image-name}@{snap-name}
			
	eg：

		rbd snap rm wfpool/wfimage2@wfsnap
		
* 删除所有快照：
	
		rbd snap purge {pool-name}/{image-name}
		
	eg：

		rbd snap purge wfpool/wfimage2
		
* 保护快照：
	
		rbd snap protect {pool-name}/{image-name}@{snap-name}
			
	eg：

		rbd snap protect wfpool/wfimage2@wfsnap
		
* 克隆快照：
	
		rbd clone {pool-name}/{parent-image}@{snap-name} {pool-name}/{child-image-name}
			
	eg：

		rbd clone wfpool/wfimage2@wfsnap wfpool/wfimage-child
		
* 取消快照保护:
	
		rbd snap unprotect {pool-name}/{image-name}@{snap-name}
			
	eg：
	
		rbd snap unprotect wfpool/wfimage2@wfsnap
		
* 罗列子快照
	
		rbd children {pool-name}/{image-name}@{snap-name}
			
	eg: 

		rbd children wfpool/wfimage2@wfsnap
		
* 压扁克隆映像:
			
		rbd flatten {pool-name}/{image-name}
			
	eg: 

		rbd flatten wfpool/wfimage2
	