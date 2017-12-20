Final Report
---



## Protocol

기존의 LoRaWAN Class A에, Server에서 donwlink를 개시할 수 있는 기능을 추가했다.
Server는 일정 시간마다 Beacon Frame을 broadcast한다. 이 Beacon Frame에는 어떤 Client가 downlink data를 받기 위해 radio를 열어야 하는지에 대한 정보가 들어있다.
해당하는 Client는 beacon을 받은 뒤 일정 시간 후에 radio를 열어야 한다.

### Beacon Interval

구현할 Server는 일정 시간마다 Beacon Frame을 broadcast할 것이다. 실험 결과 Gateway가 초당 1 packet 이상 잘 보내지지 않아서, Beacon Interval을 3000(ms)로 정했다.

### Beacon-transmit Interval

Server는 Beacon Frame을 broadcast하고 일정 시간 이후에 해당하는 Client로 downlink packet을 전송할 것이다. 역시 Gateway의 성능을 고려하여, 보수적으로 이 시간 간격을 1000(ms)로 정했다.

### Beacon Frame

Beacon은 자주 송수신되는 packet이므로, 최대한 간단해야 한다. 우리가 구현할 것은 MAC level의 protocol이기 때문에, PHYPayload를 잘 정의하면 된다.
기존의 PHYPayload는 MHDR의 종류에 따라 다음 3가지가 정의되어 있다.

![](images/final/PHYPayload.jpg)

MHDR의 앞 3bit는 MType이라 불리는데, 이 중 한 종류(`110`)는 RFU(Reserved for Future Usage)로 사용되지 않고 있다.
Beacon Frame의 MType을 이것으로 정의했으며, 나머지 bit는 0으로 정의했다. 즉, Beacon Frame의 MHDR(1byte)는 `11000000(2)`로 정의했다.

이어지는 Beacon Payload에서는 downlink를 받을 Client를 지정할 필요가 있다. Join과정 중 생성된 DevAddr(4byte)를 실어 보내기로 했다. 만약 보낼 packet이 없다면, 4byte의 0을 보낸다.

따라서 Beacon Frame은 다음과 같이 구성된다:
```
MHDR(1byte, 11000000) + Addr(4byte, DevAddr or 0)
```

### Timing, Datarate, Coding Rate, Frequency

위의 단락들에서 거의 설명했지만, 다시 설명하자면 Server는 3000ms마다 Beacon Frame을 broadcast하며, Beacon Frame transmit 1000ms 이후에 downlink data를 전송한다.
Downlink data, Beacon frame 모두 921.9MHz, Datarate "SF12BW125", Coding Rate "4/5"를 택했다.

---

## Server Implementation

기본적인 서버 코드의 구조는 다음 그림과 같다.
![](images/final/server_structure.jpg)

서버에서 추가로 구현해야 하는 것은 Beacon Frame Broadcast와, Downlink Data Unicast이다.
기본적으로, 다음과 같은 상수들을 `src/lorawan_server.app.src`에 추가했다.
```text
{beacon_freq, 921.9},
{beacon_interval, 3000},
{beacon_transmit_interval, 1000}
```

### Beacon Frame Broadcast
`erlang`의 `timer` library를 이용해 beacon frame broadcast를 구현했다. 코드 구조상 Gateway의 정보를 직접 다루는 `src/lorawan_gw_router.erl` 파일에 Beacon 전송을 구현하는 것이 좋다고 판단했다. 해당 파일의 `init/0` 함수에 다음과 같은 `timer`를 추가했다.

```erl
{ok, BInterval} = application:get_env(lorawan_server, beacon_interval),
timer:send_after(BInterval, beacon)
```

`BInterval`의 값은 3000이다. 이 시간 이후 `beacon` signal이 자기 자신에게 돌아오면서, `handle_info(beacon, State)를 호출한다. 이제 그 함수에서 beacon frame을 전송하면 될 것이다.

`handle_info(beacon, State)`는 erlang database인 `mnesia`의 `gateways` table에서 Gateway의 MAC주소를 읽어와서, 해당 Gateway에 Beacon Frame을 만들어 전송하는 역할을 했다.
해당 함수가 종료될 때, 다시 timer를 작동시켜 주기적으로 Beacon을 전송하도록 했다.

### Downlink Data Unicast
위의 `handle_info(beacon, State)` 함수가 불릴 때, table `txframes`에 보낼 frame이 있는지, 그리고 있다면 가장 예전에 store된 frame id를 반환하는 함수 `get_oldest_frid/0`을 구현했다.

만약 위와 같은 frame이 있다면,
```erl
timer:send_after(BTInterval, {beacon_transmit, Trid})
```
가 호출된다. `BTInterval`은 1000(ms)이며, `Trid`는 transmit할 frame의 `frid`이다. 위 내용대로면, 가장 오래 전 store된 frame을 보내게 된다(Last-in, First-out).

요약하자면 Beacon이 보내진 지 1초 후, 비콘 재전송과는 관계 없이 async하게 해당 파일의 `handle_info({beacon_transmit, Trid}, State)`함수가 호출되게 된다.
해당 함수에서는 transmit를 하고 `txframes` table에서 해당 frame을 pop한다.
data 전송은 `lorawan_handler:downlink/3`함수를 이용했다.
