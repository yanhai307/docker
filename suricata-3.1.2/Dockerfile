FROM debian:stretch

MAINTAINER yh@chinabluedon.cn

COPY lib /root/lib

RUN chmod +x /root/lib/* && \
    cp /root/lib/libhtp-0.5.22.so.1.0.0 /usr/local/lib/ && \
    cd /usr/local/lib/ && ln -s libhtp-0.5.22.so.1.0.0 libhtp-0.5.22.so.1 && \
    cp /root/lib/libhiredis.so.0.13 /usr/local/lib/ && \
    cp /root/lib/libmagic.so.1.0.0 /usr/lib/x86_64-linux-gnu/ && \
    cd /usr/lib/x86_64-linux-gnu/ && ln -s libmagic.so.1.0.0 libmagic.so.1 && \
    cp /root/lib/libiconv.so.2.2.0 /usr/local/lib/ && \
    cd /usr/local/lib/ && ln -s libiconv.so.2.2.0 libiconv.so.2 && \
    cp /root/lib/libpcap.so.1.6.2 /usr/lib/x86_64-linux-gnu/ && \
    cd /usr/lib/x86_64-linux-gnu/ && ln -s libpcap.so.1.6.2 libpcap.so.0.8 && \
    cp /root/lib/libnet.so.1.7.0 /usr/lib/x86_64-linux-gnu/ && \
    cd /usr/lib/x86_64-linux-gnu/ && ln -s libnet.so.1.7.0 libnet.so.1 && \
    cp /root/lib/libjansson.so.4.9.0 /usr/local/lib/ && \
    cd /usr/local/lib/ && ln -s libjansson.so.4.9.0 libjansson.so.4 && \
    cp /root/lib/libyaml-0.so.2.0.4 /usr/lib/x86_64-linux-gnu/ && \
    cd /usr/lib/x86_64-linux-gnu/ && ln -s libyaml-0.so.2.0.4 libyaml-0.so.2 && \
    cp /root/lib/libpcre.so.1.2.7 /usr/local/lib/ && \
    cd /usr/local/lib/ && ln -s libpcre.so.1.2.7 libpcre.so.1 && \
    cp /root/lib/libhs.so.4.3.1 /usr/local/lib/ && \
    cd /usr/local/lib/ && ln -s libhs.so.4.3.1 libhs.so.4 && \
    ldconfig && rm -rf /root/lib

# /etc/magic, 0: Warning: using regular magic file `/usr/share/misc/magic'
COPY sources.list /etc/apt/
RUN apt update && apt install -y libmagic1 && rm -rf /var/lib/apt/lists/*

CMD ["bash"]
