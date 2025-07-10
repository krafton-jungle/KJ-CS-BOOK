# 3.6 제어문

이전 챕터 들에서는 인스트럭션들이 하나씩 순차적으로 실행되는 “직선적인” 코드 동작만을 다뤘다.

C언어 구문들인 조건코드, 반복문, 스위치문들은 조건부 실행을 요구한다. 

기계어 코드에서 조건부 실행을 위해 두 개의 기본적인 낮은 수준의 방법을 제공한다: 

데이터 값을 시험한 후 실험결과에 따라 

1. 제어흐름을 바꾸거나,
2. 데이터흐름을 바꾼다. 

보통 C언어에서의 인스트럭션들은 프로그램에 표시되려면 “순차적”으로 진행이 돼야하는데,  jump 인스트럭션을 이용해 기계어 인스트럭션들의 실행 순서를 변경할 수 있다. 컨트롤이 다른 파트의 프로그램을 넘어가는 것을 의미한다. 컴파일러가 낮은 수준의 방법으로 인스트럭션의 코드를 생성해야 한다.  

# 3.6.1 조건 코드 [Condition codes]

## CPU의 단일 비트 조건 코드로 구성된 레지스터

- CF - Carry Flag. (비부호형) 두 개의 비부호형 숫자를 연산하며 난 오버플로우를 검출할 때 사용한다.
- ZF - Zero Flag. 연산의 값이 0일 때 표시한다.
- SF - Sign Flag. (부호형) 연산의 처음 바이트의 값이 1일 때 (음수일 때) 표시한다.
- OF - Overflow Flag. (부호형) 2의 보수의 오버플로우를 검출할 때 사용한다.

 leaq 인스트럭션은 주소 계산에 사용하기 위한 것이므로 조건 코드를 변경하지 않고, 계산된 값을 레지스터에 저장한다. 

<div align="center">
 <img src="/assets/ch3_a_assembly/leaq_insturctions.png" width="600"/>
 <br>
 <br>
 <sub>leaq 인스트럭션</sub>
 </div>

<div align="center">
 <img src="/assets/ch3_a_assembly/compare_test.png" width="600"/>
 <br>
 <br>
 <sub>비교 시험 및 시험 인스트럭션</sub>
 </div>


이 인스트럭션의 집합은 다른 레지스터 값은 수정하지 않은 채 조건코드를 바꿔준다.

CMP

- 두 오퍼랜드 차에 따라 목적지를 갱신하지 않고 조건 코드를 설정한다 (나머지는 SUB 인스트럭션과 같은 방법으로 동작).
- 두 오퍼랜드 크기가 같으면 영 플래그를 1로 설정한다.

TEST

- 목적지 오퍼랜드를 변경하지 않고 조건 코드를 설정한다 (나머지는 AND 인스트럭션과 같은 방법으로 동작).

# 3.6.2 조건 코드 사용하기[Accessing the Condition Codes]

SET 인스트럭션은 조건 코드의 조합에 따라 0 또는 1을 한 바이트에 기록한다. 

조건코드를 이용하는 보편적인 세 가지의 방법: 

1. 조건 코드의 조합에 따라 0 또는 1을 한 개의 바이트에 기록한다.
2. 조건에 따라 프로그램의 다른 부분으로 이동한다.
3. 조건에 따라 데이터를 전송한다.

<div align="center">
 <img src="/assets/ch3_a_assembly/sets_instructions" width="600"/>
 <br>
 <br>
 <sub>셋 인스트럭션</sub>
 </div>

이런 SET 인스트럭션들은 서로 다른 접미어를 갖으며, 조건 코드의 어떤 조합을 사용할 것인지를 나타낸다. 

- ex) setl: set less, setb: set below

목적지로 하위 단일 바이트 레지스터와 바이트 메모리 주소를 사용하고 이 바이트를 0이나 1로 기록한다. 

# 3.6.3 점프(Jump) 인스트럭션[Jump Instructions]

