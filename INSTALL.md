# 安装
## 环境
gonet2全部在linux + mac环境中开发，确保能在ubuntu 14.04 运行，理论上主流linux都能运行。      
开发工具链可以访问[TOOLCHAIN.md](TOOLCHAIN.md)     

## 基础设施
1. http://nsq.io/        
2. https://github.com/coreos/etcd       
3. https://www.docker.com/    
4. https://github.com/pote/gvp
5. https://github.com/pote/gpm

请预先安装好上述环境，并确保172.17.42.1是容器可访问地址，所有基础设施都应该监听这个地址， 如mongodb, nsq, etcd

## 框架
执行克隆:       

     curl -s https://raw.githubusercontent.com/gonet2/tools/master/clone_all.sh | sh      


## 启动顺序[base_service.sh](base_service.sh)     
	1. 启动基础设施
		1. nsq
		nsqlookup
			nsqlookupd --tcp-address=172.17.42.1:4160 --http-address=172.17.42.1:4161 &
		nsqd
			nsqd --lookupd-tcp-address=172.17.42.1:4160 --tcp-address=172.17.42.1:4150 --http-address=172.17.42.1:4151 &
		nsqadmin
			nsqadmin --lookupd-http-address=172.17.42.1:4161 --http-address=172.17.42.1:4171 &
		2. etcd
			etcd &
		3. gliderlabs/registrator
			docker run -d -v /var/run/docker.sock:/tmp/docker.sock gliderlabs/registrator -ip="<red>public_ip_that_all_services_can_access</red>" etcd://172.17.42.1:2379/backends
		
	2. 启动各个服务
		1. docker中运行：所有服务运行在docker中，并通过registrator自动注册；
		snowflake, auth, game, ...
		如snowflake:
			cd snowflake
			docker build -t snowflake
			docker run -d --name snowflake -e SERVICE_ID=snowflake1 -P snowflake

		2. 如果需要手动注册或不使用docker, 则需要自己把服务注册进etcd server, 格式为： /backends/SERVICE_NAME/SERVICE_ID 
		如snowflake:
			$cd snowflake
			$source gvp
			$gpm
			$go install agent
			$./startup.sh
			$etcdctl set /backends/snowflake/snowflake1 172.17.42.1:51006

	3. 启动agent[agent不需要在docker中运行]
	    $cd agent
	    $source gvp
	    $gpm
	    $go install agent
	    $./startup.sh

## 工具安装
	1.tailn 查看所有服务的日志
		go get https://github.com/gonet2/tools/tailn
		tailn
	
	2. upload_numbers 上传配置文件到etcd(以逗号分割的csv文件)
		go get https://github.com/gonet2/tools/upload_numbers
		upload_numbers numbers --addr http://172.17.42.1:4001 --dir ~/gonet2/gamedata --pattern="/*.csv"
	