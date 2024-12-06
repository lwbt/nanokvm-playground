# Makefile for Building Static FFmpeg for RISC-V
# Requires: make, wget, tar, riscv64-linux-gnu-gcc

# cspell:ignore ranlib sysroot bzlib alsa bugu buildroot github

# TODO: Where it the riscv64 toolchain we were supposed to compile before?

SHELL := /bin/bash

REMOTE_URL := $(shell git config --get remote.origin.url)
REPOSITORY_NAME := $(shell basename "$(REMOTE_URL)" .git)
COMPONENT := FFmpeg
# TODO: This needs to be escaped? It still prints the entire line.
#VERSION := $(shell awk '/FFMPEG:/ {print $2}' ../VERSIONS)
VERSION := $(shell grep  'FFMPEG:' ../VERSIONS | cut -f 2 -d' ')
BUILD_DATE := $(shell date -I)
GH_ACCOUNT := $(shell gh auth status --active | grep "Logged in to github.com account" | cut -d " " -f 9)

# Configuration
RISCV_TRIPLE := riscv64-linux-gnu
CC := $(RISCV_TRIPLE)-gcc
CXX := $(RISCV_TRIPLE)-g++
AR := $(RISCV_TRIPLE)-ar
RANLIB := $(RISCV_TRIPLE)-ranlib

# Versions
ZLIB_VERSION := 1.3.1
#FFMPEG_VERSION := 6.1
FFMPEG_VERSION := 7.1

# Directories
BUILD_DIR := $(PWD)/build
CCACHE_DIR := $(PWD)/ccache
DOWNLOAD_DIR := $(PWD)/downloads
PREFIX := $(BUILD_DIR)/sysroot

# Flags
CFLAGS := -static -O3 -fPIC
LDFLAGS := -static

# Targets
.PHONY: all
#all: zlib ffmpeg
all: zlib ffmpeg-gcc-static

.PHONY: test
test:
	@echo "Test?"

# Clean everything
.PHONY: clean
clean:
	rm -rf $(DOWNLOAD_DIR) $(BUILD_DIR)

.PHONY: prerequisites
prerequisites:
	sudo apt-get update -y
	sudo apt-get install -y wget tar gcc-riscv64-linux-gnu
#        sudo apt-get install -y wget tar gcc-riscv64-linux-gnu texinfo bison flex

# Create necessary directories
$(DOWNLOAD_DIR) $(BUILD_DIR) $(CCACHE_DIR) $(PREFIX):
	mkdir -pv $@

# Download Zlib
$(DOWNLOAD_DIR)/zlib-$(ZLIB_VERSION).tar.gz: | $(DOWNLOAD_DIR)
	wget https://zlib.net/zlib-$(ZLIB_VERSION).tar.gz -O $@

# Extract Zlib
.PHONY: zlib-extract
zlib-extract: $(DOWNLOAD_DIR)/zlib-$(ZLIB_VERSION).tar.gz | $(BUILD_DIR)
	tar -xzf $< -C $(BUILD_DIR)

# Build Zlib
.PHONY: zlib
zlib: zlib-extract | $(PREFIX)
	@echo -e "\033[7m Building zlib \033[0m"
	cd $(BUILD_DIR)/zlib-$(ZLIB_VERSION) && \
	CC="$(CC)" CFLAGS="$(CFLAGS)" ./configure \
		--prefix=$(PREFIX) \
		--static && \
	make -j$$(nproc) && \
	make install

.PHONY: gh-auth-refresh
gh-auth-refresh:
	gh auth refresh

# Only use this when you have bundles built for both platforms.
.PHONY: release
release:
	gh release create $(COMPONENT)/v$(FFMPEG_VERSION)-$(BUILD_DATE) \
	  --repo $(GH_ACCOUNT)/$(REPOSITORY_NAME) \
	  --title "$(COMPONENT) v$(FFMPEG_VERSION) $(BUILD_DATE)" \
	  --notes "Update $(REPOSITORY_NAME)/$(COMPONENT) to v$(FFMPEG_VERSION) $(BUILD_DATE). These are not CI/CD releases! The assets have been built on my workstation." \
	  --prerelease=false \
	  --draft \
	  ffmpeg-gcc-static.gz

## Download FFmpeg
#$(DOWNLOAD_DIR)/ffmpeg-$(FFMPEG_VERSION).tar.bz2: | $(DOWNLOAD_DIR)
#	wget https://ffmpeg.org/releases/ffmpeg-$(FFMPEG_VERSION).tar.bz2 -O $@
#
## Extract FFmpeg
#ffmpeg-extract: $(DOWNLOAD_DIR)/ffmpeg-$(FFMPEG_VERSION).tar.bz2 | $(BUILD_DIR)
#	tar -xjf $< -C $(BUILD_DIR)

