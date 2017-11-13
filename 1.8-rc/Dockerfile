FROM debian:stretch-slim

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        liblua5.3-0 \
        libpcre2-posix0 \
    && rm -rf /var/lib/apt/lists/*

# chromium-63.0.3239.40 (Google Chrome Beta 2017-11-13)
ENV BORINGSSL_COMMIT 664e99a6486c293728097c661332f92bf2d847c6

ENV HAPROXY_MAJOR 1.8
ENV HAPROXY_VERSION 1.8-rc3
ENV HAPROXY_MD5 aa9f67bf6f700d6688d7c861c6319735

RUN set -ex; \
    buildDeps=' \
        ca-certificates \
        cmake \
        g++ \
        gcc \
        git \
        golang \
        libc6-dev \
        liblua5.3-dev \
        libpcre2-dev \
        make \
        wget \
        zlib1g-dev \
    '; \
    apt-get update; \
    apt-get install -y --no-install-recommends $buildDeps; \
    rm -rf /var/lib/apt/lists/*; \
    \
    git clone https://boringssl.googlesource.com/boringssl /usr/src/boringssl; \
    cd /usr/src/boringssl; \
    git reset --hard $BORINGSSL_COMMIT; \
    cmake -DCMAKE_BUILD_TYPE=RelWithDebInfo .; \
    make -j "$(nproc)" VERBOSE=1; \
    \
    mkdir /usr/local/boringssl /usr/local/boringssl/lib; \
    cp -r include /usr/local/boringssl; \
    cp ssl/libssl.a crypto/libcrypto.a /usr/local/boringssl/lib; \
    cd -; \
    rm -rf /usr/src/boringssl; \
    \
    wget -O haproxy.tar.gz "http://www.haproxy.org/download/${HAPROXY_MAJOR}/src/haproxy-${HAPROXY_VERSION}.tar.gz"; \
    echo "$HAPROXY_MD5 *haproxy.tar.gz" | md5sum -c; \
    mkdir -p /usr/src/haproxy; \
    tar -xzf haproxy.tar.gz -C /usr/src/haproxy --strip-components=1; \
    rm haproxy.tar.gz; \
    \
    cd /usr/src/haproxy; \
    makeOpts=' \
        LD=g++ \
        TARGET=linux2628 \
        USE_LUA=1 LUA_INC=/usr/include/lua5.3 \
        USE_OPENSSL=1 SSL_INC=/usr/local/boringssl/include SSL_LIB=/usr/local/boringssl/lib \
        USE_PCRE2=1 USE_PCRE2_JIT=1 \
        USE_ZLIB=1 \
    '; \
    make -j "$(nproc)" all $makeOpts; \
    make install-bin $makeOpts; \
    \
    mkdir -p /usr/local/etc/haproxy; \
    cp -R examples/errorfiles /usr/local/etc/haproxy/errors; \
    cd -; \
    rm -rf /usr/src/haproxy; \
    \
    rm -rf /usr/local/boringssl; \
    apt-get purge -y --auto-remove $buildDeps

COPY docker-entrypoint.sh /
ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["haproxy", "-f", "/usr/local/etc/haproxy/haproxy.cfg"]
