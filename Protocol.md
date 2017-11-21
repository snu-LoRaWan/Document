Protocol
---

## Beacon

### Frame

* Type: beacon
* Beacon Cycle Length(ms)
* Target ID List
* Time Stamp

### Channel

### Cycle

### Downstream ACK Control

## Join Senario

### Data Stream Control

#### Data type

* Confirmed: ACK 요구
* Unconfirmed: ACK 필요 없음

#### Upstream
#### Downstream

### OTAA + Beacon

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

### Beacon 랜덤 채널 쓰기
### Beacon cycle 유동화
### Beacon Tartget ID 갯수 최적화
### Beacon 채널 활용
### Beacon Timestamp 활용
