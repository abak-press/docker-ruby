FROM ubuntu:20.04

# Установка базовых пакетов и RVM
RUN apt-get update -qq \
  && DEBIAN_FRONTEND=noninteractive apt-get install -yq --no-install-recommends \
    curl \
    gnupg2 \
    software-properties-common \
    ca-certificates \
    locales \
    make \
    gcc \
    g++ \
    less \
    libxml2-dev \
    libxslt-dev \
    pkg-config \
    libcurl4-openssl-dev \
    libssl-dev \
    libgmp-dev \
    git-core \
    ssh \
    libmagickcore-dev \
    libmagickwand-dev \
    imagemagick \
    libsqlite3-dev \
    catdoc \
    enca \
    apt-transport-https \
    libmysqlclient-dev \
    build-essential \
    vim \
    tzdata \
    dirmngr \
    netcat \
    wget \
    bzip2 \
    autoconf \
    libreadline-dev \
    zlib1g-dev \
    libncurses5-dev \
    libffi-dev \
    libyaml-dev \
    libgdbm-dev \
    libnss3-dev \
    libtool \
    libxslt1-dev \
    libxml2-dev \
    libssl-dev \
    libyaml-dev \
    libreadline6-dev \
    zlib1g-dev \
    libncurses5-dev \
    libffi-dev \
    libgdbm6 \
    libgdbm-dev \
    libdb-dev \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* \
  && truncate -s 0 /var/log/*log

# Установка RVM и Ruby 2.3.8
RUN gpg --keyserver keyserver.ubuntu.com --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB \
  && curl -sSL https://get.rvm.io | bash -s stable \
  && /bin/bash -l -c "rvm pkg install openssl" \
  && /bin/bash -l -c "rvm autolibs disable" \
  && /bin/bash -l -c "rvm install 2.3.8 --with-openssl-dir=/usr/local/rvm/usr --disable-install-doc" \
  && /bin/bash -l -c "rvm use 2.3.8 --default" \
  && /bin/bash -l -c "rvm cleanup all"

# Добавляем RVM в PATH
ENV PATH /usr/local/rvm/bin:/usr/local/rvm/rubies/ruby-2.3.8/bin:$PATH
ENV GEM_HOME /usr/local/rvm/gems/ruby-2.3.8
ENV GEM_PATH /usr/local/rvm/gems/ruby-2.3.8:/usr/local/rvm/gems/ruby-2.3.8@global

# Установка PostgreSQL client
RUN apt-get update -qq \
  && DEBIAN_FRONTEND=noninteractive apt-get install -yq --no-install-recommends \
    postgresql-client \
    libpq-dev \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Установка Node.js и Yarn
RUN curl -s https://deb.nodesource.com/gpgkey/nodesource.gpg.key | apt-key add - \
  && curl -fsSL https://deb.nodesource.com/setup_14.x | bash - \
  && apt-get install -y nodejs \
  && npm install -g yarn

# Настройка timezone и locale
RUN ln -sf /usr/share/zoneinfo/Asia/Yekaterinburg /etc/localtime \
  && dpkg-reconfigure -f noninteractive tzdata \
  && echo 'en_US.UTF-8 UTF-8' >> /etc/locale.gen \
  && locale-gen \
  && dpkg-reconfigure --frontend=noninteractive locales

ENV LANG en_US.UTF-8
ENV BUNDLE_SILENCE_ROOT_WARNING 1

# Настройка Ruby gems и bundler
RUN echo 'gem: --no-rdoc --no-ri --no-document' > /root/.gemrc \
  && ssh-keyscan -H github.com >> /etc/ssh/ssh_known_hosts \
  && ssh-keyscan -H gitlab.railsc.ru >> /etc/ssh/ssh_known_hosts \
  && /bin/bash -l -c "gem install bundler --version 1.17.3" \
  && /bin/bash -l -c "gem install executable-hooks -v '>=1.3.2'" \
  && /bin/bash -l -c "gem regenerate_binstubs" \
  && /bin/bash -l -c "bundle config --global jobs 7" \
  && /bin/bash -l -c "bundle config git.allow_insecure true" \
  && /bin/bash -l -c "bundle config --global build.nokogiri --use-system-libraries"

# Настройка magic file для SVG
RUN set -eux; \
  echo "\
0	string		\<svg			SVG Scalable Vector Graphics image\n\
!:mime	image/svg\n\
" >> /etc/magic

# Установка tini
ENV TINI_VERSION v0.18.0
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini /tini
RUN chmod +x /tini

# Установка libxl и excel2csv
ENV EXCEL2CSV_VERSION v0.1.4
ENV LIBXL_VERSION 3.8.8.2
RUN set -ex; \
  curl -o /tmp/libxl-$LIBXL_VERSION.tar.gz https://www.libxl.com/download/libxl-lin-$LIBXL_VERSION.tar.gz; \
  tar -xzf /tmp/libxl-$LIBXL_VERSION.tar.gz -C /tmp; \
  cp /tmp/libxl-$LIBXL_VERSION/lib64/libxl.so /usr/lib; \
  rm -rf /tmp/libxl-$LIBXL_VERSION.tar.gz /tmp/libxl-$LIBXL_VERSION;

ADD excel2csv /usr/bin

# Настройка SSH
RUN mkdir /root/.ssh && \
  echo 'Host sphinx\n\
StrictHostKeyChecking no\n\
UserKnownHostsFile=/dev/null\n'\
>> /root/.ssh/config

ENTRYPOINT ["/tini", "--"]

WORKDIR /app
