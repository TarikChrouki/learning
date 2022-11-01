FROM alpine:3.16.2
USER root

RUN apk add git
RUN git clone --depth 1 https://github.com/edenhill/kcat.git --branch 1.7.1 /usr/src/kcat

ENV BUILD_DEPS bash make gcc g++ cmake curl pkgconfig python perl bsd-compat-headers zlib-dev zstd-dev zstd-libs lz4-dev openssl-dev curl-dev

ENV RUN_DEPS libcurl lz4-libs zstd-libs ca-certificates

RUN echo Installing ; \
  apk add --no-cache --virtual .dev_pkgs $BUILD_DEPS $BUILD_DEPS_EXTRA && \
  apk add --no-cache $RUN_DEPS $RUN_DEPS_EXTRA && \
  echo Building && \
  cd /usr/src/kcat && \
  rm -rf tmp-bootstrap && \
  echo "Source versions:" && \
  grep ^github_download ./bootstrap.sh && \
  ./bootstrap.sh --no-install-deps --no-enable-static && \
  mv kcat /usr/bin/ && \
  echo Cleaning up && \
  cd / && \
  apk del .dev_pkgs

RUN kcat -V

RUN apk add java-11-openjdk && \
    apk add openssl && \
    apk add which && \
    apk add unzip && \
    apk add --no-cache librdkafka && \
    rm -rf /var/cache/apk/*


