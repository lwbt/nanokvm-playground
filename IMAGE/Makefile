#SHELL := /bin/bash

RISCV64_CROSS_COMPILER=riscv64-linux-musl-cross
BASE_REPO=LicheeSG-Nano-Build

.PHONY: all
all: $(RISCV64_CROSS_COMPILER) git-clone-base git-clone-submodules build-nanokvm

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
