
### Percona XtraDB Cluster + Haproxy on Ubuntu 16.04
B1: Installing PXC
Cài đặt các gói cần thiết và update :
```
wget https://repo.percona.com/apt/percona-release_0.1-4.xenial_all.deb
sudo dpkg -i percona-release_0.1-4.xenial_all.deb
sudo apt-get update
```
Cài đặt Percona XtraDB 
```
sudo apt-get install -y percona-xtradb-cluster-57
```
B2; Tiến hành cấu hình trên các node
Trước khi cài đặt ta cần stop mysql trên cả 3 node
```
/etc/init.d/mysql stop
```
Cấu hình trên node 1:
Cấu hình trong file  **/etc/mysql/percona-xtradb-cluster.conf.d/wsrep.cnf** với nội dung như sau: 


```
[mysqld]
# Path to Galera library
wsrep_provider=/usr/lib/galera3/libgalera_smm.so

# Cluster connection URL contains IPs of nodes
#If no IP is found, this implies that a new cluster needs to be created,
#in order to do that you need to bootstrap this node
wsrep_cluster_address=gcomm://192.168.1.128,192.168.1.133,192.168.1.134

# In order for Galera to work correctly binlog format should be ROW
binlog_format=ROW

# MyISAM storage engine has only experimental support
default_storage_engine=InnoDB

# Slave thread to use
wsrep_slave_threads= 8

wsrep_log_conflicts

# This changes how InnoDB autoincrement locks are managed and is a requirement for Galera
innodb_autoinc_lock_mode=2

# Node IP address
wsrep_node_address=192.168.1.128
# Cluster name
wsrep_cluster_name=pxc-cluster

#If wsrep_node_name is not specified,  then system hostname will be used
wsrep_node_name=pxc-cluster-node-1

#pxc_strict_mode allowed values: DISABLED,PERMISSIVE,ENFORCING,MASTER
pxc_strict_mode=ENFORCING

# SST method
wsrep_sst_method=xtrabackup-v2

#Authentication for SST method
wsrep_sst_auth="vinhbv:vinhbv123"
```

Start node đầu tiên với lệnh:
```
/etc/init.d/mysql bootstrap-pxc
```

```
mysql> CREATE USER 'vinhbv'@'localhost' IDENTIFIED BY 'vccloud';
mysql> GRANT PROCESS, RELOAD, LOCK TABLES, REPLICATION CLIENT ON *.* TO 'vinhbv'@'localhost';
mysql> FLUSH PRIVILEGES;
mysql> exit;
```

Trên Node DB2:
Cấu hình tương tự Node 1 trong file /etc/mysql/percona-xtradb-cluster.conf.d/wsrep.cnf và chỉ sửa dòng:
```
wsrep_node_addres=192.168.1.133
```
Khởi động MySQL lên:
```
/etc/init.d/mysql start
```

Trên Node DB3:
Cấu hình tương tự Node 1 trong file /etc/mysql/percona-xtradb-cluster.conf.d/wsrep.cnf và chỉ sửa dòng

```
wsrep_node_addres=192.168.1.134

```

Khởi động MySQL lên:
```
/etc/init.d/mysql start
```
3. Test replication
Để kiểm tra replication, chúng ta sẽ thử tạo 1 cơ sở dữ liệu mới trên node DB2, 1 bảng cho CSDL mới đó trên node DB3 và thêm một số bản ghi trên node DB1. Trên Node DB2: Ta tạo 1 CSDL mới





