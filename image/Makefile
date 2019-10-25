.PHONY: build_%

# Current apache hadoop releases: http://hadoop.apache.org/releases.html

HADOOP_30_VERSION = 3.2.1

DOCKER = docker
DOCKER_REPO = chenseanxy/hadoop

all: build

### Build hadoop versioned images with native libs.
build: build-$(HADOOP_30_VERSION)
build-%: # hadoop-native-%.tar - Disabled lib building
	$(DOCKER) build -t hadoop:$*-nolib \
	--build-arg HADOOP_VERSION=$* \
	.

tag: tag-$(HADOOP_30_VERSION)
tag-%:
	$(DOCKER) tag hadoop:$*-nolib $(DOCKER_REPO):$*-nolib

push: push-$(HADOOP_30_VERSION)
push-%:
	$(DOCKER) push $(DOCKER_REPO):$*-nolib

### Fetch source from closest mirror
hadoop-%-src.tar.gz:
	curl -LO http://archive.apache.org/dist/hadoop/common/hadoop-$*/hadoop-$*-src.tar.gz

### Fetch binary distribution from closest mirror
hadoop-%.tar.gz:
	curl -LO http://archive.apache.org/dist/hadoop/common/hadoop-$*/hadoop-$*.tar.gz

### Fetch protobuf source
protobuf-%.tar.bz2:
	curl -LO https://github.com/google/protobuf/releases/download/v$*/protobuf-$*.tar.bz2

# Keep intermediate downloads.
.PRECIOUS: protobuf-%.tar.bz2 hadoop-%-src.tar.gz hadoop-%.tar.gz

### Compile native libs (~10min)
native_libs_%: hadoop-%-src.tar.gz protobuf-$(PROTOBUF_VERSION).tar.bz2
	$(DOCKER) build -f Dockerfile-compile -t hadoop-nativelibs:$*\
		--build-arg=HADOOP_VERSION=$* \
		--build-arg=PROTOBUF_VERSION=$(PROTOBUF_VERSION) \
		.

### Extract native libs from previous compile target
hadoop-native-%.tar: native_libs_% hadoop-%.tar.gz
	$(DOCKER) run --rm \
	 	-e HADOOP_VERSION=$* \
		hadoop-nativelibs:$* > hadoop-native-$*.tar