점프 인스트럭션을 이용하면 프로그램을 완전히 새로운 위치로 실행하도록 전환할 수 있다. 

어셈블리 코드에서 점프의 목적지는 레이블(label)로 표시한다.

<div align="center">
 <img src="/assets/ch3_a_assembly/jump_instruction_ex.png" width="600"/>
 <br>
 <br>
 <sub>점프 인스트럭션 예시</sub>
 </div>

jump .L1은 프로그램 movq 인스트럭션을 건너뛰는 대신 popq로 다시 실행하게 한다.

목적 코드 파일을 만들기 위해서 어셈블러는 모든 레이블에 붙은 인스트럭션들의 주소를 결정하고, 점프 인스트럭션의 일부분인 점프 목적지를 인코딩한다.

<div align="center">
 <img src="/assets/ch3_a_assembly/jump_instructions.png" width="600"/>
 <br>
 <br>
 </div>

두 가지의 점프 방식:

1. 직접 점프: 점프 목적지가 인스트럭션의 일부로 인코딩되는 경우
    1. .L1같이 점프 대상을 레이블로 프로그램 내에서 작성한다.
2. 간접 점프: 점프 대상을 레지스터나 메모리 위치로부터 읽어 들여야 하는 경우
    1. *와 메모리 오퍼랜드 중의 하나를 이용한 오퍼랜드 식별자를 합쳐서 작성한다 (ex. jump *%rax*: %rax).

# 3.6.4 점프 인스트럭션 인코딩[Jump Instruction Encodings]

어셈블리 코드에서 점프 목적지는 심벌 레이블을 사용해 작성한다.

점프를 인코딩하는 방법 두 가지: 

1. PC 상대적 방법 (PC relative)
    1. 가장 일반적인 점프 인코딩 방식이다. 대상 인스트럭션과 점프 인스트럭션 바로 다음에 오는 인스트럭션 주소와의 차이를 인코딩한다.
2. 절대 주소
    1. 대상을 직접 명시하기 위해 4바이트를 사용한다.

어셈블리와 링커는 점프 목적지를 인코딩하는 방법을 적절히 선택한다.

# 3.6.5 조건부 분기를 조건제어로 구현하기[Implementing Conditional Branches with Conditional Control]

<div align="center">
 <img src="/assets/ch3_a_assembly/c_and_goto.png" width="600"/>
 <br>
 <br>
 </div>

C에서 조건부 수식과 문장을 기계어 코드로 번역하는 가장 일반적인 방법은 조건부 및 무조건 점프를 함께 사용하는 것이다. 

어셈블리 코드에서의 무조건 점프와 유사하도록 C의 goto문을 사용한다. 

<div align="center">
 <img src="/assets/ch3_a_assembly/c_goto_assembly.png" width="600"/>
 <br>
 <br>
 <sub>조건문의 컴파일한 부분</sub>
 </div>


<div align="center">
 <img src="/assets/ch3_a_assembly/conditional_move_ex.png" width="600"/>
 <br>
 <br>
 <sub>조건문의 다른 예시이다</sub>
 </div>

# 3.6.6 조건부 이동으로 조건부 분기 구현하기[Implementing Conditional Branches with Conditional Moves]

조건이 만족하면 프로그램의 한 가지 실행경로를 따르고, 아닌 경우 다른 경로를 따라가도록 하는 제어의 조건부 전환은 간단하지만, 일반적인 프로세서들에는 매우 비효율적일 수 있다.

조건부 전송을 이용한다면, 조건부 동작의 산출물을 모두 계산하고 조건에 따라 하나만 선택하는 방식도 있다. 이 방법은 최신 프로세서의 성능 특성과 잘 일치하는 간단부 조건 이동 move 인스트럭션으로 구현될 수 있다. 

프로세서들은 각 인스트럭션을 일련의 단계로 처리하며, 이 단계들은 작은 부분만을 실행하는 파이프라인을 통해 높은 성능을 얻는다.  

