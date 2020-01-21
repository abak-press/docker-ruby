FROM ruby:2.5-stretch


RUN apt-key adv --keyserver keyserver.ubuntu.com --recv 68576280 \
  && echo 'deb http://deb.nodesource.com/node_8.x trusty main' > /etc/apt/sources.list.d/nodesource.list \
  && apt-key adv --keyserver keyserver.ubuntu.com --recv 46C2130DFD2497F5 \
  && echo 'deb http://dl.yarnpkg.com/debian/ stable main' > /etc/apt/sources.list.d/yarn.list \
  && apt-get update -qq \
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
    ca-certificates \
    default-libmysqlclient-dev \
    nodejs \
    build-essential \
    yarn \
    vim \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* \
  && truncate -s 0 /var/log/*log

RUN echo "Asia/Yekaterinburg" > /etc/timezone \
  && dpkg-reconfigure -f noninteractive tzdata \
  && locale-gen "en_US.UTF-8" \
  && dpkg-reconfigure --frontend=noninteractive locales

ENV LANG en_US.UTF-8
ENV BUNDLE_SILENCE_ROOT_WARNING 1

RUN echo 'gem: --no-rdoc --no-ri --no-document' > /root/.gemrc \
  && ssh-keyscan -H github.com >> /etc/ssh/ssh_known_hosts \
  && gem install bundler --version 1.17.3 \
  && bundle config --global jobs 7 \
  && bundle config git.allow_insecure true \
  && bundle config --global build.nokogiri --use-system-libraries

ENV TINI_VERSION v0.18.0
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini /tini
RUN chmod +x /tini

ENV EXCEL2CSV_VERSION v0.1.4
ENV LIBXL_VERSION 3.8.8.2
RUN set -ex; \
  curl -o /tmp/libxl-$LIBXL_VERSION.tar.gz https://www.libxl.com/download/libxl-lin-$LIBXL_VERSION.tar.gz; \
  tar -xzf /tmp/libxl-$LIBXL_VERSION.tar.gz -C /tmp; \
  cp /tmp/libxl-$LIBXL_VERSION/lib64/libxl.so /usr/lib; \
  rm -rf /tmp/libxl-$LIBXL_VERSION.tar.gz /tmp/libxl-$LIBXL_VERSION;

ADD excel2csv /usr/bin

ENTRYPOINT ["/tini", "--"]

WORKDIR /app
