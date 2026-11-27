# OpenWrt Support: D-Link DIR-1260 Rev. R1 (MediaTek MT7621DAT)

[English](#english) | [繁體中文](#繁體中文) | [Русский](#русский)

---

<a name="english"></a>
## English

Device support for the **D-Link DIR-1260 Revision R1** wireless router (`ramips/mt7621` target), submitted upstream in [OpenWrt Pull Request #25233](https://github.com/openwrt/openwrt/pull/25233).

### Overview & Changes
- **Upstream Codebase Adaptation:** Reworked an [initial community patch](https://github.com/Lucky1openwrt/openwrt/commit/b03f52df9c7f1b5490607926dbf0256065705f24) by `Lucky1openwrt` (originally written for OpenWrt `24.10.x`) for the current OpenWrt `main` tree and `25.12.x` release branch (verified on `25.12.5` and `25.12.6`). Consolidated three separate DTS/DTSI files into a single self-contained `mt7621_dlink_dir-1260-r1.dts` and replaced deprecated `$(Device/dsa-migration)` and `openwrt,uimage` definitions with standard `$(Device/nand)` and `denx,uimage` bindings.
- **NAND Capacity & Partition Protection:** Replaced the hardcoded `40960k` (40 MB) `IMAGE_SIZE` limit in `mt7621.mk` with `129280k` to utilize the full `0x7e40000` (126.25 MB) `firmware` partition on the 128 MB NAND (~100 MB usable UBI overlay space), and marked the `factory` partition (`0x100000`) as `read-only` to protect radio calibration data and MAC addresses.
- **LED Configuration Fixes:** Fixed Power, WAN (Internet), and 2.4 GHz WLAN LEDs — removed invalid `gpio 37`/`38` entries from `gpio-leds`, fixed `led-wan-orange` GPIO polarity (`GPIO_ACTIVE_HIGH` to `GPIO_ACTIVE_LOW`), reassigned `led-upgrade` to the orange Power LED, and simplified `01_leds` to drive `white:wan` (`link tx rx`).
- **Device Packages & USB Support:** Included `kmod-usb3` in `DEVICE_PACKAGES` to enable the physical USB 2.0 Type-A port via the SoC xHCI controller, and added `-uboot-envtools` to omit unused U-Boot environment utilities.

### Hardware Specifications
- **SoC:** MediaTek MT7621DAT (880 MHz, 2 cores / 4 threads, MIPS 1004Kc)
- **RAM:** 128 MB DDR3
- **Storage:** 128 MB Parallel NAND (Winbond W29N01HVSINA)
- **Ethernet:** Integrated MT7621 switch, 5x 10/100/1000 Mbps (1 WAN, 4 LAN)
- **Wi-Fi 2.4 GHz:** MediaTek MT7603E (802.11b/g/n, 2T2R)
- **Wi-Fi 5 GHz:** MediaTek MT7613BEN (`mt7615e` driver / `mt7663` AP firmware, 802.11a/n/ac)
- **USB:** 1x USB 2.0 Type-A (SoC xHCI controller, requires `kmod-usb3`)
- **Buttons:** Reset (`gpio 15`), WPS (`gpio 14`)
- **LEDs:** Power (White: `gpio 16`, Orange: `gpio 4`), WAN (White: `gpio 7`, Orange: `gpio 8`), WLAN 2.4 GHz (`mt7603e`), WLAN 5 GHz (permanently off)

### Partition Layout & Hardware Configuration
- **NAND Flash Partitions:** Preserves the factory partition table:
  - `0x0000000 - 0x0080000` (512 KB): `Bootloader` (`read-only`)
  - `0x0080000 - 0x0100000` (512 KB): `config`
  - `0x0100000 - 0x0140000` (256 KB): `factory` (`read-only`)
  - `0x0140000 - 0x7f80000` (126.25 MB): `firmware` (`denx,uimage`, UBI volume providing ~100 MB usable overlay space after kernel and rootfs)
  - `0x7f80000 - 0x8000000` (512 KB): `reserve`
- **NVMEM Layout:** Interface MAC addresses and radio calibration data are parsed from the `factory` partition via Linux NVMEM (`fixed-layout`):
  - Base MAC (`macaddr_factory_4`, offset `0x4`): LAN (`gmac0`) `base - 1`, WAN (`port@4`) `base + 1`, 2.4 GHz (`wifi0`) `base + 0`.
  - 5 GHz MAC (`macaddr_factory_8004`, offset `0x8004`): `wifi1` `offset + 0`.
  - EEPROM / Calibration: `eeprom_factory_0` (offset `0x0`, size `0x4da8`) and `eeprom_factory_8000` (offset `0x8000`, size `0x4da8`).
- **Pin Multiplexing:** Configured in `&state_default` to assign `i2c`, `uart3`, `jtag`, and `wdt` pin groups to `gpio` function.

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
- Built and verified on OpenWrt `main` and `25.12.x` (`25.12.5` and `25.12.6` builds).
- `scripts/checkpatch.pl` (0 errors, 0 warnings) and upstream `FormalityCheck` CI checks passed.
- **Verified on Physical Hardware:**
  - Initial flashing via OEM recovery (`factory.bin`) and `sysupgrade` (`sysupgrade.bin`)
  - Boot process and UBI partition layout (~100 MB usable overlay space)
  - 1 Gbps Ethernet switching (WAN and LAN 1–4) with proper MAC address assignment
  - 2.4 GHz Wi-Fi (`MT7603E`) and 5 GHz Wi-Fi (`MT7613BEN` / `mt7663` AP firmware)
  - USB 2.0 host port (`kmod-usb3`)
  - Reset and WPS hardware buttons
  - Power (White/Orange), WAN (White), and WLAN 2.4 GHz LEDs
  - Upstream kernel module (`kmod-*`) installation via `apk` without ABI / `vermagic` mismatches
- **Known Issue:** The 5 GHz WLAN LED remains permanently off.

### Modified Files
- `target/linux/ramips/dts/mt7621_dlink_dir-1260-r1.dts` — Self-contained Device Tree source (peripherals, NAND partitions, NVMEM layout, GPIO).
- `target/linux/ramips/image/mt7621.mk` — Device build profile, `IMAGE_SIZE` definition, default packages, and image recipes.
- `target/linux/ramips/mt7621/base-files/etc/board.d/01_leds` — Network interface LED trigger configuration.
- `target/linux/ramips/mt7621/base-files/lib/upgrade/platform.sh` — Sysupgrade platform verification and NAND flashing routine.

---

<a name="繁體中文"></a>
## 繁體中文

針對 **D-Link DIR-1260 R1** 無線路由器的 OpenWrt 裝置支援配置（`ramips/mt7621` 目標平台），相關變更已提交至上游 [OpenWrt Pull Request #25233](https://github.com/openwrt/openwrt/pull/25233)。

### 概述與主要修改
- **上游主線與版本適配：** 基於社群開發者 `Lucky1openwrt` 針對 OpenWrt `24.10.x` 的[初始補丁](https://github.com/Lucky1openwrt/openwrt/commit/b03f52df9c7f1b5490607926dbf0256065705f24)，重構並適配至現行 OpenWrt `main` 主線與 `25.12.x` 發行分支（已於 `25.12.5` 與 `25.12.6` 驗證）。將原先分散的三個 DTS/DTSI 檔案整合為單一獨立的 `mt7621_dlink_dir-1260-r1.dts`，並將已棄用的 `$(Device/dsa-migration)` 與 `openwrt,uimage` 替換為標準 `$(Device/nand)` 與 `denx,uimage` 定義。
- **NAND 容量與分區保護：** 將 `mt7621.mk` 中寫死的 `40960k`（40 MB）`IMAGE_SIZE` 上限調整為 `129280k`，以完整利用 128 MB NAND 上的 `0x7e40000`（126.25 MB）`firmware` 分區（提供約 100 MB 可用 UBI Overlay 空間）；同時將 `factory` 分區（`0x100000`）設為唯讀（`read-only`），保護無線校準資料與 MAC 位址。
- **LED 指示燈配置修正：** 修正 Power、WAN（網際網路）與 WLAN 2.4 GHz 指示燈——移除 `gpio-leds` 中無效的 `gpio 37`/`38` 定義，修正 `led-wan-orange` 的 GPIO 極性（由 `GPIO_ACTIVE_HIGH` 改為 `GPIO_ACTIVE_LOW`），將 `led-upgrade` 改為對應橘色 Power 指示燈，並精簡 `01_leds` 僅保留 `white:wan`（`link tx rx`）。
- **預設套件與 USB 支援：** 於 `DEVICE_PACKAGES` 加入 `kmod-usb3`，透過 SoC 內建的 xHCI 控制器啟用實體 USB 2.0 Type-A 連接埠，並加入 `-uboot-envtools` 排除未使用的 U-Boot 環境工具。

### 硬體規格
- **SoC：** MediaTek MT7621DAT（880 MHz，雙核心 / 4 線程，MIPS 1004Kc）
- **RAM：** 128 MB DDR3
- **儲存空間：** 128 MB Parallel NAND（Winbond W29N01HVSINA）
- **有線網路：** 整合式 MT7621 交換器，5x 10/100/1000 Mbps（1 WAN, 4 LAN）
- **Wi-Fi 2.4 GHz：** MediaTek MT7603E（802.11b/g/n, 2T2R）
- **Wi-Fi 5 GHz：** MediaTek MT7613BEN（`mt7615e` 驅動 / `mt7663` AP 韌體，802.11a/n/ac）
- **USB：** 1x USB 2.0 Type-A（SoC xHCI 控制器，需搭配 `kmod-usb3`）
- **按鍵：** Reset (`gpio 15`), WPS (`gpio 14`)
- **LED 指示燈：** Power（白光: `gpio 16`, 橘光: `gpio 4`）、WAN（白光: `gpio 7`, 橘光: `gpio 8`）、WLAN 2.4 GHz（`mt7603e`）、WLAN 5 GHz（目前永久熄滅）

### 分區與硬體配置
- **NAND Flash 分區：** 保留原廠分區表：
  - `0x0000000 - 0x0080000`（512 KB）：`Bootloader`（唯讀）
  - `0x0080000 - 0x0100000`（512 KB）：`config`
  - `0x0100000 - 0x0140000`（256 KB）：`factory`（唯讀）
  - `0x0140000 - 0x7f80000`（126.25 MB）：`firmware`（`denx,uimage`，部署核心與根檔案系統後提供約 100 MB 可用 UBI Overlay 空間）
  - `0x7f80000 - 0x8000000`（512 KB）：`reserve`
- **NVMEM 配置：** 透過 Linux NVMEM（`fixed-layout`）從 `factory` 分區解析網路介面 MAC 位址與無線校準資料：
  - 基礎 MAC（`macaddr_factory_4`，位移 `0x4`）：LAN（`gmac0`）為 `base - 1`，WAN（`port@4`）為 `base + 1`，2.4 GHz（`wifi0`）為 `base + 0`。
  - 5 GHz MAC（`macaddr_factory_8004`，位移 `0x8004`）：`wifi1` 為 `offset + 0`。
  - EEPROM / 校準資料：`eeprom_factory_0`（位移 `0x0`，大小 `0x4da8`）與 `eeprom_factory_8000`（位移 `0x8000`，大小 `0x4da8`）。
- **引腳複用（Pin Multiplexing）：** 於 `&state_default` 節點中將 `i2c`、`uart3`、`jtag` 及 `wdt` 引腳組設為 `gpio` 模式。

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
- 已於 OpenWrt `main` 主線與 `25.12.x`（`25.12.5`、`25.12.6` 編譯版本）原始碼樹中完成編譯與驗證。
- 通過 `scripts/checkpatch.pl`（0 錯誤、0 警告）與上游 `FormalityCheck` CI 檢查。
- **實體硬體驗證項目：**
  - 透過 OEM 救援模式刷入 `factory.bin` 與執行 `sysupgrade`（`sysupgrade.bin`）
  - 開機流程與 UBI 分區配置（約 100 MB 可用 Overlay 空間）
  - 1 Gbps 有線網路交換（WAN 與 LAN 1–4）及正確的 MAC 位址指派
  - 2.4 GHz（`MT7603E`）與 5 GHz（`MT7613BEN` / `mt7663` AP 韌體）無線網路運作
  - USB 2.0 Host 連接埠（`kmod-usb3`）
  - Reset 與 WPS 硬體按鍵
  - Power（白光/橘光）、WAN（白光）與 WLAN 2.4 GHz 狀態指示燈
  - 透過 `apk` 安裝官方上游核心模組（`kmod-*`），無 ABI / `vermagic` 不相容問題
- **已知問題：** 5 GHz WLAN 指示燈目前永久熄滅。

### 修改檔案清單
- `target/linux/ramips/dts/mt7621_dlink_dir-1260-r1.dts` — 獨立整合的 Device Tree 原始碼（週邊裝置、NAND 分區、NVMEM 配置、GPIO）。
- `target/linux/ramips/image/mt7621.mk` — 裝置編譯配置、`IMAGE_SIZE` 定義、預設套件與映像檔打包規則。
- `target/linux/ramips/mt7621/base-files/etc/board.d/01_leds` — 網路介面 LED 觸發設定。
- `target/linux/ramips/mt7621/base-files/lib/upgrade/platform.sh` — Sysupgrade 平台驗證與 NAND 寫入程序。

---

<a name="русский"></a>
## Русский

Поддержка маршрутизатора **D-Link DIR-1260 ревизии R1** в OpenWrt (платформа `ramips/mt7621`), изменения отправлены в апстрим в [OpenWrt Pull Request #25233](https://github.com/openwrt/openwrt/pull/25233).

### Обзор и основные изменения
- **Адаптация под актуальную кодовую базу:** за основу взят [исходный патч сообщества](https://github.com/Lucky1openwrt/openwrt/commit/b03f52df9c7f1b5490607926dbf0256065705f24) от `Lucky1openwrt` (написанный для ветки OpenWrt `24.10.x`) и переработан под актуальный `main` и релизную ветку `25.12.x` (проверено на сборках `25.12.5` и `25.12.6`). Три отдельных файла DTS/DTSI объединены в один самодостаточный `mt7621_dlink_dir-1260-r1.dts`, а устаревшие макросы `$(Device/dsa-migration)` и `openwrt,uimage` заменены на стандартные `$(Device/nand)` и `denx,uimage`.
- **Объём NAND и защита разделов:** жёстко заданный лимит `IMAGE_SIZE := 40960k` (40 МБ) в `mt7621.mk` заменён на `129280k` для использования всего раздела `firmware` (`0x7e40000`, 126.25 МБ) на флеш-памяти NAND объёмом 128 МБ (~100 МБ доступного пространства UBI overlay). Раздел `factory` (`0x100000`) переведён в режим `read-only` для защиты калибровочных данных радиомодулей и MAC-адресов.
- **Исправление конфигурации светодиодов:** восстановлена работа индикаторов Power, WAN (Internet) и WLAN 2.4 GHz — из `gpio-leds` удалены некорректные записи `gpio 37`/`38`, исправлена полярность `led-wan-orange` (`GPIO_ACTIVE_HIGH` → `GPIO_ACTIVE_LOW`), алиас `led-upgrade` переназначен на оранжевый индикатор питания, а в `01_leds` оставлен только триггер `white:wan` (`link tx rx`).
- **Пакеты сборки и поддержка USB:** в `DEVICE_PACKAGES` включён пакет `kmod-usb3` для работы порта USB 2.0 Type-A через контроллер xHCI в SoC, а также добавлен `-uboot-envtools` для исключения неиспользуемых утилит U-Boot.

### Аппаратные характеристики
- **SoC:** MediaTek MT7621DAT (880 МГц, 2 ядра / 4 потока, MIPS 1004Kc)
- **ОЗУ (RAM):** 128 МБ DDR3
- **Накопитель:** 128 МБ Parallel NAND (Winbond W29N01HVSINA)
- **Ethernet:** встроенный коммутатор MT7621, 5 портов 10/100/1000 Мбит/с (1 WAN, 4 LAN)
- **Wi-Fi 2.4 ГГц:** MediaTek MT7603E (802.11b/g/n, 2T2R)
- **Wi-Fi 5 ГГц:** MediaTek MT7613BEN (драйвер `mt7615e` / прошивка `mt7663` AP, 802.11a/n/ac)
- **USB:** 1 порт USB 2.0 Type-A (контроллер xHCI в SoC, требуется `kmod-usb3`)
- **Кнопки:** Reset (`gpio 15`), WPS (`gpio 14`)
- **Индикаторы:** Power (Белый: `gpio 16`, Оранжевый: `gpio 4`), WAN (Белый: `gpio 7`, Оранжевый: `gpio 8`), WLAN 2.4 GHz (`mt7603e`), WLAN 5 GHz (перманентно отключён)

### Разметка памяти и конфигурация
- **Разделы NAND Flash:** сохранена заводская таблица разделов:
  - `0x0000000 - 0x0080000` (512 КБ): `Bootloader` (`read-only`)
  - `0x0080000 - 0x0100000` (512 КБ): `config`
  - `0x0100000 - 0x0140000` (256 КБ): `factory` (`read-only`)
  - `0x0140000 - 0x7f80000` (126.25 МБ): `firmware` (`denx,uimage`, том UBI, предоставляющий ~100 МБ полезного раздела overlay после размещения ядра и rootfs)
  - `0x7f80000 - 0x8000000` (512 КБ): `reserve`
- **Конфигурация NVMEM:** MAC-адреса интерфейсов и калибровочные данные радиомодулей считываются из раздела `factory` средствами Linux NVMEM (`fixed-layout`):
  - Базовый MAC (`macaddr_factory_4`, смещение `0x4`): LAN (`gmac0`) `base - 1`, WAN (`port@4`) `base + 1`, 2.4 ГГц (`wifi0`) `base + 0`.
  - MAC 5 ГГц (`macaddr_factory_8004`, смещение `0x8004`): `wifi1` `offset + 0`.
  - Калибровки EEPROM: `eeprom_factory_0` (смещение `0x0`, размер `0x4da8`) и `eeprom_factory_8000` (смещение `0x8000`, размер `0x4da8`).
- **Настройка выводов (Pinmux):** в узле `&state_default` группы выводов `i2c`, `uart3`, `jtag` и `wdt` переведены в режим `gpio`.

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
- Сборка выполнена и проверена как на актуальной ветке `main`, так и на релизной ветке `25.12.x` (сборки `25.12.5` и `25.12.6`).
- Проверки `scripts/checkpatch.pl` (0 ошибок, 0 предупреждений) и апстрим-CI `FormalityCheck` пройдены без замечаний.
- **Проверено на физическом устройстве:**
  - Первичная прошивка через режим восстановления OEM (`factory.bin`) и обновление через `sysupgrade` (`sysupgrade.bin`)
  - Загрузка системы и разметка разделов UBI (~100 МБ доступного пространства overlay)
  - Коммутация Ethernet 1 Гбит/с (WAN и LAN 1–4) и корректное назначение MAC-адресов
  - Работа Wi-Fi 2.4 ГГц (`MT7603E`) и 5 ГГц (`MT7613BEN` / прошивка `mt7663` AP)
  - Работа порта USB 2.0 Host (`kmod-usb3`)
  - Аппаратные кнопки Reset и WPS
  - Индикаторы состояния Power (Белый/Оранжевый), WAN (Белый) и WLAN 2.4 GHz
  - Установка официальных модулей ядра (`kmod-*`) из репозиториев OpenWrt через `apk` без несовпадений ABI / `vermagic`
- **Известная проблема:** светодиодный индикатор WLAN 5 GHz перманентно отключён.

### Изменённые файлы
- `target/linux/ramips/dts/mt7621_dlink_dir-1260-r1.dts` — самодостаточный исходный код Device Tree (периферия, разделы NAND, разметка NVMEM, GPIO).
- `target/linux/ramips/image/mt7621.mk` — профиль сборки устройства, лимит `IMAGE_SIZE`, пакеты по умолчанию и правила сборки образов.
- `target/linux/ramips/mt7621/base-files/etc/board.d/01_leds` — настройка триггеров светодиодной индикации сетевых интерфейсов.
- `target/linux/ramips/mt7621/base-files/lib/upgrade/platform.sh` — проверка платформы при `sysupgrade` и сценарий записи во флеш-память NAND.
