# fastdfs-guide

## install and complie

- install source

```
git clone https://github.com/happyfish100/fastdfs.git
git clone https://github.com/happyfish100/libfastcommon.git
git clone https://github.com/happyfish100/fastdfs-nginx-module.git
```
- complie

```
./make.sh
./make.sh install
```
- config

```
cd /etc/hdfs
cp client.conf.sample client.conf
cp storage.conf.sample storage.conf
cp tracker.conf.sample tracker.conf

ln -s /usr/lib64/libfastcommon.so /usr/local/lib/libfastcommon.so
ln -s /usr/lib64/libfastcommon.so /usr/lib/libfastcommon.so
ln -s /usr/lib64/libfdfsclient.so /usr/local/lib/libfdfsclient.so
ln -s /usr/lib64/libfdfsclient.so /usr/lib/libfdfsclient.so
ln -s /usr/bin/fdfs_storaged /usr/local/bin


mkdir -r /root/fastdfs/fastdfs_tracker
mkdir -r /root/fastdfs/fastdfs_storage_data
mkdir -r /root/fastdfs/fastdfs_storage
```

- tarcker.conf

```
disabled=false  #默认开启 
port=22122      #默认端口号 
base_path=/root/fastdfs/fastdfs_tracker #我刚刚创建的目录 
http.server_port=8081   #默认端口是8080
```

- storage.conf

```
disabled=false 
group_name=group1 #组名，根据实际情况修改 
port=23000 #设置storage的端口号，默认是23000，同一个组的storage端口号必须一致 
base_path=/root/fastdfs/fastdfs_storage #设置storage数据文件和日志目录
store_path_count=1 #存储路径个数，需要和store_path个数匹配 
base_path0=/root/fastdfs/fastdfs_storage_data #实际文件存储路径 
tracker_server=172.16.43.78:22122 #我CentOS7的ip地址 
http.server_port=8888 #设置 http 端口号
```

- start

```
service fdfs_trackerd start
service fdfs_storaged start
```
