
# 주소 지정


## '가상주소(VA) -> 물리주소(PA)' 변환 과정


- 물리적 주소는 메인 메모리 내의 **바이트 단위** 셀에 할당되는 고유한 주소


### 현대 시스템에서는 가상 메모리 시스템을 통해 간접적으로 접근

- **MMU(Memory Management Unit)**라는 전용 하드웨어를 통해 가상 주소를 물리적 주소로 변환
- 주소 변환 과정

    - 가상 주소 공간의 주소를 물리적 주소 공간의 주소로 매핑
    - OS가 관리하는 주 메모리의 lookup table(page table)을 사용하여 실시간으로 수행됨


> [!NOTE] Page Table
>
> - 프로세스마다 하나의 페이지 테이블 존재 (스레드 간 공유 가능)
> - 페이지 테이블은 데이터 구조로, 물리 메모리(RAM)에 저장됨, 파일로 저장되지 않음
> - 운영체제와 하드웨어가 직접 관리
> - 가상 주소의 가상 페이지 번호를 물리 메모리의 물리 페이지 번호로 매핑



### 가상주소는 MMU에 의해 물리주소로 변환된다.

- 가상주소의 두가지 주요 부분

  - **VPN (Virtual Page Number)**
    - VPN은 VA의 상위 $(n-p)$ 비트를 구성함
    - 페이지 테이블(Page Table) 내의 적절한 PTE(Page Table Entry)를 선택하는 인덱스로 사용됨

  - **VPO (Virtual Page Offset)**
    - VPO는 $p$비트를 구성하며, 페이지 내의 바이트 위치를 나타냄
    - VPO는 주소 변환 후 생성되는 물리적 주소(PA)의 PPO(Physical Page Offset)와 값이 동일함


- 물리주소의 두가지 주요 부분

  - **PPN(물리 페이지 번호)**: 페이지 테이블의 PTE(Page Table Entry)에서 가져옴
  - **PPO(물리 페이지 오프셋)**: VA의 VPO(Virtual Page Offset)를 그대로 복사, 페이지 내 정확한 바이트 위치를 지정(오프셋은 변환 없이 사용)


- 대부분의 시스템은 SRAM 캐시(L1, L2, L3)에 접근할 때도 **물리적 주소 지정을 선호**

    - 정의: 주소 변환이 먼저 수행된 다음, 물리적 주소가 SRAM 캐시로 전달되어 Cache Lookup

    - 이점: 

        - 여러 프로세스가 동시에 캐시에 블록을 가질 수 있고
        - 동일한 가상 페이지의 블록을 공유하기 쉬워지며
        - 접근 권한 확인 문제가 VA -> PA 주소 변환 단계에서 처리되어, 캐시가 복잡한 보호 문제를 처리할 필요가 없음



### ECF 및 메모리 보호


_MMU는 단순한 주소 변환뿐만 아니라 예외적 제어 흐름(ECF) 및 메모리 보호와도 깊이 연관됨._


- **페이지 폴트 처리**:
  MMU가 PTE를 확인했을 때, 유효 비트(Valid Bit)가 설정되어 있지 않다면 (DRAM에 페이지가 캐시되어 있지 않다면), MMU는 **페이지 폴트(Page Fault) 예외**를 트리거하고 OS 커널로 제어를 전환


- **메모리 보호**: PTE에 포함된 권한 비트(Permission Bits, 예: SUP, READ, WRITE)를 MMU가 검사하여, 해당 페이지에 대한 접근 권한(읽기/쓰기/커널 모드 접근)을 제어함.

  권한 위반 시 MMU는 **일반 보호 오류(General Protection Fault)**를 트리거하여 프로세스를 종료시킬 수 있음




## 더 알아보기


> [!nt] "Frame Number" == "Physical Page Number"
>
> 예를 들어, 프레임 크기가 4KB인 경우  프레임 번호 0은 물리 주소 0x0000–0x0fff에 해당



