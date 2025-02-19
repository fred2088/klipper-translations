# 常見問題

1. [我怎樣才能支援專案？](#how-can-i-donate-to-the-project)
1. [如何計算rotation_distance配置參數？](#how-do-i-calculate-the-rotation_distance-config-parameter)
1. [我的串列埠在哪裡找？](#wheres-my-serial-port)
1. [當微控制器重啟裝置更改為/dev/ttyUSB1](#when-the-micro-controller-restarts-the-device-changes-to-devttyusb1)
1. [「make flash」命令不起作用](#the-make-flash-command-doesnt-work)
1. [如何更改串列埠波特率？](#how-do-i-change-the-serial-baud-rate)
1. [我可以在 Raspberry Pi 3 以外的其他裝置上執行 Klipper 嗎？](#can-i-run-klipper-on-something-other-than-a-raspberry-pi-3)
1. [我可以在同一臺主機上執行多個 Klipper 實例嗎？](#can-i-run-multiple-instances-of-klipper-on-the-same-host-machine)
1. [我必須使用 OctoPrint 嗎？](#do-i-have-to-use-octoprint)
1. [為什麼我不能在歸位印表機之前移動步進器？](#why-cant-i-move-the-stepper-before-homing-the-printer)
1. [為什麼預設配置中的 Z position_endstop 設定為 0.5？](#why-is-the-z-position_endstop-set-to-05-in-the-default-configs)
1. [我從 Marlin 轉換了我的配置，X/Y 軸工作正常，但在 Z 軸歸位時我只會聽到刺耳的噪音](#i-converted-my-config-from-marlin-and-the-xy-axes-work-fine-but-i-just-get-a-screeching-noise-when-homing-the-z-axis)
1. [我的 TMC 電機驅動程式在列印過程中關閉](#my-tmc-motor-driver-turns-off-in-the-middle-of-a-print)
1. [我不斷收到隨機的「與 MCU 失去通訊」錯誤](#i-keep-getting-random-lost-communication-with-mcu-errors)
1. [我的樹莓派在列印過程中不斷重啟](#my-raspberry-pi-keeps-rebooting-during-prints)
1. [當我設定`restart_method=command`時，我的 AVR 裝置在重啟時掛起](#when-i-set-restart_methodcommand-my-avr-device-just-hangs-on-a-restart)
1. [如果 Raspberry Pi 崩潰，加熱器會繼續打開嗎？](#will-the-heaters-be-left-on-if-the-raspberry-pi-crashes)
1. [如何將 Marlin 引腳編號轉換為 Klipper 引腳名稱？](#how-do-i-convert-a-marlin-pin-number-to-a-klipper-pin-name)
1. [我必須將我的裝置連線到特定型別的微控制器引腳嗎？](#do-i-have-to-wire-my-device-to-a-specific-type-of-micro-controller-pin)
1. [我如何取消 M109/M190「等待溫度」請求？](#how-do-i-cancel-an-m109m190-wait-for-temperature-request)
1. [可以查一下印表機是否丟步嗎？](#can-i-find-out-whether-the-printer-has-lost-steps)
1. [為什麼Klipper會報錯？ 我列印的檔案丟了！](#why-does-klipper-report-errors-i-lost-my-print)
1. [如何升級到最新軟體？](#how-do-i-upgrade-to-the-latest-software)
1. [如何解除安裝 klipper？](#how-do-i-uninstall-klipper)

## 我如何向專案捐款？

謝謝。Kevin有一個PATREON頁面，地址是： <https://www.patreon.com/koconnor>

## 如何計算 rotation_distance 配置參數？

參見[旋轉距離文件](Rotation_Distance.md)。

## 我的串列埠在哪裡找？

查詢 USB 串列埠的一般方法是從主機上的 ssh 終端執行 `ls /dev/serial/by-id/*`。 它可能會產生類似於以下內容的輸出：

```
/dev/serial/by-id/usb-1a86_USB2.0-Serial-if00-port0
```

在上述命令中找到的名稱是穩定的，可以在配置檔案中使用它，同時可用於刷寫微控制器的程式碼。 例如，一個 flash 命令：

```
sudo service klipper stop
make flash FLASH_DEVICE=/dev/serial/by-id/usb-1a86_USB2.0-Serial-if00-port0
sudo service klipper start
```

更新后的配置可能如下所示：

```
[mcu]
serial: /dev/serial/by-id/usb-1a86_USB2.0-Serial-if00-port0
```

請務必從上面執行的「ls」命令中複製並貼上名稱，因為每個印表機的名稱都不同。

如果您使用多個微控制器並且它們沒有唯一的 ID（在帶有 CH340 USB 晶片的板上很常見），那麼請按照上面的說明使用命令 `ls /dev/serial/by-path/*` 代替。

## 當微控制器重新啟動裝置更改為/dev/ttyUSB1

按照「[我的串列埠在哪裡找？](#wheres-my-serial-port)」部分中的說明來防止這種情況發生。

## 「make flash」命令不起作用

該程式碼嘗試使用每個平臺最常用的方法來刷寫裝置。 不幸的是，刷寫方法存在很多差異，因此「make flash」命令可能不適用于所有主板。

如果您遇到間歇性故障或您確實有標準設定，請仔細檢查刷寫時 Klipper 是否未執行（sudo service klipper stop），確保 OctoPrint 未嘗試直接連線到裝置（打開 網頁中的連線選項卡，如果串列埠設定為裝置，則單擊斷開連線），並確保為您的電路板正確設定了 FLASH_DEVICE（請參閱[上面的問題](#wheres-my-serial-port)）。

但是，如果「make flash」對您的電路板不起作用，那麼您將需要手動刷寫。 檢視 [config directory](../config) 中是否有一個配置檔案，裡面有刷機的具體說明。 此外，請檢視電路板製造商的文件，看看它是否描述瞭如何刷寫裝置。 最後，可以使用諸如「avrdude」或「bossac」之類的工具手動刷寫裝置——有關更多資訊，請參閱[引導載入程式文件](Bootloaders.md)。

## 如何更改序列波特率？

Klipper的推薦波特率是 250000。這個波特率在 Klipper 支援的所有微控制器板上都很好用。如果你發現網上的指南推薦了一個不同的波特率，那麼請忽略這部分指南，繼續使用預設值 250000。

如果你還是想改變波特率，那麼需要重新配置微控制器為新的波特率（在**make menuconfig**過程中），然後將更新的程式碼編譯並刷寫到微控制器中。Klipper 的 printer.cfg 檔案也需要更新以匹配該波特率（詳見[配置參考](Config_Reference.md#mcu）)。例如：

```
[mcu]
baud: 250000
```

OctoPrint 網頁上顯示的波特率對內部 Klipper 微控制器的波特率沒有影響。使用 Klipper 時，始終將 OctoPrint 的波特率設定為250000。

Klipper 微控制器的波特率與微控制器啟動載入程式的波特率無關。有關啟動載入程式的額外資訊請參閱[啟動載入程式文件](Bootloaders.md)。

## 我可以在 Raspberry Pi 3 以外的其他裝置上執行 Klipper 嗎？

推薦的硬體是 Raspberry Pi 2、Raspberry Pi 3 或 Raspberry Pi 4。

Klipper 可以在 Raspberry Pi 1和Raspberry Pi Zero上執行，但這些板子沒有足夠的處理能力來執行 OctoPrint。在這些較慢的機器上直接從 OctoPrint 列印時，經常會出現列印停滯。(印表機的移動速度可能比 OctoPrint 發送移動命令的速度快。)如果你希望在這些較慢的板子上執行，請考慮在列印時使用 "virtual_sdcard "功能(詳情請參見[配置參考](Config_Reference.md#virtual_sdcard))。

要在Beaglebone上執行，請參閱[Beaglebone特別安裝說明](Beaglebone.md)。

Klipper 可以在其他計算機上執行。Klipper 主機軟體只需要在Linux（或類似）系統的計算機上執行 Python。然而，如果你想在其他計算機上執行它，你將需要一些 Linux 管理知識來安裝該計算機上系統的依賴包。參見 [install-octopi.sh](..../scripts/install-octopi.sh) 指令碼，以進一步瞭解必要的 Linux 安裝方法。

如果你想在低端處理器上執行 Klipper 主機軟體，請注意，你至少需要一臺具有 "雙精度浮點 "運算硬體的計算機。

如果你想在共享的通用計算機或伺服器上執行 Klipper 主機程式，請注意 Klipper 有一些實時排程要求。如果在列印過程中，主機也在執行密集型的通用計算任務（如硬碟碎片整理、3D渲染、大量swapping （虛擬記憶體交換）等），可能會導致 Klipper 報告列印錯誤。

注意：如果你沒有使用 OctoPi 映象，一些Linux發行版啟用了一個 "ModemManager"（或類似的）軟體包，它干擾序列通訊。(這可能導致 Klipper 報告看似隨機的 "與MCU失去通訊 "錯誤）。如果你在這些發行版上安裝Klipper，你可能需要禁用該軟體包。

## 我可以在同一臺主機上執行多個 Klipper 實例嗎？

可以執行 Klipper 主機軟體的多個實例，但這樣做需要一定的 Linux 知識。 Klipper 安裝指令碼最終會導致執行以下 Unix 命令：

```
~/klippy-env/bin/python ~/klipper/klippy/klippy.py ~/printer.cfg -l /tmp/klippy.log
```

只要每個實例都有自己的印表機配置檔案、自己的日誌檔案和自己的偽 tty，就可以執行上述命令的多個實例。 例如：

```
~/klippy-env/bin/python ~/klipper/klippy/klippy.py ~/printer2.cfg -l /tmp/klippy2.log -I /tmp/printer2
```

如果你選擇這樣做，你將需要實現必要的啟動、停止和安裝指令碼。[install-octopi.sh](..../scripts/install-octopi.sh)和[klipper-start.sh](..../scripts/klipper-start.sh)指令碼提供了一些例子。

## 我必須使用 OctoPrint 嗎？

Klipper 不依賴 OctoPrint。其他軟體也可以向 Klipper 發送命令，但這樣做可能需要一些 Linux 管理知識。

Klipper 通過 「/tmp/printer」 檔案建立了一個「虛擬串列埠」，該檔案模擬了一個標準的3D印表機串列埠。理論上任何能配置並使用 「/tmp/printer」 作為印表機串列埠的軟體都可以與Klipper一起工作。

## 為什麼在歸位印表機之前我不能移動步進器？

程式碼這樣做是爲了減少意外將頭撞到床或墻壁的機會。 印表機歸位后，軟體會嘗試驗證每個移動是否在配置檔案中定義的 position_min/max 範圍內。 如果電機被禁用（通過 M84 或 M18 命令），那麼電機將需要在運動前再次歸位。

如果您想在通過 OctoPrint 取消列印后移動列印頭，請考慮更改 OctoPrint 取消順序來為您執行此操作。 它是通過 Web 瀏覽器在 OctoPrint 中配置的：設定-> GCODE 指令碼

如果您想在列印完成後移動頭部，請考慮將所需的移動新增到切片機的「自定義 g 程式碼」部分。

如果印表機歸位過程中需要一些額外的移動（或者根本上沒有歸位運動），那麼可以在配置檔案中定義safe_z_home或 homing_override分段。如果你需要為診斷或除錯目的移動一個步進，可以在配置檔案中新增一個 force_move 分段。參見[配置參考](Config_Reference.md#customized_homing)以瞭解關於這些選項的詳情。

## 為什麼 Z position_endstop 在預設配置中設定為 0.5？

對於笛卡爾式印表機，Z position_endstop 定義了當限位觸發時噴嘴離床的距離。如果可能的話，建議使用 Z-max 限位使歸位時列印頭遠離床（因為這可以減少碰撞的發生）。如果必須朝向床歸位，那麼建議將限位固定在噴嘴距離床還有一小段距離時觸發。這樣，當歸位 Z 軸時，噴嘴會在接觸床之前停止。有關詳細資訊，請參閱 [列印床調平文件](Bed_Level.md)。

## 我從 Marlin 轉換了我的配置並且 X/Y 軸工作正常，但是在歸位 Z 軸時我只聽到刺耳的噪音且印表機不動

長話短說：首先，請確保您已按照 [配置檢查文件](Config_checks.md) 中描述的步驟驗證了步進驅動配置。如果問題仍然存在，請嘗試降低印表機配置中的 max_z_velocity 設定。

具體原理：在實踐中，Marlin 通常只能以每秒 10000 步左右的速度發送步進。如果要求它以需要更高的步進率的速度移動，那麼 Marlin 會盡可能快地步進。Klipper 能夠實現更高的步進率，但步進電機可能沒有足夠的扭矩以更高的速度移動。因此，對於一個具有高傳動比或高微步設定的 Z 軸，實際的最大速度可能小於 Marlin 中配置的值。

## 我的 TMC 電機驅動在列印過程中關閉了

如果在「獨立模式」中使用 TMC2208（或 TMC2224）驅動，那麼請確保您正在使用[最新版本的 Klipper](#how-do-i-upgrade-to-the-latest-software)。2020年3月中旬，Klipper 中修復了 TMC2208 「StealthChop」 驅動的問題。

## 我不斷收到隨機的"與MCU失去通訊"錯誤

這通常是由主機和微控制器之間的 USB 連線中硬體錯誤引起的。注意這些問題：

- 在主機和微控制器之間應該使用高質量USB線，並確保插頭牢固。
- 如果使用樹莓派，請為樹莓派使用[優質電源](https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#power-supply)，並使用[優質USB電纜](https://forums.raspberrypi.com/viewtopic.php?p=589877#p589877)將該電源與Pi連線。如果你從OctoPrint得到 "電壓不足 "的警告，這也與電源有關，必須解決。
- 請確保印表機的電源沒有過載。(微控制器 USB 晶片的電壓波動可能會導致該晶片欠壓復位。)
- 確認步進電機、加熱器和其他印表機電線沒有被壓壞或磨損。（印表機移動時可能對故障的電線施加壓力，導致其失去接觸、短暫短路或產生過多的噪音。）
- 有報告稱，當印表機的電源和主機的5V電源混合時，USB噪音很大。(如果你發現當印表機的電源打開或USB電纜插入時，微控制器會通電，那麼這表明5V電源被混用了）。配置微控制器，使其只使用一個電源可能會有幫助。(另外，如果微控制器主板不能配置其供電來源，也可以修改USB電纜，使其主機和微控制器之間不共享5V電源。）

## 我的樹莓派在列印時不斷重啟

這很可能是由於電壓波動引起的。請遵循["與MCU失去通訊"](#i-keep-getting-random-lost-communication-with-mcu-errors)錯誤同樣的故障排除步驟。

## 當設定`restart_method=command`時， AVR 裝置在重啟時就會宕機

一些舊版本的AVR載入程式在看門狗事件處理方面有一個已知的錯誤。這通常表現在printer.cfg檔案中的restart_method設定為 "command"的時候。當這個錯誤發生時，AVR裝置會宕機直到電源被移除並重新上電（電源或狀態LED也可能不停的閃爍直到斷電）。

解決辦法是使用 "command "以外的重啟方法，或者在AVR裝置上刷寫一個新的載入程式。刷寫一個新的載入程式是一個一次性的步驟，通常需要一個外部程式設計器--更多細節請參見[Bootloaders](Bootloaders.md)。

## 如果 Raspberry Pi 崩潰了，加熱器會一直加熱嗎？

Klipper 的設計防止了這種情況。一旦主機啟用了一個加熱器，主機軟體需要每 5 秒鐘確認一次啟用狀態。如果微控制器沒有收到每 5 秒的確認，它就會進入"關閉"狀態，該狀態會關閉所有加熱器和步進電機。

詳情請見[MCU 命令](MCU_Commands.md)文件中的 "config_digital_out" 命令。

此外，微控制器軟體在啟動時為每個加熱器配置了最低和最高溫度範圍（詳見[配置參考](Config_Reference.md#extruder)中的min_temp和max_temp參數）。如果微控制器檢測到溫度超出了該範圍，那麼它也將進入 "關機 "狀態。

另外，主機軟體還實現了檢查加熱器和溫度感測器是否正常工作的程式碼。詳情見[配置參考](Config_Reference.md#verify_heater) 。

## 如何將 Marlin 引腳編號轉換為 Klipper 引腳名稱？

簡要回答：在[sample-aliases.cfg](../config/sample-aliases.cfg)檔案中有一個對映。使用該檔案作為指引來尋找實際的微控制器引腳名稱。(也可以將相關的[board_pins](Config_Reference.md#board_pins)配置部分複製到你的配置檔案中，在你的配置中使用別名，但最好是翻譯並使用實際的微控制器引腳名稱。) 請注意，sample-aliases.cfg檔案使用以 "ar "開頭的引腳名稱替換 "D"（例如，Arduino引腳`D23`是Klipper別名`ar23`）和以 "analog "開頭的引腳名稱替換 "A"（例如，Arduino引腳`A14`是Klipper別名`analog14`）。

詳細答案：Klipper使用微控制器定義的標準引腳名稱。在 Atmega 晶片上，這些硬體引腳的名稱類似於`PA4`、`PC7`或`PD2`。

很久以前，Arduino專案決定避免使用標準的硬體名稱，而採用他們自己的基於遞增數字的引腳名稱--這些Arduino名稱一般看起來像`D23`或`A14`。這是一個不幸的選擇，導致了大量的混亂。特別是Arduino的引腳號碼經常不能轉換為相同的硬體名稱。例如，`D21`在一個常見的Arduino板上是`PD0`，但在另一個常見的Arduino板上是`PC7`。

爲了避免這種混淆，Klipper 核心程式碼使用微控制器定義的標準引腳名稱。

## 我必須將裝置連線到特定型別的微控制器引腳嗎？

這取決於裝置和引腳的型別：

ADC 引腳（或模擬引腳）：熱敏電阻和類似的「模擬」感測器必須連線到微控制器上具有「模擬」或「ADC」功能的引腳。如果您在 Klipper 上為一個模擬感測器配置為使用不具備模擬功能的引腳，Klipper 將報告「Not a valid ADC pin」錯誤。

PWM引腳（或定時器引腳）。Klipper預設不在任何裝置上使用硬體PWM。因此，一般來說，可以將熱端、風扇和類似的裝置連線到任何的通用IO引腳。然而，風扇和output_pin裝置可以選擇配置為使用`hardware_pwm: True`，在這種情況下，微控制器在這個引腳上必須支援硬體PWM（否則，Klipper將報告 "不是有效的PWM引腳 "錯誤）。

IRQ引腳（或中斷引腳）。Klipper不使用IO引腳上的硬體中斷，所以沒有必要將裝置連線到這些微控制器引腳上。

SPI 引腳：使用硬體 SPI 時，需要將引腳連線到微控制器的 SPI 引腳。但是，大多數裝置都可以配置為使用「軟體 SPI」，在這種情況下，可以使用任何通用 IO 引腳。

I2C 引腳：使用 I2C 時，必須將引腳連線到微控制器支援 I2C 的引腳。

其他裝置可以連線到任何通用的IO引腳。例如，步進電機、加熱器、風扇、Z探頭、舵機、LED、通用的hd44780/st7920液晶顯示器和Trinamic UART控制線都可以連線到任何通用的IO引腳。

## 如何取消 M109/M190 "等待達到目標溫度"的請求？

導航到OctoPrint終端標籤頁，在終端框中輸入M112命令。M112命令將使Klipper進入 "關機 "狀態，它將使OctoPrint從Klipper斷開。導航到OctoPrint連線區，點選 "連線"，使OctoPrint重新連線。導航回到終端標籤頁，輸入FIRMWARE_RESTART命令以清除Klipper的錯誤狀態。完成這一系列操作將取消先前的加熱請求，可以開始新的列印。

## 怎麼檢查印表機是否發生了丟步?

在某種程度上，是的。將印表機歸位，發出 `GET_POSITION `命令，執行列印，再次歸位，再發出 `GET_POSITION`。然後比較 `mcu: `行中的值。

這可能有助於調整步進電機電流、加速度和速度等設定，而不需要實際列印東西和浪費材料：只要在 `GET_POSITION `命令之間執行一些高速移動。

請注意，限位開關本身的觸發位置略有不同，所以導致限位開關觸發結果可能有幾個微步的誤差。步進電機本身只能以4個整步為增量損失步數。(因此，如果使用16個微步，那麼步進器電機上的失步將導致 "MCU: "步進計數器偏離64微步的倍數。）

## 為什麼 Klipper 會報錯？我收了一盤面條（我又打廢了）！

簡短的回答：我們想知道印表機是否檢測到問題，這樣就可以解決潛在的問題，就可以獲得高質量的列印件。我們絕對不希望印表機默默地生產低質量的列印件。

詳細答案：Klipper被設計成能夠自動解決許多瞬時問題。例如，它自動檢測到通訊錯誤，並會重新傳輸；它提前安排行動，並在多層緩衝命令，以便即使有間歇性干擾也能精確控制時序。然而，如果軟體檢測到它無法恢復的錯誤，或者它被命令採取無效的行動，或者它檢測到它無法執行其被命令的任務，那麼Klipper將報告錯誤。在這些情況下，產生低質量印刷件（或更糟）的風險很大。我們希望通過提醒使用者，使他們有能力解決潛在的問題，提高列印的整體質量。

有一些相關的問題。為什麼Klipper不暫停列印？為什麼不先警告？為什麼不在列印前檢查錯誤？為什麼不忽略使用者輸入的命令中的錯誤？等等？目前，Klipper使用G-Code協議讀取命令，不幸的是，G-Code命令協議不夠靈活，導致現在並不能使用這些替代方案。開發者對改善異常事件中的使用者體驗很感興趣，但預計這將需要大量的底層工作（包括從G-Code的遷移到別的方式）。

## 我怎樣才能升級到最新的軟體？

升級軟體的第一步是檢視最新的[配置變更](Config_Changes.md)檔案。作為軟體升級的一部分，偶爾有時候軟體會有一些變化需要使用者更新他們的設定。在升級前最好檢視該檔案。

當準備升級時，一般的方法是ssh進入樹莓派並執行：

```
cd ~/klipper
git pull
~/klipper/scripts/install-octopi.sh
```

然後可以重新編譯和刷寫微控制器的程式碼。例如：

```
make menuconfig
make clean
make

sudo service klipper stop
make flash FLASH_DEVICE=/dev/ttyACM0
sudo service klipper start
```

然而，通常是隻有主機軟體發生變化。這時可以只更新和重新啟動主機軟體：

```
cd ~/klipper
git pull
sudo service klipper restart
```

如果在使用這個快速方式后，軟體警告說需要重新刷寫微控制器或出現其他不常見的錯誤，那麼請按照上面所述的完整升級步驟進行。

如果任何錯誤持續存在，那麼請仔細檢查[config changes](Config_Changes.md)檔案，因為可能需要修改印表機配置。

請注意，RESTART和FIRMWARE_RESTART g-code命令不會載入新的軟體--需要使用上述 "sudo service klipper restart "和 "make flash "命令才能使軟體變化生效。

## 如何解除安裝Klipper？

在韌體端，沒有什麼特別需要做的。只要刷寫新韌體就可以了。

在樹莓派上，解除安裝指令碼在這裡[scripts/klipper-uninstall.sh](../scripts/klipper-uninstall.sh)。例如：

```
sudo ~/klipper/scripts/klipper-uninstall.sh
rm -rf ~/klippy-env ~/klipper
```
