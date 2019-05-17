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
