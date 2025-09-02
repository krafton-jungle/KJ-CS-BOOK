# 시그널 (Signals)

## 이 장에서 다루는 내용
프로세스가 다른 프로세스에게 또는 커널이 프로세스에게 어떤 이벤트가 발생했음을 알리는 고수준 소프트웨어 인터럽트인 **시그널(Signal)**에 대해 학습한다. </br>
시그널의 개념, 종류, 처리 과정과 함께 시그널을 안전하게 다루는 방법을 이해하는 것이 목표이다.

> 예외적인 제어 흐름의 한 형태로, 프로그램이 예상치 못한 이벤트에 대응하고 스스로를 보호하는 방법을 알아보자.

---

## 학습 목표

- 시그널의 개념과 역할을 이해한다.
- 시그널을 보내고, 받고, 처리하는 과정을 설명할 수 있다.
- `SIGINT`, `SIGKILL`, `SIGCHLD` 등 주요 시그널의 의미와 용도를 알 수 있다.
- `signal` 함수와 `sigaction` 함수의 차이를 이해하고, `sigaction`을 사용해 시그널 핸들러를 설치할 수 있다.
- 시그널 핸들러 작성 시 주의사항, 특히 비동기-신호-안전성(Async-Signal-Safety)의 개념을 이해한다.
- `sigprocmask` 함수를 이용해 특정 시그널을 블록(block)하는 방법을 학습한다.

---

## 1. 시그널이란?

**시그널(Signal)**은 유닉스 계열 시스템에서 사용되는 간단한 메시지 메커니즘으로, 어떤 이벤트가 발생했음을 프로세스에게 알려주기 위해 사용된다. 커널이나 다른 프로세스가 특정 프로세스에게 시그널을 보낼 수 있다.

예를 들어, 사용자가 터미널에서 `Ctrl+C`를 누르면, 포그라운드(foreground) 프로세스 그룹에 `SIGINT` 시그널이 전달되어 프로세스가 종료된다. 이처럼 시그널은 예외적인 제어 흐름(ECF)을 구현하는 고수준의 소프트웨어 형태이다.

### 시그널 관련 용어

- **전송 (Sending)**: 커널이나 한 프로세스가 다른 프로세스에게 시그널을 보내는 것
- **수신 (Receiving)**: 대상 프로세스가 시그널을 받는 것
- **처리 (Handling)**: 시그널을 받은 프로세스가 특정 동작을 취하는 것
    1.  **기본 동작 (Default Action)**: 각 시그널에 미리 정해진 동작을 수행 (대부분 프로세스 종료)
    2.  **무시 (Ignore)**: 시그널을 무시한다. (`SIGKILL`과 `SIGSTOP`은 무시할 수 없다.)
    3.  **핸들러 실행 (Catch)**: 시그널 핸들러(Signal Handler)라는 특정 함수를 실행하여 시그널을 처리한다.

- **펜딩 시그널 (Pending Signal)**: 전송되었지만 아직 수신되지 않은 시그널. 특정 종류의 시그널은 단 하나만 펜딩될 수 있다.
- **블록된 시그널 (Blocked Signal)**: 프로세스가 특정 시그널을 일시적으로 받지 않도록 막아놓은 상태. 블록된 시그널은 전달되지만, 프로세스가 해당 시그널을 언블록(unblock)할 때까지 수신되지 않는다.

---

## 2. 주요 시그널 종류

| 시그널 이름 | 기본 동작 | 설명 |
| --- | --- | --- |
| `SIGINT` | 종료 | 터미널에서 `Ctrl+C`를 눌렀을 때 전송되는 인터럽트 시그널 |
| `SIGKILL` | 종료 | 프로세스를 강제로 종료시키는 시그널 (무시하거나 잡을 수 없음) |
| `SIGTERM` | 종료 | 프로세스에게 정상적으로 종료하라고 요청하는 시그널 (가장 일반적인 종료 신호) |
| `SIGSEGV` | 종료 (코어 덤프) | 잘못된 메모리 주소에 접근했을 때 (세그멘테이션 오류) 발생 |
| `SIGCHLD` | 무시 | 자식 프로세스가 종료되거나 멈췄을 때 부모 프로세스에게 전송됨 |
| `SIGALRM` | 종료 | `alarm` 함수로 설정한 시간이 다 되었을 때 발생 |
| `SIGSTOP` | 프로세스 중지 | 프로세스를 일시 정지시킴 (무시하거나 잡을 수 없음) |
| `SIGCONT` | 계속 실행 | `SIGSTOP`이나 `SIGTSTP`로 멈춘 프로세스를 다시 실행시킴 |

---

## 3. 시그널 처리하기

### `signal` 함수

`signal` 함수는 특정 시그널에 대한 처리 방법을 지정하는 가장 간단한 방법이다.

```c
#include <signal.h>

typedef void (*sighandler_t)(int);

sighandler_t signal(int signum, sighandler_t handler);
```
- `signum`: 처리할 시그널 번호 (e.g., `SIGINT`)
- `handler`: 시그널을 처리할 핸들러 함수 포인터. `SIG_DFL`(기본 동작)이나 `SIG_IGN`(무시)을 지정할 수도 있다.

하지만 `signal` 함수는 일부 유닉스 시스템에서 신뢰성 문제가 있어, 현재는 이식성이 높고 더 많은 기능을 제공하는 `sigaction` 함수 사용이 권장된다.

