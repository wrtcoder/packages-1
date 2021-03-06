#
# Copyright (C) 2010 OpenWrt.org
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#
# $Id: Makefile 5624 2006-11-23 00:29:07Z nbd $

include $(TOPDIR)/rules.mk

PKG_NAME:=batman-adv-legacy

PKG_VERSION:=2014-06-10
BATCTL_VERSION:=2013.4.0
BATCTL_MD5SUM:=42e269cc710bbc9a8fd17628201d4258

PKG_SOURCE_PROTO:=git
PKG_SOURCE_URL:=git://github.com/freifunk-gluon/batman-adv-legacy.git
PKG_SOURCE_SUBDIR:=$(PKG_NAME)-$(PKG_VERSION)
PKG_SOURCE_VERSION:=cf06cf52ce31318419dea72f3296c50578f0255e
PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION)-$(PKG_SOURCE_VERSION).tar.gz

BATCTL_SOURCE_URL:=http://downloads.open-mesh.org/batman/releases/batman-adv-$(BATCTL_VERSION)

PKG_BUILD_DIR:=$(BUILD_DIR)/$(PKG_NAME)/$(PKG_NAME)-$(PKG_VERSION)
PKG_BATCTL_BUILD_DIR:=$(BUILD_DIR)/$(PKG_NAME)/batctl-$(BATCTL_VERSION)

include $(INCLUDE_DIR)/package.mk
include $(INCLUDE_DIR)/kernel.mk

define KernelPackage/batman-adv-legacy
  URL:=http://www.open-mesh.org/
  MAINTAINER:=Marek Lindner <lindner_marek@yahoo.de>
  SUBMENU:=Network Support
  DEPENDS:=+kmod-lib-crc16 +kmod-crypto-core +kmod-crypto-crc32c +kmod-lib-crc32c +libc
  TITLE:=B.A.T.M.A.N. Adv
  FILES:=$(PKG_BUILD_DIR)/batman-adv.$(LINUX_KMOD_SUFFIX)
  AUTOLOAD:=$(call AutoLoad,50,batman-adv)
endef

define KernelPackage/batman-adv-legacy/description
B.A.T.M.A.N. advanced is a kernel module which allows to
build layer 2 mesh networks.
endef

define KernelPackage/batman-adv-legacy/config
	source "$(SOURCE)/Config.in"
endef

MAKE_BATMAN_ADV_ARGS += \
	CROSS_COMPILE="$(TARGET_CROSS)" \
	KERNELPATH="$(LINUX_DIR)" \
	ARCH="$(LINUX_KARCH)" \
	PATH="$(TARGET_PATH)" \
	SUBDIRS="$(PKG_BUILD_DIR)" \
	PWD="$(PKG_BUILD_DIR)" \
	LINUX_VERSION="$(LINUX_VERSION)" \
	CONFIG_BATMAN_ADV_DEBUG=$(if $(CONFIG_KMOD_BATMAN_ADV_LEGACY_DEBUG_LOG),y,n) \
	CONFIG_BATMAN_ADV_BLA=$(if $(CONFIG_KMOD_BATMAN_ADV_LEGACY_BLA),y,n) \
	CONFIG_BATMAN_ADV_DAT=$(if $(CONFIG_KMOD_BATMAN_ADV_LEGACY_DAT),y,n) \
	CONFIG_BATMAN_ADV_NC=$(if $(CONFIG_KMOD_BATMAN_ADV_LEGACY_NC),y,n) \
	REVISION="" all

# The linker can identify unused sections of a binary when each symbol is stored
# in a separate section. This mostly removes unused linker sections and reduces
# the size by ~3% on mipsel.

TARGET_CFLAGS  += -ffunction-sections -fdata-sections
TARGET_LDFLAGS += -Wl,--gc-sections

# Link-time optimization allows to move parts of the optimization from the single
# source file to the global source view. This is done by emitting the GIMPLE
# representation in each object file and analyzing it again during the link step.

TARGET_CFLAGS  += -flto
TARGET_LDFLAGS += -fuse-linker-plugin

MAKE_BATCTL_ENV += \
	CPPFLAGS="$(TARGET_CPPFLAGS)" \
	CFLAGS="$(TARGET_CFLAGS)" \
	LDFLAGS="$(TARGET_LDFLAGS)"

MAKE_BATCTL_ARGS += \
	REVISION="" \
	CC="$(TARGET_CC)" \
	DESTDIR="$(PKG_INSTALL_DIR)" \
	batctl install

ifneq ($(DEVELOPER)$(CONFIG_KMOD_BATMAN_ADV_LEGACY_BATCTL),)
define Download/batctl-legacy
  FILE:=batctl-$(BATCTL_VERSION).tar.gz
  URL:=$(BATCTL_SOURCE_URL)
  MD5SUM:=$(BATCTL_MD5SUM)
endef
$(eval $(call Download,batctl-legacy))

BATCTL_EXTRACT = tar xzf "$(DL_DIR)/batctl-$(BATCTL_VERSION).tar.gz" -C "$(BUILD_DIR)/$(PKG_NAME)"
BATCTL_PATCH = $(call Build/DoPatch,"$(PKG_BATCTL_BUILD_DIR)","$(PATCH_DIR)","*batctl*")
BATCTL_BUILD = $(MAKE_BATCTL_ENV) $(MAKE) -C $(PKG_BATCTL_BUILD_DIR) $(MAKE_BATCTL_ARGS)
BATCTL_INSTALL = $(INSTALL_BIN) $(PKG_INSTALL_DIR)/usr/local/sbin/batctl $(1)/usr/sbin/
endif

KPATCH ?= $(PATCH)
define Build/DoPatch
	@if [ -d "$(2)" ]; then \
		if [ "$$$$(ls $(2) | grep -Ec $(3))" -gt 0 ]; then \
			$(KPATCH) "$(1)" "$(2)" "$(3)"; \
		fi; \
	fi
endef

define Build/Patch
	$(call Build/DoPatch,"$(PKG_BUILD_DIR)","$(PATCH_DIR)","*batman*")
	$(BATCTL_EXTRACT)
	$(BATCTL_PATCH)
endef

define Build/Prepare
	$(call Build/Prepare/Default)
	$(SED) '/#define _NET_BATMAN_ADV_MAIN_H_/a\#undef CONFIG_MODULE_STRIPPED' \
		$(PKG_BUILD_DIR)/main.h
endef


define Build/Compile
	$(MAKE) -C "$(PKG_BUILD_DIR)" $(MAKE_BATMAN_ADV_ARGS)
	$(BATCTL_BUILD)
endef

define Build/Clean
	rm -rf $(BUILD_DIR)/$(PKG_NAME)/
endef

define KernelPackage/batman-adv-legacy/install
	$(INSTALL_DIR) $(1)/etc/config $(1)/etc/hotplug.d/net $(1)/etc/hotplug.d/iface $(1)/lib/batman-adv $(1)/usr/sbin $(1)/lib/netifd/proto
	$(INSTALL_DATA) ./files/etc/config/batman-adv $(1)/etc/config
	$(INSTALL_DATA) ./files/lib/batman-adv/config.sh $(1)/lib/batman-adv
	$(INSTALL_BIN) ./files/etc/hotplug.d/net/99-batman-adv $(1)/etc/hotplug.d/net
	$(INSTALL_BIN) ./files/lib/netifd/proto/batadv.sh $(1)/lib/netifd/proto
	$(BATCTL_INSTALL)
endef

$(eval $(call KernelPackage,batman-adv-legacy))
