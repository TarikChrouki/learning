FROM alpine:3.16.2
USER root

RUN apk add git
RUN git clone --depth 1 https://github.com/edenhill/kcat.git --branch 1.7.1 /usr/src/kcat

ENV BUILD_DEPS bash make gcc g++ cmake curl pkgconfig python3 perl bsd-compat-headers zlib-dev zstd-dev zstd-libs lz4-dev openssl-dev curl-dev
ENV RUN_DEPS libcurl lz4-libs zstd-libs ca-certificates

ENV UTILS_DEPS curl openjdk11 openssl

RUN echo Installing Kcat ; \
  apk add --no-cache --virtual .dev_pkgs $BUILD_DEPS $BUILD_DEPS_EXTRA && \
  apk add --no-cache $RUN_DEPS $RUN_DEPS_EXTRA && \
  echo Building && \
  cd /usr/src/kcat && \
      rm -rf tmp-bootstrap && \
      echo "Source versions:" && \
      grep ^github_download ./bootstrap.sh && \
      ./bootstrap.sh --no-install-deps --no-enable-static && \
      mv kcat /usr/bin/ && \
  echo Installing utilities && \
        apk add --no-cache $UTILS_DEPS && \
  echo Installing confluent cli && \
        cd /usr && \
        curl -sL --http1.1 https://cnfl.io/cli | sh -s -- latest && \
        export PATH=$(pwd)/bin:$PATH && \
  echo Cleaning up && \
  cd / && \
  apk del .dev_pkgs && \
  rm -rf /var/cache/apk/*

RUN echo $(kcat -V) && \
    echo $(java -version) && \
    echo $(confluent version)



