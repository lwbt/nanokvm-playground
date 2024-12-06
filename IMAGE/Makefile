# cspell:ignore nanokvm github linux buildroot libavutil extralibs vfat czvf mips

#SHELL := /bin/bash

RISCV64_CROSS_COMPILER=riscv64-linux-musl-cross
BASE_REPO=LicheeSG-Nano-Build

REMOTE_URL := $(shell git config --get remote.origin.url)
REPOSITORY_NAME := $(shell basename "$(REMOTE_URL)" .git)
COMPONENT=Custom-OS-Image
# TODO: This needs to be escaped? It still prints the entire line.
#VERSION := $(shell awk '/FFMPEG:/ {print $2}' ../VERSIONS)
#VERSION := $(shell grep  'IMAGE:' ../VERSIONS | cut -f 2 -d' ')
BUILD_DATE := $(shell date -I)
GH_ACCOUNT := $(shell gh auth status --active | grep "Logged in to github.com account" | cut -d " " -f 9)

IMAGE_VERSION := 0.1
RELEASE_VERSION := $(IMAGE_VERSION)

.PHONY: all
all: $(RISCV64_CROSS_COMPILER) git-clone-base git-clone-submodules build-nanokvm

.PHONY: test
test:
	@echo "Test?"
.PHONY: clean
clean:
	@echo "Test?"

.PHONY: git-clone-base
git-clone-base: $(BASE_REPO)

$(BASE_REPO):
	git clone https://github.com/scpcom/$(BASE_REPO) --depth=1
	ln -sv ../$(RISCV64_CROSS_COMPILER) $(BASE_REPO)/$(RISCV64_CROSS_COMPILER)

.PHONY: git-clone-submodules
git-clone-submodules:
	cd $(BASE_REPO) && \
	git submodule update --init --recursive --depth=1

.PHONY: git-reset-submodules
git-reset-submodules:
	git submodule foreach --recursive 'git clean -fdx'
	git submodule foreach --recursive 'git reset --hard'
	git submodule update --init --recursive

# NOTE: Shorthands for some annoying issues.
.PHONY: git-fix-submodules
git-fix-submodules:
	cd $(BASE_REPO)/linux_5.10 && \
	git tag --delete $(shell date +%Y%m%d)

.PHONY: prepare-host
prepare-host:
	cd $(BASE_REPO) && \
	./host/prepare-host.sh

.PHONY: update-ffmpeg-definition
update-ffmpeg-definition:
	cp -v ffmpeg.mk $(BASE_REPO)/buildroot/package/ffmpeg/ffmpeg.mk
	rm -v $(BASE_REPO)/buildroot/package/ffmpeg/ffmpeg.hash
# on 6.1
#	rm -v $(BASE_REPO)/buildroot/package/ffmpeg/0003-libavutil-Fix-mips-build.patch
	rm -v $(BASE_REPO)/buildroot/package/ffmpeg/0004-configure-add-extralibs-to-extralibs_xxx.patch
	rm -v $(BASE_REPO)/buildroot/package/ffmpeg/0005-Fixes-assembling-with-binutils-as-2.41.patch
# on 7.1
#	rm -v $(BASE_REPO)/buildroot/package/ffmpeg/*.patch

# TODO:
# NOTE: This does not work yet, you have to enter the commands below by hand.
#  export PATH=${PWD}/riscv64-linux-musl-cross/bin:$PATH
#  cd LicheeSG-Nano-Build && time ./build-nanokvm.sh
.PHONY: build-nanokvm
build-nanokvm:
	cd $(BASE_REPO) && \
	export PATH="$(PWD)/$(RISCV64_CROSS_COMPILER)/bin:$(PATH)" && \
	time ./build-nanokvm.sh

$(RISCV64_CROSS_COMPILER).tgz:
	wget https://musl.cc/$(RISCV64_CROSS_COMPILER).tgz

# NOTE: The link is created for convenience to be less strict about extending
# the PATH variable.
$(RISCV64_CROSS_COMPILER): $(RISCV64_CROSS_COMPILER).tgz
	tar xf $(RISCV64_CROSS_COMPILER).tgz

package-image:
	mv -v \
	$(BASE_REPO)/install/soc_sg2002_licheervnano_sd/images/*.img \
	$(BASE_REPO)/install/soc_sg2002_licheervnano_sd/images/boot.vfat \
	-t .
	tar -czvf nanokvm_$(shell date +%Y%m%d-%H%M).tar.gz *.img boot.vfat
	rm -v *.img boot.vfat

.PHONY: gh-auth-refresh
gh-auth-refresh:
	gh auth refresh

# Only use this when you have bundles built for both platforms.
.PHONY: release
release:
	gh release create $(COMPONENT)/v$(RELEASE_VERSION)-$(BUILD_DATE) \
	  --repo $(GH_ACCOUNT)/$(REPOSITORY_NAME) \
	  --title "$(COMPONENT) v$(RELEASE_VERSION) $(BUILD_DATE)" \
	  --notes "Update $(REPOSITORY_NAME)/$(COMPONENT) to v$(RELEASE_VERSION) $(BUILD_DATE). These are not CI/CD releases! The assets have been built on my workstation." \
	  --prerelease=false \
	  --draft \
	  nanokvm_*.gz
