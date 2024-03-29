---
layout: archive
title:  "[Study] Operating System - Process, Thread"
date:   2023-08-16 00:05:07 +0900
categories: 
    - Study
---

## 프로세스
- 메모리 상에서 실행 중인 프로그램입니다.
- 디스크로부터 메모리에 적재되어 CPU의 할당을 받습니다. 운영체제로부터 주소 공간, 파일, 메모리 등을 할당받습니다.
- 코드 영역: 프로그램의 소스 코드 자체를 구성하는 메모리 영역
- 데이터 영역: 전역변수, 정적변수, 배열 등에 대한 메모리 영역 (초기화 데이터는 data 영역에, 초기화되지 않은 데이터는 bss 영역에 저장)
- Heap 영역: 동적 할당 시 사용하는 (`new()`, `malloc()`) 영역입니다.
- Stack 영역: 지역변수, 매개변수, 리턴 값 (임시 메모리 영역)
- 함수의 매개변수, 복귀 주소와 로컬 변수와 같은 임시 자료를 갖는 프로세스 스택, 전역 변수들을 수록하는 데이터 섹션을 포함합니다.
- 하나의 프로세스가 생성될 때, 기본적으로 하나의 스레드가 같이 생성됩니다.

### 프로세스의 5가지 상태
- 생성 (New): 프로세스의 생성 상태
- 실행 (Running): 프로세스가 CPU에 할당되어 실행 중인 상태
- 준비 (Ready): 프로세스가 CPU에 할당되는 것을 기다리는 상태
- 대기 (Waiting): 보류(Block) 상태라고도 하며, 프로세스가 입출력이나 이벤트를 기다리는 상태
- 종료 (Terminated): 프로세스 종료 상태

### 멀티프로세스
- 하나의 프로그램을 여러 개의 프로세스로 구성하여 각 프로세스가 병렬적으로 작업을 수행하도록 하는 것입니다.
- 장점: 메모리 침법 문제를 OS 차원에서 해결하기 때문에 안전성이 높습니다.
- 단점: 각각의 독립된 메모리 영역을 가지고 있기 때문에, 작업량이 많을수록 오버헤드가 발생합니다. Context Switching으로 인한 성능 저하가 발생합니다.

## 스레드
- 프로세스의 실행 단위로, 한 프로세스 내에서 동작되는 여러 실행 흐름입니다. 프로세스 내의 주소 공간이나 자원을 스레드 간에 공유할 수 있습니다.
- 스레드는 독립적인 동작을 수행하기 위해 존재하며, 독립적인 실행 흐름을 추가하기 위한 최소 조건으로 독립된 Stack 영역만 따로 할당받습니다.
- 스레드는 스레드 ID, 프로그램 카운터, 레지스터 집합, 스택으로 구성됩니다.
- 같은 프로세스에 속한 다른 스레드와 코드, 데이터 섹션, 열린 파일, 신호 등의 운영체제 자원들을 공유합니다.
- 멀티 스레딩: 하나의 프로세스를 다수의 실행 단위로 구분하여 자원을 공유하고, 자원의 생성과 관리의 중복성을 최소화하여 수행능력을 향상시킵니다.

### 스택을 스레드마다 독립적으로 할당하는 이유
- 스택은 함수 호출 시 전달되는 인자이며, 되돌아갈 주소값 및 함수 내 변수를 저장하기 위해 사용되는 메모리 공간입니다.
- 스택 메모리 공간이 독립적이라는 것은, 독립적인 함수 호출이 가능하다는 것이며 이는 독립적인 실행 흐름이 가능하다는 것입니다.
- 따라서, 독립적인 실행 흐름을 위한 최소 조건으로 독립된 스택을 할당하는 것입니다.

### PC 레지스터를 스레드마다 독립적으로 할당하는 이유
- PC 값은 스레드의 명령어 수행 위치값을 나타냅니다. 스레드는 CPU를 할당받았다가, 스케줄러에 의해 다시 선점당합니다.
- 따라서, 명령어가 연속적으로 수행되지 못하고 어느 부분까지 수행했는지 기억할 필요가 있으므로, PC 레지스터를 독립적으로 할당합니다.

### 멀티 스레드
- 장점: 공유 메모리만큼의 시간, 메모리 공간과 시스템 자원 소모가 절약되고, 전역변수와 정적 변수에 대한 자료를 공유할 수 있습니다.
- 단점: 하나의 스레드가 데이터 공간을 망가뜨리면 모든 스레드가 작동 불능 상태가 되기 때문에 안전성에 문제가 있습니다. (공유 메모리를 갖기 때문)
  (이는 뮤텍스와 세마포어를 통해 대비할 수 있습니다.)
