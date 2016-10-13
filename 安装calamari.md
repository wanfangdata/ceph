#安装calamari
>技术研发中心 许新浩 2016/10/10


	安装包列表：
	calamari-server-1.3.3-jewel.el7.centos.x86_64.rpm
	diamond-3.4.68-jewel.noarch.rpm
	romana-1.2.2-36_gc62bb5b.el7.centos.x86_64.rpm
	salt-2015.8.1-1.el7.noarch.rpm
	salt-master-2015.8.1-1.el7.noarch.rpm
	salt-minion-2015.8.1-1.el7.noarch.rpm
##一．安装calamari-server
这里由于salt版本不对会有bug,所以需要预先安装salt 2015.8.1：
	
	sudo rpm -ivh salt-2015.8.1-1.el7.noarch.rpm
	sudo rpm -ivh salt-master-2015.8.1-1.el7.noarch.rpm
	sudo rpm -ivh salt-minion-2015.8.1-1.el7.noarch.rpm
	sudo yum localinstall calamari-server-xxx
**注意：由于缺少依赖salt会安不上，这里先yum安装salt (参考第四步) 然后卸载掉，再安装salt**
##二．初始化calamari-server
 
	sudo calamari-ctl initialize 	执行改命令后需要输入用户名，密码 邮箱。
更改密码命令为：

	calamari-ctl change_password --password {password} {user-name}
##三．在所有节点安装diamond

	sudo rpm -ivh diamond-3.4.68-jewel.noarch.rpm
	sudo cp /etc/diamond/diamond.conf.example /etc/diamond/diamond.conf

修改 diamond.conf的 # Graphite server host ：

	sudo vi /etc/diamond/diamond.conf

	# Graphite server host
	host = admin-node      #calamari-server的那台机器的主机名，所有的需要收集数据的机器都需要修改

启动 diamond ： 

	sudo /etc/init.d/diamond restart
##四．在除了calamari-server的节点上安装salt-minion

	sudo vi /etc/yum.repos.d/repo-saltstack-el7.repo
	
	[repo-saltstack-el7]
	name=SaltStack EL7 Repo
	baseurl=https://repo.saltstack.com/yum/rhel7/
	skip_if_unavailable=True
	gpgcheck=0
	gpgkey=https://repo.saltstack.com/yum/rhel7/SALTSTACK-GPG-KEY.pub
	enabled=1
	enabled_metadata=1
	
	yum makecache
	
	sudo yum install salt-minion -y

启动服务(所有节点)：

	sudo systemctl start salt-minion.service
在所有节点执行：

	sudo vi /etc/salt/minion.d/calamari.conf
写入：

	master: admin-node  （salt-master位置。注意空格）
重启服务(所有节点)：

	sudo systemctl restart salt-minion.service
##五．salt-master上获取认证并认可
查询当前的认证请求： 

	sudo salt-key -L
批准请求：	

	sudo salt-key -A
配置calamari-server机器的文件权限 ：

	cd /var/log/calamari/
	sudo chmod 777 -R *
	sudo systemctl restart supervisord.service
##六．安装romana（集群的管理界面）
在calamari-server节点上安装：

	sudo rpm -ivh romana-1.2.2-36_gc62bb5b.el7.centos.x86_64.rpm
##七．登录管理界面端口80
>http://spark4:80