FROM ruby:2.3.8-jessie

RUN apt-get update -qq \
  && DEBIAN_FRONTEND=noninteractive apt-get install -yq --no-install-recommends \
    locales \
    make \
    gcc \
    g++ \
    curl \
    less \
    libxml2-dev \
    libxslt-dev \
    pkg-config \
    libcurl3-dev \
    libssl-dev \
    libgmp3-dev \
    git-core \
    ssh \
    libmagickcore-dev \
    libmagickwand-dev \
    imagemagick \
    libsqlite3-dev \
    catdoc \
    enca \
    ca-certificates \
    apt-transport-https \
    libmysqlclient-dev \
    build-essential \
    vim \
    gnupg2 \
    tzdata \
    dirmngr \
    netcat \
  && curl -L https://www.postgresql.org/media/keys/ACCC4CF8.asc | apt-key add - \
  && echo "deb https://apt-archive.postgresql.org/pub/repos/apt jessie-pgdg main 11" > /etc/apt/sources.list.d/pgdg.list \
  && apt-get update -qq \
  && DEBIAN_FRONTEND=noninteractive apt-get install -yq --no-install-recommends \
    postgresql-client-11 \
    libpq-dev \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* \
  && truncate -s 0 /var/log/*log

RUN curl -s https://deb.nodesource.com/gpgkey/nodesource.gpg.key | apt-key add - \
  && curl -fsSL https://deb.nodesource.com/setup_14.x | bash - ;\
  apt-get install -y nodejs;

RUN echo "Asia/Yekaterinburg" > /etc/timezone \
  && dpkg-reconfigure -f noninteractive tzdata \
  && locale-gen "en_US.UTF-8" \
  && dpkg-reconfigure --frontend=noninteractive locales

ENV LANG en_US.UTF-8
ENV LC_ALL en_US.UTF-8
ENV BUNDLE_SILENCE_ROOT_WARNING 1

# https://github.com/bundler/bundler/issues/4402
RUN echo 'gem: --no-rdoc --no-ri --no-document' > /root/.gemrc \
  && ssh-keyscan -H github.com >> /etc/ssh/ssh_known_hosts \
  && rm /usr/local/lib/ruby/gems/2.3.0/specifications/default/bundler*.gemspec \
  && gem install bundler --version 1.17.3 \
  && gem install executable-hooks -v ">=1.3.2" \
  && gem regenerate_binstubs \
  && bundle config --global jobs 7 \
  && bundle config git.allow_insecure true \
  && bundle config --global build.nokogiri --use-system-libraries

RUN set -eux; \
  \
echo "\
0	string		\<svg			SVG Scalable Vector Graphics image\n\
!:mime	image/svg\n\
" >> /etc/magic

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
