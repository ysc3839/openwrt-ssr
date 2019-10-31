#
# Copyright (C) 2017-2019 Yousong Zhou <yszhou4tech@gmail.com>
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#

include $(TOPDIR)/rules.mk

PKG_NAME:=shadowsocksr-libev
PKG_RELEASE:=1

PKG_SOURCE_PROTO:=git
PKG_SOURCE_URL:=https://github.com/shadowsocksrr/shadowsocksr-libev.git
PKG_SOURCE_DATE:=2018-03-07
PKG_SOURCE_VERSION:=d63ff863800a5645aca4309d5dd5962bd1e95543
PKG_MIRROR_HASH:=4d81ae70d966e54e510bf9b24627911ee2bf6b22b8bc0c3c34b7bf68e135344b

PKG_LICENSE:=GPL-3.0-or-later
PKG_LICENSE_FILES:=LICENSE

PKG_FIXUP:=autoreconf
PKG_INSTALL:=1
PKG_USE_MIPS16:=0
PKG_BUILD_PARALLEL:=1

include $(INCLUDE_DIR)/package.mk


define Package/shadowsocksr-libev-config
  SECTION:=net
  CATEGORY:=Network
  SUBMENU:=Web Servers/Proxies
  TITLE:=shadowsocksr-libev config scripts
  URL:=https://github.com/shadowsocksrr/shadowsocksr-libev
endef

define Package/shadowsocksr-libev-config/install
	$(INSTALL_DIR) $(1)/etc/config
	$(INSTALL_DATA) ./files/shadowsocksr-libev.config $(1)/etc/config/shadowsocksr-libev
	$(INSTALL_DIR) $(1)/etc/init.d
	$(INSTALL_BIN) ./files/shadowsocksr-libev.init $(1)/etc/init.d/shadowsocksr-libev
	$(INSTALL_DIR) $(1)/usr/bin
	$(INSTALL_BIN) ./files/ssr-sub $(1)/usr/bin
endef


define Package/shadowsocksr-libev/Default
  define Package/shadowsocksr-libev-$(1)
    SECTION:=net
    CATEGORY:=Network
    SUBMENU:=Web Servers/Proxies
    TITLE:=shadowsocksr-libev $(1)
    URL:=https://github.com/shadowsocksrr/shadowsocksr-libev
    VARIANT:=$(1)
    DEPENDS:=+libev +libpthread +libsodium +libudns +libpcre +shadowsocksr-libev-config $(DEPENDS_$(1))
  endef

  define Package/shadowsocksr-libev-$(1)/install
	$$(INSTALL_DIR) $$(1)/usr/bin
	$$(INSTALL_BIN) $$(PKG_INSTALL_DIR)/usr/bin/ss-local $$(1)/usr/bin/ssr-local
	$$(INSTALL_BIN) $$(PKG_INSTALL_DIR)/usr/bin/ss-redir $$(1)/usr/bin/ssr-redir
	$$(LN) ssr-local $$(1)/usr/bin/ssr-tunnel
  endef

  ifeq ($$(BUILD_VARIANT),$(1))
	CONFIGURE_ARGS += $(CONFIGURE_$(1))
  endif

endef

DEPENDS_openssl = +libopenssl
DEPENDS_mbedtls = +libmbedtls
CONFIGURE_mbedtls = --with-crypto-library=mbedtls

SHADOWSOCKSR_VARIANTS:=openssl mbedtls
define shadowsocksr-libev/templates
  $(foreach variant,$(SHADOWSOCKSR_VARIANTS),
    $(call Package/shadowsocksr-libev/Default,$(variant))
  )
endef
$(eval $(call shadowsocksr-libev/templates))


define Package/shadowsocksr-libev-rules
  SECTION:=net
  CATEGORY:=Network
  SUBMENU:=Web Servers/Proxies
  TITLE:=shadowsocksr-libev ssr-rules
  URL:=https://github.com/shadowsocksrr/shadowsocksr-libev
  DEPENDS:=+ip +ipset +iptables-mod-tproxy +resolveip +shadowsocksr-libev-config
endef

define Package/shadowsocksr-libev-rules/install
	$(INSTALL_DIR) $(1)/usr/bin
	$(INSTALL_BIN) ./files/ssr-rules $(1)/usr/bin
	$(INSTALL_DIR) $(1)/etc/uci-defaults
	$(INSTALL_DATA) ./files/firewall.ssr-rules $(1)/etc
	$(INSTALL_BIN) ./files/ssr-rules.defaults $(1)/etc/uci-defaults
endef

define Package/shadowsocksr-libev-rules/prerm
#!/bin/sh
s=firewall.ss_rules
uci get "$$s" >/dev/null || exit 0
uci batch <<-EOF
	delete $$s
	commit firewall
EOF
endef

define Build/Prepare
	$(call Build/Prepare/Default)
	$(FIND) $(PKG_BUILD_DIR) \
			   -name '*.o' \
			-o -name '*.lo' \
			-o -name '.deps' \
			-o -name '.libs' \
		| $(XARGS) rm -rvf
endef

CONFIGURE_ARGS += \
	--disable-documentation \
	--disable-silent-rules \
	--disable-assert \
	--disable-ssp \
	--enable-system-shared-lib \

TARGET_CFLAGS += -flto
TARGET_LDFLAGS += -Wl,--gc-sections,--as-needed

$(eval $(call BuildPackage,shadowsocksr-libev-config))
$(eval $(call BuildPackage,shadowsocksr-libev-rules))
$(foreach variant,$(SHADOWSOCKSR_VARIANTS), \
  $(eval $(call BuildPackage,shadowsocksr-libev-$(variant))) \
)