$(BUILD_DIR)/ffmpeg-$(FFMPEG_VERSION):
	git -c advice.detachedHead=false clone --depth 1 -b "release/$(FFMPEG_VERSION)" \
	  https://git.ffmpeg.org/ffmpeg.git $(BUILD_DIR)/ffmpeg-$(FFMPEG_VERSION)

# Build FFmpeg
# cspell:disable
ffmpeg-gcc-static: zlib $(BUILD_DIR)/ffmpeg-$(FFMPEG_VERSION)
	@echo -e "\033[7m Building FFmpeg \033[0m"
	@cd $(BUILD_DIR)/ffmpeg-$(FFMPEG_VERSION) && \
	./configure \
		--arch=riscv64 \
		--cpu=rv64imafdc \
		--cross-prefix=$(RISCV_TRIPLE)- \
		--disable-altivec \
		--disable-armv6 \
		--disable-armv6t2 \
		--disable-avisynth \
		--disable-avx \
		--disable-avx2 \
		--disable-cuda \
		--disable-cuvid \
		--disable-doc \
		--disable-dxva2 \
		--disable-extra-warnings \
		--disable-ffplay \
		--disable-ffprobe \
		--disable-fontconfig \
		--disable-frei0r \
		--disable-gnutls \
		--disable-gray \
		--disable-hardcoded-tables \
		--disable-iconv \
		--disable-libass \
		--disable-libbluray \
		--disable-libcdio \
		--disable-libdav1d \
		--disable-libdc1394 \
		--disable-libdrm \
		--disable-libfdk-aac \
		--disable-libfreetype \
		--disable-libgsm \
		--disable-libilbc \
		--disable-libmfx \
		--disable-libmodplug \
		--disable-libmp3lame \
		--disable-libopencore-amrnb \
		--disable-libopencore-amrwb \
		--disable-libopencv \
		--disable-libopenh264 \
		--disable-libopenjpeg \
		--disable-libopus \
		--disable-librtmp \
		--disable-libspeex \
		--disable-libtheora \
		--disable-libv4l2 \
		--disable-libvo-amrwbenc \
		--disable-libvpx \
		--disable-libx264 \
		--disable-libx265 \
		--disable-libxcb \
		--disable-mipsdsp \
		--disable-mipsdspr2 \
		--disable-mmal \
		--disable-mmx \
		--disable-msa \
		--disable-manpages \
		--disable-neon \
		--disable-nvenc \
		--disable-omx \
		--disable-omx-rpi \
		--disable-shared \
		--disable-small \
		--disable-sse \
		--disable-sse2 \
		--disable-sse3 \
		--disable-sse4 \
		--disable-sse42 \
		--disable-ssse3 \
		--disable-symver \
		--disable-vaapi \
		--disable-vdpau \
		--disable-version3 \
		--disable-vfp \
		--disable-x86asm \
		--enable-avcodec \
		--enable-avdevice \
		--enable-avfilter \
		--enable-avformat \
		--enable-cross-compile \
		--enable-ffmpeg \
		--enable-gpl \
		--enable-hwaccels \
		--enable-indevs \
		--enable-logging \
		--enable-network \
		--enable-nonfree \
		--enable-optimizations \
		--enable-outdevs \
		--enable-pic \
		--enable-postproc \
		--enable-pthreads \
		--enable-runtime-cpudetect \
		--enable-static \
		--enable-swscale \
		--enable-swscale-alpha \
		--enable-zlib \
		--extra-libs=-latomic \
		--extra-cflags="$(CFLAGS) -I$(PREFIX)/include" \
		--extra-ldflags="$(LDFLAGS) -L$(PREFIX)/lib" \
		--pkg-config-flags="--static" \
		--prefix=/usr \
		--sysroot=$(PREFIX) \
		--target-os=linux && \
	make -j$$(nproc) && \
	file ffmpeg && \
	cp -v ffmpeg ../../ffmpeg-gcc-static && \
	gzip --keep --force ../../ffmpeg-gcc-static
# cspell:enable

# Including this option makes the binary 100MB larger. I don't know how much this affects running on a 120MB RAM constrained platform, but I opt for stripping the binary for now.
#		--disable-stripping \
# These options did not compile, so I removed them.
#		--enable-bzlib \
#		--enable-alsa \
#		--enable-openssl \
#		--host-cc='$(CCACHE_DIR) /usr/bin/gcc' \
#		--pkg-config=/home/bugu/LicheeRV-Nano-Build/buildroot/output/per-package/ffmpeg/host/bin/pkg-config \
#	make install