### `sigaction` 함수 (권장)

`sigaction` 함수는 시그널 처리에 대한 더 정교한 제어를 제공한다.

```c
#include <signal.h>

int sigaction(int signum, const struct sigaction *act, struct sigaction *oldact);
```

`sigaction` 구조체는 다음과 같이 정의된다.
```c
struct sigaction {
    void       (*sa_handler)(int); // 시그널 핸들러 함수 포인터
    sigset_t   sa_mask;            // 핸들러 실행 중 블록할 시그널 집합
    int        sa_flags;           // 핸들러 동작을 수정하는 플래그
    void       (*sa_sigaction)(int, siginfo_t *, void *); // 실시간 시그널 핸들러
};
```
- `sa_handler`: `signal` 함수와 동일한 핸들러 함수 포인터
- `sa_mask`: 핸들러가 실행되는 동안, 여기에 지정된 시그널들은 자동으로 블록된다. 이는 핸들러 실행 중에 다른 시그널에 의해 방해받지 않도록 보장한다.
- `sa_flags`: `SA_RESTART` (시스템 콜 재시작) 등 다양한 옵션을 설정할 수 있다.

**예제 코드:**
```c
#include <stdio.h>
#include <stdlib.h>
#include <signal.h>
#include <unistd.h>

void sigint_handler(int sig) {
    printf("Caught SIGINT! Press Ctrl+C again to exit.\n");
    // SIGINT에 대한 핸들러를 기본 동작으로 복구
    signal(SIGINT, SIG_DFL); 
}

int main() {
    struct sigaction sa;
    sa.sa_handler = sigint_handler;
    sigemptyset(&sa.sa_mask);
    sa.sa_flags = 0;

    // SIGINT에 대한 핸들러로 sigint_handler를 등록
    if (sigaction(SIGINT, &sa, NULL) == -1) {
        perror("sigaction");
        exit(1);
    }

    printf("PID: %d\n", getpid());
    printf("Waiting for signal...\n");

    while(1) {
        pause(); // 시그널을 받을 때까지 프로세스를 잠시 멈춤
    }

    return 0;
}
```

---

## 4. 시그널 핸들러와 비동기-신호-안전성

시그널 핸들러는 메인 프로그램의 실행 흐름과 상관없이 언제든지 호출될 수 있는 **비동기적(asynchronous)** 특성을 가진다. 이 때문에 핸들러 함수를 작성할 때는 매우 신중해야 한다.

메인 코드와 핸들러가 같은 전역 변수에 접근하면, 경쟁 상태(race condition)가 발생할 수 있다. 예를 들어, 메인 코드가 전역 변수를 수정하는 도중에 시그널이 도착해 핸들러가 실행되면, 데이터가 깨질 수 있다.

### 비동기-신호-안전 (Async-Signal-Safe) 함수

이러한 문제를 피하기 위해, 시그널 핸들러 내에서는 비동기-신호-안전(async-signal-safe)하다고 보장된 함수만 호출해야 한다. 이런 함수들은 다음과 같은 특징을 가진다.

1.  **재진입 가능 (Reentrant)**: 함수가 자기 자신을 다시 호출해도 안전하게 동작한다.
2.  전역 변수나 정적 데이터 구조에 대한 락(lock) 없이 접근하지 않는다.

`printf`, `malloc`, `free` 같은 표준 라이브러리 함수 대부분은 **안전하지 않다**. 예를 들어, `printf`는 내부적으로 전역 버퍼를 사용하는데, 메인 코드에서 `printf`를 호출하는 도중 핸들러가 끼어들어 `printf`를 또 호출하면 버퍼가 깨질 수 있다.

**안전한 함수의 예:** `write`, `read`, `_exit`, `kill`, `getpid`, `sleep` 등

> **규칙**: 시그널 핸들러는 최대한 작고 단순하게 유지해야한다. 핸들러에서는 전역 플래그 변수 값만 바꾸고, 실제 작업은 메인 루프에서 해당 플래그를 확인하여 처리하는 것이 좋은 설계이다.

---

## 5. 시그널 블록하기

때로는 코드의 중요한 부분(critical section)을 실행하는 동안 시그널이 도착하는 것을 막고 싶을 수 있다. `sigprocmask` 함수를 사용하면 특정 시그널을 선택적으로 블록하거나 언블록할 수 있다.

```c
#include <signal.h>

int sigprocmask(int how, const sigset_t *set, sigset_t *oldset);
```
- `how`: 동작을 지정
    - `SIG_BLOCK`: `set`에 포함된 시그널을 현재 블록된 시그널 집합에 추가한다.
    - `SIG_UNBLOCK`: `set`에 포함된 시그널을 블록 해제한다.
    - `SIG_SETMASK`: 블록된 시그널 집합을 `set`으로 교체한다.
- `set`: 블록하거나 언블록할 시그널의 집합
- `oldset`: 이전의 블록된 시그널 집합을 저장할 위치 (필요 없다면 `NULL`)

`sigset_t` 타입의 시그널 집합을 다루기 위해 다음 함수들이 사용된다.
- `sigemptyset(sigset_t *set)`: 집합을 비운다.
- `sigfillset(sigset_t *set)`: 모든 시그널을 집합에 추가한다.
- `sigaddset(sigset_t *set, int signum)`: 특정 시그널을 집합에 추가한다.
- `sigdelset(sigset_t *set, int signum)`: 특정 시그널을 집합에서 제거한다.
