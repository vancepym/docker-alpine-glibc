ARG GLIBC_VERSION=2.38
ARG PREFIX_DIR=/usr/lib/glibc


FROM ubuntu:22.04 AS builder
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
    /glibc-${GLIBC_VERSION}/configure CFLAGS="-Os -Wno-maybe-uninitialized" CXXFLAGS="-Os -Wno-maybe-uninitialized" \
        --prefix="${PREFIX_DIR}" \
        --enable-stack-protector=strong && \
    make && make install
RUN sed -i '1c #!/bin/sh' ${PREFIX_DIR}/bin/ldd && \
    mkdir -p ${PREFIX_DIR}/lib/locale && ${PREFIX_DIR}/bin/localedef -c -i POSIX -f UTF-8 C.UTF-8 || true
RUN rm -rf ${PREFIX_DIR}/lib/audit && \
    rm -rf ${PREFIX_DIR}/lib/gconv && \
    rm -rf ${PREFIX_DIR}/lib/*.a && \
    rm -rf ${PREFIX_DIR}/lib/*.o


FROM alpine:3.18.3
ARG PREFIX_DIR
ENV LANG=C.UTF-8
COPY aliases.sh /etc/profile.d/
COPY --from=lib ${PREFIX_DIR}/lib ${PREFIX_DIR}/lib
COPY --from=lib ${PREFIX_DIR}/bin/ldd ${PREFIX_DIR}/bin/ldd
COPY --from=lib ${PREFIX_DIR}/bin/locale ${PREFIX_DIR}/bin/locale
COPY --from=lib ${PREFIX_DIR}/sbin/ldconfig ${PREFIX_DIR}/sbin/ldconfig
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories
RUN echo "hosts: files mdns4_minimal [NOTFOUND=return] dns mdns4" > /etc/nsswitch.conf && \
    mkdir -p /lib64 ${PREFIX_DIR}/lib64 ${PREFIX_DIR}/etc ${PREFIX_DIR}/share/locale ${PREFIX_DIR}/share/i18n/charmaps && \
    ln -s ${PREFIX_DIR}/lib/ld-linux-x86-64.so.2 ${PREFIX_DIR}/lib64/ld-linux-x86-64.so.2 && \
    ln -s ${PREFIX_DIR}/lib/ld-linux-x86-64.so.2 /lib64/ld-linux-x86-64.so.2 && \
    ln -s ../lib64/ld-linux-x86-64.so.2 /lib/ld-linux-x86-64.so.2 && \
    printf "${PREFIX_DIR}/lib\n${PREFIX_DIR}/lib64\n" > ${PREFIX_DIR}/etc/ld.so.conf && \
    ${PREFIX_DIR}/sbin/ldconfig
RUN apk update && apk upgrade --available && rm -rf /var/cache/apk/*
