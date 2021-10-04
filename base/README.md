<h1 align="center">Base images for build</h1>

## Usage

### Xenial+RVM

```bash
docker build --build-arg RVM_RUBY=ruby-2.3.8 -t abakpress/ruby-app:2.3-xenial-latest -f Dockerfile.xenial .

docker build --build-arg RVM_RUBY=ruby-2.4.10 -t abakpress/ruby-app:2.4-xenial-latest -f Dockerfile.xenial .

docker build --build-arg RVM_RUBY=ruby-2.5.8 -t abakpress/ruby-app:2.5-xenial-latest -f Dockerfile.xenial .
```