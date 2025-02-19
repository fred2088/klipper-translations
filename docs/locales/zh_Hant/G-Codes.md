# G-Codes

本文件描述了 Klipper 支援的命令。這些命令可以輸入到 OctoPrint 終端中。

## G程式碼命令

Klipper支援以下標準的G-Code命令：

- 移動 (G0 or G1): `G1 [X<pos>] [Y<pos>] [Z<pos>] [E<pos>] [F<speed>]`
- 停留時間：`G4 P<milliseconds>`
- 返回原點：`G28 [X] [Y] [Z]`
- 關閉步進電機：`M18`或`M84`
- 等待目前移動完成： `M400`
- 使用絕對/相對擠出距離：`M82`， `M83`
- 使用絕對/相對座標：`G90`, `G91`
- 設定座標：`G92 [X<座標>] [Y<座標>] [Z<座標>] [E<座標>]`
- 設定速度因子覆寫百分比：`M220 S<百分比>`
- 設定擠壓因子覆蓋百分比：`M221 S<percent>`
- 設定加速度：`M204 S<value>` 或 `M204 P<value> T<value>`
   - 注意：如果沒有指定S，同時指定了P和T，那麼加速度將被設定為P和T中的最小值。
- 獲取擠出機溫度：`M105`
- 設定擠出機溫度：`M104 [T<index>] [S<temperature>]`
- 設定擠出機溫度並等待：`M109 [T<index>] S<temperature>`
   - 注意：M109總是等待溫度穩定在請求的數值上
- 設定熱床溫度：`M140 [S<temperature>]`
- 設定熱床溫度並且等待：`M190 S<temperature>`
   - 注意：M190總是等待溫度穩定在請求的數值上
- 設定風扇速度：`M106 S<value>`
- 停止風扇：`M107`
- 緊急停止：`M112`
- 獲取目前位置：`M114`
- 獲取韌體版本：`M115`