- 스레드 간의 통신이 필요한 경우에도 별도의 자원을 이용하지 않고, Heap 영역을 통해 데이터를 주고받습니다.
- 프로세스 간 통신 방법에 비해 훨씬 간단하다는 장점이 있습니다.
- 스레드의 context switch는 캐시 메모리를 비울 필요가 없기 때문에, 프로세스 context switch보다 더 빠릅니다.
- 시스템의 throughput이 향상되고 자원 소모가 줄어들며 자연스럽게 프로그램 응답 시간이 단축됩니다.

### 멀티 스레딩의 문제
- 스레드 간 공유하는 자원에 동시에 접근하는 일이 발생하게 됩니다.
- 따라서, 동기화 작업이 필요하고, 동기화를 위해 작업 처리 순서를 컨트롤하고 공유 자원에 대한 접근을 컨트롤 해야합니다.
- 병목현상이 발생할 수 있으며, 성능이 저하될 가능성이 높습니다. 따라서, 과도한 락으로 인한 병목현상을 줄여야 합니다.
- 멀티 프로세스 방식은 하나의 프로세스가 죽더라도 다른 프로세스에는 영향을 끼치지 않지만, 멀티 스레드는 하나의 스레드가 종료될 때, 전체 스레드도 이에 대한 영향을 받을 수 있습니다.


## 프로세스 제어 블록 (Process Control Block)
- PCB는 특정 프로세스에 대한 중요한 정보를 저장하는 운영체제의 자료구조입니다.
  (관련 저장 정보 아래에 서술)
- 운영체제는 프로세스를 관리하기 위해 프로세스의 생성과 동시에 PCB를 생성합니다.
- CPU를 할당받아 작업을 처리하다가 프로세스 전환이 발생하면, 작업을 저장하고 CPU를 반환해야 하는데, 이때 진행상황을 PCB에 저장합니다.
  (즉, 앞으로 다시 수행할 대기 중인 프로세스에 대한 저장 값을 PCB에 저장합니다.)
- 다시 CPU를 할당받게 되면 이전에 종료됐던 시점부터 다시 작업을 수행합니다.
- PCB는 Linked List 방식으로 관리되어, List Head에 PCB들이 생성될 때마다 노드가 추가됩니다. 주소값으로 연결이 이루어진 연결리스트이기 때문에 삽입, 삭제가 용이합니다.
- 프로세스가 생성되면 해당 PCB가 생성되고 프로세스 완료 시 제거됩니다.

**PCB에 저장되는 정보**
- PID, 프로세스의 상태(new, ready, running, waiting, terminated)
- 프로그램 카운터: 프로세스가 다음에 실행할 명령어의 주소
- CPU 레지스터
- CPU 스케쥴링 정보: 프로세스의 우선순위, 스케줄 큐에 대한 포인터
- 메모리 관리 정보: 페이지 테이블 또는 세그먼트 테이블 등과 같은 정보 포함
- 입출력 상태 정보: 프로세스에 할당된 입출력 장치들과 열린 파일 목록
- 어카운팅 정보: 사용된 CPU 시간, 시간제한, 계정번호 등


### Context Switching
- 프로세스의 상태 정보를 저장하고 복원하는 일련의 과정으로, 동작 중인 프로세스가 대기하며 해당 프로세스의 상태를 보관하고, 대기하고 있던 다음 순번의 프로세스가 동작하며 이전에 보관했던 프로세스 상태를 복구하는 과정입니다.
- 프로세스는 각 독립된 메모리 영역을 할당받아 사용되므로, 캐시 메모리 초기화와 같은 무거운 작업이 진행될 때 오버헤드가 발생하는 문제가 있습니다. 따라서, 프로세스의 컨텍스트 스위칭이 스레드의 컨텍스트 스위칭보다 늦습니다.

- CPU가 이전의 프로세스 상태를 PCB에 보관하고, 또 다른 프로세스의 정보를 PCB에서 읽어 레지스터에 적재하는 과정입니다.  

**발생하는 경우**
- 인터럽트가 발생하는 경우
- 실행 중인 CPU의 사용 허가 시간을 모두 소모하는 경우
- 입출력을 위해 대기해야 하는 경우

## CPU Scheduling
- CPU를 잘 사용하기 위해 프로세스를 잘 배정하기 위한 방법입니다.
- 프로세스가 작업을 수행하려면, 스케줄러로부터 CPU를 할당 받아야하므로, 순서와 처리 시간을 효율적으로 정하기 위한 정책입니다.
- 조건: 오버헤드는 낮게, 사용률은 높게, 기아 현상은 낮게
- 목표  
    1. Batch System: 한 번에 하나의 프로그램만 수행하는 것을 말합니다. 따라서, 가능하면 많은 일을 수행하며, 시간보다 처리량이 중요합니다. CPU Utilization과 같은 측면을 극대화하는 것이 배치 시스템에 더 좋습니다.
    2. Interactive System: 사용자가 컴퓨터와 대화형으로 동작하기 때문에, 응답시간이 중요합니다. 응답시간을 빠르게, 대기시간은 적게 합니다.
    3. Real-time System: 일반적으로 시간이라는 제약 조건이 추가된 시스템이기 때문에, 주어진 인풋에 대한 아웃풋의 처리 시간이 중요합니다. 따라서, 시간제약 조건(Deadline)을 맞추는 것입니다.

