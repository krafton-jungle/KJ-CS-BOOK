# 저장장치 기술(Storage Technologies)

## 랜덤 접근 메모리(Random Access Memory, RAM)

- 랜덤 접근 메모리(Random Access Memory, RAM)란 저장된 데이터의 위치에 상관없이 동일한 속도로 접근할 수 있는 메모리를 뜻하며, 이를 원하는 주소에 즉시 접근하여서 데이터에 접근할 수 있다.

- RAM은 휘발성 메모리로, 전원이 꺼지면 저장된 데이터가 모두 사라지게 된다.

### 정적 램(Static Random Access Memory, SRAM)

- 한 비트를 두 개의 안정된 상태(0 또는 1) 중 하나로 유지할 수 있는 플립플롭 형태의 메모리 셀에 저장합니다.

- 여섯 개의 트랜지스터로 구성된 메모리 셀을 이용하여 무한 루프를 도는 방식으로 값이 영구적으로 유지된다.

- 구조가 단순해 읽기·쓰기가 매우 빠르지만, 셀당 트랜지스터가 많아 동일 면적당 제조 비용이 높다.

- 주로 캐시 메모리에 사용

### 동적 램(Dynamic Random Access Memory, DRAM)

- 한 비트를 캐패시터(축전기)의 전하량으로 표현하고, 이와 연결된 1개의 접근 트랜지스터가 읽고 쓰는 방식이다.

- 하나의 접근 트렌지스터와 캐패시터로 구성되어 있기 때문에, 면적을 적게 사용하기 때문에 제조 비용이 낮다.

- 캐패시터 전하는 시간에 따라 새어 나가므로, 메모리 컨트롤러 주기적 리프레시(읽은 뒤 다시 쓰기)를 수행해야 한다.

- 주로 메인메모리나 그래픽 시스템 프레임 버퍼에 사용

## 비휘발성 메모리

- 전원이 꺼져도 정보를 유지하는 비휘발성 메모리도 존재하는데, 이들을 ROM(Read Only Memory)이라 부른다

- 이러한 ROM에 저장된 프로그램들을 펌웨어라고도 부른다

### PROM(Programmable ROM)

- 비트를 기록하는 셀마다 퓨즈가 있고 퓨즈가 이어지면 1, 끊어지면 0으로 기록하는 방식이다

- 퓨즈로 구분하는 방식이기 때문에 정보를 한번만 기록할 수 있다

### EPROM(Erasable Programmable ROM)

- 저장장치 셀에 특수 파장의 빛을 쏴서 기록하는 방식이다

- 재기록 방법은 셀에 자외선을 비춰서 0으로 만들 수 있다

- 평균 1000번 정도 재기록 가능

### EEPROM(Electrically Erasable Programmable ROM)

- EPROM과 유사하나, 자외선이 아닌 전기적 신호로 재기록 가능하다

- 평균 105번 정도 재기록 가능
