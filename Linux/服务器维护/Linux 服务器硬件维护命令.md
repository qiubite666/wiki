### 1.SN号,品牌
> 设备SN

   dmidecode -t1	
   
    dmidecode | grep 'Product Name'
> 服务器品牌

    dmidecode|grep "System Information" -A9|egrep  "Manufacturer|Product" 	

### 2.硬盘
> 查看硬盘大小
    
    fdisk -l |sed s/,// |cut -d " " -f 2-4 |grep dev
> 查看硬盘UUID
    
    blkid

### 3.内存

> 查看内存或使用

    free -g
    cat /proc/meminfo |awk  'NR==1'|awk '{print $2}'
	
### 4.cpu

> 物理cpu颗数

    cat /proc/cpuinfo |grep "physical id" |sort|uniq|wc -l

> 查看每个物理CPU中core的个数(即核数)
    
    cat /proc/cpuinfo| grep "cpu cores"| uniq

> 查看物理CPU个数

    cat /proc/cpuinfo| grep "physical id"| sort| uniq| wc -l

> 查看CPU信息（型号）
    
    grep "model name" /proc/cpuinfo |awk -F ':' '{print $NF}'|uniq

### 5.网卡

> 查看网卡UUID
    
    nmcli -c

> 查看em1网卡(物理机) 速度和双工模式

    ethtool em1 |egrep 'Speed|Duplex'
> 查看em1网卡 RX下行流量，TX上行流量

    watch 'ethtool -S em1 |grep packets'
> 查询ethX网口基本设置

    ethtool em1  
> 查询ethX网口的相关信息

    ethtool -i em1

> 查询ethX网口注册性信息

    ethtool -d em1