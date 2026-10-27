# OpenWrt port for D-Link DIR-1260 R1 (MT7621DAT)

[English](#english) | [繁體中文 (香港)](#繁體中文) | [Русский](#русский)

---

<a name="english"></a>
## English

Board bring-up and upstream patch for the **D-Link DIR-1260 rev. R1** router in OpenWrt (`ramips/mt7621`).

### Specs
* **SoC:** MediaTek MT7621DAT (2 cores / 4 threads @ 880 MHz)
* **RAM:** 128 MB DDR3
* **Flash:** 128 MB Parallel NAND (Winbond W29N01HVSINA)
* **Switch:** 1x GbE WAN, 4x GbE LAN (integrated MT7621 switch)
* **Wi-Fi:** 
  * 2.4 GHz: MT7603E (802.11b/g/n, 2T2R)
  * 5 GHz: MT7613BEN (driven by `mt7615` / `mt7663` firmware, 802.11a/n/ac)
* **USB:** 1x USB 2.0 Type-A
* **Controls:** Reset, WPS buttons

### Hardware & Porting Notes
* **Flash Layout:** The stock partition table is preserved. UBI partition is mapped to `0x140000 - 0x7f80000`, giving ~126.25 MB for kernel and rootfs/overlay.
* **MAC Addresses:** Handled via modern NVMEM cells from the `factory` partition (offset `0x4` for LAN/base, offset `0x8004` for 5 GHz).
* **Wi-Fi:** Calibration data is pulled directly from the factory block (`eeprom_factory_0` and `eeprom_factory_8000`).
* **Known quirks:** 4 out of 5 front-panel LEDs work properly. One LED GPIO line is still being tracked down/verified against board traces.

### How to Flash
1. **From Stock OEM:**
   * Downgrade OEM firmware to stock version `4.0.1` (newer OEM builds block unsigned recovery uploads).
   * Unplug power, hold the **Reset** button, plug power back in.
   * Keep holding Reset (~10s) until the Power LED blinks orange to enter Recovery Mode.
   * Set your PC to static IP `192.168.0.2` (mask `255.255.255.0`).
   * Open `http://192.168.0.1` in a browser, pick `openwrt-...-factory.bin` and upload.
2. **From existing OpenWrt:**
   * Flash `openwrt-...-sysupgrade.bin` via LuCI web UI or CLI:
     ```sh
     sysupgrade -v openwrt-ramips-mt7621-dlink_dir-1260-r1-squashfs-sysupgrade.bin
     ```

### Modified Files in Tree
* `target/linux/ramips/dts/mt7621_dlink_dir-1260-r1.dts`
* `target/linux/ramips/image/mt7621.mk`
* `target/linux/ramips/mt7621/base-files/etc/board.d/01_leds`
* `target/linux/ramips/mt7621/base-files/lib/upgrade/platform.sh`

---

<a name="繁體中文"></a>
## 繁體中文

適用於 **D-Link DIR-1260 R1** 路由器之 OpenWrt 適配補丁（基於 `ramips/mt7621` 目標）。

### 硬體規格
* **SoC 處理器：** MediaTek MT7621DAT（雙核心 / 4 線程 @ 880 MHz）
* **記憶體 (RAM)：** 128 MB DDR3
* **快閃記憶體 (Flash)：** 128 MB Parallel NAND（Winbond W29N01HVSINA）
* **網路介面：** 1x GbE WAN, 4x GbE LAN（整合 MT7621 交換晶片）
* **無線網路 (Wi-Fi)：** 
  * 2.4 GHz：MT7603E（802.11b/g/n, 2T2R）
  * 5 GHz：MT7613BEN（使用 `mt7615` / `mt7663` 韌體驅動，802.11a/n/ac）
* **USB：** 1x USB 2.0 Type-A
* **按鍵：** Reset, WPS

### 適配說明與亮點
* **Flash 分割表：** 完整保留原廠分區結構。UBI 分區掛載於 `0x140000 - 0x7f80000`，提供約 126.25 MB 可用空間存放核心及系統軟體包。
* **MAC 地址與校準：** 採用現代 NVMEM 機制直接由 `factory` 分區讀取，無舊版硬編碼問題。
* **LED 指示燈狀態：** 面板 4 個主要指示燈運作正常，另有 1 個 LED GPIO 腳位目前正進一步驗證中。

### 刷機方式
1. **原廠韌體升級至 OpenWrt：**
   * 需先降級原廠韌體至 `4.0.1` 版本（較新版本限制了 Recovery 介面上傳第三方韌體）。
   * 拔掉電源線，按住 **Reset** 鍵不放並重新插上電源。
   * 持續按住約 10 秒，直到電源燈閃爍橘燈，代表已進入 Recovery Mode。
   * 將電腦網卡手動設定為固定 IP：`192.168.0.2`（遮罩 `255.255.255.0`）。
   * 使用瀏覽器開啟 `http://192.168.0.1`，上傳 `openwrt-...-factory.bin` 檔案。
2. **OpenWrt 系統內升級：**
   * 透過 LuCI 網頁介面或終端機執行 `sysupgrade`：
     ```sh
     sysupgrade -v openwrt-ramips-mt7621-dlink_dir-1260-r1-squashfs-sysupgrade.bin
     ```

### 變更檔案清單
* `target/linux/ramips/dts/mt7621_dlink_dir-1260-r1.dts`
* `target/linux/ramips/image/mt7621.mk`
* `target/linux/ramips/mt7621/base-files/etc/board.d/01_leds`
* `target/linux/ramips/mt7621/base-files/lib/upgrade/platform.sh`

---

<a name="русский"></a>
## Русский

Портирование и адаптация поддержки роутера **D-Link DIR-1260 ревизии R1** для OpenWrt (таргет `ramips/mt7621`).

### Железо
* **Процессор:** MediaTek MT7621DAT (2 ядра / 4 потока, 880 МГц)
* **Оперативная память:** 128 МБ DDR3
* **Флеш-память:** 128 МБ Parallel NAND (Winbond W29N01HVSINA)
* **Сеть:** 1 WAN, 4 LAN (все гигабитные, встроенный свитч MT7621)
* **Wi-Fi:** 
  * 2.4 ГГц: MT7603E (802.11b/g/n)
  * 5 ГГц: MT7613BEN (работает через стек `mt7615` / прошивку `mt7663`)
* **USB:** 1 порт USB 2.0 Type-A
* **Кнопки:** Reset, WPS

### Что сделано и нюансы сборки
* **Разметка NAND:** Сохранена родная структура разделов D-Link. Под прошивку (`firmware`) и UBI-волюмы выделено пространство с `0x140000` по `0x7f80000` — это дает больше 126 МБ свободного места под систему и пакеты.
* **MAC-адреса и калибровки:** Собраны через современный `nvmem-layout` прямо из раздела `factory` без устаревших костылей. Адреса портов считаются со смещениями от базового.
* **Индикация:** Работают кнопки и 4 светодиода. Один диод на лицевой панели пока в процессе отлова правильного GPIO/pinctrl.

### Как прошивать
1. **С заводской прошивки (OEM):**
   * Откатить стоковую прошивку на версию `4.0.1` (в более свежих D-Link закрыл установку сторонних сборок через веб-рекавери).
   * Отключить питание, зажать кнопку **Reset** и вставить кабель питания обратно.
   * Держать Reset около 10 секунд, пока диод питания не начнет мигать оранжевым (роутер вошел в Recovery Mode).
   * На сетевой карте ПК вручную выставить IP `192.168.0.2`, маска `255.255.255.0`.
   * Открыть в браузере `http://192.168.0.1`, выбрать файл `*factory.bin` и нажать загрузку. Дождаться перезагрузки.
2. **Обновление с OpenWrt:**
   * Шить стандартный образ `*sysupgrade.bin` через веб-морду LuCI или по SSH:
     ```sh
     sysupgrade -v <путь_к_файлу_sysupgrade.bin>
     ```

### Измененные файлы в ветке
* `target/linux/ramips/dts/mt7621_dlink_dir-1260-r1.dts` — полное описание платы, шин, pinctrl и разделов.
* `target/linux/ramips/image/mt7621.mk` — профиль устройства, пакеты и рецепт сборки образов.
* `target/linux/ramips/mt7621/base-files/etc/board.d/01_leds` — хуки индикации.
* `target/linux/ramips/mt7621/base-files/lib/upgrade/platform.sh` — поддержка безопасного sysupgrade.
