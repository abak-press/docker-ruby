FROM ubuntu:14.04
MAINTAINER Igor Okunev

ENV DEBIAN_FRONTEND noninteractive

# ensure UTF-8
RUN locale-gen en_US.UTF-8
ENV LANG       en_US.UTF-8

# YEKT locale
RUN mv /etc/localtime /etc/localtime-old &&\
    ln -sf /usr/share/zoneinfo/Asia/Yekaterinburg /etc/localtime

# add apt repositories
RUN \
  sed -i 's/# \(.*multiverse$\)/\1/g' /etc/apt/sources.list &&\
  apt-get update &&\
  apt-get -y upgrade &&\
  apt-get install -y build-essential &&\
  apt-get install -y software-properties-common &&\
  apt-get install -y byobu curl git htop man unzip vim wget sudo

# create init script
RUN touch /root/init &&\
    echo "#!/bin/sh" >> /root/init &&\
    echo "set -x" >> /root/init &&\
    chmod +x /root/init

# gem env
ENV GEM_HOME /usr/local/gems
ENV GEM_PATH /usr/local/gems
ENV PATH $GEM_PATH/bin:$PATH

# install time command
RUN apt-get -y -q install time &&\
    ssh-keyscan -H github.com >> /etc/ssh/ssh_known_hosts

# install ruby
ENV RVM_PATH /usr/local/rvm
ENV RVM ${RVM_PATH}/bin/rvm
ENV RVM_RUBY ruby-2.2.7

RUN gpg --keyserver hkp://pool.sks-keyservers.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB &&\
    curl -L https://get.rvm.io | bash -s stable &&\
    $RVM install $RVM_RUBY

ENV PATH ${RVM_PATH}/rubies/${RVM_RUBY}/bin:$PATH

# project deps
RUN apt-get install -y -q libxslt-dev libxml2-dev &&\
    apt-get install -y -q libcurl3-dev &&\
    apt-get install -y -q libmagickwand-dev imagemagick &&\
    apt-get install -y -q catdoc &&\
    apt-get install -y -q enca &&\
    apt-get install -y -q libmysqlclient-dev &&\
    apt-get install -y -q libpq-dev

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

# postgres
RUN wget -O - http://apt.postgresql.org/pub/repos/apt/ACCC4CF8.asc | apt-key add - &&\
    echo "deb http://apt.postgresql.org/pub/repos/apt/ squeeze-pgdg main" > /etc/apt/sources.list.d/pgdg.list &&\
    apt-get update &&\
    chmod 666 /dev/null &&\
    apt-get -y -q install pgdg-keyring &&\
    apt-get -y -q install python-software-properties software-properties-common &&\
    apt-get -y -q install postgresql-9.3 postgresql-client-9.3 postgresql-contrib-9.3 postgresql-plperl-9.3 &&\
    pg_dropcluster --stop 9.3 main &&\
    pg_createcluster --locale $LC_ALL --start 9.3 main &&\
    echo "rm -f /var/run/postgresql/*" >> /root/init &&\
    echo "sudo -u postgres /etc/init.d/postgresql start" >> /root/init &&\
    echo "host all all 0.0.0.0/0 trust" > /etc/postgresql/9.3/main/pg_hba.conf &&\
    echo "local all all trust" >> /etc/postgresql/9.3/main/pg_hba.conf &&\
    echo "listen_addresses='*'" >> /etc/postgresql/9.3/main/postgresql.conf &&\
    echo "ssl=false" >> /etc/postgresql/9.3/main/postgresql.conf &&\
    echo "plperl.on_init = 'use utf8; use re; package utf8; require \"utf8_heavy.pl\";'" >> /etc/postgresql/9.3/main/postgresql.conf

# sphinx
RUN curl -o /root/sphinx.deb http://sphinxsearch.com/files/sphinxsearch_2.1.9-release-0ubuntu11~precise_amd64.deb &&\
    apt-get -y -q install libodbc1 unixodbc &&\
    dpkg -i /root/sphinx.deb &&\
    rm /root/sphinx.deb

# redis
RUN apt-get -y install redis-server &&\
    echo "/etc/init.d/redis-server start" >> /root/init

# memcached
RUN apt-get -y install memcached &&\
    echo "/etc/init.d/memcached start" >> /root/init

# Project DB
ADD db/structure/ /root/db/structure/
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
