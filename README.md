
Aerospike(以下简称AS）是一个以分布式为核心基础，可基于行随机存取内存中索引、数据或SSD存储中数据的数据库。它主要用于百G、数T等大数据量并且在数万以上高并发情况下，对性能也有ms读取插入要求的场景。目前主要集中于互联网广告行业，如eXelate、BlueKai、MediaV、 InMobi、 applovin等。

# Aerospike

## Aerospike server install

- 安装服务端可以直接使用dockerhub上的aerospike镜像，参照  https://hub.docker.com/_/aerospike/
- 启动之前需要在aerospike.conf文件中预定义好namespace。
- 启动的时候，请把aerospike.conf文件network中的mesh-port注释掉。

## Aerospike Tools -- Aerospike Query Language 

1. 在aerospike官网 下载tools--> aerospike-tools-3.9.1-ubuntu14.04。 然后执行  
    
    ```bash
        ./asinstall
    ```
2. 安装好了之后 就可以用aql去连接aerospike server：

    ```bash
        aql -h [aerospike server ip] -p [port default 3000]
    ```

3. aerospike是key-value存储的，在网表里写数据的时候 一定要有一个Key。

    ```sql
        insert into test.demoset (PK, bin1) values ('key', 'value1')
    ```

## Aerospike Client -- java

1. 从github上下载 aerospike client java的代码：然后用maven构建一下就可以使用。

    ```bash
        mvn clean install
    ```

2. 工程中有一个example模块，有一个GUI 主类是Main.class可以调试 aerospike client java api

## Aerospike clustering

Aerospike Clustering主要是修改aerospike.conf文件中的network部分中的service和heartbeat部分：

1. access-address 是aerospike server 暴露给集群中其他node的ip，通常是本机ip，如果是container的话，启动的时候需要加上参数 --net=host

2. mesh-seed-address-port 是集群中其他aerospike server 的ip 和端口 信息。

3. cluster中的每个节点上都会备份其他节点上的数据信息。即使摸个节点挂了，在其他节点上还可以查询到之前的数据。aerospike.conf例子如下：
    
        # Aerospike database configuration file.
        # This stanza must come first.
        service {
	        user root
	        group root
	        paxos-single-replica-limit 1 # Number of nodes where the replica count is automatically reduced to 1.
	        pidfile /var/run/aerospike/asd.pid
	        service-threads 4
	        transaction-queues 4
	        transaction-threads-per-queue 4
	        proto-fd-max 15000
        }
        logging {
	         # Log file must be an absolute path.
	         file /var/log/aerospike/aerospike.log {
		         context any info
	         }
	        # Send log messages to stdout
	        console {
		          context any critical
	        }
        }
        network {
	             service {
		                  address any
		                  port 3000
		                  # Uncomment the following to set the `access-address` parameter to the
		                  # IP address of the Docker host. This will the allow the server to correctly
		                  # publish the address which applications and other nodes in the cluster to
		                  # use when addressing this node.
		                  access-address 10.62.57.239 # IP Address to be used by applications and other nodes in the cluster.
	             }
	            heartbeat {
		              # mesh is used for environments that do not support multicast
		
		              mode mesh
		              port 3002
                      mesh-seed-address-port 10.62.57.240 3002 # the other node ip and port in the cluster.
		            # use asinfo -v 'tip:host=<ADDR>;port=3002' to inform cluster of
		            # other mesh nodes
		
		            # mesh-port 3002
		            interval 150
		            timeout 10
	            }
	           fabric {
		              port 3001
	           }
	           info {
		             port 3003
	           }
         }
         namespace test {
	             replication-factor 2
	             memory-size 1G
	             default-ttl 5d # 5 days, use 0 to never expire/evict.
	             #	storage-engine memory
	             # To use file storage backing, comment out the line above and use the
	             # following lines instead.
	             storage-engine device {
		                 file /opt/aerospike/data/test.dat
		                 filesize 4G
		                 data-in-memory true # Store data in memory in addition to file.
	             }
          }
          namespace moon {
	               replication-factor 2
	               memory-size 1G
	               default-ttl 5d # 5 days, use 0 to never expire/evict.
	               #	storage-engine memory
	               # To use file storage backing, comment out the line above and use the
	               # following lines instead.
 	                 storage-engine device {
	         	          file /opt/aerospike/data/moon.dat
		                  filesize 4G
		                  data-in-memory true # Store data in memory in addition to file.
	                 }
           }
