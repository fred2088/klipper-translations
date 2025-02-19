# 구성 확인

이 문서는 Klipper printer.cfg 파일에서 핀 설정을 확인하는 데 도움이 되는 단계 목록을 제공합니다. [installation document](Installation.md)의 단계를 수행한 후 이 단계를 실행하는 것이 좋습니다.

이 가이드를 진행하는 동안 Klipper 구성 파일을 변경해야 할 수도 있습니다. 구성 파일을 변경할 때마다 RESTART 명령을 실행하여 변경 사항이 적용되도록 하십시오(Octoprint 터미널 탭에 "restart"를 입력한 다음 "보내기"를 클릭하십시오). 구성 파일이 성공적으로 로드되었는지 확인하기 위해 모든 RESTART 후에 STATUS 명령을 실행하는 것도 좋은 방법입니다.

## 온도 확인

온도가 제대로 보고되고 있는지 확인하세요. Octoprint 온도 탭으로 이동합니다.

![octoprint-temperature](img/octoprint-temperature.png)

노즐과 베드(해당되는 경우)의 온도가 있고 증가하지 않는지 확인합니다. 증가하는 경우 프린터에서 전원을 제거하십시오. 온도가 정확하지 않으면 노즐 및/또는 베드에 대한 "sensor_type" 및 "sensor_pin" 설정을 검토하십시오.

## M112 확인

Octoprint 터미널 탭으로 이동하여 터미널 상자에서 M112 명령을 실행합니다. 이 명령은 Klipper가 "종료" 상태가 되도록 요청합니다. Octoprint와 Klipper의 연결이 끊어질 것입니다. 연결 영역으로 이동한 다음 "연결"을 클릭하면 Octoprint가 다시 연결됩니다. 그런 다음 Octoprint 온도 탭으로 이동하여 온도가 계속 업데이트되고 온도가 증가하지 않는지 확인합니다. 온도가 올라가면 프린터의 전원을 차단하십시오.

M112 명령은 Klipper를 "종료" 상태로 만듭니다. 이 상태를 초기화 하려면 Octoprint 터미널 탭에서 FIRMWARE_RESTART 명령을 실행하십시오.

## 히터 확인

Octoprint 온도 탭으로 이동하여 "Tool"의 온도에 50을 입력합니다. 그래프의 압출기 온도가 증가하기 시작해야 합니다(약 30초 이내). 그런 다음 "Tool"의 온도에 드롭다운 메뉴에서 "끄기"를 선택합니다. 몇 분 후에 온도가 초기 실내 온도 값으로 돌아가기 시작해야 합니다. 온도가 증가하지 않으면 구성에서 "heater_pin" 설정을 확인하십시오.

프린터에 히팅 베드가 있는 경우 베드를 사용하여 위의 테스트를 다시 수행하십시오.

## 스테퍼 모터 enable pin 확인

모든 프린터 축이 수동으로 자유롭게 움직일 수 있는지 확인합니다(스테퍼 모터가 비활성화됨). 그렇지 않은 경우 M84 명령을 실행하여 모터를 비활성화하십시오. 여전히 자유롭게 이동할 수 없는 축이 있으면 해당 축에 대한 스테퍼 "enable_pin" 구성을 확인하십시오. 대부분의 상용 스테퍼 모터 드라이버에서 모터 enable pin은 "active low"이므로 enable pin 앞에 "!"가 있어야 합니다. (예: "enable_pin: !ar38").

## endstops 확인

모든 프린터 축을 수동으로 이동하여 어느 것도 endstop과 접촉하지 않도록 합니다. Octoprint 터미널 탭을 통해 QUERY_ENDSTOPS 명령을 보냅니다. 구성된 모든 endstop의 현재 상태를 응답해야 하며 모두 "open" 상태이어야 합니다. 각 endstop에 대해 endstop을 수동으로 누른 상태에서 QUERY_ENDSTOPS 명령을 다시 실행하십시오. QUERY_ENDSTOPS 명령은 endstop을 "TRIGGERED"로 보고해야 합니다.

endstop이 거꾸로 표시되면(눌렀을 때 "open"으로 나오고 그 반대의 경우도 마찬가지임) "!"를 핀 정의에 추가하거나(예: "endstop_pin: ^!ar3") "!" 이미 있는 경우, 제거 합니다.

endstop이 전혀 변경되지 않으면 일반적으로 endstop이 다른 핀에 연결되었음을 나타냅니다. 핀의 풀업 설정을 변경해야 할 수도 있습니다. (endstop_pin 이름 시작 부분의 '^' - 대부분의 프린터는 풀업 저항을 사용하며 '^'이 있어야 함).

## 스테퍼 모터 확인

