# User configuration
WGET = wget
BLOCK_DEVICE ?= /dev/null
FIND ?= find

ARCH_TARBALL = ArchLinuxARM-rpi-3-latest.tar.gz

MOUNT_POINT_ROOT = mnt_root
MOUNT_POINT_BOOT = mnt_boot

ALL = $(ARCH_TARBALL)

all: $(ALL)

$(ARCH_TARBALL):
	$(WGET) http://archlinuxarm.org/os/$@

define part
$(foreach pf,$(shell $(FIND) /sys/block/$(shell basename $(1))/ -maxdepth 2 -name "partition"),$(if $(shell grep ^$(2)$$ $(pf)),/dev/$(shell basename $(pf:/partition=))))
endef

define prepare_mount_point
mkdir -p $(2) || true
sudo umount $(2) || true
sudo mount $(1) $(2)
endef

define remove_mount_point
sudo umount $(1) || true
sudo rmdir $(1) || true
endef

install: P1=$(call part,$(BLOCK_DEVICE),1)
install: P2=$(call part,$(BLOCK_DEVICE),2)
install: $(ALL) fdisk.cmd
ifeq ($(BLOCK_DEVICE),/dev/null)
	@echo You must set BLOCK_DEVICE option
else
	sudo dd if=/dev/zero of=$(BLOCK_DEVICE) bs=1M count=8
	sudo fdisk $(BLOCK_DEVICE) < fdisk.cmd
	sync
	sudo mkfs.vfat $(P1)
	sudo mkfs.ext4 $(P2)
	$(call prepare_mount_point,$(P2),$(MOUNT_POINT_ROOT))
	sudo bsdtar -xpf $(ARCH_TARBALL) -C $(MOUNT_POINT_ROOT)
	sync
	$(call prepare_mount_point,$(P1),$(MOUNT_POINT_BOOT))
	sudo mv $(MOUNT_POINT_ROOT)/boot/* $(MOUNT_POINT_BOOT)
	sync
	$(call remove_mount_point,$(MOUNT_POINT_BOOT))
	$(call remove_mount_point,$(MOUNT_POINT_ROOT))
endif

clean:
	$(RM) $(ALL)

.PHONY: all clean install