有關上述命令的更多詳細資訊，請參閱 [RepRap G-Code documentation](http://reprap.org/wiki/G-code)

Klipper的目標是支援常見第三方軟體（如OctoPrint、Printrun、Slic3r、Cura等）在其標準配置中產生的 G 程式碼命令。支援所有可能的 G-Code 命令並不是我們的目標。相反，Klipper更喜歡人類可讀的["擴充套件G-Code命令"](#additional-commands)。

如果一個人需要一個不太常見的G-Code命令，那麼可以用一個自定義的[gcode_macro config section](Config_Reference.md#gcode_macro)來實現它。例如，我們可以用這個來實現。`G12`, `G29`, `G30`, `G31`, `M42`, `M80`, `M81`, `T1` ，etc.

## 其他命令

Klipper使用 "extended" 的G程式碼命令來進行一般的配置和狀態。這些擴充套件命令都遵循一個類似的格式--它們以一個命令名開始，後面可能有一個或多個參數。比如說：`SET_SERVO SERVO=myservo ANGLE=5.3`。在本檔案中，命令和參數以大寫字母顯示，但它們不分大小寫。(所以，"SET_SERVO "和 "set_servo "都是執行同一個命令）

This section is organized my Klipper module name, which generally follows the section names specified in the [printer configuration file](Config_Reference.md). Note that some modules are automatically loaded.

### [adxl345]

The following commands are available when an [adxl345 config section](Config_Reference.md#adxl345) is enabled.

#### ACCELEROMETER_MEASURE

`ACCELEROMETER_MEASURE [CHIP=<config_name>] [NAME=<value>]` 。以要求的每秒採樣數啟動加速度計測量。如果沒有指定CHIP，則預設為 "adxl345"。該命令以啟動-停止模式工作：第一次執行時，它開始測量，下次執行時停止測量。測量結果被寫入一個名為`/tmp/adxl345-<chip>-<name>的檔案中。csv`，其中`<chip>`是加速度計晶片的名稱（`my_chip_name`來自`[adxl345 my_chip_name]`），`<name>`是可選NAME參數。如果沒有指定NAME，則預設為目前時間，格式為 "YYYMMDD_HHMMSS"。如果加速度計在其配置部分沒有名稱（只是`[adxl345]`），那麼`<chip >`部分的名稱就不會產生。

#### ACCELEROMETER_QUERY

`ACCELEROMETER_QUERY [CHIP=<config_name>] [RATE=<value>]`: 查詢加速度計的當前值。如果沒有指定晶片，則預設為 "adxl345"。如果沒有指定RATE，則使用預設值。該命令對於測試與ADXL345加速度計的連線非常有用：返回的數值之一應該是自由落體加速度（+/-晶片的一些噪聲）。

#### ACCELEROMETER_DEBUG_READ

`ACCELEROMETER_DEBUG_READ [CHIP=<配置名>] REG=<暫存器>`：查詢ADXL345的暫存器"REG"（例如44或0x2C）。可以用於debug。

#### ACCELEROMETER_DEBUG_WRITE

`ACCELEROMETER_DEBUG_WRITE [CHIP=<配置名>] REG=<暫存器> VAL=<值>`：將原始的"值"寫進暫存器"暫存器"。"值"和"暫存器"都可以是一個十進制或十六進制的整數。請謹慎使用，並參考 ADXL345 數據手冊。

### [bed_mesh]

The following commands are available when the [bed_mesh config section](Config_Reference.md#bed_mesh) is enabled (also see the [bed mesh guide](Bed_Mesh.md)).

#### BED_MESH_CALIBRATE

`BED_MESH_CALIBRATE [METHOD=manual] [<probe_parameter>=<value>] [<mesh_parameter>=<value>]`: 此命令使用通過配置參數指定並產生的探測點探測列印床。在探測后，一個網格將被產生，z 軸移動將根據網格調整。有關可選探測參數，請見 PROBE命令。如果指定 METHOD=manual ，則會啟動手動探測工具 - 有關此工具活躍時可用的額外命令，詳見 MANUAL_PROBE 命令。

#### BED_MESH_OUTPUT

`BED_MESH_OUTPUT PGP=[<0:1>]`：該命令將目前探測到的 Z 值和目前網格的值輸出到終端。如果指定 PGP=1，則將bed_mesh產生的X、Y座標，以及它們關聯的指數，輸出到終端。

#### BED_MESH_MAP

`BED_MESH_MAP`：類似 BED_MESH_OUTPUT，這個命令在終端中顯示網格的當前狀態。它不以人類可讀格式列印，而是被序列化為 json 格式。這允許 Octoprint 外掛捕獲數據並產生描繪列印床表面的高度圖。

#### BED_MESH_CLEAR

`BED_MESH_CLEAR`：此命令清除床網並移除所有 z 調整。建議把它放在你的 end-gcode （結束G程式碼）中。

#### BED_MESH_PROFILE

`BED_MESH_PROFILE LOAD=<名稱> SAVE=<名稱> REMOVE=<名稱>`：此命令提供了網床配置管理功能。LOAD 將從與所提供的名稱相符的配置檔案中恢復網格狀態。SAVE 將會把目前的網格狀態儲存到與提供的名稱相符的配置檔案中。REMOVE（移除）將從永續性記憶體中刪除與所提供名稱相符的配置檔案。請注意，在 SAVE 或 REMOVE 操作后，必須發送SAVE_CONFIG G程式碼，以儲存變更到永續性記憶體。

#### BED_MESH_OFFSET

`BED_MESH_OFFSET [X=<value>] [Y=<value>]`。將X和/或Y的偏移量應用於網格查詢。這對具有多個獨立擠出頭的印表機很有用，因為偏移量對切換工具頭后產生正確的 Z 值調整是至關重要的。

### [bed_screws]

The following commands are available when the [bed_screws config section](Config_Reference.md#bed_screws) is enabled (also see the [manual level guide](Manual_Level.md#adjusting-bed-leveling-screws)).

#### BED_SCREWS_ADJUST

`BED_SCREWS_ADJUST`：該命令將呼叫列印床螺絲調整工具。它將命令噴嘴移動到不同的位置（在配置檔案中定義），並允許對列印床螺絲進行手動調整，使列印床與噴嘴的距離保持不變。

### [bed_tilt]

The following commands are available when the [bed_tilt config section](Config_Reference.md#bed_tilt) is enabled.

#### BED_TILT_CALIBRATE

`BED_TILT_CALIBRATE [Method=manual] [<probe_parameter>=<value>] `:該命令將探測配置中指定的點，然後建議更新X和Y的傾斜調整。有關可選探測參數的詳細資訊，請參見PROBE命令。如果指定METHOD=manual，那麼手動探測工具就會被啟用 - 關於該工具啟用時可用的附加命令，請參見上面的MANUAL_PROBE命令。

### [bltouch]

The following command is available when a [bltouch config section](Config_Reference.md#bltouch) is enabled (also see the [BL-Touch guide](BLTouch.md)).

#### BLTOUCH_DEBUG

`BLTOUCH_DEBUG COMMAND=<命令>`：向BLTouch發送一個指定的命令，可以用於除錯。可用的命令有：`pin_down`、`touch_mode`、`pin_up`、`self_test`和`reset`。BL-TOUCH V3.0 或 V3.1 也可能支援`set_5V_output_mode`、`set_OD_output_mode`和`output_mode_store`命令。

#### BLTOUCH_STORE

`BLTOUCH_STORE MODE=<output_mode>`:這將在BLTouch V3.1的EEPROM中儲存一個輸出模式 可用的輸出模式有`5V`, `OD`

### [configfile]

configfile模組被自動載入.

#### SAVE_CONFIG

`SAVE_CONFIG`：該命令將覆蓋印表機的主配置檔案，並重新啟動主機軟體。該命令與其他校準命令一起使用，用於儲存校準測試的結果。

### [delayed_gcode]

The following command is enabled if a [delayed_gcode config section](Config_Reference.md#delayed_gcode) has been enabled (also see the [template guide](Command_Templates.md#delayed-gcodes)).

#### UPDATE_DELAYED_GCODE

`UPDATE_DELAYED_GCODE [ID=<名稱>] [DURATION=<秒>]`：更新目標 [delayed_gcode] 的延遲並啟動G程式碼執行的計時器。為0的值會取消準備執行的延遲G程式碼。

### [delta_calibrate]

The following commands are available when the [delta_calibrate config section](Config_Reference.md#linear-delta-kinematics) is enabled (also see the [delta calibrate guide](Delta_Calibrate.md)).

#### DELTA_CALIBRATE

`DELTA_CALIBRATE [Method=manual] [<probe_parameter>=<value>] `:這條命令將探測床身的七個點，並建議更新限位位置、塔架角度和半徑。有關可選探測參數的詳細資訊，請參見PROBE命令。如果指定METHOD=manual，那麼手動探測工具將被啟用 - 關於該工具啟用時可用的附加命令的詳細資訊，請參見上面的MANUAL_PROBE命令。

#### DELTA_ANALYZE

`DELTA_ANALYZE`:這個命令在增強的delta校準過程中使用。詳情見[Delta Calibrate](Delta_Calibrate.md)。

### [display]

The following command is available when a [display config section](Config_Reference.md#gcode_macro) is enabled.

#### SET_DISPLAY_GROUP

`SET_DISPLAY_GROUP [DISPLAY=<display>] GROUP=<group>`:設定一個lcd顯示器的活動顯示組。這允許在配置中定義多個顯示數據組，例如`[display_data <group> <elementname>]`並使用這個擴充套件的gcode命令在它們之間切換。如果沒有指定DISPLAY，則預設為 "display"（主顯示）。

### [display_status]

The display_status module is automatically loaded if a [display config section](Config_Reference.md#display) is enabled. It provides the following standard G-Code commands:

- 顯示資訊： `M117 <message> `
- 設定構建百分比：`M73 P<percent>`

### [dual_carriage]

The following command is available when the [dual_carriage config section](Config_Reference.md#dual_carriage) is enabled.

#### SET_DUAL_CARRIAGE

`SET_DUAL_CARRIAGE CARRIAGE=[0|1]`:該命令將設定活動的滑塊。它通常是在一個多擠出機配置中從 activate_gcode和deactivate_gcode欄位呼叫。

### [endstop_phase]

The following commands are available when an [endstop_phase config section](Config_Reference.md#endstop_phase) is enabled (also see the [endstop phase guide](Endstop_Phase.md)).

#### ENDSTOP_PHASE_CALIBRATE

`ENDSTOP_PHASE_CALIBRATE [STEPPER=<config_name>]` 。如果沒有提供STEPPER參數，那麼該命令將報告在過去的歸位操作中對端停步進相的統計。當提供STEPPER參數時，它會安排將給定的終點站相位設定寫入配置檔案中（與SAVE_CONFIG命令一起使用）。

### [extruder]

The following commands are available if an [extruder config section](Config_Reference.md#extruder) is enabled:

#### ACTIVATE_EXTRUDER

`ACTIVATE_EXTRUDER EXTRUDER=<config_name>`：這個命令在具有多個擠出機的印表機中用於更改活動擠出機。

#### SET_PRESSURE_ADVANCE

`SET_PRESSURE_ADVANCE [EXTRUDER=<擠出機名稱>] [ADVANCE=<pressure_advance>] [SMOOTH_TIME=<pressure_advance_smooth_time>]` ：設定壓力提前的參數。如果沒有指定擠出機，則預設為活動的擠出機。

#### SET_EXTRUDER_ROTATION_DISTANCE

`SET_EXTRUDER_ROTATION_DISTANCE EXTRUDER=<config_name> [DISTANCE=<distance>]`: Set a new value for the provided extruder's "rotation distance". If the rotation distance is a negative number then the stepper motion will be inverted (relative to the stepper direction specified in the config file). Changed settings are not retained on Klipper reset. Use with caution as small changes can result in excessive pressure between extruder and hot end. Do proper calibration with filament before use. If 'DISTANCE' value is not included command will return current rotation distance.

#### SYNC_EXTRUDER_MOTION

`SYNC_EXTRUDER_MOTION EXTRUDER=<name> MOTION_QUEUE=<name>`: This command will cause the stepper specified by EXTRUDER (as defined in an [extruder](Config_Reference#extruder) or [extruder_stepper](Config_Reference#extruder_stepper) config section) to become synchronized to the movement of an extruder specified by MOTION_QUEUE (as defined in an [extruder](Config_Reference#extruder) config section). If MOTION_QUEUE is an empty string then the stepper will be desynchronized from all extruder movement.

#### SET_EXTRUDER_STEP_DISTANCE

This command is deprecated and will be removed in the near future.

#### SYNC_STEPPER_TO_EXTRUDER

This command is deprecated and will be removed in the near future.

### [fan_generic]

The following command is available when a [fan_generic config section](Config_Reference.md#fan_generic) is enabled.

#### SET_FAN_SPEED

`SET_FAN_SPEED FAN=config_name SPEED=<速度>`該命令設定風扇的速度。"速度" 必須在0.0到1.0之間。

### [firmware_retraction]

The following standard G-Code commands are available when the [firmware_retraction config section](Config_Reference.md#firmware_retraction) is enabled. These commands allow you to utilize the firmware retraction feature available in many slicers, to reduce stringing during non-extrusion moves from one part of the print to another. Appropriately configuring pressure advance reduces the length of retraction required.

- `G10`：使用目前配置的參數回抽擠出機。
- `G11`：不使用目前配置的參數回抽擠出機。

還可以使用以下額外命令.

#### SET_RETRACTION

`SET_RETRACTION [RETRACT_LENGTH=<毫米>] [RETRACT_SPEED=<毫米每秒>] [UNRETRACT_EXTRA_LENGTH=<毫米>] [UNRETRACT_SPEED=<毫米每秒>]`：調整韌體回抽所使用的參數。RETRACT_LENGTH 決定回抽和回填的耗材長度。回抽的速度通過 RETRACT_SPEED 調整，通常設定得比較高。回填的速度通過 UNRETRACT_SPEED 調整，雖然經常比RETRACT_SPEED 低，但不是特別重要。在某些情況下，在回填時增加少量的額外長度的耗材可能有益，這可以通過 UNRETRACT_EXTRA_LENGTH 設定。SET_RETRACTION 通常作為切片機耗材配置的一部分來設定，因為不同的耗材需要不同的參數設定。

#### GET_RETRACTION

`GET_RETRACTION`:查詢目前韌體回抽所使用的參數並在終端顯示。

### [filament_switch_sensor]

The following command is available when a [filament_switch_sensor](Config_Reference.md#filament_switch_sensor) or [filament_motion_sensor](Config_Reference.md#filament_motion_sensor) config section is enabled.

#### QUERY_FILAMENT_SENSOR

`QUERY_FILAMENT_SENSOR SENSOR=<感測器名>`：查詢耗材感測器的當前狀態。在終端中顯示的數據將取決於配置中定義的感測器型別。

#### SET_FILAMENT_SENSOR

`SET_FILAMENT_SENSOR SENSOR=<sensor_name> ENABLE=[0|1]` ：設定燈絲感測器的開/關。如果 ENABLE 設定為 0，耗材感測器將被禁用，如果設定為 1是啟用。

### [firmware_retraction]

如果啟用了[firmware_retraction 配置分段](Config_Reference.md#firmware_retraction)，則以下標準G程式碼命令可用：

- 回抽：`G10`
- 回填：`G11`

### [force_move]

The force_move module is automatically loaded, however some commands require setting `enable_force_move` in the [printer config](Config_Reference#force_move).

#### STEPPER_BUZZ

`STEPPER_BUZZ STEPPER=<配置名>`：移動指定的步進電機前後運動一毫米，重複的10次。這是一個用於驗證步進電機接線的工具.

#### FORCE_MOVE

`FORCE_MOVE STEPPER=<config_name> DISTANCE=<value> VELOCITY=<value> [ACCEL=<value>]` 。該命令將以給定的恒定速度（mm/s）強制移動給定的步進器，移動距離（mm）。如果指定了ACCEL並且大於零，那麼將使用給定的加速度（單位：mm/s^2）；否則不進行加速。不執行邊界檢查；不進行運動學更新；一個軸上的其他平行步進器將不會被移動。請謹慎使用，因為不正確的命令可能會導致損壞使用該命令幾乎肯定會使低階運動學處於不正確的狀態；隨後發出G28命令以重置運動學。該命令用於低階別的診斷和除錯。

#### SET_KINEMATIC_POSITION

`SET_KINEMATIC_POSITION [X=<值>] [Y=<值>] [Z=<值>]`：強制設定低階運動學程式碼使用的列印頭位置。這是一個診斷和除錯工具，常規的軸轉換應該使用 SET_GCODE_OFFSET 和/或 G92指令。如果沒有指定一個軸，則使用上一次命令的列印頭位置。設定一個不合理的數值可能會導致內部程式問題。這個命令可能會使邊界檢查失效，使用后發送 G28 來重置運動學。

### [gcode]

The gcode module is automatically loaded.

#### RESTART

`RESTART`：這將導致主機軟體重新載入其配置並執行內部重置。此命令不會從微控制器清除錯誤狀態（請參閱 FIRMWARE_RESTART），也不會載入新軟體（請參閱 [常見問題](FAQ.md#how-do-i-upgrade-to-the-latest-software)） .

#### FIRMWARE_RESTART

`FIRMWARE_RESTART`：這類似於重啟命令，但它也清除了微控制器的任何錯誤狀態。

#### STATUS

`STATUS`：報告Klipper主機程式的狀態。

#### HELP

`HELP`：報告可用的擴充套件G-Code命令列表。

### [gcode_arcs]

如果啟用了[gcode_arcs 配置分段](Config_Reference.md#gcode_arcs)，下列標準G程式碼命令可用：

- 受控弧線移動（G2或G3）。`G2 [X<pos>] [Y<pos>] [Z<pos>] [E<pos>] [F<speed>] I<value> J<value>`

### [gcode_macro]

The following command is available when a [gcode_macro config section](Config_Reference.md#gcode_macro) is enabled (also see the [command templates guide](Command_Templates.md)).

#### SET_GCODE_VARIABLE

`SET_GCODE_VARIABLE MACRO=<macro_name> VARIABLE=<name> VALUE=<value>`：這條命令允許人們在執行時改變 gcode_macro 變數的值。所提供的 VALUE 會被解析為一個 Python 字面。

### [gcode_move]

The gcode_move module is automatically loaded.

#### GET_POSITION

`GET_POSITION`：返回工具目前位置資訊.

#### SET_GCODE_OFFSET

`SET_GCODE_OFFSET [X=<pos>|X_ADJUST=<adjust>] [Y=<pos>|Y_ADJUST=<adjust>] [Z=<pos>|Z_ADJUST=<adjust>] [MOVE=1 [MOVE_SPEED=<speed>]]` 。設定一個位置偏移，以應用於未來的G程式碼命令。這通常用於實際改變Z床的偏移量或在切換擠出機時設定噴嘴的XY偏移量。例如，如果發送 "SET_GCODE_OFFSET Z=0.2"，那麼未來的G程式碼移動將在其Z高度上增加0.2mm。如果使用X_ADJUST風格的參數，那麼調整將被新增到任何現有的偏移上（例如，"SET_GCODE_OFFSET Z=-0.2"，然後是 "SET_GCODE_OFFSET Z_ADJUST=0.3"，將導致總的Z偏移為0.1）。如果指定了 "MOVE=1"，那麼將發出一個工具頭移動來應用給定的偏移量（否則偏移量將在指定給定軸的下一次絕對G-Code移動中生效）。如果指定了 "MOVE_SPEED"，那麼刀頭移動將以給定的速度（mm/s）執行；否則，列印頭移動將使用最後指定的G-Code速度。

#### SAVE_GCODE_STATE

`SAVE_GCODE_STATE [NAME=<state_name>]`：儲存目前的g-code座標解析狀態。儲存和恢復g-code狀態在指令碼和宏中很有用。該命令儲存目前g-code絕對座標模式（G90/G91）絕對擠出模式（M82/M83）原點（G92）偏移量（SET_GCODE_OFFSET）速度覆蓋（M220）擠出機覆蓋（M221）移動速度。目前XYZ位置和相對擠出機 "E "位置。如果提供NAME，它允許人們將儲存的狀態命名為給定的字串。如果沒有提供NAME，則預設為 "default".

#### RESTORE_GCODE_STATE

`RESTORE_GCODE_STATE [NAME=<state_name>] [MOVE=1 [MOVE_SPEED=<speed>]]`：恢復之前通過 SAVE_GCODE_STATE 儲存的狀態。如果指定「MOVE=1」，則將發出刀頭移動以返回到先前的 XYZ 位置。如果指定了「MOVE_SPEED」，則刀頭移動將以給定的速度（以mm/s為單位）執行；否則工具頭移動將使用恢復的G-Code速度。

### [hall_filament_width_sensor]

The following commands are available when the [tsl1401cl filament width sensor config section](Config_Reference.md#tsl1401cl_filament_width_sensor) or [hall filament width sensor config section](Config_Reference.md#hall_filament_width_sensor) is enabled (also see [TSLl401CL Filament Width Sensor](TSL1401CL_Filament_Width_Sensor.md) and [Hall Filament Width Sensor](Hall_Filament_Width_Sensor.md)):

#### QUERY_FILAMENT_WIDTH

`QUERY_FILAMENT_WIDTH`：返回目前測量的耗材直徑。

#### RESET_FILAMENT_WIDTH_SENSOR

`RESET_FILAMENT_WIDTH_SENSOR`：清除全部感測器讀數。在更換耗材後有用。

#### DISABLE_FILAMENT_WIDTH_SENSOR

`DISABLE_FILAMENT_WIDTH_SENSOR`：關閉耗材直徑感測器並停止使用它進行流量控制。

#### ENABLE_FILAMENT_WIDTH_SENSOR

`ENABLE_FILAMENT_WIDTH_SENSOR`：啟用耗材直徑感測器並使用它進行流量控制。

#### QUERY_RAW_FILAMENT_WIDTH

`QUERY_RAW_FILAMENT_WIDTH`：返回目前 ADC 通道讀數和校準點的 RAW 感測器值。

#### ENABLE_FILAMENT_WIDTH_LOG

`ENABLE_FILAMENT_WIDTH_LOG`：開啟直徑記錄。

#### DISABLE_FILAMENT_WIDTH_LOG

`DISABLE_FILAMENT_WIDTH_LOG`：停止直徑記錄。

### [heaters]

The heaters module is automatically loaded if a heater is defined in the config file.

#### TURN_OFF_HEATERS

`TURN_OFF_HEATERS`：關閉全部加熱器。

#### TEMPERATURE_WAIT

`TEMPERATURE_WAIT SENSOR=<配置名> [MINIMUM=<目標>] [MAXIMUM=<目標>]`：等待指定溫度感測器讀數高於 MINIMUM 和或低於 MAXIMUM。

#### SET_HEATER_TEMPERATURE

`SET_HEATER_TEMPERATURE HEATER=<加熱器名稱> [TARGET=<目標溫度>]`：設定一個加熱器的目標溫度。如果沒有提供目標溫度，則目標溫度為 0。

### [idle_timeout]

The idle_timeout module is automatically loaded.

#### SET_IDLE_TIMEOUT

`SET_IDLE_TIMEOUT [TIMEOUT=<超時>]`：允許使用者設定空閑超時（以秒為單位）。

### [input_shaper]

The following command is enabled if an [input_shaper config section](Config_Reference.md#input_shaper) has been enabled (also see the [resonance compensation guide](Resonance_Compensation.md)).

#### SET_INPUT_SHAPER

`SET_INPUT_SHAPER [SHAPER_FREQ_X=<shaper_freq_x>] [SHAPER_FREQ_Y=<shaper_freq_y>] [DAMPING_RATIO_X=<damping_ratio_x>] [DAMPING_RATIO_Y=<damping_ratio_y>] [SHAPER_TYPE=<shaper>] [SHAPER_TYPE_X=<shaper_type_x>] [SHAPER_TYPE_Y=<shaper_type_y>]`：修改輸入整形參數。注意 SHAPER_TYPE 參數會同時覆寫 X 和 Y 軸的整形器型別，即使它們在 [input_shaper] 配置分段中有不同的整形器型別。SHAPER_TYPE 不能和 SHAPER_TYPE_X 和 SHAPER_TYPE_Y 參數同時使用。這些參數的細節請見[配置參考](Config_Reference.md#input_shaper)。

### [manual_probe]

The manual_probe module is automatically loaded.

#### MANUAL_PROBE

`MANUAL_PROBE [SPEED=<speed>]`：執行一個輔助指令碼，對測量給定位置的噴嘴高度有用。如果指定了速度，它將設定TESTZ命令的速度（預設為5mm/s）。在手動探測過程中，可使用以下附加命令：

- `ACCEPT`：該命令接受目前的Z位置，並結束手動探測工具。
- `ABORT`：該命令終止手動探測工具。
- `TESTZ Z=<值>`：這個命令可以將噴嘴上升或下降給定值，以毫米為單位。例如，`TESTZ Z=-.1` 會將噴嘴下降 0.1 毫米，而 `TESTZ Z=.1` 會將噴嘴上升 0.1 毫米，參數可以帶有`+`, `-`, `++`, or `--`來根據上次嘗試相對的移動噴嘴。

#### Z_ENDSTOP_CALIBRATE

`Z_ENDSTOP_CALIBRATE [SPEED=<速度>]`：執行一個校準 Z position_endstop 參數的輔助指令碼。有關更多參數和額外命令的資訊，請檢視 MANUAL_PROBE 命令。

#### Z_OFFSET_APPLY_ENDSTOP

`Z_OFFSET_APPLY_ENDSTOP`：將目前的Z 的 G 程式碼偏移量（就是 babystepping）從 stepper_z 的 endstop_position 中減去。該命令將持久化一個常用babystepping 微調值。需要執行 `SAVE_CONFIG`才能生效。

### [manual_stepper]

當[manual_stepper 配置分段](Config_Reference.md#manual_stepper)被啟用時，以下命令可用。

#### MANUAL_STEPPER

`MANUAL_STEPPER STEPPER=config_name [ENABLE=[0|1]] [SET_POSITION=<pos>] [SPEED=<speed>] [ACCEL=<accel>] [MOVE=<pos> [STOP_ON_ENDSTOP=[1|2|-1|-2]] [SYNC=0]]`：該命令將改變步進器的狀態。使用ENABLE參數來啟用/禁用步進。使用SET_POSITION參數，迫使步進認為它處於給定的位置。使用MOVE參數，要求移動到給定位置。如果指定了SPEED或者ACCEL，那麼將使用給定的值而不是配置檔案中指定的預設值。如果指定ACCEL為0，那麼將不執行加速。如果STOP_ON_ENDSTOP=1被指定，那麼如果止動器報告被觸發，動作將提前結束（使用STOP_ON_ENDSTOP=2來完成動作，即使止動器沒有被觸發也不會出錯，使用-1或-2來在止動器報告沒有被觸發時停止）。通常情況下，未來的G-Code命令將被安排在步進運動完成後執行，但是如果手動步進運動使用SYNC=0，那麼未來的G-Code運動命令可能與步進運動平行執行。

### [neopixel]

The following command is available when a [neopixel config section](Config_Reference.md#neopixel) or [dotstar config section](Config_Reference.md#dotstar) is enabled.

#### SET_LED

`SET_LED LED=<配置中的名稱> RED=<值> GREEN=<值> BLUE=<值> WHITE=<值> [INDEX=<索引>] [TRANSMIT=0] [SYNC=1]`：這條指令設定LED的輸出。每個顏色的 `<值>` 必須在0.0和1.0之間。WHITE（白色）參數僅在 RGBW LED上有效果。如果多個 LED 晶片是通過菊鏈連線的，可以用INDEX參數來指定改變指定晶片的顏色（1是第一顆晶片，2是第二顆晶片…）。如果不提供 INDEX 則全部菊鏈中的 LED 將被設定為給定的顏色。如果指定了 TRANSMIT=0 ，色彩變化只會在下一條不是 TRANSMIT=0 的 SET_LED命令發送后發生。這個特性和 INDEX 參數一起使用可以在菊鏈中批量改變LED的顏色。預設情況下，SET_LED命令將與其他正在進行的G程式碼命令同步其變化。 因為修改LED狀態會重置空閑超時，這可能在不列印時造成不期望的行為。 如果精確的時序不重要，可選的 SYNC=0 參數可以立刻應用LED變化而不重置LED超時。

### [output_pin]

The following command is available when an [output_pin config section](Config_Reference.md#output_pin) is enabled.

#### SET_PIN

`SET_PIN PIN=config_name VALUE=<值> CYCLE_TIME=<週期時間>`: 注意 - 硬體PWM目前不支援CYCLE_TIME參數，將使用配置中定義的週期時間。

### [palette2]

The following commands are available when the [palette2 config section](Config_Reference.md#palette2) is enabled.

Palette列印通過在GCode檔案中嵌入特殊的OCodes（Omega Codes）來工作:

- `O1`...`O32`：這些程式碼從G-Code流中讀出並且傳遞給Palette 2裝置進行處理。

還可以使用以下額外命令.

#### PALETTE_CONNECT

`PALETTE_CONNECT`：該命令初始化與Palette 2的連線。

#### PALETTE_DISCONNECT

`PALETTE_DISCONNECT`：該命令斷開與Palette 2的連線。

#### PALETTE_CLEAR

`PALETTE_CLEAR`:該命令指示 Palette 2 清除所有耗材的輸入或者輸出。

#### PALETTE_CUT

`PALETTE_CUT`:該命令指引Palette 2切割耗材並且裝載分段的耗材。

#### PALETTE_SMART_LOAD

`PALETTE_SMART_LOAD`：該命令在Palette 2上啟動智慧載入序列。通過在裝置上為印表機校準的距離擠壓，自動載入耗材，並在載入完成後指示Palette 2。該命令與耗材載入完成後直接在Palette 2螢幕上按**Smart Load**相同。

### [pid_calibrate]

The pid_calibrate module is automatically loaded if a heater is defined in the config file.

#### PID_CALIBRATE

`PID_CALIBRATE HEATER=<config_name> TARGET=<temperature> [WRITE_FILE=1]`：執行一個PID校準測試。指定的加熱器將被啟用，直到達到指定的目標溫度，然後加熱器將被關閉和開啟幾個週期。如果WRITE_FILE參數被啟用，那麼將建立檔案/tmp/heattest.txt，其中包含測試期間所有溫度樣本的日誌。

### [pause_resume]

當[pause_resume 配置分段](Config_Reference.md#pause_resume)被啟用時，以下命令可用：

#### PAUSE

`PAUSE`：暫停當前的列印。目前的位置被報錯以便在恢復時恢復。

#### RESUME

`RESUME [VELOCITY=<value>]`：從暫停中恢復列印，首先恢復以前保持的位置。VELOCITY參數決定了工具返回到原始捕捉位置的速度。

#### CLEAR_PAUSE

`CLEAR_PAUSE`:清除目前的暫停狀態而不恢復列印。如果一個人決定在暫停后取消列印，這很有用。建議將其新增到你的啟動程式碼中，以確保每次列印時的暫停狀態是新的。

#### CANCEL_PRINT

`CANCEL_PRINT`：取消目前的列印。

### [probe]

The following commands are available when a [probe config section](Config_Reference.md#probe) or [bltouch config section](Config_Reference.md#bltouch) is enabled (also see the [probe calibrate guide](Probe_Calibrate.md)).

#### PROBE

`PROBE [PROBE_SPEED=<mm/s>] [LIFT_SPEED=<mm/s>] [SAMPLES=<count>] [SAMPLE_RETRACT_DIST=<mm>] [SAMPLES_TOLERANCE=<mm>] [SAMPLES_TOLERANCE_RETRIES=<count>] [SAMPLES_RESULT=median|average]`：向下移動噴嘴直到探針觸發。如果提供了任何可選參數，它們將覆蓋 [probe config section](Config_Reference.md#probe) 中的等效設定。

#### QUERY_PROBE

`QUERY_PROBE`:報告探針的當前狀態（"triggered"或 "open"）。

#### PROBE_ACCURACY

`PROBE_ACCURACY [PROBE_SPEED=<mm/s>] [SAMPLES=<count>] [SAMPLE_RETRACT_DIST=<mm>]`：計算多個探針樣本的最大、最小、平均、中位數和標準偏差。預設情況下采樣10次。否則可選參數預設為探針配置部分的同等設定。

#### PROBE_CALIBRATE

`PROBE_CALIBRATE [SPEED=<speed>] [<probe_parameter>=<value>] `:執行一個對校準測頭的z_offset有用的輔助指令碼。有關可選測頭參數的詳細資訊，請參見PROBE命令。參見MANUAL_PROBE命令，瞭解SPEED參數和工具啟用時可用的附加命令的詳細資訊。請注意，PROBE_CALIBRATE命令使用速度變數在XY方向以及Z方向上移動。

#### Z_OFFSET_APPLY_PROBE

`Z_OFFSET_APPLY_PROBE`：將目前的Z 的 G 程式碼偏移量（就是 babystepping）從 probe 的 z_offset 中減去。該命令將持久化一個常用babystepping 微調值。需要執行 `SAVE_CONFIG`才能生效。

### [query_adc]

The query_endstops module is automatically loaded.

#### QUERY_ADC

`QUERY_ADC [NAME=<config_name>] [PULLUP=<value>]` ：返回為配置的模擬引腳收到的最後一個模擬值。如果NAME沒有被提供，將報告可用的adc名稱列表。如果提供了PULLUP（以歐姆為單位的數值），將會返回原始模擬值和給定的等效電阻。

### [query_endstops]

The query_endstops module is automatically loaded. The following standard G-Code commands are currently available, but using them is not recommended:

- 獲取限位狀態：`M119` (使用QUERY_ENDSTOPS代替)

#### QUERY_ENDSTOPS

`QUERY_ENDSTOPS`：檢測限位並返回限位是否被 "triggered"或處於"open"。該命令通常用於驗證一個限位是否正常工作。

### [resonance_tester]

The following commands are available when a [resonance_tester config section](Config_Reference.md#resonance_tester) is enabled (also see the [measuring resonances guide](Measuring_Resonances.md)).

#### MEASURE_AXES_NOISE

`MEASURE_AXES_NOISE`：測量並輸出所有啟用的加速度計晶片的所有軸的噪聲。

#### TEST_RESONANCES

`TEST_RESONANCES AXIS=<axis> OUTPUT=<resonances,raw_data> [NAME=<name>] [FREQ_START=<min_freq>] [FREQ_END=<max_freq>] [HZ_PER_SEC=<hz_per_sec>] [INPUT_SHAPING=[<0:1>]]`: Runs the resonance test in all configured probe points for the requested "axis" and measures the acceleration using the accelerometer chips configured for the respective axis. "axis" can either be X or Y, or specify an arbitrary direction as `AXIS=dx,dy`, where dx and dy are floating point numbers defining a direction vector (e.g. `AXIS=X`, `AXIS=Y`, or `AXIS=1,-1` to define a diagonal direction). Note that `AXIS=dx,dy` and `AXIS=-dx,-dy` is equivalent. If `INPUT_SHAPING=0` or not set (default), disables input shaping for the resonance testing, because it is not valid to run the resonance testing with the input shaper enabled. `OUTPUT` parameter is a comma-separated list of which outputs will be written. If `raw_data` is requested, then the raw accelerometer data is written into a file or a series of files `/tmp/raw_data_<axis>_[<point>_]<name>.csv` with (`<point>_` part of the name generated only if more than 1 probe point is configured). If `resonances` is specified, the frequency response is calculated (across all probe points) and written into `/tmp/resonances_<axis>_<name>.csv` file. If unset, OUTPUT defaults to `resonances`, and NAME defaults to the current time in "YYYYMMDD_HHMMSS" format.

#### SHAPER_CALIBRATE

`SHAPER_CALIBRATE [AXIS=<軸>] [NAME=<名稱>] [FREQ_START=<最小頻率>] [FREQ_END=<最大頻率>] [HZ_PER_SEC=<hz_per_sec>] [MAX_SMOOTHING=<max_smoothing>]`：類似`TEST_RESONANCES`，該命令按配置執行共振測試並嘗試尋找給定軸最佳的輸入整形參數。如果沒有選擇`AXIS`，則測量X和Y軸。如果沒有選擇 `MAX_SMOOTHING` ，會使用 `[resonance_tester]` 的預設參數。有關該特性的使用方法，詳見共振量測量指南中的 [最大平滑](Measuring_Resonances.md#max-smoothing)章節。測試結果會被輸出到終端，而頻響和不同輸入整形器的參數將被寫入到一個或多個 CSV 檔案中。它們的名稱是`/tmp/calibration_data_<軸>_<名稱>.csv`。如果沒有指定，名稱預設為「YYYYMMDD_HHMMSS」格式的當前時間。注意，可以通過請求 `SAVE_CONFIG` 命令將推薦的輸入整形器參數直接儲存到配置檔案中。

### [respond]

The following standard G-Code commands are available when the [respond config section](Config_Reference.md#respond) is enabled:

- `M118 <message>`：回顯配置了預設字首的資訊（如果沒有配置字首，則返回`echo: `）。

還可以使用以下額外命令.

#### RESPOND

- `RESPOND MSG="<message>"`：回顯帶有配置的預設字首的訊息（沒有配置字首則預設 `echo: `為字首 ）。
- `RESPOND TYPE=echo MSG="<訊息>"`：回顯`echo:`開頭訊息。
- `RESPOND TYPE=command MSG="<訊息>"`：回顯以`/`為字首的訊息。可以配置 OctoPrint 對這些訊息進行響應（例如，`RESPOND TYPE=command MSG=action:pause`）。
- `RESPOND TYPE=error MSG="<訊息>"`：回顯以 `!!`開頭的訊息。
- `RESPOND PREFIX=<prefix> MSG="<message>"`: 迴應以`<prefix>`為字首的資訊。(`PREFIX`參數將優先於`TYPE`參數)

### [save_variables]

The following command is enabled if a [save_variables config section](Config_Reference.md#save_variables) has been enabled.

#### SAVE_VARIABLE

`SAVE_VARIABLE VARIABLE=<name> VALUE=<value>`：將變數儲存到磁碟，以便在重新啟動時使用。所有儲存的變數都會在啟動時載入到 `printer.save_variables.variables` 目錄中，並可以在 gcode 宏中使用。所提供的 VALUE 會被解析為一個 Python 字面。

### [screws_tilt_adjust]

The following commands are available when the [screws_tilt_adjust config section](Config_Reference.md#screws_tilt_adjust) is enabled (also see the [manual level guide](Manual_Level.md#adjusting-bed-leveling-screws-using-the-bed-probe)).

#### SCREWS_TILT_CALCULATE

`SCREWS_TILT_CALCULATE [DIRECTION=CW|CCW] [<probe_parameter>=<value>]`：該命令將呼叫熱床絲桿調整工具。它將命令噴嘴到不同的位置（如配置檔案中定義的）探測z高度，並計算出調整床面水平的旋鈕旋轉次數。如果指定了DIRECTION，旋鈕的轉動方向都是一樣的，順時針（CW）或逆時針（CCW）。有關可選探頭參數的詳細資訊，請參見PROBE命令。重要的是：在使用這個命令之前，你必須先做一個G28。

### [sdcard_loop]

When the [sdcard_loop config section](Config_Reference.md#sdcard_loop) is enabled, the following extended commands are available.

#### SDCARD_LOOP_BEGIN

`SDCARD_LOOP_BEGIN COUNT=<count>`：SD 列印中開始循環的部分。計數為0表示該部分應無限期地循環。

#### SDCARD_LOOP_END

`SDCARD_LOOP_END`：結束SD列印中的一個循環部分。

#### SDCARD_LOOP_DESIST

`SDCARD_LOOP_DESIST`：完成現有的循環，不再繼續迭代。

### [servo]

The following commands are available when a [servo config section](Config_Reference.md#servo) is enabled.

#### SET_SERVO

`SET_SERVO SERVO=配置名 [ANGLE=<角度> | WIDTH=<秒>]`：將舵機位置設定為給定的角度（度）或脈衝寬度（秒）。使用 `WIDTH=0` 來禁用舵機輸出。

### [skew_correction]

The following commands are available when the [skew_correction config section](Config_Reference.md#skew_correction) is enabled (also see the [Skew Correction](Skew_Correction.md) guide).

#### SET_SKEW

`SET_SKEW [XY=<ac_length,bd_length,ad_length>] [XZ=<ac,bd,ad>] [YZ=<ac,bd,ad>] [CLEAR=<0|1>]`：用從校準列印中測量的數據（以毫米為單位）配置 [skew_correction] 模組。可以輸入任何組合的平面，沒有新輸入的平面將保持它們原有的數值。如果傳入 `CLEAR=1`，則全部偏斜校正將被禁用。

#### GET_CURRENT_SKEW

`GET_CURRENT_SKEW`:以弧度和度數報告每個平面的當前印表機偏移。斜度是根據通過`SET_SKEW`程式碼提供的參數計算的。

#### CALC_MEASURED_SKEW

`CALC_MEASURED_SKEW [AC=<ac 長度>] [BD=<bd 長度>] [AD=<ad 長度>]`：計算並報告基於一個列印件測量的偏斜度（以弧度和角度為單位）。它可以用來驗證應用校正後印表機的當前偏斜度。它也可以用來確定是否有必要進行偏斜矯正。有關偏斜矯正列印模型和測量方法詳見[偏斜校正文件](Skew_Correction.md)。

#### SKEW_PROFILE

`SKEW_PROFILE [LOAD=<名稱>] [SAVE=<名稱>] [REMOVE=<名稱>]`：skew_correction 配置管理命令。 LOAD 將從與提供的名稱匹配的配置中載入偏斜狀態。 SAVE 會將目前偏斜狀態儲存到與提供的名稱匹配的配置中。 REMOVE 將從持久記憶體中刪除與提供的名稱匹配的配置。請注意，在執行 SAVE 或 REMOVE 操作后，必須執行 SAVE_CONFIG G程式碼才能儲存更改。

### [stepper_enable]

The stepper_enable module is automatically loaded.

#### SET_STEPPER_ENABLE

`SET_STEPPER_ENABLE STEPPER=<配置名> ENABLE=[0|1]` 。啟用或禁用指定的步進電機。這是一個診斷和除錯工具，必須謹慎使用。因為禁用一個軸電機不會重置歸位資訊，手動移動一個被禁用的步進可能會導致機器在安全限值外操作電機。這可能導致軸結構、熱端和列印件的損壞。

### [temperature_fan]

The following command is available when a [temperature_fan config section](Config_Reference.md#temperature_fan) is enabled.

#### SET_TEMPERATURE_FAN_TARGET

`SET_TEMPERATURE_FAN_TARGET temperature_fan=<temperature_fan_名稱> [target=<目標溫度>] [min_speed=<最小速度>] [max_speed=<最大速度>]`：設定一個溫度控制風扇的目標溫度。如果沒有提供目標溫度，它將被設為配置檔案中定義的溫度。如果沒有提供速度，則不會進行任何更改。

### [tmcXXXX]

The following commands are available when any of the [tmcXXXX config sections](Config_Reference.md#tmc-stepper-driver-configuration) are enabled.

#### DUMP_TMC

`DUMP_TMC STEPPER=<name>`。該命令將讀取TMC驅動暫存器並報告其值。

#### INIT_TMC

`INIT_TMC STEPPER=<名稱>`：此命令將初始化 TMC 暫存器。如果晶片的電源關閉然後重新打開，則需要重新啟用該驅動。

#### SET_TMC_CURRENT

`SET_TMC_CURRENT STEPPER=<名稱> CURRENT=<安培> HOLDCURRENT=<安培>`：該命令修改TMC驅動的執行和保持電流（HOLDCURRENT 在 tmc2660 驅動上不起效）。

#### SET_TMC_FIELD

`SET_TMC_FIELD STEPPER=<名稱> FIELD=<欄位> VALUE=<值>`：這將修改指定 TMC 步進驅動暫存器欄位的值。該命令僅適用於低階別的診斷和除錯，因為在執行期間改變欄位可能會導致印表機出現不符合預期的、有潛在危險的行為。常規修改應當通過印表機配置檔案進行。該命令不會對給定的值進行越界檢查。

### [toolhead]

The toolhead module is automatically loaded.

#### SET_VELOCITY_LIMIT

`SET_VELOCITY_LIMIT [VELOCITY=<值>] [ACCEL=<值>] [ACCEL_TO_DECEL=<值>] [SQUARE_CORNER_VELOCITY=<值>]`：修改印表機速度限制。

### [tuning_tower]

The tuning_tower module is automatically loaded.

#### TUNING_TOWER

`TUNING_TOWER COMMAND=<命令> PARAMETER=<名稱> START=<值> [SKIP=<值>] [FACTOR=<值> [BAND=<值>]] | [STEP_DELTA=<值> STEP_HEIGHT=<值>]`：根據Z高度調整參數的工具。該工具將定期執行一個 `PARAMETER` 不斷根據 `Z` 的公式變化的 `COMMAND`（命令）。如果使用一把尺子或者遊標卡尺測量 Z來獲得最佳值，你可以用`FACTOR`。如果列印件帶有帶狀標識或者使用離散數值（例如溫度塔），可以用`STEP_DELTA`和 `STEP_HEIGHT` 。如果 `SKIP=<值>` 被定義，則調整隻會在到達 Z 高度 `<值>` 后才開始。在此之前，參數會被設定為 `START`；在這種情況下，下面公式中`z_height`用`max(z - skip, 0)`替代。這些選項有三種不同的組合：

- `FACTOR`：數值以`factor`每毫米的速度變化。使用的公式是：`value = start + factor * z_height`。你可以將最佳的 Z 高度直接插入該公式，以確定最佳的參數值。
- `FACTOR` 和 `BAND`：該值以`factor`每毫米的平均速度變化，但在離散的環上，每`BAND`毫米的Z高度才會進行調整。使用的公式是：`value = start + factor * ((floor(z_height / band) + .5) * band)`。
- `STEP_DELTA` and `STEP_HEIGHT`: The value changes by `STEP_DELTA` every `STEP_HEIGHT` millimeters. The formula used is: `value = start + step_delta * floor(z_height / step_height)`. You can simply count bands or read tuning tower labels to determine the optimum value.

### [virtual_sdcard]

如果啟用了 [virtual_sdcard 配置分段](Config_Reference.md#virtual_sdcard)，Klipper 支援以下標準 G-Code 命令：

- 列出SD卡：`M20`
- 初始化SD卡：`M21`
- 選擇SD卡檔案：`M23 <filename>`
- 開始/暫停 SD 卡列印：`M24`
- 暫停 SD 卡列印： `M25`
- 設定 SD 塊位置：`M26 S<偏移>`
- 報告SD卡列印狀態：`M27`

此外，當啟用"virtual_sdcard"配置分段時，以下擴充套件命令可用。

#### SDCARD_PRINT_FILE

`SDCARD_PRINT_FILE FILENAME=<檔名>`：載入一個檔案並開始 SD 列印.

#### SDCARD_RESET_FILE

`SDCARD_RESET_FILE`：解除安裝檔案並清除SD狀態。

### [z_tilt]

The following commands are available when the [z_tilt config section](Config_Reference.md#z_tilt) is enabled.

#### Z_TILT_ADJUST

`Z_TILT_ADJUST [<probe_參數>=<值>]`：該命令將探測配置中指定的座標並對每個Z步進電機進行獨立的調整以抵消傾斜。有關可選的探針參數，詳見 PROBE 命令。