STEPPER_BUZZ 명령을 사용하여 각 스테퍼 모터의 연결을 확인합니다. 주어진 축을 중간 지점에 수동으로 배치하여 시작한 다음 `STEPPER_BUZZ STEPPER=stepper_x`를 실행합니다. STEPPER_BUZZ 명령은 주어진 스테퍼 모터가 양의 방향으로 1밀리미터 이동하도록 한 다음 시작 위치로 돌아갑니다. (만약 endstop이 position_endstop=0에 정의되어 있다면, 각 움직임이 시작될 때 스테퍼는 endstop에서 멀어질 것입니다.) 이 진동을 10번 수행할 것입니다.

스테퍼 모터가 전혀 움직이지 않으면 "enable_pin" 및 "step_pin" 설정을 확인하십시오. 스테퍼 모터가 움직이지만 원래 위치로 돌아가지 않으면 "dir_pin" 설정을 확인하십시오. 스테퍼 모터가 잘못된 방향으로 진동하는 경우 일반적으로 축의 "dir_pin"을 반전해야 함을 나타냅니다. 이것은 프린터 구성 파일의 "dir_pin"에'!'를 추가하면 됩니다. (또는 이미 있는 경우 제거). 모터가 1밀리미터보다 훨씬 많거나 훨씬 적게 움직이는 경우 "rotation_distance" 설정을 확인하십시오.

구성 파일에 정의된 각 스테퍼 모터에 대해 위의 테스트를 실행합니다. (STEPPER_BUZZ 명령의 STEPPER 매개변수를 테스트할 구성 섹션의 이름으로 설정합니다.) 압출기에 필라멘트가 없으면 STEPPER_BUZZ를 사용하여 압출기 모터 연결을 확인할 수 있습니다(STEPPER=extruder 사용). 아니면, 압출기 모터를 별도로 테스트하는 것이 가장 좋습니다.(다음 섹션 참조).

모든 endstop을 확인하고 모든 스테퍼 모터를 확인한 후 원점 복귀 메커니즘을 테스트해야 합니다. 모든 축을 홈으로 이동하려면 G28 명령을 실행하십시오. 홈으로 가지 않으면 프린터에서 전원을 제거하십시오. endstop 및 스테퍼 모터 확인 단계를 다시 실행합니다.

## 압출기 모터 확인

압출기 모터를 테스트하려면 압출기를 인쇄 온도로 가열해야 합니다. Octoprint 온도 탭으로 이동하여 온도 드롭다운 상자에서 목표 온도를 선택합니다(또는 적절한 온도를 수동으로 입력). 프린터가 원하는 온도에 도달할 때까지 기다립니다. 그런 다음 Octoprint 제어 탭으로 이동하여 "Extrude" 버튼을 클릭합니다. 압출기 모터가 올바른 방향으로 회전하는지 확인합니다. 그렇지 않은 경우 이전 섹션의 문제 해결 팁을 참조하여 압출기의 "enable_pin", "step_pin" 및 "dir_pin" 설정을 확인하십시오.

## PID 설정 보정

Klipper supports [PID control](https://en.wikipedia.org/wiki/PID_controller) for the extruder and bed heaters. In order to use this control mechanism, it is necessary to calibrate the PID settings on each printer (PID settings found in other firmwares or in the example configuration files often work poorly).

압출기를 보정하려면 OctoPrint 터미널 탭으로 이동하여 PID_CALIBRATE 명령을 실행하십시오. 예시: `PID_CALIBRATE HEATER=extruder TARGET=170`

조정 테스트가 완료되면 `SAVE_CONFIG`를 실행하여 printer.cfg 파일을 업데이트하여 새 PID 설정을 업데이트합니다.

프린터에 히팅베드가 있고 PWM(Pulse Width Modulation) 구동을 지원하는 경우 베드에 PID 제어를 사용하는 것이 좋습니다. (베드 히터가 PID 알고리즘을 사용하여 제어될 경우 1초에 10번씩 켜지고 꺼질 수 있으므로 기계식 스위치를 사용하는 히터에는 적합하지 않을 수 있습니다.) 일반적인 베드 PID 교정 명령은 다음과 같습니다: `PID_CALIBRATE HEATER=heater_bed TARGET=60`

## 다음 단계

이 가이드는 Klipper 구성 파일의 핀 설정에 대한 기본적인 확인을 돕기 위한 것입니다. [bed leveling](Bed_Level.md) 가이드를 반드시 읽어주세요. 또한 Klipper로 슬라이서를 구성하는 방법에 대한 정보는 [Slicers](Slicers.md) 문서를 참조하십시오.

기본 인쇄가 작동하는지 확인한 후 [pressure advance](Pressure_Advance.md)보정을 고려하는 것이 좋습니다.

다른 유형의 자세한 프린터 보정을 수행해야 할 수도 있습니다. 이를 돕기 위해 온라인에서 여러 가이드를 사용할 수 있습니다(예: "3d 프린터 보정"에 대한 웹 검색 수행). 예를 들어 링잉(ringing)이라는 효과가 발생한다면 [resonance compensation](Resonance_Compensation.md) 튜닝 가이드를 따라 해보시면 됩니다.
