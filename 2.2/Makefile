PROJECT = ci
DEPLOY = docker
RAILS_ENV = test
SPHINX_PATH = tmp/sphinx
BUNDLE = RAILS_ENV=${RAILS_ENV} bundle
BUNDLE_OPTIONS = -j 3 --quiet --without=development
BUNDLER_VERSION = 1.17.3
BUILD_OPTIONS = VERBOSE=1 NO_VERBOSE_RAILS_LOGS=1
RAKE = ${BUNDLE} exec rake

all: test

test: package/symlinks bundler/install project/env sphinx/env
	${BUILD_OPTIONS} ${RAKE} --trace build 2>&1

package/symlinks:
	DEPLOY=${DEPLOY} PROJECT=${PROJECT} script/config_env

bundler/install:
	gem list -i bundler -v ${BUNDLER_VERSION} > /dev/null || gem install bundler -v ${BUNDLER_VERSION}
	${BUNDLE} install ${BUNDLE_OPTIONS}

project/env:
	mkdir -p tmp

sphinx/env:
	mkdir -p db/sphinx/test
	mkdir -p db/sphinx/test/binlog
