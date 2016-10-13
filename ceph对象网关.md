#ceph对象网关
>技术研发中心 许新浩 2016/10/9 

* 安装ceph对象网关:
	
	在管理节点执行：
		
		ceph-deploy install --rgw  <gateway-node1> [<gateway-node2> ...]
	
	eg：

		ceph-deploy install --rgw spark3
	
	新建ceph对象网关实例(在管理节点的工作目录---myceph中):
	
		ceph-deploy rgw create <gateway-node1>
	
	eg: 

		ceph-deploy rgw create spark3

	一旦网关开始运行，你就可以通过 7480 端口来访问它（eg: **http://spark3:7480** ）
	
* 修改默认端口:
	
	修改 /etc/ceph/ceph.conf 在 [global] 节点后增加名为 [client.rgw.<client-node>] 的节点
	
	eg:
		
		[client.rgw.spark3]
		rgw_frontends = "civetweb port=80"
		
	修改 ~/myceph/ceph.conf 将该配置文件推送到你的 Ceph 对象网关节点(也包括其他 Ceph 节点):

		ceph-deploy --overwrite-conf config push <gateway-node> [<other-nodes>]
			
	把/etc/ceph/ceph.conf 覆盖工作目录下的ceph.conf
			
		ceph-deploy --overwrite-conf config pull {hostname}
		
* 重启对象网关：
		
		sudo systemctl restart ceph-radosgw.target
		
* 配置bucket最大分片数：

	在[global] 节点里增加：
			
		rgw_override_bucket_index_max_shards = {N}
			
			

			
		
		
		
		
		
		
		
		