### 프로세스의 상태 전이
- 승인 (Admitted): 프로세스 생성이 가능하여 승인.
- 스케줄러 디스패치(Scheduler Dispatch): 준비 상태에 있는 프로세스 중 하나를 선택하여 실행시키는 것.
- 인터럽트(interrupt): 예외, 입출력, 이벤트 등이 발생하여 현재 실행 중인 프로세스를 준비 상태로 바꾸고, 해당 작업을 먼저 처리하는 것.
- 입출력 또는 이벤트 대기 (I/O or Event wait): 실행 중인 프로세스가 입출력이나 이벤트를 처리해야 하는 경우, 입출력/이벤트가 모두 끝날 때까지 대기 상태로 만드는 것.
- 입출력 또는 이벤트 완료 (I/O or Event Completion): 입출력/이벤트가 끝난 프로세스를 준비 상태로 전환하여 스케줄러에 의해 선택될 수 있도록 만드는 것.

### 선점 스케줄링
- 한 프로세스가 CPU를 할당받아 실행하고 있을 때, 다른 프로세스가 CPU를 사용하고 있는 프로세스를 중지시키고 CPU를 차지할 수 있는 스케줄링 기법입니다.  
  (처리시간에 대한 예측이 어렵습니다.)
- 우선순위가 높은 프로세스를 먼저 수행할 때 유리하며, 빠른 응답 시간을 요구하는 대화식 시분할 시스템에 유용합니다.
- 많은 오버헤드를 초래합니다.
- Interrupt, I/O or Event Completion, I/O or Event Wait, Exit

Priority Scheduling
- 정적/동적으로 우선순위를 부여하여 우선순위가 높은 순서대로 처리합니다.
- 우선순위가 낮은 프로세스가 무한정 기다리는 Starvation이 발생할 수 있습니다.  
  (Aging 방법으로 Starvation 문제를 해결할 수 있음)

Round Robin
- FCFS에 의해 프로세스들이 보내지면 각 프로세스는 동일한 시간의 Time Quantum만큼 CPU를 할당받습니다.  
  (time quantum, time slice: 실행의 최소 단위 시간)
- 할당 시간(time quantum)이 크면 FCFS와 같게 되고, 작으면 컨텍스트 스위칭이 잦아져서 오버헤드가 증가합니다.

Multilevel-Queue
- 작업들을 여러 종류의 그룹으로 나누어 여러 개의 큐를 이용하는 기법입니다.
- 우선순위가 낮은 큐들이 실행하지 못하는 것을 방지하고자 각 큐마다 다른 `time quantum`을 설정해 주는 방식이 사용됩니다.
- 우선순위가 높은 큐는 작은 `time quantum`을, 우선순위가 낮은 큐는 큰 `time quantum`을 할당합니다.

Multilevel-Feedback-Queue
-  Multilevel-Queue에서 자신의 `time quantum`을 다 채운 프로세스는 밑으로 내려가고, `time quantum`을 다 채운 프로세스는 원래 큐에 그대로 있습니다.
- `time quantum`을 다 채운 프로세스는 CPU burst 프로세스로 판단됩니다.
- 짧은 작업에 유리하며, interrupt가 잦은 입출력 위주의 작업에 우선권을 줍니다.
- 처리 시간이 짧은 프로세스를 먼저 처리하기 때문에 turn around 평균 시간을 줄여줍니다.

### 비선점 스케줄링
- 이미 사용되는 CPU를 회수하지는 못하고, 끝날 때까지 기다리는 스케줄링 기법입니다.
- 따라서, 프로세스 종료 또는 I/O 이벤트가 있을 때까지 실행이 보장됩니다. 처리시간 예측이 용이합니다.
- 일괄 처리방식이 적합합니다.
- 모든 프로세스 요구에 대해 공정하며, 중요도가 높은 작업이 낮은 작업을 기다리는 경우가 발생할 수도 있습니다.
- I/O or Event Wait, Exit

FCFS (First Come First Served)
- 큐에 도착한 순서대로 CPU를 할당합니다.
- 실행 시간이 짧은 작업이 뒤로 가면 평균 대기시간이 길어집니다.

