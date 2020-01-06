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

mkdir -r /root/fastdfs/fastdfs_tracker
```

- tarcker.conf

```
disabled=false  #默认开启 
port=22122      #默认端口号 
base_path=/root/fastdfs/fastdfs_tracker #我刚刚创建的目录 
http.server_port=8081   #默认端口是8080
```
- start

```
service fdfs_trackerd start
```
