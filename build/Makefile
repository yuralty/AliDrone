################################################################################
# Following variables defines how the NS_USER (Non Secure User - Client
# Application), NS_KERNEL (Non Secure Kernel), S_KERNEL (Secure Kernel) and
# S_USER (Secure User - TA) are compiled
################################################################################
override COMPILE_NS_USER   := 32
override COMPILE_NS_KERNEL := 32
override COMPILE_S_USER    := 32
override COMPILE_S_KERNEL  := 32

-include common.mk

################################################################################
# Paths to git projects and various binaries
################################################################################
BIOS_QEMU_PATH			?= $(ROOT)/bios_qemu_tz_arm
QEMU_PATH			?= $(ROOT)/qemu

SOC_TERM_PATH			?= $(ROOT)/soc_term

DEBUG = 1

################################################################################
# Targets
################################################################################
all: bios-qemu qemu soc-term
all-clean: bios-qemu-clean busybox-clean linux-clean optee-os-clean \
	optee-client-clean qemu-clean soc-term-clean check-clean

-include toolchain.mk

################################################################################
# QEMU
################################################################################
define bios-qemu-common
	+$(MAKE) -C $(BIOS_QEMU_PATH) \
		CROSS_COMPILE=$(CROSS_COMPILE_NS_USER) \
		O=$(ROOT)/out/bios-qemu \
		BIOS_NSEC_BLOB=$(LINUX_PATH)/arch/arm/boot/zImage \
		BIOS_NSEC_ROOTFS=$(GEN_ROOTFS_PATH)/filesystem.cpio.gz \
		BIOS_SECURE_BLOB=$(OPTEE_OS_BIN) \
		PLATFORM_FLAVOR=virt
endef

bios-qemu: update_rootfs optee-os
	$(call bios-qemu-common)

bios-qemu-clean:
	$(call bios-qemu-common) clean

qemu:
	cd $(QEMU_PATH); ./configure --target-list=arm-softmmu\
			$(QEMU_CONFIGURE_PARAMS_COMMON)
	$(MAKE) -C $(QEMU_PATH)

qemu-clean:
	$(MAKE) -C $(QEMU_PATH) distclean

################################################################################
# Busybox
################################################################################
BUSYBOX_COMMON_TARGET = vexpress
BUSYBOX_CLEAN_COMMON_TARGET = vexpress clean

busybox: busybox-common

busybox-clean: busybox-clean-common

busybox-cleaner: busybox-cleaner-common

################################################################################
# Linux kernel
################################################################################
LINUX_DEFCONFIG_COMMON_ARCH := arm
LINUX_DEFCONFIG_COMMON_FILES := \
		$(LINUX_PATH)/arch/arm/configs/vexpress_defconfig \
		$(CURDIR)/kconfigs/qemu.conf

linux-defconfig: $(LINUX_PATH)/.config

LINUX_COMMON_FLAGS += ARCH=arm

linux: linux-common

linux-defconfig-clean: linux-defconfig-clean-common

LINUX_CLEAN_COMMON_FLAGS += ARCH=arm

linux-clean: linux-clean-common

LINUX_CLEANER_COMMON_FLAGS += ARCH=arm

linux-cleaner: linux-cleaner-common

################################################################################
# OP-TEE
################################################################################
OPTEE_OS_COMMON_FLAGS += PLATFORM=vexpress-qemu_virt
optee-os: optee-os-common

OPTEE_OS_CLEAN_COMMON_FLAGS += PLATFORM=vexpress-qemu_virt
optee-os-clean: optee-os-clean-common

optee-client: optee-client-common

optee-client-clean: optee-client-clean-common

################################################################################
# Soc-term
################################################################################
soc-term:
	$(MAKE) -C $(SOC_TERM_PATH)

soc-term-clean:
	$(MAKE) -C $(SOC_TERM_PATH) clean

################################################################################
# xtest / optee_test
################################################################################
xtest: xtest-common

xtest-clean: xtest-clean-common

xtest-patch: xtest-patch-common

################################################################################
# hello_world
################################################################################
#helloworld: helloworld-common

#helloworld-clean: helloworld-clean-common

################################################################################
# sgps
################################################################################
sgps: sgps-common

sgps-clean: sgps-clean-common

################################################################################
# Root FS
################################################################################
filelist-tee: filelist-tee-common

update_rootfs: update_rootfs-common

################################################################################
# Run targets
################################################################################
.PHONY: run
# This target enforces updating root fs etc
run: all
	$(MAKE) run-only

.PHONY: run-only
run-only:
	$(call check-terminal)
	$(call run-help)
	$(call launch-terminal,54320,"Normal World")
	$(call launch-terminal,54321,"Secure World")
	$(call wait-for-ports,54320,54321)
	$(QEMU_PATH)/arm-softmmu/qemu-system-arm \
		-nographic \
		-serial tcp:localhost:54320 -serial tcp:localhost:54321 \
		-s -S -machine virt -machine secure=on -cpu cortex-a15 \
		-m 1057 \
		-bios $(ROOT)/out/bios-qemu/bios.bin \
		$(QEMU_EXTRA_ARGS)


ifneq ($(filter check,$(MAKECMDGOALS)),)
CHECK_DEPS := all
endif

check-args := --bios $(ROOT)/out/bios-qemu/bios.bin
ifneq ($(TIMEOUT),)
check-args += --timeout $(TIMEOUT)
endif

check: $(CHECK_DEPS)
	expect qemu-check.exp -- $(check-args) || \
		(if [ "$(DUMP_LOGS_ON_ERROR)" ]; then \
			echo "== $$PWD/serial0.log:"; \
			cat serial0.log; \
			echo "== end of $$PWD/serial0.log:"; \
			echo "== $$PWD/serial1.log:"; \
			cat serial1.log; \
			echo "== end of $$PWD/serial1.log:"; \
		fi; false)

check-only: check

check-clean:
	rm -f serial0.log serial1.log