## Initial code generated by Claude
# cspell:disable
#ffmpeg_org: zlib ffmpeg-extract | $(PREFIX)
#ffmpeg_1: zlib $(BUILD_DIR)/ffmpeg-$(FFMPEG_VERSION)
#	cd $(BUILD_DIR)/ffmpeg-$(FFMPEG_VERSION) && \
#	./configure \
#		--arch=riscv64 \
#		--cross-prefix=$(RISCV_TRIPLE)- \
#		--disable-doc \
#		--disable-ffplay \
#		--disable-ffprobe \
#		--disable-manpages \
#		--disable-shared \
#		--enable-gpl \
#		--enable-static \
#		--enable-version3 \
#		--extra-cflags="$(CFLAGS) -I$(PREFIX)/include" \
#		--extra-ldflags="$(LDFLAGS) -L$(PREFIX)/lib" \
#		--pkg-config-flags="--static" \
#		--prefix=$(PREFIX) \
#		--target-os=linux && \
#	make -j$$(nproc) && \
#	make install

## Reconstructed from `ffmpeg -buildconf`
#ffmpeg_2: zlib $(BUILD_DIR)/ffmpeg-$(FFMPEG_VERSION)
#	cd $(BUILD_DIR)/ffmpeg-$(FFMPEG_VERSION) && \
#	./configure \
#		--arch=riscv64 \
#		--cpu=rv64imafdc \
#		--cross-prefix=/home/bugu/LicheeRV-Nano-Build/buildroot/output/per-package/ffmpeg/host/bin/riscv64-unknown-linux-musl- \
#		--disable-altivec \
#		--disable-armv6 \
#		--disable-armv6t2 \
#		--disable-avisynth \
#		--disable-avx \
#		--disable-avx2 \
#		--disable-crystalhd \
#		--disable-cuda \
#		--disable-cuvid \
#		--disable-doc \
#		--disable-dxva2 \
#		--disable-extra-warnings \
#		--disable-ffplay \
#		--disable-fontconfig \
#		--disable-frei0r \
#		--disable-gnutls \
#		--disable-gray \
#		--disable-hardcoded-tables \
#		--disable-iconv \
#		--disable-libass \
#		--disable-libbluray \
#		--disable-libcdio \
#		--disable-libdav1d \
#		--disable-libdc1394 \
#		--disable-libdrm \
#		--disable-libfdk-aac \
#		--disable-libfreetype \
#		--disable-libgsm \
#		--disable-libilbc \
#		--disable-libmfx \
#		--disable-libmodplug \
#		--disable-libmp3lame \
#		--disable-libopencore-amrnb \
#		--disable-libopencore-amrwb \
#		--disable-libopencv \
#		--disable-libopenh264 \
#		--disable-libopenjpeg \
#		--disable-libopus \
#		--disable-librtmp \
#		--disable-libspeex \
#		--disable-libtheora \
#		--disable-libv4l2 \
#		--disable-libvo-amrwbenc \
#		--disable-libvpx \
#		--disable-libx264 \
#		--disable-libx265 \
#		--disable-libxcb \
#		--disable-mipsdsp \
#		--disable-mipsdspr2 \
#		--disable-mmal \
#		--disable-mmx \
#		--disable-msa \
#		--disable-neon \
#		--disable-nvenc \
#		--disable-omx \
#		--disable-omx-rpi \
#		--disable-small \
#		--disable-sse \
#		--disable-manpages \
#		--disable-sse2 \
#		--disable-sse3 \
#		--disable-sse4 \
#		--disable-sse42 \
#		--disable-ssse3 \
#		--disable-stripping \
#		--disable-symver \
#		--disable-vaapi \
#		--disable-vdpau \
#		--disable-version3 \
#		--disable-vfp \
#		--disable-x86asm \
#		--enable-alsa \
#		--enable-avcodec \
#		--enable-avdevice \
#		--enable-avfilter \
#		--enable-avformat \
#		--enable-avresample \
#		--enable-bzlib \
#		--enable-cross-compile \
#		--enable-dct \
#		--enable-ffmpeg \
#		--enable-ffprobe \
#		--enable-fft \
#		--enable-gpl \
#		--enable-hwaccels \
#		--enable-indevs \
#		--enable-logging \
#		--enable-mdct \
#		--enable-network \
#		--enable-nonfree \
#		--enable-optimizations \
#		--enable-outdevs \
#		--enable-pic \
#		--enable-postproc \
#		--enable-pthreads \
#		--enable-rdft \
#		--enable-runtime-cpudetect \
#		--enable-shared \
#		--enable-swscale \
#		--enable-swscale-alpha \
#		--enable-zlib \
#		--extra-libs=-latomic \
#		--host-cc='/home/bugu/LicheeRV-Nano-Build/buildroot/output/per-package/ffmpeg/host/bin/ccache /usr/bin/gcc' \
#		--pkg-config=/home/bugu/LicheeRV-Nano-Build/buildroot/output/per-package/ffmpeg/host/bin/pkg-config --disable-static \
#		--prefix=/usr \
#		--sysroot=/home/bugu/LicheeRV-Nano-Build/buildroot/output/per-package/ffmpeg/host/riscv64-buildroot-linux-musl/sysroot \
#		--target-os=linux \
#		--target-os=linux && \
#	make -j$$(nproc) && \
#	make install
# cspell:enable