SJF (Shortest Job First)
- 수행시간이 가장 짧다고 판단되는 작업을 먼저 수행합니다.
- FCFS 보다 평균 대기 시간이 감소하고, 짧은 작업에 유리합니다.

HRN (Highest Response-ratio Next)
- 우선순위를 계산하여 점유 불평등을 보완하는 방법으로, SJF의 단점을 보완합니다.
- 우선순위 = (대기시간 + 실행시간) / 실행시간


## Deadlock(교착상태)
- 둘 이상의 프로세스가 다른 프로세스가 점유하고 있는 자원을 기다리기 위해 무한 대기에 빠지는 상황을 의미합니다.

### 발생조건
이 중 하나라도 만족하지 않으면 교착상태는 발생하지 않습니다.
- 상호배제(Mutual exclusion): **한 번에 프로세스 하나만 해당 자원을 사용할 수 있다**. 사용 중인 자원을 다른 프로세스가 사용하려면 요청한 자원이 해제될 때까지 기다려야 한다.
- 점유대기(Hold and Wait): 자원을 최소한 하나 보유하고, 다른 프로세스에 할당된 자원을 점유하기 위해 대기하는 프로세스가 존재해야 한다.
- 비선점(No preemption): 프로세스가 어떤 자원의 사용이 끝날 때까지 그 자원을 뺏을 수 없습니다.
- 순환대기(Circular wait): Hold and Wait 관계의 프로세스들이 서로를 기다립니다.

### 교착상태 방지방법
- 예방(Prevention): 할당 구조 측면에서, 교착상태가 발생할 수 있는 요구조건을 만족시키지 않도록 함으로써 교착상태를 방지합니다.
- 회피(Avoidance): 교착상태가 발생할 가능성이 있는 자원을 할당(unsafe allocation)하지 않습니다. (대표적으로, 은행원 알고리즘, 자원 할당 그래프)
- 탐지 및 회복(Detection and Recovery): 교착상태가 발생할 수 있도록 놔두고 교착상태가 발생할 경우 이를 찾아내어 고칩니다.

**예방**
- 상호 배제 조건 방지: 한 번에 여러 프로세스가 공유 자원을 사용할 수 있도록 합니다.
- 점유 대기 조건 방지
  - 프로세스 대기를 없애기 위해서 프로세스가 실행되기 전에 필요한 모든 자원을 할당합니다. (자원 낭비 발생)
  - 자원을 점유하지 않고 있을 때에만 다른 자원을 요청할 수 있도록 합니다. 한 프로세스가 추가적인 자원을 필요로 하면, 자신의 자원을 모두 해제하여 빈상태에서 다시 요청합니다. (기아상태가 될 수 있음)
- 비선점 조건 방지: 이미 다른 프로세스에 할당된 자원에 대해 선점권이 없다면, 높은 우선순위의 프로세스가 해당 자원을 선점할 수 있도록 합니다.
- 순환대기 조건 방지: 자원을 순환 형태로 대기하도록 하지 않고, 일정한 방향으로 자원을 요구할 수 있도록 합니다. (임의로 순서를 부여)

**회피**
- 시스템의 프로세스들이 데드락을 발생시키지 않도록 요청되는 자원을 모두에게 할당할 수 있다면, `안정 상태(safe state)`에 있다고 말합니다.
- 프로세스들에게 자원을 할당하고 실행 및 종료 등의 작업을 할 때, 데드락을 발생시키지 않게 하는 특정한 순서를 찾는다면, 이를 `안전 순서(safe sequence)`라고 합니다.
- 데드락은 불안정 상태에 있을 때 발생합니다. 따라서, 불안정 상태가 데드락보다 더 큰 집합입니다.
- 회피 알고리즘은 자원을 할당한 후에도 시스템이 항상 Safe state에 있도록 할당을 허용하게 됩니다.

**탐지 및 회복**

교착상태가 발생하도록 두고, 교착상태가 발생한다면 그 후에 해결하는 방식입니다.

탐지 기법
- Allocation, Request, Available 등으로 시스템에 데드락이 발생했는지 여부를 탐색합니다.
- 자원 할당 그래프를 통해 탐지하는 방법도 있습니다.

회복 기법
- 교착상태에 빠진 모든 프로세스를 중단시키는 방법이 있지만, 연산중이던 프로세스들도 모두 일시에 중단되므로, 작업 중인 부분 결과가 폐기될 수 있습니다.
- 프로세스를 중단시킬 때마다 탐지 알고리즘으로 데드락을 탐지하며 회복시키는 방법이 있습니다. 매번 탐지 알고리즘을 호출 및 수행해야하기 때문에 부담되는 작업일 수 있습니다.
- 프로세스에 할당된 자원을 선점하여 데드락을 해결할 때까지 해당 자원을 다른 프로세스에 할당하는 방법이 있습니다.
