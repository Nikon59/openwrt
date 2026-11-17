# OpenWrt Support: D-Link DIR-1260 Rev. R1 (MediaTek MT7621DAT)

[English](#english) | [繁體中文](#繁體中文) | [Русский](#русский)

---

<a name="english"></a>
## English

Device support for the **D-Link DIR-1260 Revision R1** wireless router (`ramips/mt7621` target), submitted upstream in [OpenWrt Pull Request #25233](https://github.com/openwrt/openwrt/pull/25233).

### Overview & Changes
- **Upstream Adaptation:** Adapted an initial community patch by `Lucky1openwrt` (originally targeting OpenWrt `24.10.x`) to the current OpenWrt tree.
- **NAND Overlay Capacity Fix:** Corrected the NAND flash partition layout in DTS to provide **~100 MB of usable writable overlay space** (resolving a 40 MB limitation in the earlier patch).
- **LED Configuration:** Fixed GPIO definitions for the Power, WAN, and WLAN 2.4 GHz indicator LEDs and removed non-existent per-port Ethernet LED entries.
- **USB Host Enablement:** Enabled the physical USB 2.0 Type-A port (routed to the SoC xHCI controller) using community-verified DTS definitions and `kmod-usb3`.

### Hardware Specifications
- **SoC:** MediaTek MT7621DAT (880 MHz, 2 cores / 4 threads, MIPS 1004Kc)
- **RAM:** 128 MB DDR3
- **Storage:** 128 MB SPI Parallel NAND (Winbond W29N01HVSINA)
- **Ethernet:** Integrated MT7621 switch, 5x 10/100/1000 Mbps (1 WAN, 4 LAN)
- **Wi-Fi 2.4 GHz:** MediaTek MT7603E (802.11b/g/n, 2T2R)
- **Wi-Fi 5 GHz:** MediaTek MT7613BEN (`mt7615` / `mt7663` firmware, 802.11a/n/ac)
- **USB:** 1x USB 2.0 Type-A (xHCI controller, requires `kmod-usb3`)
- **Buttons:** Reset (`gpio 15`), WPS (`gpio 14`)
- **LEDs:** Power (White: `gpio 16`, Orange: `gpio 4`), WAN (White: `gpio 7`, Orange: `gpio 8`), WLAN 2.4G, WLAN 5G

### Partition Layout & Hardware Configuration
- **NAND Flash Partitioning:** Preserves the factory partition table. The `firmware` partition spans `0x140000 - 0x7f80000` (126.25 MB UBI space), leaving **~100 MB of usable overlay space** after deploying the kernel and rootfs.
- **NVMEM Layout:** Interface MAC addresses and radio calibration data are parsed from the `factory` partition via Linux NVMEM (`fixed-layout`):
  - Base MAC (`macaddr_factory_4`, offset `0x4`): LAN `base - 1`, WAN `base + 1`, 2.4 GHz `base + 0`.
  - 5 GHz MAC (`macaddr_factory_8004`, offset `0x8004`): `offset + 0`.
  - EEPROM / Calibration: `eeprom_factory_0` (offset `0x0`) and `eeprom_factory_8000` (offset `0x8000`).
- **Pin Multiplexing:** Configured in `&state_default` to assign `i2c`, `uart3`, `jtag`, and `wdt` pin groups to GPIO mode.

### Installation & Upgrade

#### Installation from OEM Firmware
1. Downgrade OEM firmware to stock version `4.0.1` (newer vendor versions block flashing third-party firmware via recovery mode).
2. Power off the router.
3. Press and hold the **Reset** button while plugging in the power cord.
4. Keep holding Reset until the Power LED starts blinking orange (~10 seconds) to enter U-Boot Web Recovery Mode.
5. Set the host PC network adapter to static IP `192.168.0.2/24` (`255.255.255.0`).
6. Open `http://192.168.0.1` in a web browser.
7. Select the OpenWrt `*factory.bin` image and click **Upload**.
8. Wait for flashing to complete and the router to automatically reboot.

#### Upgrade from OpenWrt
Standard `sysupgrade` is supported using the `*sysupgrade.bin` image (identical to D-Link DIR-2150), via LuCI web interface or CLI:
```sh
sysupgrade -v openwrt-ramips-mt7621-dlink_dir-1260-r1-squashfs-sysupgrade.bin
```

### Testing & Verification
- Built and verified on a clean OpenWrt tree with full target manifest.
- `scripts/checkpatch.pl` clean (0 errors, 0 warnings).
- **Verified Functionality:**
  - Boot process and UBI partition layout (~100 MB overlay)
  - 1 Gbps Ethernet (LAN & WAN) with proper MAC address assignment
  - 2.4 GHz Wi-Fi (`MT7603E`)
  - 5 GHz Wi-Fi (`MT7613BEN` / `mt7663` AP firmware)
  - USB 2.0 host (`xHCI` via `kmod-usb3`)
  - Reset and WPS hardware buttons
  - Power, WAN, and WLAN 2.4 GHz LEDs
- **Known Issue:** The 5 GHz WLAN LED currently remains off.

### Modified Files
- `target/linux/ramips/dts/mt7621_dlink_dir-1260-r1.dts` — Device Tree source (peripherals, NAND partitions, NVMEM layout, GPIO).
- `target/linux/ramips/image/mt7621.mk` — Device build profile, default packages (`kmod-usb3`, Wi-Fi drivers), and image recipes.
- `target/linux/ramips/mt7621/base-files/etc/board.d/01_leds` — Network interface LED trigger configuration.
- `target/linux/ramips/mt7621/base-files/lib/upgrade/platform.sh` — Sysupgrade platform verification and NAND flashing routine.

---

<a name="繁體中文"></a>
## 繁體中文

針對 **D-Link DIR-1260 R1** 無線路由器的 OpenWrt 裝置支援配置（`ramips/mt7621` 目標平台），相關變更已提交至上游 [OpenWrt Pull Request #25233](https://github.com/openwrt/openwrt/pull/25233)。

### 概述與主要修改
- **上游主線適配：** 基於社群開發者 `Lucky1openwrt` 針對 OpenWrt `24.10.x` 的初始補丁，移植並適配至現行 OpenWrt 主線。
- **修正 NAND Overlay 空間限制：** 修正 DTS 中的 NAND 快閃記憶體分區配置，提供 **約 100 MB 可用寫入空間（Overlay）**（解決早期補丁僅有 40 MB 的限制）。
- **LED 指示燈配置：** 修正 Power、WAN 與 WLAN 2.4 GHz 指示燈的 GPIO 定義，並移除硬體上不存在的各別 Ethernet 埠指示燈項目。
- **啟用 USB Host：** 採用社群驗證的 DTS 定義與 `kmod-usb3` 模組，啟用實體 USB 2.0 Type-A 連接埠（線路連接至 SoC 的 xHCI 控制器）。

### 硬體規格
- **SoC：** MediaTek MT7621DAT（880 MHz，雙核心 / 4 線程，MIPS 1004Kc）
- **RAM：** 128 MB DDR3
- **儲存空間：** 128 MB SPI Parallel NAND（Winbond W29N01HVSINA）
- **有線網路：** 整合式 MT7621 交換器，5x 10/100/1000 Mbps（1 WAN, 4 LAN）
- **Wi-Fi 2.4 GHz：** MediaTek MT7603E（802.11b/g/n, 2T2R）
- **Wi-Fi 5 GHz：** MediaTek MT7613BEN（`mt7615` / `mt7663` 韌體，802.11a/n/ac）
- **USB：** 1x USB 2.0 Type-A（xHCI 控制器，需搭配 `kmod-usb3`）
- **按鍵：** Reset (`gpio 15`), WPS (`gpio 14`)
- **LED 指示燈：** Power（白光: `gpio 16`, 橘光: `gpio 4`）、WAN（白光: `gpio 7`, 橘光: `gpio 8`）、WLAN 2.4G、WLAN 5G

### 分區與硬體配置
- **NAND Flash 分區：** 保留原廠分區表。`firmware` 分區範圍為 `0x140000 - 0x7f80000`（126.25 MB UBI 空間），部署核心與根檔案系統後提供 **約 100 MB 可用 Overlay 空間**。
- **NVMEM 配置：** 透過 Linux NVMEM（`fixed-layout`）從 `factory` 分區解析網路介面 MAC 位址與無線校準資料：
  - 基礎 MAC（`macaddr_factory_4`，位移 `0x4`）：LAN 為 `base - 1`，WAN 為 `base + 1`，2.4 GHz 為 `base + 0`。
  - 5 GHz MAC（`macaddr_factory_8004`，位移 `0x8004`）：`offset + 0`。
  - EEPROM / 校準資料：`eeprom_factory_0`（位移 `0x0`）與 `eeprom_factory_8000`（位移 `0x8000`）。
- **引腳複用（Pin Multiplexing）：** 於 `&state_default` 節點中將 `i2c`、`uart3`、`jtag` 及 `wdt` 引腳組設為 GPIO 模式。

### 安裝與升級

#### 從原廠（OEM）韌體安裝
1. 將原廠韌體降級至 `4.0.1` 版本（較新版本會封鎖透過救援模式刷入第三方韌體）。
2. 關閉路由器電源。
3. 按住 **Reset** 鍵不放，同時接上電源線。
4. 持續按住 Reset 鍵直到電源燈開始閃爍橘光（約 10 秒），進入 U-Boot Web Recovery 模式。
5. 將電腦網路介面卡設為固定 IP `192.168.0.2/24`（子網路遮罩 `255.255.255.0`）。
6. 使用瀏覽器開啟 `http://192.168.0.1`。
7. 選擇 OpenWrt `*factory.bin` 映像檔並點擊 **Upload**。
8. 等待刷寫完成，路由器將會自動重新啟動。

#### 從 OpenWrt 升級
支援透過 LuCI 網頁介面或 CLI 使用 `*sysupgrade.bin` 映像檔進行標準 `sysupgrade` 升級（流程與 D-Link DIR-2150 相同）：
```sh
sysupgrade -v openwrt-ramips-mt7621-dlink_dir-1260-r1-squashfs-sysupgrade.bin
```

### 測試與驗證
- 已於乾淨的 OpenWrt 原始碼樹中完成完整目標清單編譯與驗證。
- 通過 `scripts/checkpatch.pl` 檢查（0 錯誤、0 警告）。
- **已驗證功能：**
  - 開機流程與 UBI 分區配置（約 100 MB Overlay）
  - 1 Gbps 有線網路（LAN 與 WAN）及正確的 MAC 位址指派
  - 2.4 GHz Wi-Fi（`MT7603E`）
  - 5 GHz Wi-Fi（`MT7613BEN` / `mt7663` AP 韌體）
  - USB 2.0 Host（透過 `kmod-usb3` 驅動 `xHCI`）
  - Reset 與 WPS 硬體按鍵
  - Power、WAN 與 WLAN 2.4 GHz 指示燈
- **已知問題：** 5 GHz WLAN 指示燈目前不會亮起。

### 修改檔案清單
- `target/linux/ramips/dts/mt7621_dlink_dir-1260-r1.dts` — Device Tree 原始碼（週邊裝置、NAND 分區、NVMEM 配置、GPIO）。
- `target/linux/ramips/image/mt7621.mk` — 裝置編譯配置、預設套件（`kmod-usb3`、Wi-Fi 驅動）與映像檔打包規則。
- `target/linux/ramips/mt7621/base-files/etc/board.d/01_leds` — 網路介面 LED 觸發設定。
- `target/linux/ramips/mt7621/base-files/lib/upgrade/platform.sh` — Sysupgrade 平台驗證與 NAND 寫入程序。

---

<a name="русский"></a>
## Русский

Поддержка маршрутизатора **D-Link DIR-1260 ревизии R1** в OpenWrt (платформа `ramips/mt7621`), изменения отправлены в апстрим в [OpenWrt Pull Request #25233](https://github.com/openwrt/openwrt/pull/25233).

### Обзор и основные изменения
- **Адаптация под актуальную ветку:** за основу взят исходный патч сообщества от `Lucky1openwrt` (для ветки OpenWrt `24.10.x`) и перенесён в актуальную кодовую базу OpenWrt.
- **Исправление объёма раздела overlay:** скорректирована таблица разделов NAND в DTS — после установки ядра и rootfs доступно **~100 МБ свободного пространства overlay** (исправлено ограничение в 40 МБ из раннего патча).
- **Настройка светодиодов:** исправлены определения GPIO для индикаторов Power, WAN и WLAN 2.4 GHz, а также удалены записи о несуществующих светодиодах портов Ethernet.
- **Поддержка USB:** включён физический порт USB 2.0 Type-A (подключённый к контроллеру xHCI в SoC) с использованием проверенной сообществом конфигурации DTS и пакета `kmod-usb3`.

### Аппаратные характеристики
- **SoC:** MediaTek MT7621DAT (880 МГц, 2 ядра / 4 потока, MIPS 1004Kc)
- **ОЗУ (RAM):** 128 МБ DDR3
- **Накопитель:** 128 МБ SPI Parallel NAND (Winbond W29N01HVSINA)
- **Ethernet:** встроенный коммутатор MT7621, 5 портов 10/100/1000 Мбит/с (1 WAN, 4 LAN)
- **Wi-Fi 2.4 ГГц:** MediaTek MT7603E (802.11b/g/n, 2T2R)
- **Wi-Fi 5 ГГц:** MediaTek MT7613BEN (прошивка `mt7615` / `mt7663`, 802.11a/n/ac)
- **USB:** 1 порт USB 2.0 Type-A (контроллер xHCI, требуется `kmod-usb3`)
- **Кнопки:** Reset (`gpio 15`), WPS (`gpio 14`)
- **Индикаторы:** Power (Белый: `gpio 16`, Оранжевый: `gpio 4`), WAN (Белый: `gpio 7`, Оранжевый: `gpio 8`), WLAN 2.4G, WLAN 5G

### Разметка памяти и конфигурация
- **Разделы NAND Flash:** сохранена заводская таблица разделов. Раздел `firmware` занимает диапазон `0x140000 - 0x7f80000` (126.25 МБ пространства UBI), оставляя **~100 МБ доступного раздела overlay** после развёртывания ядра и корневой файловой системы.
- **Конфигурация NVMEM:** MAC-адреса интерфейсов и калибровочные данные радиомодулей считываются из раздела `factory` средствами Linux NVMEM (`fixed-layout`):
  - Базовый MAC (`macaddr_factory_4`, смещение `0x4`): LAN `base - 1`, WAN `base + 1`, 2.4 ГГц `base + 0`.
  - MAC 5 ГГц (`macaddr_factory_8004`, смещение `0x8004`): `offset + 0`.
  - Калибровки EEPROM: `eeprom_factory_0` (смещение `0x0`) и `eeprom_factory_8000` (смещение `0x8000`).
- **Настройка выводов (Pinmux):** в узле `&state_default` группы выводов `i2c`, `uart3`, `jtag` и `wdt` переведены в режим GPIO.

### Установка и обновление

#### Первичная установка со штатной прошивки (OEM)
1. Понизьте версию заводской прошивки до штатной `4.0.1` (более поздние версии блокируют загрузку сторонних образов через режим восстановления).
2. Отключите питание маршрутизатора.
3. Зажмите кнопку **Reset** и, удерживая её, подключите кабель питания.
4. Удерживайте Reset до тех пор, пока индикатор Power не начнёт мигать оранжевым (~10 секунд) для входа в режим U-Boot Web Recovery.
5. Назначьте сетевому адаптеру ПК статический IP-адрес `192.168.0.2/24` (`255.255.255.0`).
6. Откройте `http://192.168.0.1` в браузере.
7. Выберите образ OpenWrt `*factory.bin` и нажмите **Upload**.
8. Дождитесь завершения прошивки и автоматической перезагрузки устройства.

#### Обновление из OpenWrt
Поддерживается стандартное обновление `sysupgrade` с помощью образа `*sysupgrade.bin` (аналогично D-Link DIR-2150) через веб-интерфейс LuCI или командную строку:
```sh
sysupgrade -v openwrt-ramips-mt7621-dlink_dir-1260-r1-squashfs-sysupgrade.bin
```

### Тестирование и проверка
- Сборка выполнена и проверена на чистом дереве OpenWrt с полным манифестом целевой платформы.
- Проверка `scripts/checkpatch.pl` пройдена без замечаний (0 ошибок, 0 предупреждений).
- **Проверенный функционал:**
  - Загрузка системы и разметка разделов UBI (~100 МБ overlay)
  - Проводная сеть 1 Гбит/с (LAN и WAN) с корректным назначением MAC-адресов
  - Wi-Fi 2.4 ГГц (`MT7603E`)
  - Wi-Fi 5 ГГц (`MT7613BEN` / прошивка `mt7663` AP)
  - Порт USB 2.0 (`xHCI` через `kmod-usb3`)
  - Аппаратные кнопки Reset и WPS
  - Индикаторы Power, WAN и WLAN 2.4 GHz
- **Известная проблема:** светодиодный индикатор WLAN 5 GHz в текущей версии не активен.

### Изменённые файлы
- `target/linux/ramips/dts/mt7621_dlink_dir-1260-r1.dts` — исходный код Device Tree (периферия, разделы NAND, разметка NVMEM, GPIO).
- `target/linux/ramips/image/mt7621.mk` — профиль сборки устройства, пакеты по умолчанию (`kmod-usb3`, драйверы Wi-Fi) и правила сборки образов.
- `target/linux/ramips/mt7621/base-files/etc/board.d/01_leds` — настройка триггеров светодиодной индикации сетевых интерфейсов.
- `target/linux/ramips/mt7621/base-files/lib/upgrade/platform.sh` — проверка платформы при `sysupgrade` и сценарий записи во флеш-память NAND.
