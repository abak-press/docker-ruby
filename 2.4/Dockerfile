FROM ruby:2.4.10-buster

RUN sed -i 's/mozilla\/DST_Root_CA_X3\.crt/!mozilla\/DST_Root_CA_X3.crt/' /etc/ca-certificates.conf \
  && update-ca-certificates

RUN curl -L https://www.postgresql.org/media/keys/ACCC4CF8.asc | apt-key add - \
  && echo "deb http://apt.postgresql.org/pub/repos/apt buster-pgdg main 11" > /etc/apt/sources.list.d/pgdg.list \
  && apt-get update -qq \
  && DEBIAN_FRONTEND=noninteractive apt-get install -yq --no-install-recommends \
    locales \
    make \
    gcc \
    g++ \
    curl \
    less \
    libxml2-dev \
    libxslt-dev \
    libssl-dev \
    pkg-config \
    libcurl3-dev \
    libpq-dev \
    libgmp3-dev \
    postgresql-client-11 \
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
    gnupg2 \
    netcat \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* \
  && truncate -s 0 /var/log/*log

RUN curl -fsSL https://deb.nodesource.com/setup_14.x | bash - ;\
  apt-get install -y nodejs && \
  npm install -g yarn

RUN ln -sf /usr/share/zoneinfo/Asia/Yekaterinburg /etc/localtime \
  && dpkg-reconfigure -f noninteractive tzdata \
  && echo 'en_US.UTF-8 UTF-8' >> /etc/locale.gen \
  && locale-gen \
  && dpkg-reconfigure --frontend=noninteractive locales

ENV LANG en_US.UTF-8
ENV BUNDLE_SILENCE_ROOT_WARNING 1

# https://github.com/bundler/bundler/issues/4402
RUN echo 'gem: --no-rdoc --no-ri --no-document' > /root/.gemrc \
  && ssh-keyscan -H github.com >> /etc/ssh/ssh_known_hosts \
  && ssh-keyscan -H gitlab.railsc.ru >> /etc/ssh/ssh_known_hosts \
  && gem install bundler --version 1.17.3 \
  && gem install executable-hooks -v ">=1.3.2" \
  && gem regenerate_binstubs \
  && bundle config --global jobs 7 \
  && bundle config git.allow_insecure true \
  && bundle config --global build.nokogiri --use-system-libraries

ENV TINI_VERSION v0.18.0
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini /tini
RUN chmod +x /tini

ENV EXCEL2CSV_VERSION v0.1.4
ENV LIBXL_VERSION 3.8.8.2
RUN sed -i 's/DEFAULT@SECLEVEL=2/DEFAULT@SECLEVEL=1/' /etc/ssl/openssl.cnf
RUN set -ex; \
  curl -o /tmp/libxl-$LIBXL_VERSION.tar.gz https://www.libxl.com/download/libxl-lin-$LIBXL_VERSION.tar.gz; \
  tar -xzf /tmp/libxl-$LIBXL_VERSION.tar.gz -C /tmp; \
  cp /tmp/libxl-$LIBXL_VERSION/lib64/libxl.so /usr/lib; \
  rm -rf /tmp/libxl-$LIBXL_VERSION.tar.gz /tmp/libxl-$LIBXL_VERSION;
RUN sed -i 's/DEFAULT@SECLEVEL=1/DEFAULT@SECLEVEL=2/' /etc/ssl/openssl.cnf

ADD excel2csv /usr/bin

RUN mkdir /root/.ssh && \
  echo 'Host sphinx\n\
StrictHostKeyChecking no\n\
UserKnownHostsFile=/dev/null\n'\
>> /root/.ssh/config

ENTRYPOINT ["/tini", "--"]

RUN git config --global --add safe.directory "*"
WORKDIR /app
