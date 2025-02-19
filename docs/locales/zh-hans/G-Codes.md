# G-Codes

本文档描述了 Klipper 支持的命令。这些命令可以输入到 OctoPrint 终端中。

## G代码命令

Klipper支持以下标准的G-Code命令：

- 移动 (G0 or G1): `G1 [X<pos>] [Y<pos>] [Z<pos>] [E<pos>] [F<speed>]`
- 停留时间：`G4 P<milliseconds>`
- 返回原点：`G28 [X] [Y] [Z]`
- 关闭步进电机：`M18`或`M84`
- 等待当前移动完成： `M400`
- 使用绝对/相对挤出距离：`M82`， `M83`
- 使用绝对/相对坐标：`G90`, `G91`
- 设置坐标：`G92 [X<坐标>] [Y<坐标>] [Z<坐标>] [E<坐标>]`
- 设置速度因子覆写百分比：`M220 S<百分比>`
- 设置挤压因子覆盖百分比：`M221 S<percent>`
- 设置加速度：`M204 S<value>` 或 `M204 P<value> T<value>`
   - 注意：如果没有指定S，同时指定了P和T，那么加速度将被设置为P和T中的最小值。
- 获取挤出机温度：`M105`
- 设置挤出机温度：`M104 [T<index>] [S<temperature>]`
- 设置挤出机温度并等待：`M109 [T<index>] S<temperature>`。
   - 注意：M109总是等待温度稳定在请求的数值上。
- 设置热床温度：`M140 [S<temperature>]`
- 设置热床温度并且等待：`M190 S<temperature>`
   - 注意：M190总是等待温度稳定在请求的数值上。
- 设置风扇速度：`M106 S<value>`
- 停止风扇：`M107`
- 紧急停止：`M112`
- 获取当前位置：`M114`
- 获取固件版本：`M115`

