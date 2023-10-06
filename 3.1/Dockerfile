FROM ruby:3.1.4-bullseye

RUN sed -i 's/mozilla\/DST_Root_CA_X3\.crt/!mozilla\/DST_Root_CA_X3.crt/' /etc/ca-certificates.conf \
  && update-ca-certificates

RUN apt-get update -qq \
  && DEBIAN_FRONTEND=noninteractive apt-get install -yq --no-install-recommends \
    locales \
    less \
    libxml2-dev \
    libxslt-dev \
    pkg-config \
    libcurl3-dev \
    libpq-dev \
    libssl-dev \
    libgmp3-dev \
    postgresql-client \
    git-core \
    ssh \
    libmagickcore-dev \
    libmagickwand-dev \
    imagemagick \
    libsqlite3-dev \
    catdoc \
    enca \
    default-libmysqlclient-dev \
    build-essential \
    vim \
    netcat \
    gnupg \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* \
  && truncate -s 0 /var/log/*log

ENV NODE_MAJOR 18
ENV NPM_VER 10.2.0

RUN mkdir -p /etc/apt/keyrings \
  && curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg \
  && echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_MAJOR.x nodistro main" > /etc/apt/sources.list.d/nodesource.list \
  && apt-get update -qq \
  && apt-get -y install --no-install-recommends nodejs \
  && npm install -g npm@${NPM_VER} \
  && npm install -g yarn

RUN ln -sf /usr/share/zoneinfo/Asia/Yekaterinburg /etc/localtime \
  && dpkg-reconfigure -f noninteractive tzdata \
  && echo 'en_US.UTF-8 UTF-8' >> /etc/locale.gen \
  && locale-gen \
  && dpkg-reconfigure --frontend=noninteractive locales

ENV LANG en_US.UTF-8
ENV BUNDLE_SILENCE_ROOT_WARNING 1

RUN echo 'gem: --no-rdoc --no-ri --no-document' > /root/.gemrc \
  && ssh-keyscan -H github.com >> /etc/ssh/ssh_known_hosts \
  && ssh-keyscan -H gitlab.railsc.ru >> /etc/ssh/ssh_known_hosts \
  && gem install bundler \
  && bundle config --global jobs 7 \
  && bundle config git.allow_insecure true \
  && bundle config --global build.nokogiri --use-system-libraries

ENV TINI_VERSION v0.19.0
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini /tini
RUN chmod +x /tini

ENV EXCEL2CSV_VERSION v0.1.4
ENV LIBXL_VERSION 3.8.8.2
RUN sed -i 's/DEFAULT@SECLEVEL=2/DEFAULT@SECLEVEL=1/' /etc/ssl/openssl.cnf
RUN set -ex; \
  curl -k1 -o /tmp/libxl-$LIBXL_VERSION.tar.gz https://www.libxl.com/download/libxl-lin-$LIBXL_VERSION.tar.gz; \
  tar -xzf /tmp/libxl-$LIBXL_VERSION.tar.gz -C /tmp; \
  cp /tmp/libxl-$LIBXL_VERSION/lib64/libxl.so /usr/lib; \
  rm -rf /tmp/libxl-$LIBXL_VERSION.tar.gz /tmp/libxl-$LIBXL_VERSION;
RUN sed -i 's/DEFAULT@SECLEVEL=1/DEFAULT@SECLEVEL=2/' /etc/ssl/openssl.cnf

ADD excel2csv /usr/bin

ENTRYPOINT ["/tini", "--"]

WORKDIR /app
