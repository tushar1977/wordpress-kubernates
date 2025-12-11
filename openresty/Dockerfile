#Builder that will compile from source
FROM ubuntu:22.04 AS builder

ENV OPENRESTY_VERSION=1.27.1.2

RUN apt-get update && \
    apt-get install -y \
        build-essential \
        zlib1g-dev \
        wget \
        postgresql-server-dev-14 \
        libpcre3 \
        libpcre3-dev \
        libssl-dev \
        perl \
        make \
        curl && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /tmp

RUN wget https://openresty.org/download/openresty-${OPENRESTY_VERSION}.tar.gz && \
    tar -xzf openresty-${OPENRESTY_VERSION}.tar.gz && \
    cd openresty-${OPENRESTY_VERSION} && \
      ./configure \
        --prefix=/opt/openresty \
        --with-pcre-jit \
        --with-ipv6 \
        --without-http_redis2_module \
        --with-http_iconv_module \
        --with-http_postgres_module \
        -j8 && \
    make -j8 && \
    make install

#Runtime used by kubernates
FROM ubuntu:22.04 AS runtime

RUN apt-get update && \
    apt-get install -y \
        libpcre3 \
        zlib1g \
        libssl3 \
        curl && \
    rm -rf /var/lib/apt/lists/*

COPY --from=builder /opt/openresty /opt/openresty

ENV PATH="/opt/openresty/bin:${PATH}"

COPY openresty.conf /opt/openresty/nginx/conf/nginx.conf

EXPOSE 80 443

CMD ["openresty", "-g", "daemon off;"]
