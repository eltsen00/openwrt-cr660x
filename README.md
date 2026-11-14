# OpenWrt 云编译（ImageBuilder）

使用 **OpenWrt 官方 ImageBuilder** 组装固件（CR660X 开启 MiniEAP 时会额外使用官方 SDK 编译 `minieap`），由 GitHub Actions 云端构建，产物自动发布到 **Releases**。

> ImageBuilder 只做「官方预编译软件包 + 第三方包 + 配置」的组装，不重新编译完整内核和工具链，
> 因此构建时间通常只需 **2–10 分钟**，稳定可靠。

---

## 🚀 支持的设备与工作流

仓库拆分为两个独立工作流，可分别触发构建：

| 方案 | 工作流 | 目标平台 | 适用设备 |
| --- | --- | --- | --- |
| **CR660X** | `OpenWrt CR660X ImageBuilder (ramips/mt7621)` | ramips/mt7621 | 小米 CR6606 / CR6608 / CR6609 |
| **x86_64** | `OpenWrt x86_64 ImageBuilder (x86/64)` | x86/64 | 通用 x86_64 设备（软路由、虚拟机、工控机） |

---

## 🚀 使用方法

### 构建 CR660X 固件

1. 打开仓库 **Actions** 页面。
2. 选择 **OpenWrt CR660X ImageBuilder (ramips/mt7621)** → **Run workflow**。
3. 按需填写参数（默认值即可直接构建）：
   - `version`：OpenWrt 版本，例如 `25.12.6`；留空自动使用最新 `25.12.x`。
   - `device`：`all`（默认，同时构建 CR6606 / CR6608 / CR6609）/ `cr6606` / `cr6608` / `cr6609`。
   - `include_minieap`：布尔开关（默认勾选 `true`），是否编译并内置 MiniEAP（**取消勾选则编译不含 `minieap` 的固件**）。
4. 等待构建完成，进入 **Releases**（或 Actions 产物）下载固件。

### 构建 x86_64 固件

1. 打开仓库 **Actions** 页面。
2. 选择 **OpenWrt x86_64 ImageBuilder (x86/64)** → **Run workflow**。
3. 可选填写 `version`（留空自动使用最新 `25.12.x`）。
4. 等待构建完成，进入 **Releases** 下载固件。

---

## 📦 固件包含对比

### CR660X 方案 (ramips/mt7621)

| 项目 | 说明 |
| --- | --- |
| OpenWrt | 25.12.x（默认自动取最新版本） |
| 管理地址 | `http://192.168.6.1`（子网掩码 `255.255.255.0`，默认无密码） |
| WiFi | 双频独立 SSID，密码 `12345678`（WPA2-PSK / `psk2+ccmp`）：<br>- 2.4G (`radio0`)：SSID `cr660x`，固定信道 `6`<br>- 5G (`radio1`)：SSID `cr660x-5G`，固定信道 `149`（非 DFS） |
| WiFi 驱动 | 开源 `mt76` (`kmod-mt7915e` + `kmod-mt7915-firmware`) + 调优参数（`US` 地区码、`txpower='23'`、`band_steering='1'`、`he_bss_color='1'`、MU-MIMO） |
| 校园网认证 | **MiniEAP (SYSU 自定义版)**：默认通过 SDK 编译并内置 `minieap`，同时在 Release 中附带 `minieap-*.apk`；触发构建时取消勾选 `include_minieap` 可生成不含 MiniEAP 的固件 |
| 主题 | Argon（第三方，`luci-theme-argon` + `luci-app-argon-config` 中文配置） |
| 其他 | `luci`、中文界面（`luci-i18n-base-zh-cn`）、时区 `Asia/Shanghai`、主机名 `CR660X`（已移除 MosDNS） |

### x86_64 方案 (x86/64)

| 项目 | 说明 |
| --- | --- |
| OpenWrt | 25.12.x（默认自动取最新版本） |
| 管理地址 | `http://192.168.31.66`（默认无密码，静态 IP） |
| 主题 | Argon（第三方，含中文配置） |
| DNS | MosDNS v5（第三方，含中文）+ SmartDNS（含中文）+ AdGuard Home |
| 其他 | LuCI、中文界面、软件包管理中文、主机名 `OpenWrt-x86`、时区 `Asia/Shanghai`、`ROOTFS_PARTSIZE=1024` |
| 固件格式 | `.img.gz`（UEFI + Legacy BIOS 双支持） |

---

## 📁 固件下载与刷机

### CR660X 固件文件（以 25.12.6 为例）

```text
openwrt-25.12.6-ramips-mt7621-xiaomi_mi-router-cr6606-squashfs-sysupgrade.bin
openwrt-25.12.6-ramips-mt7621-xiaomi_mi-router-cr6606-squashfs-firmware.bin
openwrt-25.12.6-ramips-mt7621-xiaomi_mi-router-cr6606.manifest
minieap-0.94.0.apk        # 启用 include_minieap 时一并发布
SHA256SUMS
```

