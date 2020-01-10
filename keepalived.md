
## install

```shell
wget https://www.keepalived.org/software/keepalived-2.0.19.tar.gz
tar -zxvf keepalived-2.0.19.tar.gz
cd /usr/local/keepalived
yum -y install openssl-devel
yum install -y libnl3-devel.x86_64
make && make install
```

## config

- nginx_check

vim /etc/keepalived/nginx_check.sh

```
#!/bin/bash
A=`ps -C nginx –no-header |wc -l`
if [ $A -eq 0 ];then
    /usr/local/nginx/sbin/nginx
    sleep 2
    if [ `ps -C nginx --no-header |wc -l` -eq 0 ];then
        killall keepalived
    fi
fi
赋予执行权限
chmod +x /etc/keepalived/nginx_check.sh
```

- keepalived.conf

vim /etc/keepalived/keepalived.conf

```
! Configuration File for keepalived  
# 全局配置，配置收件人  
global_defs {  
   notification_email {               ##通知机制，收件人  
     110@qq.com  
   }  
   notification_email_from keepalived@domain.com ####发件人  
   smtp_server 192.168.1.115                     ##发件服务器  
   smtp_connect_timeout 30                       ##服务器连接超时时间  
   router_id LVS_DEVEL                           ##路由器标志  
}  
# 集群资源监控，组合track_script进行  
vrrp_script check_haproxy {  
script "/etc/keepalived/nginx_check.sh"  #检测 nginx 状态的脚本路径
interval 2  #检测时间间隔
weight -20  #条件成立 权重减20
}  
vrrp_instance HAPROXY_HA {  
# 设置当前主机为主节点，如果是备用节点，则设置为BACKUP 
state MASTER  
# 指定HA监测网络接口，可以用ifconfig查看来决定设置哪一个  
interface eno16777736  
# 虚拟路由标识，同一个VRRP实例要使用同一个标识，主备机  
virtual_router_id 80  
# 因为当前环境中VRRP组播有问题，改为使用单播发送VRRP报文  如果VRRP组播没问题，以下这块的内容可以注释掉。
# 这个地方需要关注，之前未做此设置，结果主备节点互相不能发现，因此主备节点都升级成了MASTER，并且绑定了VIP  
# 主节点时，内容为：  
#unicast_src_ip 192.168.1.115  
# unicast_peer {  
# 192.168.1.120 
#}  
# 设置优先级，确保主节点的优先级高过备用节点
priority 100  
# 用于设定主备节点间同步检查时间间隔  
advert_int 2  
# 设置高可用集群中不抢占功能，在主机down后，从机接管，当主机重新恢复后，设置此功能，备机将继续提供服务，从而避免因切换导致的隐患  
nopreempt  
# 设置主备节点间的通信验证类型及密码，同一个VRRP实例中需一致  
authentication {  
auth_type PASS  
auth_pass 1234  
}  
# 集群资源监控，组合vrrp_script进行  
track_script {  
check_haproxy  
}  
# 设置虚拟IP地址，当keepalived状态切换为MASTER时，此IP会自动添加到系统中  
# 当状态切换到BACKUP时，此IP会自动从系统中删除  
# 可以通过命令ip add查看切换后的状态  
virtual_ipaddress {  
192.168.1.155  #虚拟ip配置完之后就用它访问  
}  
}
```
- backup

```
! Configuration File for keepalived  
# 全局配置，配置收件人  
global_defs {  
   notification_email {               ##通知机制，收件人  
     110@qq.com  
   }  
   notification_email_from keepalived@domain.com ####发件人  
   smtp_server 192.168.1.120                     ##发件服务器  
   smtp_connect_timeout 30                       ##服务器连接超时时间  
   router_id LVS_DEVEL                           ##路由器标志  
}  
# 集群资源监控，组合track_script进行  
vrrp_script check_haproxy {  
script "/etc/keepalived/nginx_check.sh"  #检测 nginx 状态的脚本路径
interval 2  #检测时间间隔
weight -20  #条件成立 权重减20
}
vrrp_instance HAPROXY_HA {  
# 设置当前主机为主节点，如果是备用节点，则设置为BACKUP 
state BACKUP  
# 指定HA监测网络接口，可以用ifconfig查看来决定设置哪一个  
interface eno16777736  
# 虚拟路由标识，同一个VRRP实例要使用同一个标识，主备机  
virtual_router_id 80  
# 因为当前环境中VRRP组播有问题，改为使用单播发送VRRP报文  如果VRRP组播没问题，以下这块的内容可以注释掉。
# 这个地方需要关注，之前未做此设置，结果主备节点互相不能发现，因此主备节点都升级成了MASTER，并且绑定了VIP  
# 主节点时，内容为：  
#unicast_src_ip 192.168.1.120  
# unicast_peer {  
# 192.168.1.115 
#}  
# 设置优先级，确保主节点的优先级高过备用节点
priority 90  
# 用于设定主备节点间同步检查时间间隔  
advert_int 2  
# 设置主备节点间的通信验证类型及密码，同一个VRRP实例中需一致  
authentication {  
auth_type PASS  
auth_pass 1234  
}  
# 集群资源监控，组合vrrp_script进行  
track_script {  
check_haproxy  
}  
# 设置虚拟IP地址，当keepalived状态切换为MASTER时，此IP会自动添加到系统中  
# 当状态切换到BACKUP时，此IP会自动从系统中删除  
# 可以通过命令ip add查看切换后的状态  
virtual_ipaddress {  
192.168.1.155  #虚拟ip配置完之后就用它访问  
}  
}
```

