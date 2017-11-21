Protocol
---

## Beacon

### Frame

* Type: beacon
* Beacon Cycle Length(ms)
* Target ID List
* Time Stamp(option)

### Channel

고정 채널 사용

### Cycle

* cycle = listen duration + beacon window
  * listen duration = guard time + reserved time
  * beacon window

## Join Senario - OTAA + Beacon

### 1. Beacon 1 cycle 동안 Beacon을 기다림
### 2-1. Beacon 수신 실패

* 서버가 데이터를 보내고 있는지 확인 &rarr; 이 경우 계속 listen 하면서 대기
* 아닌 경우 2~3 cycle 대기 후 서버가 없다고 판단

### 2-2. Beacon 수신 성공

해당 beacon cycle을 계산하고 window 시간에 OTAA join

## Data Stream Control

### Data type

* Confirmed: ACK 요구
* Unconfirmed: ACK 필요 없음

### Upstream

* Confirmed: Packet 전송 - ACK (TO가 날 경우 재전송)
* Unconfirmed: Packet 전송 - (listen)

### Downstream

* Confirmed: Beacon - Packet 전송 - ACK (TO가 날 경우 재전송)
* Unconfirmed: Beacon - Packet 전송 - (listen)

## Edge Case

프로토콜이 잘못 되었거나, 그렇지 않아도 생길 수 있는 문제에 대해서 다룬다.

### 어긋난 beacon waiting

### 서버 다운

### 신호가 약해짐

### beacon이 충돌
* beacon이 랜덤 채널로 보낼 경우에 충돌 가능
* 그 타이밍에 join 하려고 하는 경우 &rarr; beacon을 하나 받은 뒤 join

### endnode가 downstream 받지 않음

## Improvement

### Beacon cycle 유동화

* downstream이 적은 경우 cycle을 늘림

### Beacon Tartget ID 갯수 최적화

* 한 번에 모든 end-node에게 대기를 기다리지 않도록 함

### Beacon 채널 활용

* Beacon window에 해당 채널에서 전송 허용

### Beacon Timestamp 활용

* 시간 동기화에 활용
