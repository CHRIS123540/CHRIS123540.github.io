---
sort: 5
---


# 记录一些实验过程中的常用命令

***网卡工具***
```
lshw -class network
ethtool
```
***巨页***
```
echo 1024 > /sys/kernel/mm/hugepages/hugepages-2048kB/nr_hugepages
lsb_release -a 显示系统
```

***notice***
```
太网帧尾部的帧校验序列（Frame Check Sequence, FCS）4 字节
```

***SF***
```
/opt/mellanox/iproute2/sbin/mlxdevm port add pci/0000:03:00.1 flavour pcisf pfnum 1 sfnum 4
/opt/mellanox/iproute2/sbin/mlxdevm port function set pci/0000:03:00.1/294945 hw_addr 00:00:00:00:04:0 trust on state active

```


***OVS操作***
```
cq dpu
pci 03:00.1上两个sf
en3f1pf1sf5
en3f1pf1sf6
打印现有网桥
ovs-vsctl show
删除网桥
ovs-vsctl del-br hlx
增加网桥
ovs-vsctl add-br hlx_bridge1
删除端口
ovs-vsctl del-port bridge1 p1
增加端口
ovs-vsctl add-port bridge2 p1
查看、增加、删除流表
ovs-ofctl dump-flows ovsbr2

sudo ovs-ofctl del-flows ovsbr1
sudo ovs-ofctl add-flow ovsbr1 "in_port=p0 action=pf0hpf"
sudo ovs-ofctl add-flow ovsbr1 "in_port=pf0hpf action=p0"

sudo ovs-ofctl add-flow ovsbr2 "in_port=p1 action=en3f1pf1sf0"
sudo ovs-ofctl add-flow ovsbr2 "in_port=en3f1pf1sf0 action=p1"
```


***pktgen简单使用***
```
启动
/home/markchen/markchen/work/SPLB/dpdk/pktgen-dpdk-pktgen-21.11.0/build/app/pktgen -a 01:00.1 -- -m [1:2].0 -P
yy->cq

set 0 dst ip 192.168.201.2
set 0 src ip 192.168.201.1/24
set 0 dst mac 08:c0:eb:bf:ef:9b
set 0 size 1000
start 0

set 1 dst ip 192.168.0.1
set 1 src ip 192.168.204.101/24
set 1 dst mac a0:88:c2:32:06:6a
set 1 size 1000
start 1

enable 1 range
range 1 size 1086 1086 1086 0
range 1 dst ip 192.168.200.200 192.168.200.200 192.168.200.200 0.0.0.0
range 1 src port 1 2 3 1 
range 1 dst port 5001 5001 5001 0
range 1 dst mac  b8:ce:f6:d5:d6:f8 b8:ce:f6:d5:d6:f8 b8:ce:f6:d5:d6:f8 00:00:00:00:00:00
start 1

也可以部署脚本，推荐pkt不是lua，因为简单


```




***命名管道的同步和阻塞***
```
阻塞行为：在标准的命名管道中，如果管道的读取端（consumer）没有打开，写入端（producer）在尝试写入数据时会被阻塞。同样地，如果写入端没有打开，尝试从管道读取数据的读取端也会被阻塞。

同步启动：因此，通常需要同时运行管道的两端以避免阻塞。在实际应用中，通常先启动读取端，然后再启动写入端。
```


[DPU上安装DPDK]("https://www.cnblogs.com/JoshuaYu/p/17612711.html")

***控制版本***
```
DPDK
pkg-config --modversion libdpdk

选择使用哪个dpdk，把需要的路径加入到其中
export PKG_CONFIG_PATH=/usr/local/lib/x86_64-linux-gnu/pkgconfig:$PKG_CONFIG_PATH
```
```
https://doc.dpdk.org/guides/gpus/cuda.html
./bashrc中加入cuda的路径，包括头文件
export PATH="/usr/local/cuda-11.7/bin:$PATH"
export LD_LIBRARY_PATH="/usr/local/cuda-11.7/lib64:$LD_LIBRARY_PATH"
export C_INCLUDE_PATH=/usr/local/cuda/include:$C_INCLUDE_PATH
export C_INCLUDE_PATH=/opt/mellanox/gdrcopy/include:$C_INCLUDE_PATH
export CPLUS_INCLUDE_PATH=/usr/local/cuda/include:$CPLUS_INCLUDE_PATH
export LD_LIBRARY_PATH=${LD_LIBRARY_PATH}:/opt/mellanox/gdrcopy/src
gdrdrv kernel module is active and running on the system
nvidia-peermem kernel module is active and running on the system
```
