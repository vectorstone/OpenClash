# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

OpenClash is a LuCI (Lua Configuration Interface) application for OpenWrt that provides a web UI for the Clash proxy client. It supports Shadowsocks, ShadowsocksR, Vmess, Trojan, Snell and other protocols.

## Build System

This project uses the OpenWrt SDK build system:

```bash
# Compile po2lmo (translation tool) - one time setup
cd luci-app-openclash/tools/po2lmo
make && sudo make install

# Build the package (within OpenWrt SDK)
make package/luci-app-openclash/compile V=99
```

The CI/CD workflow (`.github/workflows/compile_new_ipk.yml`) automatically builds IPK/APK packages on version changes.

## Architecture

### LuCI Framework Structure (MVC Pattern)

The web interface follows LuCI's MVC architecture under `luci-app-openclash/luasrc/`:

- **Controller** (`controller/openclash.lua`): Defines URL routes, menu entries, and dispatch handlers. Entry point for all web requests.
- **Model/CBI** (`model/cbi/openclash/*.lua`): Configuration Binding Interface forms - each file defines a configuration page mapped to UCI (Unified Configuration Interface) options.
- **View** (`view/openclash/*.htm`): HTML templates, mostly containing JavaScript for dynamic UI updates.
- **Library** (`openclash.lua`): Shared filesystem utilities used by models.

### Runtime Components

- **Init Script** (`root/etc/init.d/openclash`): Service control (START=99), firewall rules, DNS configuration, cron management.
- **Shell Scripts** (`root/usr/share/openclash/`): Core logic including:
  - `openclash.sh`: Main runtime logic
  - `openclash_*.sh`: Feature modules (rules, geodata, subscriptions, etc.)
  - `openclash_*.lua`: Lua helpers for specific tasks
- **UCI Config** (`root/etc/config/openclash`): Default configuration values.
- **Custom Files** (`root/etc/openclash/custom/`): User customizable rules and lists.

### Key Directories (Runtime on OpenWrt)

- `/etc/openclash/`: Configuration files, cores, rules
- `/usr/share/openclash/`: Scripts and UI files
- `/www/luci-static/resources/openclash/`: Web assets (status page, etc.)

## Internationalization (i18n)

Translations are in `po/zh-cn/openclash.zh-cn.po`. Build converts `.po` to `.lmo` (LuCI Machine Object) format using `po2lmo`.

## Version Management

Version is defined in `luci-app-openclash/Makefile` (`PKG_VERSION`).
Format: `0.47.055` (follows this pattern in commits).

## Common Development Patterns

### Adding a New Settings Page

1. Add UCI options to `root/etc/config/openclash` (defaults)
2. Create CBI form in `luasrc/model/cbi/openclash/<page>.lua`
3. Register in `luasrc/controller/openclash.lua` with `entry()` call
4. Create view template in `luasrc/view/openclash/<page>.htm` if needed
5. Add translations to `po/zh-cn/openclash.zh-cn.po`

### Adding Shell Script Functions

Place in appropriate `root/usr/share/openclash/openclash_*.sh` file. Source shared functions at top:
```bash
. $IPKG_INSTROOT/usr/share/openclash/log.sh
. $IPKG_INSTROOT/usr/share/openclash/uci.sh
```

### UCI Configuration Access

Use the wrapper functions from `uci.sh`:
```bash
uci_get_config "option_name"
uci_set_config "section" "option" "value"
```

## Dependencies

Runtime: `dnsmasq-full`, `bash`, `curl`, `ca-bundle`, `ip-full`, `ruby`, `ruby-yaml`, `kmod-tun`, `unzip`, `iptables` or `nftables` (FW4).

Build: OpenWrt SDK, `po2lmo` tool for translations.
