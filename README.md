# OpenWrt Port: D-Link DIR-1260 Rev. R1 (MediaTek MT7621DAT)

[English](#english) | [繁體中文 (香港)](#繁體中文) | [Русский](#русский)

---

<a name="english"></a>
## English

Target board bring-up, Device Tree integration, and system build configuration for the **D-Link DIR-1260 Revision R1** wireless router within the OpenWrt tree (`ramips/mt7621`).

### Hardware Specifications
* **SoC:** MediaTek MT7621DAT (Dual-core / 4 threads @ 880 MHz, MIPS 1004Kc)
* **RAM:** 128 MB DDR3
* **Storage:** 128 MB Parallel NAND (Winbond W29N01HVSINA)
* **Ethernet:** Integrated MT7621 Switch, 5x 10/100/1000 Mbps (1 WAN, 4 LAN)
* **Wireless:** 
  * 2.4 GHz: MediaTek MT7603E (802.11b/g/n, 2T2R)
  * 5 GHz: MediaTek MT7613BEN (driven via `mt7615` / `mt7663` firmware framework, 802.11a/n/ac)
* **USB:** 1x USB 2.0 Type-A (xHCI controller via `kmod-usb3`)
* **Buttons:** Reset (`gpio 15`), WPS (`gpio 14`)
* **LEDs:** Power (White: `gpio 16`, Orange: `gpio 4`), WAN (White: `gpio 7`, Orange: `gpio 8`)

### Architecture & Implementation Details
* **NAND Flash Partitioning:** Preserves the factory partition table. The `firmware` partition spans `0x140000 - 0x7f80000` (126.25 MB UBI space), yielding **~100 MB of usable writable overlay space** after deploying the kernel and rootfs.
* **NVMEM Architecture:** MAC address assignments and radio calibration offsets are parsed directly from the `factory` partition using standard Linux NVMEM layouts (`fixed-layout`):
  * Base MAC (`macaddr_factory_4`, offset `0x4`): LAN assigned `base - 1`, WAN assigned `base + 1`, 2.4 GHz assigned `base + 0`.
  * 5 GHz MAC (`macaddr_factory_8004`, offset `0x8004`): assigned `offset + 0`.
  * EEPROM / Calibration: `eeprom_factory_0` (offset `0x0`) and `eeprom_factory_8000` (offset `0x8000`).
* **Pin Multiplexing:** Configured in `&state_default` to assign `i2c`, `uart3`, `jtag`, and `wdt` pin groups to standard GPIO mode.

### Installation & Recovery Procedures
1. **Initial Flashing from Vendor (OEM) Firmware:**
   * Downgrade OEM firmware to version `4.0.1` (subsequent vendor releases block third-party firmware flashing via recovery mode).
   * Power off the router. Depress and hold the **Reset** button while reconnecting the power input.
   * Hold Reset for ~10 seconds until the Power LED blinks orange to access U-Boot Web Recovery Mode.
   * Assign a static IPv4 address to the host PC: `192.168.0.2` (mask `255.255.255.0`).
   * Navigate to `http://192.168.0.1` in a browser, supply the compiled `*factory.bin` image, and execute upload.
2. **Upgrade from OpenWrt (Sysupgrade):**
   * Apply updates using the standard `*sysupgrade.bin` target via LuCI web UI or CLI:
     ```sh
     sysupgrade -v openwrt-ramips-mt7621-dlink_dir-1260-r1-squashfs-sysupgrade.bin
     ```

### Modified Subsystem Files
* `target/linux/ramips/dts/mt7621_dlink_dir-1260-r1.dts` — Device Tree Source (peripherals, NAND partitions, NVMEM, GPIO).
* `target/linux/ramips/image/mt7621.mk` — Device definition, package dependencies, and image recipes.
* `target/linux/ramips/mt7621/base-files/etc/board.d/01_leds` — Network interface LED runtime trigger hooks.
* `target/linux/ramips/mt7621/base-files/lib/upgrade/platform.sh` — Sysupgrade platform verification and NAND flashing routine.

---

<a name="繁體中文"></a>
## 繁體中文

針對 **D-Link DIR-1260 R1** 無線路由器平台之 OpenWrt 架構移植、裝置樹（Device Tree）整合及映像檔構建設定（基於 `ramips/mt7621` 目標架構）。

### 硬體系統規格
* **SoC 處理器：** MediaTek MT7621DAT（雙核心 / 4 線程 @ 880 MHz，MIPS 1004Kc）
* **主記憶體 (RAM)：** 128 MB DDR3
* **非揮發性儲存器 (Flash)：** 128 MB Parallel NAND（Winbond W29N01HVSINA）
* **網路介面：** 整合式 MT7621 交換晶片，5x 10/100/1000 Mbps（1 WAN, 4 LAN）
* **無線射頻模組：** 
  * 2.4 GHz 頻段：MediaTek MT7603E（802.11b/g/n, 2T2R）
  * 5 GHz 頻段：MediaTek MT7613BEN（由 `mt7615` / `mt7663` 韌體架構驅動，802.11a/n/ac）
* **USB 介面：** 1x USB 2.0 Type-A（透過 `kmod-usb3` 支援 xHCI 控制器）
* **硬體按鍵：** Reset (`gpio 15`), WPS (`gpio 14`)
* **LED 指示燈：** 電源燈（白光: `gpio 16`, 橘光: `gpio 4`）、WAN 網路燈（白光: `gpio 7`, 橘光: `gpio 8`）

### 系統工程細節
* **Flash 分區拓撲：** 嚴格保留原廠分區結構。韌體分區 (`firmware`) 配置範圍為 `0x140000 - 0x7f80000`（126.25 MB UBI 分區），在部署核心及根檔案系統後，**提供約 100 MB 可用空間 (Overlay)** 供擴充套件安裝。
* **NVMEM 記憶體單元機制：** MAC 地址與 Wi-Fi 校準參數全面改採 Linux NVMEM 架構（`fixed-layout`）自 `factory` 分區解析：
  * 基礎 MAC (`macaddr_factory_4`, 位移 `0x4`)：LAN 指派 `base - 1`，WAN 指派 `base + 1`，2.4 GHz 指派 `base + 0`。
  * 5 GHz MAC (`macaddr_factory_8004`, 位移 `0x8004`)：指派 `offset + 0`。
  * EEPROM / 校準資料：`eeprom_factory_0`（位移 `0x0`）與 `eeprom_factory_8000`（位移 `0x8000`）。
* **腳本與週邊設定：** 於 `&state_default` 節點中將 `i2c`、`uart3`、`jtag` 及 `wdt` 引腳組配置為標準 GPIO 模式。

### 韌體部署與升級作業
1. **原廠韌體初始刷機：**
   * 須先將原廠韌體降級至 `4.0.1` 版本（較新版本限制於 Recovery 介面上傳未簽署韌體）。
   * 拔掉電源線，按住 **Reset** 鍵並重新接通電源。
   * 持續按住約 10 秒至電源燈呈現橘燈閃爍，進入 U-Boot Web Recovery 模式。
   * 將電腦網卡手動設定為固定 IP：`192.168.0.2`（遮罩 `255.255.255.0`）。
   * 使用瀏覽器開啟 `http://192.168.0.1`，上傳 `*factory.bin` 檔案並等待系統重啟。
2. **OpenWrt 系統內核升級 (Sysupgrade)：**
   * 透過命令列執行升級指令：
     ```sh
     sysupgrade -v openwrt-ramips-mt7621-dlink_dir-1260-r1-squashfs-sysupgrade.bin
     ```

### 原始碼異動清單
* `target/linux/ramips/dts/mt7621_dlink_dir-1260-r1.dts` — 裝置樹設定檔（SoC 匯流排、NAND 分區、NVMEM 與 GPIO）。
* `target/linux/ramips/image/mt7621.mk` — 平台編譯規格、套件相依性及映像檔封裝流程。
* `target/linux/ramips/mt7621/base-files/etc/board.d/01_leds` — 指示燈網路觸發腳本。
* `target/linux/ramips/mt7621/base-files/lib/upgrade/platform.sh` — Sysupgrade 平台校驗與寫入腳本。

---

<a name="русский"></a>
## Русский

Интеграция аппаратной поддержки платформы, дерева устройств (Device Tree) и конфигурации сборки образов для беспроводного маршрутизатора **D-Link DIR-1260 аппаратной ревизии R1** в состав дерева OpenWrt (`ramips/mt7621`).

### Архитектура аппаратной части
* **Центральный процессор (SoC):** MediaTek MT7621DAT (2 ядра / 4 потока @ 880 МГц, микроархитектура MIPS 1004Kc)
* **Оперативная память (RAM):** 128 МБ DDR3
* **Энергонезависимая память (NAND):** 128 МБ Parallel NAND (Winbond W29N01HVSINA)
* **Сетевая подсистема:** Встроенный коммутатор MT7621, 5 портов 10/100/1000 Mbps (1 WAN, 4 LAN)
* **Беспроводные интерфейсы:** 
  * Диапазон 2.4 ГГц: MediaTek MT7603E (802.11b/g/n, 2T2R)
  * Диапазон 5 ГГц: MediaTek MT7613BEN (аппаратный уровень под управлением драйвера `mt7615` и микрокода `mt7663`, 802.11a/n/ac)
* **Интерфейсы расширения:** 1 порт USB 2.0 Type-A (контроллер xHCI через модуль `kmod-usb3`)
* **Элементы управления:** Кнопки Reset (`gpio 15`) и WPS (`gpio 14`)
* **Светодиодная индикация:** Power (Белый: `gpio 16`, Оранжевый: `gpio 4`), WAN (Белый: `gpio 7`, Оранжевый: `gpio 8`)

### Инженерные особенности реализации
* **Топология разделов NAND Flash:** Полностью сохранена заводская разметка. Раздел `firmware` занимает диапазон `0x140000 - 0x7f80000` (126.25 МБ UBI-пространства), что обеспечивает **~100 МБ свободного overlay-пространства** после развертывания ядра и базовой файловой системы.
* **Подсистема NVMEM:** Конфигурирование MAC-адресов интерфейсов и калибровочных таблиц радиомодулей считывается напрямую из раздела `factory` через современные ячейки NVMEM (`fixed-layout`):
  * Базовый MAC (`macaddr_factory_4`, смещение `0x4`): для LAN задано `base - 1`, для WAN `base + 1`, для 2.4 GHz `base + 0`.
  * 5 GHz MAC (`macaddr_factory_8004`, смещение `0x8004`): задано `offset + 0`.
  * Данные калибровки EEPROM: `eeprom_factory_0` (смещение `0x0`) и `eeprom_factory_8000` (смещение `0x8000`).
* **Мультиплексирование выводов:** В узле `&state_default` группы выводов `i2c`, `uart3`, `jtag` и `wdt` переведены в стандартный режим GPIO.

### Регламент прошивки и обновления
1. **Первичная установка из штатной прошивки вендора (OEM):**
   * Выполнить предварительный даунгрейд заводской прошивки до ревизии `4.0.1` (более поздние версии блокируют загрузку сторонних бинарников через веб-интерфейс восстановления).
   * Отключить кабель питания роутера. Зажать тактовую кнопку **Reset** и подать питание.
   * Удерживать Reset около 10 секунд до перехода индикатора питания в режим оранжевой пульсации (вход в U-Boot Web Recovery).
   * Назначить на сетевом адаптере ПК статический IPv4-адрес: `192.168.0.2` (маска `255.255.255.0`).
   * В веб-обозревателе перейти по адресу `http://192.168.0.1`, загрузить образ `*factory.bin` и дождаться перезагрузки.
2. **Обновление системы (Sysupgrade):**
   * Последующие сборки обновляются через веб-интерфейс LuCI либо посредством утилиты командной строки:
     ```sh
     sysupgrade -v openwrt-ramips-mt7621-dlink_dir-1260-r1-squashfs-sysupgrade.bin
     ```

### Перечень модифицированных файлов
* `target/linux/ramips/dts/mt7621_dlink_dir-1260-r1.dts` — Дерево устройств (периферия, разметка NAND, ячейки NVMEM, GPIO).
* `target/linux/ramips/image/mt7621.mk` — Спецификация сборки платформы, состав пакетов и рецепт сборки образов.
* `target/linux/ramips/mt7621/base-files/etc/board.d/01_leds` — Хуки системных триггеров сетевой индикации.
* `target/linux/ramips/mt7621/base-files/lib/upgrade/platform.sh` — Сценарий верификации платформы и записи в память при вызове sysupgrade.