（若 `device=all`，会同时包含 `cr6606` / `cr6608` / `cr6609` 对应型号的 `.bin` 与 `.manifest` 文件。）

| 场景 | 使用文件 | 说明 |
| --- | --- | --- |
| **已在 OpenWrt** | `*-squashfs-sysupgrade.bin` | 系统 → 备份/升级 → 刷写（跨大版本或更换配置建议不保留配置） |
| **原厂 SSH / PB-Boot 首次刷入** | `*-squashfs-firmware.bin` | 按设备教程刷入对应型号的 `firmware.bin` |
| **单独安装/更新 MiniEAP** | `minieap-*.apk` | 适用于同版本同架构（OpenWrt 25.12 / `mipsel_24kc`）固件的离线包安装 |

### x86_64 固件文件

```text
openwrt-25.12.6-x86-64-generic-ext4-combined-efi.img.gz
openwrt-25.12.6-x86-64-generic-ext4-combined.img.gz
openwrt-25.12.6-x86-64-generic-squashfs-combined-efi.img.gz
openwrt-25.12.6-x86-64-generic-kernel.bin
openwrt-25.12.6-x86-64-generic-ext4-rootfs.img.gz
openwrt-25.12.6-x86-64-generic.manifest
SHA256SUMS
```

| 文件 | 适用场景 | 推荐度 |
| --- | --- | --- |
| `*-ext4-combined-efi.img.gz` | **UEFI 启动，完整分区表** | ⭐⭐⭐ 推荐 |
| `*-ext4-combined.img.gz` | Legacy BIOS 启动，完整分区表 | ⭐⭐ |
| `*-squashfs-combined-efi.img.gz` | UEFI，squashfs 只读根分区（可用 overlay） | ⭐ |
| `*-kernel.bin` + `*-rootfs.img.gz` | 内核+根分区分离（高级用途） | — |

#### x86_64 写盘/刷机步骤

1. 解压 `.gz` 文件得到 `.img`（如 `gunzip openwrt-*.img.gz`）
2. 使用工具写入 USB/硬盘：
   - Linux/macOS: `sudo dd if=openwrt-*.img of=/dev/sdX bs=4M status=progress && sync`
   - Windows: `balenaEtcher` / `Rufus` / `Ventoy`
3. UEFI 机器用 `-efi.img`，老旧 BIOS 用 `-combined.img`
4. 启动后：LAN 默认 **静态 IP `192.168.31.66`**，网关 `192.168.31.1`，DNS `192.168.31.1`
5. 请确保上级路由网段为 `192.168.31.x`，或首次启动后进入 LuCI 修改网络设置

> ⚠️ 刷机有风险，请先备份原厂固件与重要数据。
> 首次登录建议立即修改管理密码。

---

## 🛠 修改默认配置

- CR660X 方案编辑 `files/etc/uci-defaults/99-cr660x`，修改后重新触发 `OpenWrt CR660X ImageBuilder (ramips/mt7621)` 即可生效。
- x86_64 方案编辑 `files-x86/etc/uci-defaults/99-x86-static-ip`，修改后重新触发 `OpenWrt x86_64 ImageBuilder (x86/64)` 即可生效。

### CR660X 首启脚本说明与常见修改示例

`99-cr660x` 会在配置无线前等待最多 15 秒（必要时执行 `/sbin/wifi config`），确保 `radio0` / `radio1` / `default_radio0` / `default_radio1` 初始化完成再写入配置，避免首次启动竞态导致无线配置丢失。

当前默认配置已包含 `192.168.6.1`、2.4G 信道 `6`（`cr660x`）、5G 信道 `149`（`cr660x-5G`）及 `country='US'`；如需自定义，可修改对应行：

```sh
# 改 LAN IP
uci set network.lan.ipaddr='192.168.10.1'

# 改 WiFi 名称/密码
uci set wireless.default_radio0.ssid='mywifi'
uci set wireless.default_radio0.key='mypassword'
uci set wireless.default_radio1.ssid='mywifi-5G'
uci set wireless.default_radio1.key='mypassword'

# 如需双频合一（将两个接口设为相同 SSID）
uci set wireless.default_radio0.ssid='cr660x'
uci set wireless.default_radio1.ssid='cr660x'

# 调整信道（5G 建议使用 149/153/157/161 等非 DFS 信道，2.4G 建议 1/6/11）
uci set wireless.radio0.channel='6'
uci set wireless.radio1.channel='149'
```

---

## 📋 工作流参数说明

### `build-cr660x.yml`