프로세서가 조건부 점프를 만나면, 프로세서는 분기 조건에 대한 계산이 완료될 때까지 분기의 방향을 결정할 수 없다. 

만약 잘못 예측한다면 인스트럭션들을 위해 이미 실행한 작업 결과들을 버려야 할 수 있고, 정확한 위치에서 다시 인스트럭션들을 파이프라인에 채우는 작업을 해야 한다. 

<div align="center">
 <img src="/assets/ch3_a_assembly/conditional_move_instruction.png" width="600"/>
 <br>
 <br>
 <sub>조건부 이동 move 인스트럭션</sub>
 </div>

<div align="center">
 <img src="/assets/ch3_a_assembly/cmov.png" width="600"/>
 <br>
 <br>
 <sub>위는 x86-64로 가능한 조건부 이동 move 인스트럭션이다.</sub>
 </div>

- move 인스트럭션의 결과는 조건 코드 값에 따라 달라진다.
- 소스 값은 메모리나 소스 레지스터로부터 읽히지만, 목적지에는 명시된 조건이 만족될 때만 복사한다.
- 어셈블러는 목적지 레지스터의 이름으로부터 조건부 이동 인스트럭션의 오퍼랜드 길이를 추정한다.
- 동일한 인스트럭션 이름이 모든 오퍼랜드 길이에 대해 사용될 수 있다.
- 프로세서는 테스트의 결과를 예측하지 않고서도 조건부 이동 인스트럭션을 실행할 수 있다.

<div align="center">
 <img src="/assets/ch3_a_assembly/conditional_branch_ex.png" width="600"/>
 <br>
 <br>
 <sub>분기점 예시 코드 </sub>
 </div>
 
# 3.6.7 반복문[Loops]

C에서 do-while, while, for 과 같은 여러ㅊ가지 반복문 구문을 제공한다. 기계어에는 여기에 대응하는 인트스럭션은 없다. 그 대신 조건부 테스트와 점프를 함께 사용해서 반복문의 효과를 구현한다. 

### “Do-While”문 C 코드

```c
//do-while
do 
	Body
	while (Test) ;
```

### “Do-While”문의 Goto 버전

```c
loop:
	Body
	if (Test)
		goto loop
```

### While 문

```c
while (test-expr)
	Body
```

- while문은 test-expr을 먼저 계산해서, Body가 실행되기전 종료될수 있다는 점에서 do-while문과 다르다.

### While 문의 Goto 버전

```c
	goto test;
loop:
	Body
test:
	t = test-expr;
	if (t)
		goto loop;
```

- 루프 이전의 goto 테스트가 result나 n 값을 변경하기 전에 n에 대한 테스트를 먼저 수행하도록 한다

### For 문

```c
for (init-expr; test-expr; update-expr)
	Body
```

- 먼저 초기화 수식인 init-expr을 계산한다. test-expr이 테스트 조건이자 루프에 들어가는 부분이다. 테스트가 실패하면 루프에서 빠져나오고 Body를 실행하고 update-expr을 실행하게 된다.

# 3.6.7 Switch문[Switch Statements]

<div align="center">
 <img src="/assets/ch3_a_assembly/switch_statements.png" width="600"/>
 <br>
 <br>
 <sub>Switch 문 예시</sub>
 </div>

Siwtch문은 정수 인덱스값에 따라 다중 분기 기능을 제공하므로, 테스트해야 할 때에 수가 많은 경우에 제일 유용하다. 

점프 인스트럭션의 목적지를 찾아내기 위해 switch문의 인덱스를 사용하여 점프 테이블을 배열처럼 참조한다. 

점프테이블은 array처럼 사용되기 때문에 어디로 점프할지 효율적으로 찾을 수 있다. 

<div align="center">
 <img src="/assets/ch3_a_assembly/jump_and_switch.png" width="600"/>
 <br>
 <br>
 <sub>점프테이블과 switch문 예시</sub>
 </div>

점프테이블을 사용하면 다중분기를 매우 효율적인 방법으로 구현할 수 있게 된다.