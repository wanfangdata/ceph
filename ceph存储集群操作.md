#ceph存储集群操作
> 技术研发中心 许新浩 2016/10/9 

* 查看存储池：
	
		ceph osd lspools  
或者 
			
		rados lspools
		
* 创建存储池：

		ceph osd pool create {pool-name} {pg-num} [{pgp-num}] [replicated] [crush-ruleset-name] [expected-num-objects]
		
		其中： pg-num默认是8
			   少于 5 个 OSD 时可把 pg_num 设置为 128
			   OSD 数量在 5 到 10 个时，可把 pg_num 设置为 512
			   OSD 数量在 10 到 50 个时，可把 pg_num 设置为 4096
		
	eg:  

		ceph osd pool create wfpool 256

* 设置每存储池最大对象数：

		ceph osd pool set-quota {pool-name} [max_objects {obj-count}] [max_bytes {bytes}]
		
	eg:  

		ceph osd pool set-quota wfpool max_objects 20 
	
* 查看存储池的对象和字节限制：

		ceph osd pool get-quota <poolname>
		
	eg：  

		ceph osd pool get-quota wfpool
	
* 删除一存储池:

		ceph osd pool delete {pool-name} [{pool-name} --yes-i-really-really-mean-it]
		
	eg：  

		ceph osd pool delete wfpool wfpool --yes-i-really-really-mean-it
	
* 重命名一个存储池:
	
		ceph osd pool rename {current-pool-name} {new-pool-name}
		
	eg:  

		ceph osd pool rename wfpool wfpool2
	
* 查看所有存储池的使用信息列表:

		ceph df
* 显示集群中pool的详细信息

		rados df
		
* 拍下某存储池的快照：

		ceph osd pool mksnap {pool-name} {snap-name}
		
	eg:  

		ceph osd pool mksnap wfpool wfplsnap
	
* 查看快照：

		rados -p {pool-name} lssnap
		
	eg:   

		rados -p wfpool2 lssnap
	
* 删除存储池快照:

		ceph osd pool rmsnap {pool-name} {snap-name}
		
	eg：  

		ceph osd pool rmsnap wfpool wfplsnap
	
* 调整存储池选项:

		ceph osd pool set {pool-name} {key} {value}
		
	eg： 

		ceph osd pool set wfpool pg_num 512

* 获取存储池选项:

		ceph osd pool get {pool-name} {key}
		
	eg:  

		ceph osd pool get wfpool pg_num
	
* 把对象存入存储池：

		rados put {object-name} {file-path} -p{pool-name}
	
	eg： 

		rados put wfobj2 createOSD.sh -p wfpool2
	
* 查看已存入的对象：

		rados ls -p {pool-name}
		
	eg:  

		rados ls -p wfpool2

* 查看对象内容：

		rados -p {pool-name} get {object-name} -
		
	eg:  

		rados -p wfpool get wfobj -
	
* 定位对象:

		ceph osd map {pool-name} {object-name}
		
	eg：  

		ceph osd map wfpool2 wfobj2  会输出对象的位置
	
* 删除对象：

		rados rm {object-name} -p {pool-name}
		
	eg: 

		rados rm wfobj2 -p wfpool2
	
* 查看归置组状态：

		ceph pg stat