| 参数 | 说明 | 默认值 |
| --- | --- | --- |
| `version` | OpenWrt 版本，如 `25.12.6` | 自动取最新 `25.12.x` |
| `device` | `all` / `cr6606` / `cr6608` / `cr6609` | `all` |
| `include_minieap` | 是否通过 SDK 编译并内置 `minieap`（及随 Release 发布 `.apk`） | `true` |

### `build-x86_64.yml`

| 参数 | 说明 | 默认值 |
| --- | --- | --- |
| `version` | OpenWrt 版本，如 `25.12.6` | 自动取最新 `25.12.x` |

### 第三方组件来源

构建时自动拉取或编译最新包，无需手动维护本地二进制：

- **Argon 主题**：<https://github.com/jerrykuku/luci-theme-argon>（CR660X 与 x86_64 共用最新 Release `.apk`）
- **MiniEAP (SYSU)**：<https://github.com/undefined443/openwrt-minieap-sysu>（仅 CR660X；在 `include_minieap=true` 时由对应版本 OpenWrt SDK 现场编译为 `.apk`）
- **MosDNS v5**：<https://github.com/sbwml/luci-app-mosdns>（仅 x86_64，`x86_64-openwrt-25.12.tar.gz`）
- **SmartDNS / AdGuard Home**：OpenWrt 官方软件源（仅 x86_64）

---

## 📖 WiFi 性能调优建议（仅 CR660X）

如果 WiFi 速度不理想，按顺序排查：

1. **确认客户端能力**：WiFi 6 (802.11ax)、支持 80MHz 频宽、2×2 MIMO 以上
2. **选清净信道**：默认已锁定 2.4G `6` 与 5G `149`（非 DFS，可避免部分地区/驱动 DFS 雷达检测失败导致 5G 无法启动）；如需切换，5G 推荐 `149/153/157/161`，2.4G 推荐 `1/6/11`
3. **距离与遮挡**：5G 穿墙衰减大，近距测速；2.4G 抗遮挡但速率低
4. **散热**：路由器过热会触发降频，注意通风
5. **本地测速排除宽带因素**：`iperf3 -c 192.168.6.1 -P 4 -t 30`
6. **固定频宽**：在 LuCI 无线设置中按需锁定 80MHz（5G）/ 40MHz（2.4G）
7. **CR660X 硬件上限参考**：5G 近距理想值 600–900Mbps，2.4G 100–250Mbps

---

## 📂 仓库结构

```text
.github/workflows/build-cr660x.yml           CR660X ImageBuilder 工作流（可选编译 MiniEAP）
.github/workflows/build-x86_64.yml           x86_64 ImageBuilder 工作流
files/etc/uci-defaults/99-cr660x             CR660X 首启脚本（192.168.6.1 / 无线等待初始化 / 6&149 信道 / 中文 / 调优）
files-x86/etc/uci-defaults/99-x86-static-ip  x86_64 首启脚本（静态 IP 192.168.31.66 / 中文 / 时区）
README.md                                    本文件
```

---

## 📝 更新日志

- **2026-11-14** CR660X 方案移除 MosDNS，接入 `openwrt-minieap-sysu` 通过 OpenWrt SDK 自动编译并在 Release 中发布 `.apk`；新增工作流参数 `include_minieap` 支持手动勾选构建无 MiniEAP 固件；默认 LAN 改为 `192.168.6.1`，WiFi 改为 `cr660x`（ch 6）/ `cr660x-5G`（ch 149），并在 `99-cr660x` 增加无线节初始化等待逻辑避免首启 5G 竞态丢失
- **2026-11-10** 修复 x86_64 中文语言包配置（增加 SmartDNS 中文与 Software 菜单中文包）
- **2026-11-09** 拆分为 `build-cr660x.yml` 与 `build-x86_64.yml` 两个独立工作流；新增 x86_64 通用方案（`ROOTFS_PARTSIZE=1024`，预置 SmartDNS + AdGuard Home + MosDNS + Argon）
- **2026-11-09** 移除闭源 mt_wifi 编译方案，保留官方 ImageBuilder 开源方案为唯一构建方式
- **2026-11-08** 从源码全量编译迁移到官方 ImageBuilder，新增 mt76 调优参数并修复 `kmod-mt76-xqca` 依赖问题

---

## ⚖️ 许可与致谢

- OpenWrt 官方：<https://openwrt.org>
- Argon Theme：<https://github.com/jerrykuku/luci-theme-argon>
- MiniEAP (SYSU)：<https://github.com/undefined443/openwrt-minieap-sysu>
- MosDNS：<https://github.com/sbwml/luci-app-mosdns>
- SmartDNS：<https://github.com/pymumu/smartdns>
- AdGuard Home：<https://github.com/AdguardTeam/AdGuardHome>
- 开源 mt76 驱动：Linux 内核主线 / OpenWrt 官方源

本项目仅提供自动化构建配置，固件组件版权归各上游所有。
