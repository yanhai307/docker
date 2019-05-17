
# 构建dpdk运行环境
## 编写Dockerfile文件

```dockerfile
FROM debian:stretch

MAINTAINER yanhai<yh@chinabluedon.cn>

COPY sources.list /etc/apt/
COPY dpdk-18.11.tar.xz /opt/

# 解压xz格式的文件，需要先安装xz-utils
RUN apt-get update && \
    apt-get install -y xz-utils && \
    apt-get install -y gcc make && \
    apt-get install -y libnuma-dev && \
    apt-get install -y libpcap-dev && \
    rm -rf /var/lib/apt/lists/*

# 关闭掉一些编译开关，因为我们不需要这些功能，打开会由于某些原因编译失败
# 如果使能PMD_PCAP，需要先安装libpcap开发包
RUN cd /opt && tar -xf dpdk-18.11.tar.xz && \
    cd dpdk-18.11 && \
    export RTE_SDK=/opt/dpdk-18.11 && \
    export RTE_TARGET=x86_64-native-linuxapp-gcc && \
    sed -i 's/CONFIG_RTE_EAL_IGB_UIO=y/CONFIG_RTE_EAL_IGB_UIO=n/' config/common_linuxapp && \
    sed -i 's/CONFIG_RTE_LIBRTE_KNI=y/CONFIG_RTE_LIBRTE_KNI=n/' config/common_linuxapp && \
    sed -i 's/CONFIG_RTE_KNI_KMOD=y/CONFIG_RTE_KNI_KMOD=n/' config/common_linuxapp && \
    sed -i 's/CONFIG_RTE_APP_TEST=y/CONFIG_RTE_APP_TEST=n/' config/common_base && \
    sed -i 's/CONFIG_RTE_TEST_PMD=y/CONFIG_RTE_TEST_PMD=n/' config/common_base && \
    sed -i 's/CONFIG_RTE_EAL_IGB_UIO=y/CONFIG_RTE_EAL_IGB_UIO=n/' config/common_base && \
    sed -i 's/CONFIG_RTE_LIBRTE_IGB_PMD=y/CONFIG_RTE_LIBRTE_IGB_PMD=n/' config/common_base && \
    sed -i 's/CONFIG_RTE_LIBRTE_IXGBE_PMD=y/CONFIG_RTE_LIBRTE_IXGBE_PMD=n/' config/common_base && \
    sed -i 's/CONFIG_RTE_LIBRTE_I40E_PMD=y/CONFIG_RTE_LIBRTE_I40E_PMD=n/' config/common_base && \
    sed -ri 's,(PMD_PCAP=).*,\1y,' config/common_base && \
    make config T=$RTE_TARGET && \
    make -j4 install T=$RTE_TARGET DESTDIR=/usr/local && \
    cd / && \
    rm -rf /opt/dpdk-18.11.tar.xz && \
    rm -rf /opt/dpdk-18.11

```

## build

我们将镜像命名为`dpdk-dev`

    docker build -t dpdk-dev .

# 测试
## 首先在宿主机配置dpdk环境
前提，在宿主机上已经编译了dpdk源码，在`/opt/dpdk-18.11/`下
### 安装igb驱动

    modprobe uio
    insmod /opt/dpdk-18.11/x86_64-native-linuxapp-gcc/kmod/igb_uio.ko

### 配置大页内存

    mkdir -p /mnt/huge
    mount -t hugetlbfs nodev /mnt/huge

```bash
# 分配系统总内存的4分之1用作2MB大页内存
total_mem=`cat /proc/meminfo | grep MemTotal | awk '{print $2}'`
if [[ -d /sys/devices/system/node/node1 ]]; then
    hugepages_mem=$(($total_mem/8000000*1024/2))
    echo $hugepages_mem > /sys/devices/system/node/node0/hugepages/hugepages-2048kB/nr_hugepages
    echo $hugepages_mem > /sys/devices/system/node/node1/hugepages/hugepages-2048kB/nr_hugepages
else
    hugepages_mem=$(($total_mem/8000000*1024))
    echo $hugepages_mem > /sys/devices/system/node/node0/hugepages/hugepages-2048kB/nr_hugepages
fi
```

## 创建容器

    docker run -it -v /sys/bus/pci/devices:/sys/bus/pci/devices -v /sys/kernel/mm/hugepages:/sys/kernel/mm/hugepages -v /sys/devices/system/node:/sys/devices/system/node -v /dev:/dev dpdk-dev

## 进入examples目录
这里编译运行helloworld程序
```bash
export RTE_SDK=/opt/dpdk-18.11
export RTE_TARGET=x86_64-native-linuxapp-gcc
cd /opt/dpdk-18.11/examples/helloworld
make
./build/app/helloworld
```

# 案例
现在有这样一个需求，在宿主机上运行一个dpdk的读取网卡数据包的一个进程（相当于生产者），将收到的数据包放入一个队列(rx_queue)，
在docker里面有一个相当于消费者的进程，读取rx_queue中的数据包进行分析处理，处理完毕后放入tx_queue队列，生产者从tx_queue取出数据包，
再回收内存（放入包池）。

那么如何让docker内外的2个进程共享dpdk队列呢？

这种需求是可以实现的，

在创建容器时，除了共享如上面测试helloworld程序时的`/sys/bus/pci/devices`、`/sys/kernel/mm/hugepages`、
`/sys/devices/system/node`、`/dev`外，还需要共享`/var/run/dpdk/`目录

即

    -v /sys/bus/pci/devices:/sys/bus/pci/devices \
    -v /sys/kernel/mm/hugepages:/sys/kernel/mm/hugepages \
    -v /sys/devices/system/node:/sys/devices/system/node \
    -v /dev:/dev \
    -v /var/run/dpdk:/var/run/dpdk

`/var/run/dpdk/`目录存放了dpdk创建的ring mempool等其他的一些信息

## 注意
本例使用的是dpdk-18.11版本，共享/var/run/dpdk/目录就可以了

在dpdk-stable-16.11.1版本中，配置信息是存放在`/var/run/.rte_config`和`/var/run/.rte_hugepage_info`目录的  
因此在dpdk-stable-16.11.1版本中需要共享

    -v /sys/bus/pci/devices:/sys/bus/pci/devices \
    -v /sys/kernel/mm/hugepages:/sys/kernel/mm/hugepages \
    -v /sys/devices/system/node:/sys/devices/system/node \
    -v /dev:/dev \
    -v /var/run/.rte_config:/var/run/.rte_config \
    -v /var/run/.rte_hugepage_info:/var/run/.rte_hugepage_info

# 参考
- [在Docker中运行DPDK](https://blog.csdn.net/nachtz/article/details/52832956)