> [!nt] PCB(Process Control Block)는 페이지 테이블을 가리키는 포인터를 가짐
>
> - PCB는 커널 메모리 영역에 저장됨
> - PCB는 프로세스 정보를 저장하는 커널 데이터 구조
>   - 상태, 레지스터, 스케줄링 정보 등 포함
>   - 프로세스의 **페이지 테이블**을 가리키는 포인터 포함
>     - 컨텍스트 스위칭 시 **PTBR**(Page Table Base Register)에 로드됨



> [!nt] Page Table vs Page Table Entry
>
> - **Page Table:**
>   - 운영체제에서 가상 메모리 주소를 실제 물리 메모리 주소로 변환하기 위해 사용하는 자료구조
>   - _각 프로세스마다 하나씩 존재하며, 여러 개의 Page Table Entry로 구성_
>   - 쉽게 말해, Page Table은 여러 Page Table Entry를 모아놓은 배열
> 
> - **Page Table Entry:**
>   - Page Table을 구성하는 하나의 "항목"
>   - 각 PTE는 하나의 가상 페이지에 대한 정보를 담고 있음
> 
>     (대체로) 아래를 포함한다:
> 
>       > 1. **Physical Frame Number (PFN) / Physical Address (aka. Physical Page Number (PPN))**
>       >    - 해당 가상 페이지가 매핑되는 실제 물리 메모리 프레임의 번호 또는 주소.
>       > 2. **Present/Valid Bit**
>       >    - 페이지가 실제로 물리 메모리에 존재하는지 여부.  
>       >    - 0이면 페이지 폴트가 발생할 수 있음.
>       > 3. **Access Rights (Protection Bits)**
>       >    - 읽기/쓰기/실행 권한 등 접근 제어 정보.
>       > 4. **Dirty Bit**
>       >    - 페이지가 수정(쓰기)되었는지 표시.  
>       >    - 페이지 교체 시 디스크에 다시 기록해야 하는지 결정.
>       > 5. **Referenced/Accessed Bit**
>       >    - 최근에 접근(읽기/쓰기)된 적이 있는지 표시.  
>       >    - 페이지 교체 알고리즘에서 사용.
>       > 6. **Cache/Write-Through/Write-Back Control Bits**
>       >    - 캐시 사용 여부, 쓰기 정책 등 메모리 동작 방식 제어.
>       > 7. **User/Supervisor Bit**
>       >    - 사용자 모드/커널 모드에서 접근 가능한지 여부.
>       > 8. **Other OS or Architecture-specific Bits**
>       >    - 예: 페이지 크기, 글로벌 페이지 여부, NX(Non-Executable) 비트 등.



> [!nt] TLB(Translation Lookaside Buffer)
>
> - MMU 내부의 **캐시**
> - 최근 접근한 페이지의 데이터를 저장
>   - 키-값 쌍 형태로 저장  
>     (예: PTE의 정보, VPN → PPN + valid/permissions 등 플래그)
>     → 빠른 변환 가능
>
> - 주소 변환 과정에서 접근  
>   - 히트 시(TLB에 변환 정보가 있을 때) TLB에서 바로 PPN을 가져옴(빠름)
>   - 미스 시  
>     1. PTBR을 통해 RAM의 페이지 테이블 접근(하드웨어가 먼저 L1/L2/L3 캐시에서 PTE 탐색)
>     2. PTE를 캐시 또는 RAM에서 가져옴
>     3. 변환 정보(VPN → PPN + 플래그)를 TLB에 캐싱
>     4. 이후 과정 진행



> [!nt]
>
> 리눅스 x86-64 시스템에서 프로세스의 VM은 일반적으로 아래와 같이 구성됨
>
>   - 사용자 스택
>   - 공유 라이브러리
>   - 힙
>   - 초기화되지 않은 데이터(.bss)
>   - 초기화된 데이터(.data)
>   - 코드(.text)


