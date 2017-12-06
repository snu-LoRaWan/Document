# Project process



## Client

### Code Struct 분석

main.c 코드에서는 while문 안에서 `lora_fsm()` 함수를 통해서 상태를 전환한다. 이때 `lora_fsm()`은 `Core/lora.h`에 정의되어 있으며, `Core/lora.c`에 구현되어 있다.

특히 OTAA를 사용할 때는 join한 이후 바로 sleep 상태로 전환된다. 이 때 메시지를 보내기 위해서 콜백 함수가 호출되기를 기다린다. 이 콜백 함수는 초기화시 `TimerInit`에 아래와 같이 시간과 함께 `OnTxNextPacketTimerEvent()` 함수 포인터를 넘겨줌으로써 다음과 같이 작동한다.

`TimerInit` (`TimerEvent_t`,  `OnTxNextPacketTimerEvent()`) &rarr; `OnSendEvent()` &rarr; `DEVICE_STATE_SEND`로 상태 변경

### Changes

* `lora_fsm()` 함수에서 OTAA를 사용할 때 초기화 이후 Sync state로 넘어가도록 함
* Sync state에서 Sync 완료 후 join으로 넘어가도록 함

### Remains

* Sync state에서 비콘 채널을 수신하도록 변경


* 서버 비콘 송신 구현 후, join이 성공하는지 테스트
* Timer (재)정의
  * Beacon Timer
  * Send Timer
* Beacon Sync 이후 state 이전을 위하여 Timer reset
  * Beacon Timer reset
  * Send Timer reset
* listen state 정의
* listen state에서 fsm이 동작하도록 수정
* send할 때 Send에서 Beacon Timer 이상 시간을 보내지 않도록 조정

## Server

### Code Struct 분석

`https://github.com/gotthardp/lorawan-server/blob/master/doc/Development.md` 문서를 참고해서 확인한 결과, gateway로부터 들어온 packet들은 udp link를 통해 `lorawan_gw_forwarder.erl` 파일(프로세스)에서 처리하고 있다.

이 파일에서 패킷을 받은 후, gateway protocol(`https://github.com/Lora-net/packet_forwarder/blob/master/PROTOCOL.TXT`)에 맞춰 `lorawan_gw_router.erl`의 함수로 넘겨준다.

`lorawan_gw_router.erl`에서는 self-async message를 보냄으로써 위의 요청들을 async하게 처리하기 시작한다.

Downlink의 경우, `lorawan_gw_router.erl`에서의 downlink request에서 시작된다.

### Remains

* `lorawan_gw_router.erl`의 downlink를 누가 invoke하는 지 찾기 (worker쪽 코드 추가 조사)
* 서버 비콘을 어디에 정의하면 되고, 스케쥴링은 어떻게 하면 될 지 찾아보기
