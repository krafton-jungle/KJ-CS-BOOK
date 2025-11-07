# 시스템 수준 입출력

---

## 이 장에서 다루는 내용

> 이 장에서는 시스템 수준 입출력의 개념을 학습한다.

* \[간단한 설명] 유닉스 시스템에서 입출력이 어떻게 동작하는지 이해한다.
* \[학습 이유] 파일 입출력의 동작을 이해하고, 응용 수준에서 효율적으로 파일의 입출력을 수행하기 위해 학습한다.

## 주요 키워드

| 키워드                                                |
| ----------------------------------------------------|
| [Unix I/O System](#1-unix-io-system)                |
| [File Descriptor Table](#2-file-descriptor-table)   |
| [open, close, read, write](#3-open-close-read-write)|
| [Rio package](#4-rio-package)                       |


## 1. Unix I/O System

유닉스 시스템에서 모든 I/O 디바이스들은 파일로 모델링된다. 따라서 네트워크, 디스크, 터미널 등을 유형을 나누어 접근할 필요가 없다. 소켓, 파일, 디렉토리 등을 모두 파일로 취급하고 있기에, 모두 파일과 관련한 연산을 할 수 있다. 파일 읽기, 쓰기, 정보 조회, 파일 닫기 등을 동일한 인터페이스로 수행할 수 있다. 예를 들어, 일반 파일과 소켓 모두 read()함수로 다룰 수 있다.

## 2. File Descriptor Table

하나의 프로세스는 파일들을 파일 디스크립터 테이블이라는 것을 통해 관리한다. 파일 디스크립터라는 정수형 인덱스를 통해서 파일들 각각에 접근할 수 있다. 파일의 구체적인 정보는 드러내지 않고, 간편하게 인덱스로 관리하는 추상화가 이루어져 있다. 

파일 디스크립터 테이블의 파일 디스크립터들은 각각 오픈 파일 테이블 엔트리를 가리킨다. 그리고 각각의 오픈 파일 테이블 엔트리는 v-node 테이블을 참조한다.

## 3. open, close, read, write

```
#include <fcntl.h>
int open(const char *path, int flags, ... /*mode_t mode*/);
```
주어진 경로의 파일을 지정된 방식으로 열고, 파일 디스크립터를 반환한다. 새로운 파일을 만들 경우, 추가 인자로 권한이 필요하다.

```
#include <fcntl.h>
int close(int fd);
```
파일 디스크립터를 닫아 해당 파일에 대한 연결을 해제한다. 

```
#include <unistd.h>
ssize_t read(int fd, void buf[count], size_t count);
```
파일 디스크립터를 통해 파일의 내용을 읽어서 버퍼에 저장한다.

```
#include <unistd.h>
ssize_t write(int fd, const void buf[count], size_t count);
```
파일 디스크립터를 통해 파일에 버퍼의 바이트를 전송한다.

## 4. Rio package

Rio package(Robust I/O package)는 CSAPP에서 제공하는 I/O 패키지이다. 이 패키지를 사용하는 이유는 안정적으로 I/O 함수를 실행하기 위해서이다. 

read(), write()는 요청한 크기만큼 수행하는 것을 보장하지 않는다. 함수 실행 중에 시그널, 인터럽트로 인해 중간에 중단될 수 있다. 이를 고려하여 관리하는 것이 요구된다. 따라서 해당 패키지를 사용해서 원하는 만큼 읽고 쓸 수 있도록 해주는 것이 좋다. 

