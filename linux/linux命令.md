#### 

[TOC]

## 基础命令

### ls命令

作用:列出指定位置的文件和文件夹 

#### 参数

- **-a** :**列出所有的文件或者文件夹** 
- **-l** **列出详细信息** 
- **-h** **增加可读性** 
- **-R** **递归访问** 
- **-Q** **文件名用双引号包裹** 

### **echo** **命令** 

#### 参数

- **显示字符串**  echo hello world 或者 echo “hello world”
- **显示转义字符**  echo \” 或者 \’ \`
- **显示变量** echo $PATH
- **显示换行**  **/不换行**  echo -e “\n”换行 echo -e “\c”不换行 
- **显示原样字符串** echo '$PATH' 
- **显示命令结果** echo `data`

### **head** **命令** 

#### 参数

- **-c n** **显示头部的指定** **n** **个字符**

- **-n x** **显示头部的指定的** **x** **行** 

- **-v** **显示文件名** 

- **-q** **不显示文件名** 

  

## 网关路由

### 查看信息网关

```shell
root@ubuntu_server1:/opt/install/zk-dev# netstat -rn
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
0.0.0.0         192.168.3.1   0.0.0.0         UG        0 0          0 eno1
172.17.0.0      0.0.0.0         255.255.0.0     U         0 0          0 docker0
192.168.3.0     0.0.0.0         255.255.255.0   U         0 0          0 eno1
或者
root@ubuntu_server1:/opt/install/zk-dev# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.3.110   0.0.0.0         UG    0      0        0 eno1
172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0 docker0
192.168.3.0     0.0.0.0         255.255.255.0   U     0      0        0 eno1


```



### route命令将默认路由器设置为192.168.3.1

```shell
root@ubuntu_server1:/opt/install/zk-dev# route add default gw 192.168.3.1
root@ubuntu_server1:/opt/install/zk-dev# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.3.1     0.0.0.0         UG    0      0        0 eno1
0.0.0.0         192.168.3.110   0.0.0.0         UG    0      0        0 eno1
172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0 docker0
192.168.3.0     0.0.0.0         255.255.255.0   U     0      0        0 eno1
```



### ip命令将默认路由器设置为192.168.3.1

```shell
 ip route add default via 192.168.1.254
```



### ubuntu 防火墙

1. 防火墙的打开 **sudo ufw enable**
2. 防火墙的重启 **sudo ufw reload**
3. 打开想要的端口（以9000为例）**ufw allow 9000**
4. .查看本机端口使用情况 **ufw status**

