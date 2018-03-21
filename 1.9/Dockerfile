FROM golang:1.9.4-stretch

RUN apt-get update && apt-get install -y --no-install-recommends \
    autoconf \
    automake \
    clang \
    g++ \
    gcc \
    graphviz \
    jq \
    libc6-dev \
    libssl-dev \
    libsqlite3-dev \
    libtool \
    make \
    pkg-config \
    tcl \
    tk \
    unzip

### INSTALL PYTHON

ENV PYTHON_VERSION 2.7.14
ENV PYTHON_GPG_KEY C01E1CAD5EA2C4F0B8E3571504C367C218ADD4FF
RUN set -ex \
  && buildDeps=' \
    dpkg-dev \
    tcl-dev \
    tk-dev \
  ' \
  && apt-get update && apt-get install -y $buildDeps --no-install-recommends && rm -rf /var/lib/apt/lists/* \
  \
  && wget -O python.tar.xz "https://www.python.org/ftp/python/${PYTHON_VERSION%%[a-z]*}/Python-$PYTHON_VERSION.tar.xz" \
  && wget -O python.tar.xz.asc "https://www.python.org/ftp/python/${PYTHON_VERSION%%[a-z]*}/Python-$PYTHON_VERSION.tar.xz.asc" \
  && export GNUPGHOME="$(mktemp -d)" \
  && gpg --keyserver ha.pool.sks-keyservers.net --recv-keys "$PYTHON_GPG_KEY" \
  && gpg --batch --verify python.tar.xz.asc python.tar.xz \
  && rm -rf "$GNUPGHOME" python.tar.xz.asc \
  && mkdir -p /usr/src/python \
  && tar -xJC /usr/src/python --strip-components=1 -f python.tar.xz \
  && rm python.tar.xz \
  \
  && cd /usr/src/python \
  && gnuArch="$(dpkg-architecture --query DEB_BUILD_GNU_TYPE)" \
  && ./configure \
    --build="$gnuArch" \
    --enable-shared \
    --enable-unicode=ucs4 \
  && make -j "$(nproc)" \
  && make install \
  && ldconfig \
  \
  && apt-get purge -y --auto-remove $buildDeps \
  \
  && find /usr/local -depth \
    \( \
      \( -type d -a \( -name test -o -name tests \) \) \
      -o \
      \( -type f -a \( -name '*.pyc' -o -name '*.pyo' \) \) \
    \) -exec rm -rf '{}' + \
  && rm -rf /usr/src/python

ENV GOPATH /go
RUN mkdir -p "$GOPATH/src" "$GOPATH/bin" && chmod -R 777 "$GOPATH"

### INSTALL PROTOBUF

ENV PROTOBUF_VERSION 3.5.1

RUN wget -O protobuf.tar.gz "https://github.com/google/protobuf/releases/download/v${PROTOBUF_VERSION}/protobuf-cpp-${PROTOBUF_VERSION}.tar.gz"
RUN tar xzf protobuf.tar.gz && cd "protobuf-${PROTOBUF_VERSION}" && ./autogen.sh && ./configure && make && make install && ldconfig

RUN /usr/local/go/bin/go get -u github.com/golang/protobuf/protoc-gen-go

### INSTALL GOOGLE_CLOUD_SDK

ENV CLOUD_SDK_VERSION 193.0.0

RUN echo "deb http://packages.cloud.google.com/apt cloud-sdk-jessie main" | tee -a /etc/apt/sources.list.d/google-cloud-sdk.list
RUN curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
RUN apt-get update && apt-get install -y --no-install-recommends google-cloud-sdk=${CLOUD_SDK_VERSION}-0 google-cloud-sdk-app-engine-go google-cloud-sdk-datastore-emulator

ENV GO_APPENGINE_SDK_VERSION 1.9.63

RUN wget -O go_appengine.zip "https://storage.googleapis.com/appengine-sdks/featured/go_appengine_sdk_linux_amd64-${GO_APPENGINE_SDK_VERSION}.zip"
RUN unzip go_appengine.zip && rm go_appengine.zip && mv go_appengine /usr/local/go_appengine

RUN /usr/local/go/bin/go get -u google.golang.org/appengine

RUN rm -rf /var/lib/apt/lists/*

ENV GOPATH /go
ENV PATH $GOPATH/bin:/usr/local/go/bin:$PATH

RUN mkdir -p "$GOPATH/src" "$GOPATH/bin" && chmod -R 777 "$GOPATH"
WORKDIR $GOPATH
