# docker-haproxy-boringssl

[![Docker Pulls](https://img.shields.io/docker/pulls/jamiehewland/haproxy-boringssl.svg?style=flat-square)](https://hub.docker.com/r/jamiehewland/haproxy-boringssl/)
[![Travis branch](https://img.shields.io/travis/JayH5/docker-haproxy-boringssl/master.svg?style=flat-square)](https://travis-ci.org/JayH5/docker-haproxy-boringssl)

HAProxy built with BoringSSL in a Docker image

Why might you want to use BoringSSL instead of OpenSSL?
* Match the TLS features available in Google Chrome, e.g. support the TLS version 1.3 draft that Chrome supports.
* Use [BoringSSL's cipher groups](https://boringssl.googlesource.com/boringssl/+/858a88daf27975f67d9f63e18f95645be2886bfb%5E!/) which allow the server to choose the client's preferred cipher in certain circumstances (e.g. when a client lacks hardware support for AES, then a faster software implementation of ChaCha20 can be used instead).
* Some other reason you may have for preferring BoringSSL over OpenSSL :-)

This image is _somewhat_ inspired by ["nginx-boringssl"](https://github.com/nginx-modules/docker-nginx-boringssl), but of course uses HAProxy instead of Nginx. Also, while nginx-boringssl enables many extra features and optimisations, this image does fewer fancy things.

Compared to the [official HAProxy image](https://hub.docker.com/_/haproxy/), this image:
* Builds and statically links BoringSSL, tracking* the BoringSSL version used in Chromium beta (as opposed to using the operating system's OpenSSL).
* Builds against PCRE2 instead of the older "PCRE 3".
* Enables use of the PCRE2 JIT engine.
* The Alpine Linux image is based on Alpine 3.7 vs. (currently) Alpine 3.6 in the official image.

\* _No promises about speedy updates to HAProxy or BoringSSL. I'm just one person._
