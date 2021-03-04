ARG GLIBC_VERSION=2.32
ARG PREFIX_DIR=/usr/lib/glibc


FROM ubuntu:20.04 AS builder
ARG GLIBC_VERSION
ENV DEBIAN_FRONTEND noninteractive
ENV DEBCONF_NONINTERACTIVE_SEEN true
RUN echo "Acquire::Retries \"3\";" | tee -a /etc/apt/apt.conf
RUN sed -i 's/archive.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list && \
    sed -i 's/security.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list
RUN apt-get -q update && apt-get -y upgrade
RUN apt-get -y install bison build-essential gawk gettext openssl python3 texinfo wget
RUN wget -qO- "https://ftpmirror.gnu.org/libc/glibc-${GLIBC_VERSION}.tar.gz" | tar zxf -


FROM builder AS lib
ARG GLIBC_VERSION
ARG PREFIX_DIR
RUN mkdir -p /glibc-build && cd /glibc-build && \
    /glibc-${GLIBC_VERSION}/configure CFLAGS="-Os" CXXFLAGS="-Os" \
        --prefix="${PREFIX_DIR}" \
        --enable-stack-protector=strong && \
    make && make install
RUN sed -i '1c #!/bin/sh' ${PREFIX_DIR}/bin/ldd && \
    mkdir -p ${PREFIX_DIR}/lib/locale && ${PREFIX_DIR}/bin/localedef -c -i POSIX -f UTF-8 C.UTF-8 || true


FROM builder AS lib32
ARG GLIBC_VERSION
ARG PREFIX_DIR
RUN apt-get -y install gcc-multilib g++-multilib
RUN mkdir -p /glibc-build && cd /glibc-build && \
    /glibc-${GLIBC_VERSION}/configure BUILD_CC='gcc' CC='gcc -m32' CXX='g++ -m32' CFLAGS="-Os" CXXFLAGS="-Os" \
        --host=i686-linux-gnu \
        --prefix="${PREFIX_DIR}" \
        --enable-stack-protector=strong && \
    make && make install
RUN rm -rf ${PREFIX_DIR}/lib/audit && \
    rm -rf ${PREFIX_DIR}/lib/gconv && \
    rm -rf ${PREFIX_DIR}/lib/*.a && \
    rm -rf ${PREFIX_DIR}/lib/*.o


FROM builder AS lib64
ARG GLIBC_VERSION
ARG PREFIX_DIR
RUN apt-get -y install gcc-multilib g++-multilib
RUN mkdir -p /glibc-build && cd /glibc-build && \
    /glibc-${GLIBC_VERSION}/configure BUILD_CC='gcc' CC='gcc -m64' CXX='g++ -m64' CFLAGS="-Os" CXXFLAGS="-Os" \
        --prefix="${PREFIX_DIR}" \
        --enable-stack-protector=strong && \
    make && make install
RUN rm -rf ${PREFIX_DIR}/lib/audit && \
    rm -rf ${PREFIX_DIR}/lib/gconv && \
    rm -rf ${PREFIX_DIR}/lib/*.a && \
    rm -rf ${PREFIX_DIR}/lib/*.o


FROM builder AS libx32
ARG GLIBC_VERSION
ARG PREFIX_DIR
RUN apt-get -y install gcc-multilib g++-multilib
RUN mkdir -p /glibc-build && cd /glibc-build && \
    /glibc-${GLIBC_VERSION}/configure BUILD_CC='gcc' CC='gcc -mx32' CXX='g++ -mx32' CFLAGS="-Os" CXXFLAGS="-Os" \
        --prefix="${PREFIX_DIR}" \
        --enable-stack-protector=strong && \
    make && make install
RUN rm -rf ${PREFIX_DIR}/lib/audit && \
    rm -rf ${PREFIX_DIR}/lib/gconv && \
    rm -rf ${PREFIX_DIR}/lib/*.a && \
    rm -rf ${PREFIX_DIR}/lib/*.o


FROM alpine:3.12 AS prebuilt
ARG PREFIX_DIR
COPY aliases.sh /etc/profile.d/
COPY --from=lib32  ${PREFIX_DIR}/lib ${PREFIX_DIR}/lib32
COPY --from=lib64  ${PREFIX_DIR}/lib ${PREFIX_DIR}/lib64
COPY --from=libx32 ${PREFIX_DIR}/lib ${PREFIX_DIR}/libx32
COPY --from=lib ${PREFIX_DIR}/bin/ldd ${PREFIX_DIR}/bin/ldd
COPY --from=lib ${PREFIX_DIR}/bin/locale ${PREFIX_DIR}/bin/locale
COPY --from=lib ${PREFIX_DIR}/lib/locale ${PREFIX_DIR}/lib/locale
COPY --from=lib ${PREFIX_DIR}/sbin/ldconfig ${PREFIX_DIR}/sbin/ldconfig
RUN echo "hosts: files mdns4_minimal [NOTFOUND=return] dns mdns4" > /etc/nsswitch.conf && \
    mkdir -p /lib32 /lib64 /libx32 ${PREFIX_DIR}/etc ${PREFIX_DIR}/share/locale ${PREFIX_DIR}/share/i18n/charmaps && \
    ln -s ${PREFIX_DIR}/lib32/ld-linux.so.2 /lib32/ld-linux.so.2 && \
    ln -s ${PREFIX_DIR}/lib64/ld-linux-x86-64.so.2 /lib64/ld-linux-x86-64.so.2 && \
    ln -s ${PREFIX_DIR}/libx32/ld-linux-x32.so.2 /libx32/ld-linux-x32.so.2 && \
    ln -s ../lib32/ld-linux.so.2 /lib/ld-linux.so.2 && \
    ln -s ../lib64/ld-linux-x86-64.so.2 /lib/ld-linux-x86-64.so.2 && \
    ln -s ../libx32/ld-linux-x32.so.2 /lib/ld-linux-x32.so.2 && \
    ln -s ../lib32/ld-linux.so.2 ${PREFIX_DIR}/lib/ld-linux.so.2 && \
    ln -s ../lib64/ld-linux-x86-64.so.2 ${PREFIX_DIR}/lib/ld-linux-x86-64.so.2 && \
    ln -s ../libx32/ld-linux-x32.so.2 ${PREFIX_DIR}/lib/ld-linux-x32.so.2 && \
    printf "${PREFIX_DIR}/lib\n${PREFIX_DIR}/lib32\n${PREFIX_DIR}/lib64\n${PREFIX_DIR}/libx32\n" > ${PREFIX_DIR}/etc/ld.so.conf && \
    ${PREFIX_DIR}/sbin/ldconfig


FROM alpine:3.12
ARG PREFIX_DIR
ENV LANG=C.UTF-8
COPY --from=prebuilt /lib32 /lib32
COPY --from=prebuilt /lib64 /lib64
COPY --from=prebuilt /libx32 /libx32
COPY --from=prebuilt ${PREFIX_DIR} ${PREFIX_DIR}
COPY --from=prebuilt /etc/nsswitch.conf /etc/nsswitch.conf
COPY --from=prebuilt /etc/profile.d/aliases.sh /etc/profile.d/aliases.sh
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories
RUN ln -s ../lib32/ld-linux.so.2 /lib/ld-linux.so.2 && \
    ln -s ../lib64/ld-linux-x86-64.so.2 /lib/ld-linux-x86-64.so.2 && \
    ln -s ../libx32/ld-linux-x32.so.2 /lib/ld-linux-x32.so.2
RUN apk update && apk upgrade --available && rm -rf /var/cache/apk/*
