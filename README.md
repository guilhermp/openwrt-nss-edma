# OpenWrt NSS — Linksys MX Series (SPNMX56 & MX2000)

A downstream distribution of **[kuncy7/openwrt-nss-edma](https://github.com/kuncy7/openwrt-nss-edma)**, focused and hardened specifically for the **Linksys SPNMX56** and **Linksys MX2000** (Qualcomm IPQ5018 WiSoC).

Maintained by **@guilhermp** with the help of **Gemini 4.0** and **Claude Opus 5.5**.

---

## Upstream & Lineage

This tree builds upon [kuncy7/openwrt-nss-edma](https://github.com/kuncy7/openwrt-nss-edma), which ported the Qualcomm NSS data plane (`qca-nss-drv`, ECM, NSS qdiscs, and ath11k offload) to run alongside the upstream `stmmac` ethernet driver on Linux 6.18 without vendor DP/SSDK blobs.

This fork optimizes and trims that work specifically for the Linksys MX router architecture.

---

## What This Fork Does

### 1. Hardware-Specific Focus (Linksys MX / IPQ5018)
- **Targeted Compilation**: Builds exclusively for `linksys_spnmx56` and `linksys_mx2000` using the 512 MB `std` memory group.
- **QCN9074 Firmware Memory Mode 1 (SPNMX56)**: Configures the discrete PCIe 5 GHz radio (`ipq5018-spnmx56.dts`) to run in memory mode 1, reducing host DMA reservation from ~45 MB to ~20.9 MB while maintaining full 160 MHz throughput and client capacity.
- **CPU Boost**: Retains the 1.296 GHz CPU operating point (`0974-arm64-dts-qcom-ipq5018-let-the-CPU-boost-to-1296-MHz.patch`).

### 2. Lean Kernel & Driver Pruning
- **Diagnostic Bloat Removed**: Production kernel stripped of heavy development debug instrumentation (`CONFIG_SLUB_DEBUG_ON`, `CONFIG_DEBUG_KMEMLEAK`, `CONFIG_PAGE_OWNER`, `CONFIG_DYNAMIC_DEBUG`) while enabling `/proc/config.gz` (`CONFIG_IKCONFIG=y`, `CONFIG_IKCONFIG_PROC=y`) for live inspection.
- **Unused Peripherals Pruned**:
  - **No USB / Block Storage**: SPNMX56 and MX2000 do not support USB data storage (SPNMX56's port is factory power-only; MX2000 has none). Removed `kmod-usb3`, `kmod-usb-dwc3`, `e2fsprogs`, `kmod-fs-ext4`, and `losetup`.
  - **Bluetooth Disabled**: Factory firmware only used BLE for initial mobile-app onboarding during mesh pairing. OpenWrt manages Wi-Fi mesh natively via `wpad-mesh-mbedtls` (802.11s), rendering in-kernel Bluetooth drivers obsolete.
  - **Switch & Bus Cleanups**: Removed drivers for SPI-NOR, MMC/SDHCI, GPIO LEDs (`kmod-leds-gpio`), and Realtek switches (`kmod-rtl8367s-nss`). Both boards communicate with the switch fabric through `kmod-qca8337-nss`.

### 3. Integrated Userland & Clean TLS
- **Single TLS Stack**: Uses `mbedtls` across the entire system (`wpad-mesh-mbedtls` and `luci-ssl`), avoiding duplicate OpenSSL libraries in SquashFS.
- **Baked Packages**: Pre-installs `luci`, `luci-ssl`, `luci-app-nss`, `luci-app-firewall`, `luci-app-usteer`, `usteer`, `luci-app-advanced-reboot`, `luci-app-https-dns-proxy`, `https-dns-proxy`, `umdns`, `rpcd-mod-rrdns`, `sqm-scripts-nss`, `tc-full`, `ip-full`, `iperf3`, and `dumpimage`.

---

## Supported Devices

| Model | SoC | Switch | 2.4 GHz Radio | 5 GHz Radio | Sysupgrade Image |
|---|---|---|---|---|---|
| **Linksys SPNMX56** | IPQ5018 (2x A53 @ 1.3 GHz) | QCA8337 | QCN6122 (AHB, 2x2) | QCN9074 (PCIe, 4x4, 160 MHz) | `openwrt-qualcommax-ipq50xx-linksys_spnmx56-squashfs-sysupgrade.bin` |
| **Linksys MX2000** | IPQ5018 (2x A53 @ 1.3 GHz) | QCA8337 | QCN6122 (AHB, 2x2) | QCN6122 (AHB, 2x2, 160 MHz) | `openwrt-qualcommax-ipq50xx-linksys_mx2000-squashfs-sysupgrade.bin` |

---

## Upgrading

Download the matching sysupgrade image from the [Releases page](https://github.com/guilhermp/openwrt-nss-edma/releases) and flash over a running OpenWrt installation:

```sh
sysupgrade -v /tmp/openwrt-qualcommax-ipq50xx-<device>-squashfs-sysupgrade.bin
```

Kernel module packages are indexed on this repository's GitHub Pages feed:

```text
https://guilhermp.github.io/openwrt-nss-edma/<release-tag>/std/packages.adb
```
