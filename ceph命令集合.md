#Ceph 命令集合
> 技术研发中心 许新浩 2016/10/10
##一、集群
1. 启动一个  ceph  进程
	
	启动 mon 进程

		systemctl restart ceph-mon@{host}.service
	启动 msd 进程

		systemctl restart ceph-mds.target
	启动 osd 进程
	
		systemctl restart ceph-osd@{num}.service
2. 查看集群的监控状态
		
		ceph health

3. 查看ceph的实时运行状态
	
		ceph -w

4. 检查信息状态信息
		
		ceph -s


5. 查看ceph存储空间

		ceph df

6. 查看ceph集群中的认证用户及相关的key

		ceph auth list
7. 删除集群中的一个认证用户
		
		ceph auth del osd.0
##二、monitor

1. 删除一个mon节点

		ceph mon remove node1

2. 获得一个正在运行的mon map，并保存在1.txt文件中

		ceph mon getmap -o 1.txt
	>查看上面获得的map
	>       
	>     monmaptool --print 1.txt

	>把上面的mon map 注入新加入的节点
	>
	>     ceph-mon -i node4 --inject-monmap 1.txt

3. 查看mon的详细状态

		ceph daemon mon.node1 mon_status

4. 删除一个mon节点

		ceph mon remove os-node1
5. 查看mon的选举状态

		ceph quorum_status
6. 查看mon的映射信息

		ceph mon dump
##三、osd

1. 查看osd映射信息

		ceph osd dump

2. 查看osd的目录树

		ceph osd tree

3. down掉一个osd硬盘

		ceph osd down 0  #down 掉 osd.0 节点
4. 在集群中删除一个osd硬盘

		ceph osd rm 0

5. 在集群中删除一个osd硬盘crush map

		ceph osd crush rm osd.0
6. 在集群中删除一个osd的host节点

		ceph osd crush rm node1

7. 查看最大 osd 的个数

		ceph osd getmaxosd

8. 设置最大的osd的个数 （当扩大osd节点的时候必须扩大这个值）

		ceph osd setmaxosd N
9. 设置osd crush的权重为  1.0
	
		ceph osd crush set {id} {weight} [{location1} 
	eg:
	
		[root@spark5 ~]# ceph osd crush set 3 3.0 host=spark3
		set item id 3 name 'osd.3' weight 3 at location {host=spark3} to crush map
		[root@spark5 ~]# ceph osd tree
		ID WEIGHT   TYPE NAME       UP/DOWN REWEIGHT PRIMARY-AFFINITY 
		-1 23.00346 root default                                      
		-2  8.45549     host spark3                                   
		 0  1.81850         osd.0        up  1.00000          1.00000 
		 1  1.81850         osd.1        up  1.00000          1.00000 
		 2  1.81850         osd.2        up  1.00000          1.00000 
		 3  3.00000         osd.3        up  1.00000          1.00000 
		-3  7.27399     host spark4                                   
		 4  1.81850         osd.4        up  1.00000          1.00000 
		 5  1.81850         osd.5        up  1.00000          1.00000 
		 6  1.81850         osd.6        up  1.00000          1.00000 
		 7  1.81850         osd.7        up  1.00000          1.00000 
		-4  7.27399     host spark5                                   
		 8  1.81850         osd.8        up  1.00000          1.00000 
		 9  1.81850         osd.9        up  1.00000          1.00000 
		10  1.81850         osd.10       up  1.00000          1.00000 
		11  1.81850         osd.11       up  1.00000          1.00000


	>或者

		[root@spark5 ~]# ceph osd crush reweight osd.3 1.81850
		reweighted item id 3 name 'osd.3' to 1.8185 in crush map
		[root@spark5 ~]# ceph osd tree
		ID WEIGHT   TYPE NAME       UP/DOWN REWEIGHT PRIMARY-AFFINITY 
		-1 21.82196 root default                                      
		-2  7.27399     host spark3                                   
		 0  1.81850         osd.0        up  1.00000          1.00000 
		 1  1.81850         osd.1        up  1.00000          1.00000 
		 2  1.81850         osd.2        up  1.00000          1.00000 
		 3  1.81850         osd.3        up  1.00000          1.00000 
		-3  7.27399     host spark4                                   
		 4  1.81850         osd.4        up  1.00000          1.00000 
		 5  1.81850         osd.5        up  1.00000          1.00000 
		 6  1.81850         osd.6        up  1.00000          1.00000 
		 7  1.81850         osd.7        up  1.00000          1.00000 
		-4  7.27399     host spark5                                   
		 8  1.81850         osd.8        up  1.00000          1.00000 
		 9  1.81850         osd.9        up  1.00000          1.00000 
		10  1.81850         osd.10       up  1.00000          1.00000 
		11  1.81850         osd.11       up  1.00000          1.00000 

10. 设置osd的权重

		osd reweight <int[0-]> <float[0.0-1.0]>
eg:

		[root@spark5 ~]# ceph osd reweight 3 0.5
		reweighted osd.3 to 0.5 (8000)
		[root@spark5 ~]# ceph osd tree
		ID WEIGHT   TYPE NAME       UP/DOWN REWEIGHT PRIMARY-AFFINITY 
		-1 21.82196 root default                                      
		-2  7.27399     host spark3                                   
		 0  1.81850         osd.0        up  1.00000          1.00000 
		 1  1.81850         osd.1        up  1.00000          1.00000 
		 2  1.81850         osd.2        up  1.00000          1.00000 
		 3  1.81850         osd.3        up  0.50000          1.00000 
		-3  7.27399     host spark4                                   
		 4  1.81850         osd.4        up  1.00000          1.00000 
		 5  1.81850         osd.5        up  1.00000          1.00000 
		 6  1.81850         osd.6        up  1.00000          1.00000 
		 7  1.81850         osd.7        up  1.00000          1.00000 
		-4  7.27399     host spark5                                   
		 8  1.81850         osd.8        up  1.00000          1.00000 
		 9  1.81850         osd.9        up  1.00000          1.00000 
		10  1.81850         osd.10       up  1.00000          1.00000 
		11  1.81850         osd.11       up  1.00000          1.00000 
		reweighted osd.3 to 0.5 (8327682)

11. 把一个osd节点逐出集群
	
		ceph osd out osd.3
	> osd.3 的 reweight 变为 0 了就不再分配数据，但是设备还是存活的

12. 把逐出的osd加入集群

		ceph osd in osd.3

13. 暂停osd （暂停后整个集群不再接收数据）

		ceph osd pause

14. 再次开启osd （开启后再次接收数据）

		ceph osd unpause

##四、PG组
1. 查看pg组的映射信息

		ceph pg dump

2. 查看一个PG的 map

		ceph pg map {pgid}


3. 查看PG状态

		ceph pg stat

4. 查询一个pg的详细信息

	   ceph pg {pgid} query
5. 显示非正常状态的pg

		ceph pg dump_stuck {inactive|unclean|stale}

6. 显示一个集群中的所有的pg统计

		ceph pg dump --format plain
7. 恢复一个丢失的pg

		ceph pg {pg-id} mark_unfound_lost revert

