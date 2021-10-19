# k8s-hw2
K8S Homework 2 (Module 3)


# 思考并讨论容器的劣势  by Harry Zhu


# 1. 与用户态/内核态/VM场景相比

1. 为了支持完整的cgroup/namespace实现，(LTS)内核版本必须为3.10以上，即CentOS7的默认内核，CentOS6及以下旧系统不再适用。相对于传统用户态应用，通常也无法正常在OpenVZ/LXC 正常完整运行。

2. 由于和宿主OS共用内核，一般情况下无法使用内核态应用(Wireguard,iptables(netfilter)),也不可能有单独的内核模块(modprobe)及参数设置。

3. 主流容器的通用实现强依赖于Linux内核特性(cgroup/namespace/UNIX socket/iptables)和开源生态，Windows容器是额外的独立生态且旧软件移植并不容易(例如Delphi程序)。而大多数种类的传统VM都能让不同内核的OS(Linux/Windows/FreeBSD/类Solaris)以简单的逻辑稳定共存。

4. 一般无法承载复杂的GUI(X window)应用，通常只用于服务端CLI应用。
 
5. 网络实现大多聚焦在四层或七层细粒度控制，即使是Calico和MacVLAN这样二/三层的CNI也未必能和虚拟机直通网卡的特性相比
-----
# 2. 隔离性及安全性

1. 没有内核级彻底隔离，提权漏洞时有发生
2. 默认情况下能从宿主机看到实际在容器中运行的进程名及参数
3. 容器应用的实质仍然基于进程，大规模落地时可能产生海量僵尸进程
-----

# 3. 有状态应用及存储读写
 1. 容器镜像基于Overlay只读层, Volume基于UnionFS,都不是传统的Linux文件系统(ext4,xfs),如不配置额外CSI插件或数据源，很难保证数据的持久和其它底层的文件系统特性。
 2. Openstack/KVM的生态一般都可以配置限制IO的高级参数，但容器若同时对宿主存储进行密集读写，会大幅降低性能，更可能故障。
 3. 不易恢复有状态应用(以数据库为代表)故障前的状态，需要更大规模且复杂的分布式算法(强一致，原子，选主机制)来保证。


-----
# 4. 便利性
1. Go是编译型语言，主流容器生态的Go二进制需要按CPU指令集分不同的版本，再小的executable，大小通常也是以MB为单位计。相比C++/MicroPython，对嵌入式/微控制器领域并不友好。
2. runc以上有多层设计，不同供应商的OCI容器引擎和shim的实现(Docker-cli工具及compose/containerd/CRI-O/Podman/iSula)所依赖和掌控到的层级并不相同，如果共存使用则Debug相当复杂
