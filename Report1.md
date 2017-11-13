Report 1
---



## Study

### LoRaWAN Specification Class A

LoRaWAN은 Class A, B, C로 구성되어 있다.
이때 Class A는 모든 LoRaWAN device들이 구현해야 하는 기본적 클래스이다.
기본적으로 Physical Message Format 및 MAC Message Format이 정의되어 있으며,
이에 따른 MAC 명령어 종류와 End-Device 활성화 방법이 명시되어 있다.
이때 본 과제에서는 Over-the-Air Activaction(OTAA) 방식으로 End device를 활성화해야 한다.

### LoRaWAN OTAA join procedure

OTAA는 서버와 두 개의 MAC message를 교환함으로써 시작된다.
하나는 join request이며, 나머지는 join accept이다.

#### Join-request message

Join-request message는 AppEUI, DevEUI, DevNonce의 세 가지 인자로 구성되어 있다.
DevNonce는 랜덤 값이다.

* AppEUI(8 byte)

AppEUI는 IEEE EUI64 주소 공간에서 유일하게 정의되는 app ID이다.
이 값은 활성화 전의 end-device에 저장된다.

* AppKey(8 byte)

AppKey는 IEEE EUI64 주소 공간에서 유일하게 정의되는 device ID이다.

* DevNonce(2 byte)

각 end-device마다 사용한 DevNonce를 저장하고,
다른 기기가 해당 값으로 요청할 경우 무시한다.
Message Integrity Code(MIC)는 아래와 같이 계산된다.

```
cmac = aes128_cmac(AppKey, MHDR | AppEUI | DevEUI | DevNonce)
18 MIC = cmac[0..3]
```

이 join-request 메시지는 암호화되지 않는다.
또한 전송시의 data rate는 랜덤이지만,
다수가 사용하는 전송률을 사용할 것이 권장된다.

#### Join-accept message

join-request 메시지가 적합한 경우, 서버는 join-accept 메시지로 응답한다.
다른 점은 평범한 downlink 메시지와 동일하나,
JOIN_ACCEPT_DELAY1 혹은 JOIN_ACCEPT_DELAY2만큼 지연 후 전송하는 차이가 있다.
이 두 값은 지역별 Parameter에 정의된 RX1과 RX2와 같다.

join-request가 받아들여지지 않을 경우 응답은 없다.

AppKey를 이용해서 암호화된다.

Join-accept message의 인자는 아래와 같다.

* AppNonce(3 byte)

네트워크 서버가 제공하고, end-device가 사용하는 임의값 혹은 ID이다.
이 값을 통해 NwkSKey, AppSKey를 만들어낸다.

* NetID(3 byte)

네트워크 ID이다.
NetID의 LSB 7 bit NwkID이다.
이웃하거나 중첩된 네트워크는 다른 NwkID를 지녀야 한다.
나머지 MSB는 네트워크 operator에 의해 자유롭게 선택된다.

* DevAddr(4 byte)

* DLSettings(1 byte)

RFU(1 bit), RX1DRoffset(3 bits), RX2 Data rate(4 bits)로 구성되어 있다.
RX1DRoffset은 첫 번째 reception slot(RX1)에서 사용되는 data rate이다.


* RxDelay(1 byte)

RXTimingSetupReq 명령어와 같은 컨벤션을 따른다.

* CFList(16 byte, Optional)

CFList는 end-device joining을 위한 것으로, LoRaWAN 지역별 Parameter에 정의되어 있다.

### Source Code

## Project Plan

- 11/15: Install & run end node, gateway, network server
- 11/16: Design beacon based bi-directional communications
  * Design protocol
  * Design program structure
  * Design test scenarios
- 11/22: Progress Report 2
- 11/29: Implement program
- 12/4: Test code with scenarios and modify errors
- 12/5: Performance evaluation 1
- 12/6: Progress Report 3
- 12/13: Improve performance
  * Investigate the code where to improve
  * performance evaluation to find bottleneck
- 12/17: Performace evaluation 2
- 12/18: Final Report

