# === Armbian 兼容的 GPU Makefile ===
# 强制设置必要变量
KERNEL_SRC_DIR := $(srctree)
KERNEL_OUT_DIR := $(O)
CONFIG_OS_TYPE := linux
CPU_ARCH := $(shell echo $(ARCH) | sed 's/arm64/aarch64/')

# 强制 GPU 类型
GPU_TYPE := bxm
GPU_BUILD_TYPE := release

# 模块安装路径（extra 目录）
MODULE_INS_DIR ?= $(INSTALL_MOD_PATH)/lib/modules/$(shell $(MAKE) -s -C $(srctree) kernelversion)/extra

# 避免错误中断
ifeq ($(KERNEL_SRC_DIR),)
  $(error KERNEL_SRC_DIR not set, but should be $(srctree))
endif

export KDIR = $(KERNEL_SRC_DIR)

# 读取 .config
-include .config

# 架构检测
ifeq ($(CONFIG_ARM64), y)
  CPU_ARCH := arm64
else ifeq ($(CONFIG_ARM), y)
  CPU_ARCH := arm
else
  $(warning Unsupported ARCH: $(ARCH))
  GPU_TYPE := unsupport
endif

# GPU 类型优先级
GPU_TYPE := $(word 1, $(GPU_TYPE) $(CONFIG_AW_GPU_TYPE))

# ==============================
#       GPU 构建路径设置
# ==============================
GPU_BUILD_DIR :=
GPU_KO_NAME :=
KO_DIR :=

ifneq ($(findstring $(GPU_TYPE), "bxm"),)
  GPU_BUILD_DIR = img-bxm/$(CONFIG_OS_TYPE)/rogue_km/build/linux/sunxi_$(CONFIG_OS_TYPE)

  ifeq ($(CONFIG_OS_TYPE), linux)
    ifeq ($(CPU_ARCH), arm)
      KO_DIR := img-bxm/$(CONFIG_OS_TYPE)/rogue_km/binary_sunxi_$(CONFIG_OS_TYPE)_nulldrmws_$(GPU_BUILD_TYPE)/target_armv7-a/kbuild
    else
      KO_DIR := img-bxm/$(CONFIG_OS_TYPE)/rogue_km/binary_sunxi_$(CONFIG_OS_TYPE)_nulldrmws_$(GPU_BUILD_TYPE)/target_aarch64/kbuild
    endif
  endif

  GPU_KO_NAME = $(KO_DIR)/pvrsrvkm.ko
endif

# ==============================
#       构建目标
# ==============================
.PHONY: all build modules_install clean

ifneq ($(GPU_BUILD_DIR),)

build:
	$(MAKE) -C $(GPU_BUILD_DIR) BUILD=$(GPU_BUILD_TYPE)

modules_install: $(GPU_KO_NAME)
	@if [ -z "$(MODULE_INS_DIR)" ]; then \
		echo "MODULE_INS_DIR not configured, nothing to do"; \
	else \
		mkdir -p $(MODULE_INS_DIR); \
		echo "Installing GPU module to $(MODULE_INS_DIR)"; \
		cp -v $(GPU_KO_NAME) $(MODULE_INS_DIR); \
	fi

clean:
	$(MAKE) -C $(GPU_BUILD_DIR) clean

else
all:
	@echo "Nothing to do for unsupport configuration"

modules_install: all
clean: all
endif
