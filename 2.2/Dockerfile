FROM ruby:2.2.7-jessie

RUN sed -i 's/mozilla\/DST_Root_CA_X3\.crt/!mozilla\/DST_Root_CA_X3.crt/' /etc/ca-certificates.conf \
  && update-ca-certificates

RUN apt-get update -qq \
  && DEBIAN_FRONTEND=noninteractive apt-get install -yq --no-install-recommends \
    locales \
    make \
    gcc \
    g++ \
    curl \
    less \
    libxml2-dev \
    libssl-dev \
    libxslt-dev \
    pkg-config \
    libcurl3-dev \
    libgmp3-dev \
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
    gnupg2 \
    tzdata \
    dirmngr \
    netcat \
    libodbc1 \
    unixodbc \
    upstart-job \
  && curl -L https://www.postgresql.org/media/keys/ACCC4CF8.asc | apt-key add - \
  && echo "deb https://apt-archive.postgresql.org/pub/repos/apt jessie-pgdg main 9.6" > /etc/apt/sources.list.d/pgdg.list \
  && apt-get update -qq \
  && DEBIAN_FRONTEND=noninteractive apt-get install -yq --no-install-recommends \
    postgresql-client-9.6 \
    postgresql-plperl-9.6 \
    software-properties-common \
    postgresql-9.6-pgq3 \
    skytools3 \
    skytools3-ticker \
    postgresql-server-dev-9.6 \
    postgresql-common \
    wget \
    unzip \
    make \
    build-essential \
    libssl-dev \
    libkrb5-dev \
    libpq-dev \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* \
  && truncate -s 0 /var/log/*log

RUN curl -s https://deb.nodesource.com/gpgkey/nodesource.gpg.key | apt-key add - \
  && curl -fsSL https://deb.nodesource.com/setup_14.x | bash - ;\
  apt-get install -y nodejs

RUN echo "Asia/Yekaterinburg" > /etc/timezone \
  && dpkg-reconfigure -f noninteractive tzdata \
  && sed -i -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen \
  && echo 'LANG="en_US.UTF-8"' > /etc/default/locale \
  && dpkg-reconfigure --frontend=noninteractive locales \
  && update-locale LANG=en_US.UTF-8

ENV LANG en_US.UTF-8
ENV LC_ALL en_US.UTF-8
ENV BUNDLE_SILENCE_ROOT_WARNING 1

# https://github.com/bundler/bundler/issues/4402
RUN echo 'gem: --no-rdoc --no-ri --no-document' > /root/.gemrc \
  && ssh-keyscan -H github.com >> /etc/ssh/ssh_known_hosts \
  && gem update --system 2.6.1 \
  && gem install bundler --version 1.17.3 \
  && bundle config --global jobs 7 \
  && bundle config git.allow_insecure true \
  && bundle config --global build.nokogiri --use-system-libraries

RUN stat /usr/lib/x86_64-linux-gnu/ImageMagick-6.8.9/bin-Q16
ENV PATH /usr/lib/x86_64-linux-gnu/ImageMagick-6.8.9/bin-Q16:$PATH
RUN gem install rmagick -v 2.13.2

RUN set -eux; \
  \
echo "\
0   string      \<svg           SVG Scalable Vector Graphics image\n\
!:mime  image/svg\n\
" >> /etc/magic

# dnsmasq
RUN apt-get install -y -q dnsmasq-base dnsutils &&\
    mkdir /etc/dnsmasq.d &&\
    echo "address=/dev/127.0.0.1" >> /etc/dnsmasq.d/0hosts &&\
    echo '$(which dnsmasq) -C /etc/dnsmasq.conf' >> /root/init &&\
    echo 'user=root' >> /etc/dnsmasq.conf &&\
    echo 'listen-address=127.0.0.1' >> /etc/dnsmasq.conf &&\
    echo 'resolv-file=/etc/resolv.dnsmasq.conf' >> /etc/dnsmasq.conf &&\
    echo 'conf-dir=/etc/dnsmasq.d' >> /etc/dnsmasq.conf &&\
    echo 'nameserver 8.8.8.8' >> /etc/resolv.dnsmasq.conf &&\
    echo 'nameserver 8.8.4.4' >> /etc/resolv.dnsmasq.conf

# sphinx
RUN curl -o /root/sphinx.deb http://sphinxsearch.com/files/sphinxsearch_2.1.9-release-0ubuntu11~precise_amd64.deb &&\
    dpkg -i /root/sphinx.deb &&\
    rm /root/sphinx.deb

# redis
RUN apt-get -y install redis-server &&\
    echo "/etc/init.d/redis-server start" >> /root/init

# Project DB
RUN echo "listen_addresses = '*'" >> /usr/share/postgresql/9.6/postgresql.conf.sample && \
  echo "shared_preload_libraries='pg_pathman'" >> /usr/share/postgresql/9.6/postgresql.conf.sample && \
  echo "plperl.on_init = 'use utf8; use re; package utf8; require \"utf8_heavy.pl\";'" >> /usr/share/postgresql/9.6/postgresql.conf.sample && \
  echo "host all all 0.0.0.0/0 trust" > /etc/postgresql/9.6/main/pg_hba.conf && \
  echo "local all all trust" >> /etc/postgresql/9.6/main/pg_hba.conf

COPY db/structure/dk_ci.sql /root/db/structure/dk_ci.sql
COPY db/structure/migrations_data.sql /root/db/structure/migrations_data.sql
COPY db/structure/table_partitions_data.sql /root/db/structure/table_partitions_data.sql
RUN /etc/init.d/postgresql start &&\
    apt-get -y -q install sudo &&\
    while ps aux | grep "postgres: [s]tartup process" > /dev/null; do sleep 1;done &&\
    sudo -u postgres psql -c "CREATE USER docker WITH SUPERUSER PASSWORD 'docker';" &&\
    sudo -u postgres createdb -O docker dk_ci &&\
    sudo -u postgres createlang plperl dk_ci &&\
    cat /root/db/structure/dk_ci.sql | psql -U docker dk_ci &&\
    cat /root/db/structure/migrations_data.sql | psql -U docker dk_ci &&\
    cat /root/db/structure/table_partitions_data.sql | psql -U docker dk_ci &&\
    /etc/init.d/postgresql stop

# Rails ENV
ENV RAILS_ENV test
ENV TEST_DATABASE_NAME dk_ci
ENV TEST_DATABASE_USERNAME docker
ENV TEST_DATABASE_HOST db.dev
ENV TEST_DATABASE_PORT 5432

WORKDIR /app
