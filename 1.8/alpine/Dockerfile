FROM alpine:3.6

ENV LUA_VERSION=5.3.4 \
    LUA_SHA1=79790cfd40e09ba796b01a571d4d63b52b1cd950

# chromium-63.0.3239.70 (Google Chrome Beta 2017-11-29)
ENV BORINGSSL_COMMIT 664e99a6486c293728097c661332f92bf2d847c6

ENV HAPROXY_MAJOR 1.8
ENV HAPROXY_VERSION 1.8.0
ENV HAPROXY_MD5 6ccea4619b7183fbcc8c98bae1f9823d

RUN set -ex; \
    apk add --no-cache --virtual .build-deps \
        ca-certificates \
        cmake \
        g++ \
        gcc \
        git \
        go \
        libc-dev \
        libressl \
        linux-headers \
        make \
        pcre2-dev \
        perl \
        readline-dev \
        tar \
        zlib-dev \
    ; \
# install Lua
    wget -O lua.tar.gz "https://www.lua.org/ftp/lua-$LUA_VERSION.tar.gz"; \
    echo "$LUA_SHA1 *lua.tar.gz" | sha1sum -c; \
    mkdir -p /usr/src/lua; \
    tar -xzf lua.tar.gz -C /usr/src/lua --strip-components=1; \
    rm lua.tar.gz; \
    \
    cd /usr/src/lua; \
    make -j "$(getconf _NPROCESSORS_ONLN)" linux; \
    make install \
        INSTALL_TOP=/usr/local/lua; \
    cd -; \
    rm -rf /usr/src/lua; \
    \
# BoringSSL
    git clone https://boringssl.googlesource.com/boringssl /usr/src/boringssl; \
    cd /usr/src/boringssl; \
    git reset --hard $BORINGSSL_COMMIT; \
    cmake -DCMAKE_BUILD_TYPE=RelWithDebInfo .; \
    make -j "$(getconf _NPROCESSORS_ONLN)" VERBOSE=1; \
    \
    mkdir /usr/local/boringssl /usr/local/boringssl/lib; \
    cp -r include /usr/local/boringssl; \
    cp ssl/libssl.a crypto/libcrypto.a /usr/local/boringssl/lib; \
    cd -; \
    rm -rf /usr/src/boringssl; \
    \
# install HAProxy
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
        USE_LUA=1 LUA_INC=/usr/local/lua/include LUA_LIB=/usr/local/lua/lib \
        USE_OPENSSL=1 SSL_INC=/usr/local/boringssl/include SSL_LIB=/usr/local/boringssl/lib \
        USE_PCRE2=1 USE_PCRE2_JIT=1 \
        USE_ZLIB=1 \
    '; \
    make -j "$(getconf _NPROCESSORS_ONLN)" all $makeOpts; \
    make install-bin $makeOpts; \
    \
# purge the remnants of our static Lua and BoringSSL
    rm -rf /usr/local/lua; \
    rm -rf /usr/local/boringssl; \
    \
    mkdir -p /usr/local/etc/haproxy; \
    cp -r examples/errorfiles /usr/local/etc/haproxy/errors; \
    cd -; \
    rm -rf /usr/src/haproxy; \
    \
    runDeps="$( \
        scanelf --needed --nobanner --format '%n#p' --recursive /usr/local \
            | tr ',' '\n' \
            | sort -u \
            | awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
    )"; \
    apk add --virtual .haproxy-rundeps $runDeps; \
    apk del .build-deps

COPY docker-entrypoint.sh /
ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["haproxy", "-f", "/usr/local/etc/haproxy/haproxy.cfg"]