有关上述命令的更多详细信息，请参阅 [RepRap G-Code documentation](http://reprap.org/wiki/G-code)

Klipper的目标是支持常见第三方软件（如OctoPrint、Printrun、Slic3r、Cura等）在其标准配置中产生的 G 代码命令。支持所有可能的 G-Code 命令并不是我们的目标。相反，Klipper更喜欢人类可读的["扩展G-Code命令"](#additional-commands)。

如果一个人需要一个不太常见的G-Code命令，那么可以用一个自定义的[gcode_macro config section](Config_Reference.md#gcode_macro)来实现它。例如，我们可以用这个来实现。`G12`, `G29`, `G30`, `G31`, `M42`, `M80`, `M81`, `T1` ，etc

## 其他命令

Klipper使用 "extended" 的G代码命令来进行一般的配置和状态。这些扩展命令都遵循一个类似的格式--它们以一个命令名开始，后面可能有一个或多个参数。比如说：`SET_SERVO SERVO=myservo ANGLE=5.3`。在本文件中，命令和参数以大写字母显示，但它们不分大小写。(所以，"SET_SERVO "和 "set_servo "都是运行同一个命令）

This section is organized my Klipper module name, which generally follows the section names specified in the [printer configuration file](Config_Reference.md). Note that some modules are automatically loaded.

### [adxl345]

The following commands are available when an [adxl345 config section](Config_Reference.md#adxl345) is enabled.

#### ACCELEROMETER_MEASURE

`ACCELEROMETER_MEASURE [CHIP=<config_name>] [NAME=<value>]` 。以要求的每秒采样数启动加速度计测量。如果没有指定CHIP，则默认为 "adxl345"。该命令以启动-停止模式工作：第一次执行时，它开始测量，下次执行时停止测量。测量结果被写入一个名为`/tmp/adxl345-<chip>-<name>的文件中。csv`，其中`<chip>`是加速度计芯片的名称（`my_chip_name`来自`[adxl345 my_chip_name]`），`<name>`是可选NAME参数。如果没有指定NAME，则默认为当前时间，格式为 "YYYMMDD_HHMMSS"。如果加速度计在其配置部分没有名称（只是`[adxl345]`），那么`<chip >`部分的名称就不会生成。

#### ACCELEROMETER_QUERY

`ACCELEROMETER_QUERY [CHIP=<config_name>] [RATE=<value>]`: 查询加速度计的当前值。如果没有指定芯片，则默认为 "adxl345"。如果没有指定RATE，则使用默认值。该命令对于测试与ADXL345加速度计的连接非常有用：返回的数值之一应该是自由落体加速度（+/-芯片的一些噪声）。

#### ACCELEROMETER_DEBUG_READ

`ACCELEROMETER_DEBUG_READ [CHIP=<配置名>] REG=<寄存器>`：查询ADXL345的寄存器"REG"（例如44或0x2C）。可以用于debug。

#### ACCELEROMETER_DEBUG_WRITE

`ACCELEROMETER_DEBUG_WRITE [CHIP=<配置名>] REG=<寄存器> VAL=<值>`：将原始的"值"写进寄存器"寄存器"。"值"和"寄存器"都可以是一个十进制或十六进制的整数。请谨慎使用，并参考 ADXL345 数据手册。

### [bed_mesh]

The following commands are available when the [bed_mesh config section](Config_Reference.md#bed_mesh) is enabled (also see the [bed mesh guide](Bed_Mesh.md)).

#### BED_MESH_CALIBRATE

`BED_MESH_CALIBRATE [METHOD=manual] [<probe_parameter>=<value>] [<mesh_parameter>=<value>]`: 此命令使用通过配置参数指定并生成的探测点探测打印床。在探测后，一个网格将被生成，z 轴移动将根据网格调整。有关可选探测参数，请见 PROBE命令。如果指定 METHOD=manual ，则会启动手动探测工具 - 有关此工具活跃时可用的额外命令，详见 MANUAL_PROBE 命令。

#### BED_MESH_OUTPUT

`BED_MESH_OUTPUT PGP=[<0:1>]`：该命令将当前探测到的 Z 值和当前网格的值输出到终端。如果指定 PGP=1，则将bed_mesh产生的X、Y坐标，以及它们关联的指数，输出到终端。

#### BED_MESH_MAP

`BED_MESH_MAP`：类似 BED_MESH_OUTPUT，这个命令在终端中显示网格的当前状态。它不以人类可读格式打印，而是被序列化为 json 格式。这允许 Octoprint 插件捕获数据并生成描绘打印床表面的高度图。

#### BED_MESH_CLEAR

`BED_MESH_CLEAR`：此命令清除床网并移除所有 z 调整。建议把它放在你的 end-gcode （结束G代码）中。

#### BED_MESH_PROFILE

`BED_MESH_PROFILE LOAD=<名称> SAVE=<名称> REMOVE=<名称>`：此命令提供了网床配置管理功能。LOAD 将从与所提供的名称相符的配置文件中恢复网格状态。SAVE 将会把目前的网格状态保存到与提供的名称相符的配置文件中。REMOVE（移除）将从持久性内存中删除与所提供名称相符的配置文件。请注意，在 SAVE 或 REMOVE 操作后，必须发送SAVE_CONFIG G代码，以保存变更到持久性内存。

#### BED_MESH_OFFSET

`BED_MESH_OFFSET [X=<value>] [Y=<value>]`。将X和/或Y的偏移量应用于网格查找。这对具有多个独立挤出头的打印机很有用，因为偏移量对切换工具头后产生正确的 Z 值调整是至关重要的。

### [bed_screws]

The following commands are available when the [bed_screws config section](Config_Reference.md#bed_screws) is enabled (also see the [manual level guide](Manual_Level.md#adjusting-bed-leveling-screws)).

#### BED_SCREWS_ADJUST

`BED_SCREWS_ADJUST`：该命令将调用打印床螺丝调整工具。它将命令喷嘴移动到不同的位置（在配置文件中定义），并允许对打印床螺丝进行手动调整，使打印床与喷嘴的距离保持不变。

### [bed_tilt]

The following commands are available when the [bed_tilt config section](Config_Reference.md#bed_tilt) is enabled.

#### BED_TILT_CALIBRATE

`BED_TILT_CALIBRATE [Method=manual] [<probe_parameter>=<value>] `:该命令将探测配置中指定的点，然后建议更新X和Y的倾斜调整。有关可选探测参数的详细信息，请参见PROBE命令。如果指定METHOD=manual，那么手动探测工具就会被激活 - 关于该工具激活时可用的附加命令，请参见上面的MANUAL_PROBE命令。

### [bltouch]

The following command is available when a [bltouch config section](Config_Reference.md#bltouch) is enabled (also see the [BL-Touch guide](BLTouch.md)).

#### BLTOUCH_DEBUG

`BLTOUCH_DEBUG COMMAND=<命令>`：向BLTouch发送一个指定的命令，可以用于调试。可用的命令有：`pin_down`、`touch_mode`、`pin_up`、`self_test`和`reset`。BL-TOUCH V3.0 或 V3.1 也可能支持`set_5V_output_mode`、`set_OD_output_mode`和`output_mode_store`命令。

#### BLTOUCH_STORE

`BLTOUCH_STORE MODE=<output_mode>`:这将在BLTouch V3.1的EEPROM中存储一个输出模式 可用的输出模式有`5V`, `OD`

### [configfile]

configfile模块被自动加载。

#### SAVE_CONFIG

`SAVE_CONFIG`：该命令将覆盖打印机的主配置文件，并重新启动主机软件。该命令与其他校准命令一起使用，用于存储校准测试的结果。

### [delayed_gcode]

The following command is enabled if a [delayed_gcode config section](Config_Reference.md#delayed_gcode) has been enabled (also see the [template guide](Command_Templates.md#delayed-gcodes)).

#### UPDATE_DELAYED_GCODE

`UPDATE_DELAYED_GCODE [ID=<名称>] [DURATION=<秒>]`：更新目标 [delayed_gcode] 的延迟并启动G代码执行的计时器。为0的值会取消准备执行的延迟G代码。

### [delta_calibrate]

The following commands are available when the [delta_calibrate config section](Config_Reference.md#linear-delta-kinematics) is enabled (also see the [delta calibrate guide](Delta_Calibrate.md)).

#### DELTA_CALIBRATE

`DELTA_CALIBRATE [Method=manual] [<probe_parameter>=<value>] `:这条命令将探测床身的七个点，并建议更新限位位置、塔架角度和半径。有关可选探测参数的详细信息，请参见PROBE命令。如果指定METHOD=manual，那么手动探测工具将被激活 - 关于该工具激活时可用的附加命令的详细信息，请参见上面的MANUAL_PROBE命令。

#### DELTA_ANALYZE

`DELTA_ANALYZE`:这个命令在增强的delta校准过程中使用。详情见[Delta Calibrate](Delta_Calibrate.md)。

### [display]

The following command is available when a [display config section](Config_Reference.md#gcode_macro) is enabled.

#### SET_DISPLAY_GROUP

`SET_DISPLAY_GROUP [DISPLAY=<display>] GROUP=<group>`:设置一个lcd显示器的活动显示组。这允许在配置中定义多个显示数据组，例如`[display_data <group> <elementname>]`并使用这个扩展的gcode命令在它们之间切换。如果没有指定DISPLAY，则默认为 "display"（主显示）。

### [display_status]

The display_status module is automatically loaded if a [display config section](Config_Reference.md#display) is enabled. It provides the following standard G-Code commands:

- 显示信息： `M117 <message> `
- 设置构建百分比：`M73 P<percent>`

### [dual_carriage]

The following command is available when the [dual_carriage config section](Config_Reference.md#dual_carriage) is enabled.

#### SET_DUAL_CARRIAGE

`SET_DUAL_CARRIAGE CARRIAGE=[0|1]`:该命令将设置活动的滑块。它通常是在一个多挤出机配置中从 activate_gcode和deactivate_gcode字段调用。

### [endstop_phase]

The following commands are available when an [endstop_phase config section](Config_Reference.md#endstop_phase) is enabled (also see the [endstop phase guide](Endstop_Phase.md)).

#### ENDSTOP_PHASE_CALIBRATE

`ENDSTOP_PHASE_CALIBRATE [STEPPER=<config_name>]` 。如果没有提供STEPPER参数，那么该命令将报告在过去的归位操作中对端停步进相的统计。当提供STEPPER参数时，它会安排将给定的终点站相位设置写入配置文件中（与SAVE_CONFIG命令一起使用）。

### [extruder]

The following commands are available if an [extruder config section](Config_Reference.md#extruder) is enabled:

#### ACTIVATE_EXTRUDER

`ACTIVATE_EXTRUDER EXTRUDER=<config_name>`：这个命令在具有多个挤出机的打印机中用于更改活动挤出机。

#### SET_PRESSURE_ADVANCE

`SET_PRESSURE_ADVANCE [EXTRUDER=<挤出机名称>] [ADVANCE=<pressure_advance>] [SMOOTH_TIME=<pressure_advance_smooth_time>]` ：设置压力提前的参数。如果没有指定挤出机，则默认为活动的挤出机。

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

`SET_FAN_SPEED FAN=config_name SPEED=<速度>`该命令设置风扇的速度。"速度" 必须在0.0到1.0之间。

### [firmware_retraction]

The following standard G-Code commands are available when the [firmware_retraction config section](Config_Reference.md#firmware_retraction) is enabled. These commands allow you to utilize the firmware retraction feature available in many slicers, to reduce stringing during non-extrusion moves from one part of the print to another. Appropriately configuring pressure advance reduces the length of retraction required.

- `G10`：使用当前配置的参数回抽挤出机。
- `G11`：不使用当前配置的参数回抽挤出机。

还可以使用以下额外命令：

#### SET_RETRACTION

`SET_RETRACTION [RETRACT_LENGTH=<毫米>] [RETRACT_SPEED=<毫米每秒>] [UNRETRACT_EXTRA_LENGTH=<毫米>] [UNRETRACT_SPEED=<毫米每秒>]`：调整固件回抽所使用的参数。RETRACT_LENGTH 决定回抽和回填的耗材长度。回抽的速度通过 RETRACT_SPEED 调整，通常设置得比较高。回填的速度通过 UNRETRACT_SPEED 调整，虽然经常比RETRACT_SPEED 低，但不是特别重要。在某些情况下，在回填时增加少量的额外长度的耗材可能有益，这可以通过 UNRETRACT_EXTRA_LENGTH 设置。SET_RETRACTION 通常作为切片机耗材配置的一部分来设置，因为不同的耗材需要不同的参数设置。

#### GET_RETRACTION

`GET_RETRACTION`:查询当前固件回抽所使用的参数并在终端显示。

### [filament_switch_sensor]

The following command is available when a [filament_switch_sensor](Config_Reference.md#filament_switch_sensor) or [filament_motion_sensor](Config_Reference.md#filament_motion_sensor) config section is enabled.

#### QUERY_FILAMENT_SENSOR

`QUERY_FILAMENT_SENSOR SENSOR=<传感器名>`：查询耗材传感器的当前状态。在终端中显示的数据将取决于配置中定义的传感器类型。

#### SET_FILAMENT_SENSOR

`SET_FILAMENT_SENSOR SENSOR=<sensor_name> ENABLE=[0|1]` ：设置灯丝传感器的开/关。如果 ENABLE 设置为 0，耗材传感器将被禁用，如果设置为 1是启用。

### [firmware_retraction]

如果启用了[firmware_retraction 配置分段](Config_Reference.md#firmware_retraction)，则以下标准G代码命令可用：

- 回抽：`G10`
- 回填：`G11`

### [force_move]

The force_move module is automatically loaded, however some commands require setting `enable_force_move` in the [printer config](Config_Reference#force_move).

#### STEPPER_BUZZ

`STEPPER_BUZZ STEPPER=<配置名>`：移动指定的步进电机前后运动一毫米，重复的10次。这是一个用于验证步进电机接线的工具

#### FORCE_MOVE

`FORCE_MOVE STEPPER=<config_name> DISTANCE=<value> VELOCITY=<value> [ACCEL=<value>]` 。该命令将以给定的恒定速度（mm/s）强制移动给定的步进器，移动距离（mm）。如果指定了ACCEL并且大于零，那么将使用给定的加速度（单位：mm/s^2）；否则不进行加速。不执行边界检查；不进行运动学更新；一个轴上的其他平行步进器将不会被移动。请谨慎使用，因为不正确的命令可能会导致损坏使用该命令几乎肯定会使低级运动学处于不正确的状态；随后发出G28命令以重置运动学。该命令用于低级别的诊断和调试。

#### SET_KINEMATIC_POSITION

`SET_KINEMATIC_POSITION [X=<值>] [Y=<值>] [Z=<值>]`：强制设定低级运动学代码使用的打印头位置。这是一个诊断和调试工具，常规的轴转换应该使用 SET_GCODE_OFFSET 和/或 G92指令。如果没有指定一个轴，则使用上一次命令的打印头位置。设定一个不合理的数值可能会导致内部程序问题。这个命令可能会使边界检查失效，使用后发送 G28 来重置运动学。

### [gcode]

The gcode module is automatically loaded.

#### RESTART

`RESTART`：这将导致主机软件重新加载其配置并执行内部重置。此命令不会从微控制器清除错误状态（请参阅 FIRMWARE_RESTART），也不会加载新软件（请参阅 [常见问题](FAQ.md#how-do-i-upgrade-to-the-latest-software)） .

#### FIRMWARE_RESTART

`FIRMWARE_RESTART`：这类似于重启命令，但它也清除了微控制器的任何错误状态。

#### STATUS

`STATUS`：报告Klipper主机程序的状态。

#### HELP

`HELP`：报告可用的扩展G-Code命令列表。

### [gcode_arcs]

如果启用了[gcode_arcs 配置分段](Config_Reference.md#gcode_arcs)，下列标准G代码命令可用：

- 受控弧线移动（G2或G3）。`G2 [X<pos>] [Y<pos>] [Z<pos>] [E<pos>] [F<speed>] I<value> J<value>` 。

### [gcode_macro]

The following command is available when a [gcode_macro config section](Config_Reference.md#gcode_macro) is enabled (also see the [command templates guide](Command_Templates.md)).

#### SET_GCODE_VARIABLE

`SET_GCODE_VARIABLE MACRO=<macro_name> VARIABLE=<name> VALUE=<value>`：这条命令允许人们在运行时改变 gcode_macro 变量的值。所提供的 VALUE 会被解析为一个 Python 字面。

### [gcode_move]

The gcode_move module is automatically loaded.

#### GET_POSITION

`GET_POSITION`：返回工具当前位置信息

#### SET_GCODE_OFFSET

`SET_GCODE_OFFSET [X=<pos>|X_ADJUST=<adjust>] [Y=<pos>|Y_ADJUST=<adjust>] [Z=<pos>|Z_ADJUST=<adjust>] [MOVE=1 [MOVE_SPEED=<speed>]]` 。设置一个位置偏移，以应用于未来的G代码命令。这通常用于实际改变Z床的偏移量或在切换挤出机时设置喷嘴的XY偏移量。例如，如果发送 "SET_GCODE_OFFSET Z=0.2"，那么未来的G代码移动将在其Z高度上增加0.2mm。如果使用X_ADJUST风格的参数，那么调整将被添加到任何现有的偏移上（例如，"SET_GCODE_OFFSET Z=-0.2"，然后是 "SET_GCODE_OFFSET Z_ADJUST=0.3"，将导致总的Z偏移为0.1）。如果指定了 "MOVE=1"，那么将发出一个工具头移动来应用给定的偏移量（否则偏移量将在指定给定轴的下一次绝对G-Code移动中生效）。如果指定了 "MOVE_SPEED"，那么刀头移动将以给定的速度（mm/s）执行；否则，打印头移动将使用最后指定的G-Code速度。

#### SAVE_GCODE_STATE

`SAVE_GCODE_STATE [NAME=<state_name>]`：保存当前的g-code坐标解析状态。保存和恢复g-code状态在脚本和宏中很有用。该命令保存当前g-code绝对坐标模式（G90/G91）绝对挤出模式（M82/M83）原点（G92）偏移量（SET_GCODE_OFFSET）速度覆盖（M220）挤出机覆盖（M221）移动速度。当前XYZ位置和相对挤出机 "E "位置。如果提供NAME，它允许人们将保存的状态命名为给定的字符串。如果没有提供NAME，则默认为 "default"

#### RESTORE_GCODE_STATE

`RESTORE_GCODE_STATE [NAME=<state_name>] [MOVE=1 [MOVE_SPEED=<speed>]]`：恢复之前通过 SAVE_GCODE_STATE 保存的状态。如果指定“MOVE=1”，则将发出刀头移动以返回到先前的 XYZ 位置。如果指定了“MOVE_SPEED”，则刀头移动将以给定的速度（以mm/s为单位）执行；否则工具头移动将使用恢复的G-Code速度。

### [hall_filament_width_sensor]

The following commands are available when the [tsl1401cl filament width sensor config section](Config_Reference.md#tsl1401cl_filament_width_sensor) or [hall filament width sensor config section](Config_Reference.md#hall_filament_width_sensor) is enabled (also see [TSLl401CL Filament Width Sensor](TSL1401CL_Filament_Width_Sensor.md) and [Hall Filament Width Sensor](Hall_Filament_Width_Sensor.md)):

#### QUERY_FILAMENT_WIDTH

`QUERY_FILAMENT_WIDTH`：返回当前测量的耗材直径。

#### RESET_FILAMENT_WIDTH_SENSOR

`RESET_FILAMENT_WIDTH_SENSOR`：清除全部传感器读数。在更换耗材后有用。

#### DISABLE_FILAMENT_WIDTH_SENSOR

`DISABLE_FILAMENT_WIDTH_SENSOR`：关闭耗材直径传感器并停止使用它进行流量控制。

#### ENABLE_FILAMENT_WIDTH_SENSOR

`ENABLE_FILAMENT_WIDTH_SENSOR`：启用耗材直径传感器并使用它进行流量控制。

#### QUERY_RAW_FILAMENT_WIDTH

`QUERY_RAW_FILAMENT_WIDTH`：返回当前 ADC 通道读数和校准点的 RAW 传感器值。

#### ENABLE_FILAMENT_WIDTH_LOG

`ENABLE_FILAMENT_WIDTH_LOG`：开启直径记录。

#### DISABLE_FILAMENT_WIDTH_LOG

`DISABLE_FILAMENT_WIDTH_LOG`：停止直径记录。

### [heaters]

The heaters module is automatically loaded if a heater is defined in the config file.

#### TURN_OFF_HEATERS

`TURN_OFF_HEATERS`：关闭全部加热器。

#### TEMPERATURE_WAIT

`TEMPERATURE_WAIT SENSOR=<配置名> [MINIMUM=<目标>] [MAXIMUM=<目标>]`：等待指定温度传感器读数高于 MINIMUM 和或低于 MAXIMUM。

#### SET_HEATER_TEMPERATURE

`SET_HEATER_TEMPERATURE HEATER=<加热器名称> [TARGET=<目标温度>]`：设置一个加热器的目标温度。如果没有提供目标温度，则目标温度为 0。

### [idle_timeout]

The idle_timeout module is automatically loaded.

#### SET_IDLE_TIMEOUT

`SET_IDLE_TIMEOUT [TIMEOUT=<超时>]`：允许用户设置空闲超时（以秒为单位）。

### [input_shaper]

The following command is enabled if an [input_shaper config section](Config_Reference.md#input_shaper) has been enabled (also see the [resonance compensation guide](Resonance_Compensation.md)).

#### SET_INPUT_SHAPER

`SET_INPUT_SHAPER [SHAPER_FREQ_X=<shaper_freq_x>] [SHAPER_FREQ_Y=<shaper_freq_y>] [DAMPING_RATIO_X=<damping_ratio_x>] [DAMPING_RATIO_Y=<damping_ratio_y>] [SHAPER_TYPE=<shaper>] [SHAPER_TYPE_X=<shaper_type_x>] [SHAPER_TYPE_Y=<shaper_type_y>]`：修改输入整形参数。注意 SHAPER_TYPE 参数会同时覆写 X 和 Y 轴的整形器类型，即使它们在 [input_shaper] 配置分段中有不同的整形器类型。SHAPER_TYPE 不能和 SHAPER_TYPE_X 和 SHAPER_TYPE_Y 参数同时使用。这些参数的细节请见[配置参考](Config_Reference.md#input_shaper)。

### [manual_probe]

The manual_probe module is automatically loaded.

#### MANUAL_PROBE

`MANUAL_PROBE [SPEED=<speed>]`：运行一个辅助脚本，对测量给定位置的喷嘴高度有用。如果指定了速度，它将设置TESTZ命令的速度（默认为5mm/s）。在手动探测过程中，可使用以下附加命令：

- `ACCEPT`：该命令接受当前的Z位置，并结束手动探测工具。
- `ABORT`：该命令终止手动探测工具。
- `TESTZ Z=<值>`：这个命令可以将喷嘴上升或下降给定值，以毫米为单位。例如，`TESTZ Z=-.1` 会将喷嘴下降 0.1 毫米，而 `TESTZ Z=.1` 会将喷嘴上升 0.1 毫米，参数可以带有`+`, `-`, `++`, or `--`来根据上次尝试相对的移动喷嘴。

#### Z_ENDSTOP_CALIBRATE

`Z_ENDSTOP_CALIBRATE [SPEED=<速度>]`：运行一个校准 Z position_endstop 参数的辅助脚本。有关更多参数和额外命令的信息，请查看 MANUAL_PROBE 命令。

#### Z_OFFSET_APPLY_ENDSTOP

`Z_OFFSET_APPLY_ENDSTOP`：将当前的Z 的 G 代码偏移量（就是 babystepping）从 stepper_z 的 endstop_position 中减去。该命令将持久化一个常用babystepping 微调值。需要执行 `SAVE_CONFIG`才能生效。

### [manual_stepper]

当[manual_stepper 配置分段](Config_Reference.md#manual_stepper)被启用时，以下命令可用。

#### MANUAL_STEPPER

`MANUAL_STEPPER STEPPER=config_name [ENABLE=[0|1]] [SET_POSITION=<pos>] [SPEED=<speed>] [ACCEL=<accel>] [MOVE=<pos> [STOP_ON_ENDSTOP=[1|2|-1|-2]] [SYNC=0]]`：该命令将改变步进器的状态。使用ENABLE参数来启用/禁用步进。使用SET_POSITION参数，迫使步进认为它处于给定的位置。使用MOVE参数，要求移动到给定位置。如果指定了SPEED或者ACCEL，那么将使用给定的值而不是配置文件中指定的默认值。如果指定ACCEL为0，那么将不执行加速。如果STOP_ON_ENDSTOP=1被指定，那么如果止动器报告被触发，动作将提前结束（使用STOP_ON_ENDSTOP=2来完成动作，即使止动器没有被触发也不会出错，使用-1或-2来在止动器报告没有被触发时停止）。通常情况下，未来的G-Code命令将被安排在步进运动完成后运行，但是如果手动步进运动使用SYNC=0，那么未来的G-Code运动命令可能与步进运动平行运行。

### [neopixel]

The following command is available when a [neopixel config section](Config_Reference.md#neopixel) or [dotstar config section](Config_Reference.md#dotstar) is enabled.

#### SET_LED

`SET_LED LED=<配置中的名称> RED=<值> GREEN=<值> BLUE=<值> WHITE=<值> [INDEX=<索引>] [TRANSMIT=0] [SYNC=1]`：这条指令设置LED的输出。每个颜色的 `<值>` 必须在0.0和1.0之间。WHITE（白色）参数仅在 RGBW LED上有效果。如果多个 LED 芯片是通过菊链连接的，可以用INDEX参数来指定改变指定芯片的颜色（1是第一颗芯片，2是第二颗芯片…）。如果不提供 INDEX 则全部菊链中的 LED 将被设置为给定的颜色。如果指定了 TRANSMIT=0 ，色彩变化只会在下一条不是 TRANSMIT=0 的 SET_LED命令发送后发生。这个特性和 INDEX 参数一起使用可以在菊链中批量改变LED的颜色。默认情况下，SET_LED命令将与其他正在进行的G代码命令同步其变化。 因为修改LED状态会重置空闲超时，这可能在不打印时造成不期望的行为。 如果精确的时序不重要，可选的 SYNC=0 参数可以立刻应用LED变化而不重置LED超时。

### [output_pin]

The following command is available when an [output_pin config section](Config_Reference.md#output_pin) is enabled.

#### SET_PIN

`SET_PIN PIN=config_name VALUE=<值> CYCLE_TIME=<周期时间>`: 注意 - 硬件PWM目前不支持CYCLE_TIME参数，将使用配置中定义的周期时间。

### [palette2]

The following commands are available when the [palette2 config section](Config_Reference.md#palette2) is enabled.

Palette打印通过在GCode文件中嵌入特殊的OCodes（Omega Codes）来工作。

- `O1`...`O32`：这些代码从G-Code流中读出并且传递给Palette 2设备进行处理。

还可以使用以下额外命令：

#### PALETTE_CONNECT

`PALETTE_CONNECT`：该命令初始化与Palette 2的连接。

#### PALETTE_DISCONNECT

`PALETTE_DISCONNECT`：该命令断开与Palette 2的连接。

#### PALETTE_CLEAR

`PALETTE_CLEAR`:该命令指示 Palette 2 清除所有耗材的输入或者输出。

#### PALETTE_CUT

`PALETTE_CUT`:该命令指引Palette 2切割耗材并且装载分段的耗材。

#### PALETTE_SMART_LOAD

`PALETTE_SMART_LOAD`：该命令在Palette 2上启动智能加载序列。通过在设备上为打印机校准的距离挤压，自动加载耗材，并在加载完成后指示Palette 2。该命令与耗材加载完成后直接在Palette 2屏幕上按**Smart Load**相同。

### [pid_calibrate]

The pid_calibrate module is automatically loaded if a heater is defined in the config file.

#### PID_CALIBRATE

`PID_CALIBRATE HEATER=<config_name> TARGET=<temperature> [WRITE_FILE=1]`：执行一个PID校准测试。指定的加热器将被启用，直到达到指定的目标温度，然后加热器将被关闭和开启几个周期。如果WRITE_FILE参数被启用，那么将创建文件/tmp/heattest.txt，其中包含测试期间所有温度样本的日志。

### [pause_resume]

当[pause_resume 配置分段](Config_Reference.md#pause_resume)被启用时，以下命令可用：

#### PAUSE

`PAUSE`：暂停当前的打印。当前的位置被报错以便在恢复时恢复。

#### RESUME

`RESUME [VELOCITY=<value>]`：从暂停中恢复打印，首先恢复以前保持的位置。VELOCITY参数决定了工具返回到原始捕捉位置的速度。

#### CLEAR_PAUSE

`CLEAR_PAUSE`:清除当前的暂停状态而不恢复打印。如果一个人决定在暂停后取消打印，这很有用。建议将其添加到你的启动代码中，以确保每次打印时的暂停状态是新的。

#### CANCEL_PRINT

`CANCEL_PRINT`：取消当前的打印。

### [probe]

The following commands are available when a [probe config section](Config_Reference.md#probe) or [bltouch config section](Config_Reference.md#bltouch) is enabled (also see the [probe calibrate guide](Probe_Calibrate.md)).

#### PROBE

`PROBE [PROBE_SPEED=<mm/s>] [LIFT_SPEED=<mm/s>] [SAMPLES=<count>] [SAMPLE_RETRACT_DIST=<mm>] [SAMPLES_TOLERANCE=<mm>] [SAMPLES_TOLERANCE_RETRIES=<count>] [SAMPLES_RESULT=median|average]`：向下移动喷嘴直到探针触发。如果提供了任何可选参数，它们将覆盖 [probe config section](Config_Reference.md#probe) 中的等效设置。

#### QUERY_PROBE

`QUERY_PROBE`:报告探针的当前状态（"triggered"或 "open"）。

#### PROBE_ACCURACY

`PROBE_ACCURACY [PROBE_SPEED=<mm/s>] [SAMPLES=<count>] [SAMPLE_RETRACT_DIST=<mm>]`：计算多个探针样本的最大、最小、平均、中位数和标准偏差。默认情况下采样10次。否则可选参数默认为探针配置部分的同等设置。

#### PROBE_CALIBRATE

`PROBE_CALIBRATE [SPEED=<speed>] [<probe_parameter>=<value>] `:运行一个对校准测头的z_offset有用的辅助脚本。有关可选测头参数的详细信息，请参见PROBE命令。参见MANUAL_PROBE命令，了解SPEED参数和工具激活时可用的附加命令的详细信息。请注意，PROBE_CALIBRATE命令使用速度变量在XY方向以及Z方向上移动。

#### Z_OFFSET_APPLY_PROBE

`Z_OFFSET_APPLY_PROBE`：将当前的Z 的 G 代码偏移量（就是 babystepping）从 probe 的 z_offset 中减去。该命令将持久化一个常用babystepping 微调值。需要执行 `SAVE_CONFIG`才能生效。

### [query_adc]

The query_endstops module is automatically loaded.

#### QUERY_ADC

`QUERY_ADC [NAME=<config_name>] [PULLUP=<value>]` ：返回为配置的模拟引脚收到的最后一个模拟值。如果NAME没有被提供，将报告可用的adc名称列表。如果提供了PULLUP（以欧姆为单位的数值），将会返回原始模拟值和给定的等效电阻。

### [query_endstops]

The query_endstops module is automatically loaded. The following standard G-Code commands are currently available, but using them is not recommended:

- 获取限位状态：`M119` (使用QUERY_ENDSTOPS代替)

#### QUERY_ENDSTOPS

`QUERY_ENDSTOPS`：检测限位并返回限位是否被 "triggered"或处于"open"。该命令通常用于验证一个限位是否正常工作。

### [resonance_tester]

The following commands are available when a [resonance_tester config section](Config_Reference.md#resonance_tester) is enabled (also see the [measuring resonances guide](Measuring_Resonances.md)).

#### MEASURE_AXES_NOISE

`MEASURE_AXES_NOISE`：测量并输出所有启用的加速度计芯片的所有轴的噪声。

#### TEST_RESONANCES

`TEST_RESONANCES AXIS=<axis> OUTPUT=<resonances,raw_data> [NAME=<name>] [FREQ_START=<min_freq>] [FREQ_END=<max_freq>] [HZ_PER_SEC=<hz_per_sec>] [INPUT_SHAPING=[<0:1>]]`: Runs the resonance test in all configured probe points for the requested "axis" and measures the acceleration using the accelerometer chips configured for the respective axis. "axis" can either be X or Y, or specify an arbitrary direction as `AXIS=dx,dy`, where dx and dy are floating point numbers defining a direction vector (e.g. `AXIS=X`, `AXIS=Y`, or `AXIS=1,-1` to define a diagonal direction). Note that `AXIS=dx,dy` and `AXIS=-dx,-dy` is equivalent. If `INPUT_SHAPING=0` or not set (default), disables input shaping for the resonance testing, because it is not valid to run the resonance testing with the input shaper enabled. `OUTPUT` parameter is a comma-separated list of which outputs will be written. If `raw_data` is requested, then the raw accelerometer data is written into a file or a series of files `/tmp/raw_data_<axis>_[<point>_]<name>.csv` with (`<point>_` part of the name generated only if more than 1 probe point is configured). If `resonances` is specified, the frequency response is calculated (across all probe points) and written into `/tmp/resonances_<axis>_<name>.csv` file. If unset, OUTPUT defaults to `resonances`, and NAME defaults to the current time in "YYYYMMDD_HHMMSS" format.

#### SHAPER_CALIBRATE

`SHAPER_CALIBRATE [AXIS=<轴>] [NAME=<名称>] [FREQ_START=<最小频率>] [FREQ_END=<最大频率>] [HZ_PER_SEC=<hz_per_sec>] [MAX_SMOOTHING=<max_smoothing>]`：类似`TEST_RESONANCES`，该命令按配置运行共振测试并尝试寻找给定轴最佳的输入整形参数。如果没有选择`AXIS`，则测量X和Y轴。如果没有选择 `MAX_SMOOTHING` ，会使用 `[resonance_tester]` 的默认参数。有关该特性的使用方法，详见共振量测量指南中的 [最大平滑](Measuring_Resonances.md#max-smoothing)章节。测试结果会被输出到终端，而频响和不同输入整形器的参数将被写入到一个或多个 CSV 文件中。它们的名称是`/tmp/calibration_data_<轴>_<名称>.csv`。如果没有指定，名称默认为“YYYYMMDD_HHMMSS”格式的当前时间。注意，可以通过请求 `SAVE_CONFIG` 命令将推荐的输入整形器参数直接保存到配置文件中。

### [respond]

The following standard G-Code commands are available when the [respond config section](Config_Reference.md#respond) is enabled:

- `M118 <message>`：回显配置了默认前缀的信息（如果没有配置前缀，则返回`echo: `）。

还可以使用以下额外命令：

#### RESPOND

- `RESPOND MSG="<message>"`：回显带有配置的默认前缀的消息（没有配置前缀则默认 `echo: `为前缀 ）。
- `RESPOND TYPE=echo MSG="<消息>"`：回显`echo:`开头消息。
- `RESPOND TYPE=command MSG="<消息>"`：回显以`/`为前缀的消息。可以配置 OctoPrint 对这些消息进行响应（例如，`RESPOND TYPE=command MSG=action:pause`）。
- `RESPOND TYPE=error MSG="<消息>"`：回显以 `!!`开头的消息。
- `RESPOND PREFIX=<prefix> MSG="<message>"`: 回应以`<prefix>`为前缀的信息。(`PREFIX`参数将优先于`TYPE`参数)

### [save_variables]

The following command is enabled if a [save_variables config section](Config_Reference.md#save_variables) has been enabled.

#### SAVE_VARIABLE

`SAVE_VARIABLE VARIABLE=<name> VALUE=<value>`：将变量保存到磁盘，以便在重新启动时使用。所有存储的变量都会在启动时加载到 `printer.save_variables.variables` 目录中，并可以在 gcode 宏中使用。所提供的 VALUE 会被解析为一个 Python 字面。

### [screws_tilt_adjust]

The following commands are available when the [screws_tilt_adjust config section](Config_Reference.md#screws_tilt_adjust) is enabled (also see the [manual level guide](Manual_Level.md#adjusting-bed-leveling-screws-using-the-bed-probe)).

#### SCREWS_TILT_CALCULATE

`SCREWS_TILT_CALCULATE [DIRECTION=CW|CCW] [<probe_parameter>=<value>]`：该命令将调用热床丝杆调整工具。它将命令喷嘴到不同的位置（如配置文件中定义的）探测z高度，并计算出调整床面水平的旋钮旋转次数。如果指定了DIRECTION，旋钮的转动方向都是一样的，顺时针（CW）或逆时针（CCW）。有关可选探头参数的详细信息，请参见PROBE命令。重要的是：在使用这个命令之前，你必须先做一个G28。

### [sdcard_loop]

When the [sdcard_loop config section](Config_Reference.md#sdcard_loop) is enabled, the following extended commands are available.

#### SDCARD_LOOP_BEGIN

`SDCARD_LOOP_BEGIN COUNT=<count>`：SD 打印中开始循环的部分。计数为0表示该部分应无限期地循环。

#### SDCARD_LOOP_END

`SDCARD_LOOP_END`：结束SD打印中的一个循环部分。

#### SDCARD_LOOP_DESIST

`SDCARD_LOOP_DESIST`：完成现有的循环，不再继续迭代。

### [servo]

The following commands are available when a [servo config section](Config_Reference.md#servo) is enabled.

#### SET_SERVO

`SET_SERVO SERVO=配置名 [ANGLE=<角度> | WIDTH=<秒>]`：将舵机位置设置为给定的角度（度）或脉冲宽度（秒）。使用 `WIDTH=0` 来禁用舵机输出。

### [skew_correction]

The following commands are available when the [skew_correction config section](Config_Reference.md#skew_correction) is enabled (also see the [Skew Correction](Skew_Correction.md) guide).

#### SET_SKEW

`SET_SKEW [XY=<ac_length,bd_length,ad_length>] [XZ=<ac,bd,ad>] [YZ=<ac,bd,ad>] [CLEAR=<0|1>]`：用从校准打印中测量的数据（以毫米为单位）配置 [skew_correction] 模块。可以输入任何组合的平面，没有新输入的平面将保持它们原有的数值。如果传入 `CLEAR=1`，则全部偏斜校正将被禁用。

#### GET_CURRENT_SKEW

`GET_CURRENT_SKEW`:以弧度和度数报告每个平面的当前打印机偏移。斜度是根据通过`SET_SKEW`代码提供的参数计算的。

#### CALC_MEASURED_SKEW

`CALC_MEASURED_SKEW [AC=<ac 长度>] [BD=<bd 长度>] [AD=<ad 长度>]`：计算并报告基于一个打印件测量的偏斜度（以弧度和角度为单位）。它可以用来验证应用校正后打印机的当前偏斜度。它也可以用来确定是否有必要进行偏斜矫正。有关偏斜矫正打印模型和测量方法详见[偏斜校正文档](Skew_Correction.md)。

#### SKEW_PROFILE

`SKEW_PROFILE [LOAD=<名称>] [SAVE=<名称>] [REMOVE=<名称>]`：skew_correction 配置管理命令。 LOAD 将从与提供的名称匹配的配置中载入偏斜状态。 SAVE 会将当前偏斜状态保存到与提供的名称匹配的配置中。 REMOVE 将从持久内存中删除与提供的名称匹配的配置。请注意，在运行 SAVE 或 REMOVE 操作后，必须运行 SAVE_CONFIG G代码才能保存更改。

### [stepper_enable]

The stepper_enable module is automatically loaded.

#### SET_STEPPER_ENABLE

`SET_STEPPER_ENABLE STEPPER=<配置名> ENABLE=[0|1]` 。启用或禁用指定的步进电机。这是一个诊断和调试工具，必须谨慎使用。因为禁用一个轴电机不会重置归位信息，手动移动一个被禁用的步进可能会导致机器在安全限值外操作电机。这可能导致轴结构、热端和打印件的损坏。

### [temperature_fan]

The following command is available when a [temperature_fan config section](Config_Reference.md#temperature_fan) is enabled.

#### SET_TEMPERATURE_FAN_TARGET

`SET_TEMPERATURE_FAN_TARGET temperature_fan=<temperature_fan_名称> [target=<目标温度>] [min_speed=<最小速度>] [max_speed=<最大速度>]`：设置一个温度控制风扇的目标温度。如果没有提供目标温度，它将被设为配置文件中定义的温度。如果没有提供速度，则不会进行任何更改。

### [tmcXXXX]

The following commands are available when any of the [tmcXXXX config sections](Config_Reference.md#tmc-stepper-driver-configuration) are enabled.

#### DUMP_TMC

`DUMP_TMC STEPPER=<name>`。该命令将读取TMC驱动寄存器并报告其值。

#### INIT_TMC

`INIT_TMC STEPPER=<名称>`：此命令将初始化 TMC 寄存器。如果芯片的电源关闭然后重新打开，则需要重新启用该驱动。

#### SET_TMC_CURRENT

`SET_TMC_CURRENT STEPPER=<名称> CURRENT=<安培> HOLDCURRENT=<安培>`：该命令修改TMC驱动的运行和保持电流（HOLDCURRENT 在 tmc2660 驱动上不起效）。

#### SET_TMC_FIELD

`SET_TMC_FIELD STEPPER=<名称> FIELD=<字段> VALUE=<值>`：这将修改指定 TMC 步进驱动寄存器字段的值。该命令仅适用于低级别的诊断和调试，因为在运行期间改变字段可能会导致打印机出现不符合预期的、有潜在危险的行为。常规修改应当通过打印机配置文件进行。该命令不会对给定的值进行越界检查。

### [toolhead]

The toolhead module is automatically loaded.

#### SET_VELOCITY_LIMIT

`SET_VELOCITY_LIMIT [VELOCITY=<值>] [ACCEL=<值>] [ACCEL_TO_DECEL=<值>] [SQUARE_CORNER_VELOCITY=<值>]`：修改打印机速度限制。

### [tuning_tower]

The tuning_tower module is automatically loaded.

#### TUNING_TOWER

`TUNING_TOWER COMMAND=<命令> PARAMETER=<名称> START=<值> [SKIP=<值>] [FACTOR=<值> [BAND=<值>]] | [STEP_DELTA=<值> STEP_HEIGHT=<值>]`：根据Z高度调整参数的工具。该工具将定期运行一个 `PARAMETER` 不断根据 `Z` 的公式变化的 `COMMAND`（命令）。如果使用一把尺子或者游标卡尺测量 Z来获得最佳值，你可以用`FACTOR`。如果打印件带有带状标识或者使用离散数值（例如温度塔），可以用`STEP_DELTA`和 `STEP_HEIGHT` 。如果 `SKIP=<值>` 被定义，则调整只会在到达 Z 高度 `<值>` 后才开始。在此之前，参数会被设定为 `START`；在这种情况下，下面公式中`z_height`用`max(z - skip, 0)`替代。这些选项有三种不同的组合：

- `FACTOR`：数值以`factor`每毫米的速度变化。使用的公式是：`value = start + factor * z_height`。你可以将最佳的 Z 高度直接插入该公式，以确定最佳的参数值。
- `FACTOR` 和 `BAND`：该值以`factor`每毫米的平均速度变化，但在离散的环上，每`BAND`毫米的Z高度才会进行调整。使用的公式是：`value = start + factor * ((floor(z_height / band) + .5) * band)`。
- `STEP_DELTA` and `STEP_HEIGHT`: The value changes by `STEP_DELTA` every `STEP_HEIGHT` millimeters. The formula used is: `value = start + step_delta * floor(z_height / step_height)`. You can simply count bands or read tuning tower labels to determine the optimum value.

### [virtual_sdcard]

如果启用了 [virtual_sdcard 配置分段](Config_Reference.md#virtual_sdcard)，Klipper 支持以下标准 G-Code 命令：

- 列出SD卡：`M20` 。
- 初始化SD卡：`M21`
- 选择SD卡文件：`M23 <filename>`
- 开始/暂停 SD 卡打印：`M24`
- 暂停 SD 卡打印： `M25`
- 设置 SD 块位置：`M26 S<偏移>`。
- 报告SD卡打印状态：`M27`

此外，当启用"virtual_sdcard"配置分段时，以下扩展命令可用。

#### SDCARD_PRINT_FILE

`SDCARD_PRINT_FILE FILENAME=<文件名>`：载入一个文件并开始 SD 打印

#### SDCARD_RESET_FILE

`SDCARD_RESET_FILE`：卸载文件并清除SD状态。

### [z_tilt]

The following commands are available when the [z_tilt config section](Config_Reference.md#z_tilt) is enabled.

#### Z_TILT_ADJUST

`Z_TILT_ADJUST [<probe_参数>=<值>]`：该命令将探测配置中指定的坐标并对每个Z步进电机进行独立的调整以抵消倾斜。有关可选的探针参数，详见 PROBE 命令。
