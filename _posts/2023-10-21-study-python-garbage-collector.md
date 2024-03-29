---
layout: archive
title:  "[Study] 파이썬에서는 어떻게 가비지 컬렉팅을 수행할까?"
date:   2023-10-21 00:05:07 +0900
categories: 
    - Study
---

# 가비지 컬렉터

### 메모리를 관리하는 방법
- 더 이상 사용하지 않는 메모리를 해제하지 않으면 memory leak이 발생합니다.
- 사용 중이던 메모리를 해제하면 프로그램이 중단되고 데이터가 손실될 수 있습니다.

### GC의 동작 원리
- OS가 프로그램을 프로세스로 실행하게 되면, 프로세스는 메모리에 Code, Data, Heap, Stack 영역을 할당받게 됩니다.
- 이 때, 힙과 스택 영역에 할당된 메모리들을 해제하는 동작을 수행하는 것이 GC 입니다.

## 파이썬의 GC
- 파이썬의 GC는 레퍼런스 카운팅, 제너레이셔널 가비지 컬렉션 두가지 방식으로 동작합니다.

### Reference Counting
- 어떤 객체가 참조되어 사용될 때마다 횟수를 카운팅하여 증가시키고, 해제할 때 1씩 감소시킵니다. 0이 되면 객체의 메모리 할당이 해제됩니다.
- 메모리 교체 방식에서 LRU(Least Recently Used)가 참조 횟수를 카운트하여 victim을 고릅니다.
- 레퍼런스 카운트를 증가시키는 방법은 1. 객체를 변수에 할당, 2. 객체를 리스트와 튜플과 같은 자료구조에 추가하거나, 인스턴스 프로퍼티로 추가하는 경우, 3. 함수에 파라미터로 전달하는 경우입니다.

```python
import sys
a = "Hello world!"
sys.getrefcount(a)
# a의 레퍼런스 카운트: 2

b = [a]
c = { 'key': a }
sys.getrefcount(a)
# a의 레퍼런스 카운트: 4
```
- a가 위치한 스택의 주소가 콜스택에서 참조되기 때문에 1번, `getrefcount()`에 매개변수로 전달될 때 한번 카운트가 증가됩니다.

### Generational Garbage Collection
- 어떤 동적 객체가 서로를 순환참조하고 있다면, Reference counting 방식에서는 해당 객체들이 메모리에서 해제될 수 없습니다.
- 이를 방지하기 위해, 제너레이셔널 가비지 컬렉션은 순환 참조를 탐지하고 메모리에서 해제합니다.
- 레퍼런스 카운팅과 다르게 가비지 컬렉팅은 개발자가 임계값을 직접 설정하여 동작시킬 수 있습니다.

#### 핵심 개념
- Generation: 가비지 컬렉터 객체를 0~2세대로 분리하여 관리하고, 세대가 낮을수록 더욱 자주 가비지 컬렉팅을 수행합니다. 생성된지 오래되지 않은 객체가 오래된 객체보다 해제될 가능성이 높다는 가설에 근거합니다.
- Threshold: 세대별 객체의 수가 정해진 스레스홀드를 초과하면, 임계치가 초과된 세대의 객체에 대해 수행합니다.
    
    ```python
    import gc
    gc.get_threshold() # 1. stdout: (700, 10, 10)
    gc.get_count()     # 2. stdout: (121, 9, 2)
    ```
    
    - 1번 결과에서 threshold 0이 700이라는 의미는, 0세대 객체가 700개를 초과하면 가비지 컬렉팅이 수행된다는 뜻입니다. 남은 객체들은 1세대 객체로 옮기고 1세대의 count를 1 증가시킵니다.
    - threshold 1, 2가 10이라는 의미는, 객체 수의 임계값이 아닌 가비지 컬렉팅이 발생한 횟수에 대한 임계치입니다. 따라서, 1세대는 7000번 만에, 2세대는 70000번 만에 가비지 컬렉션이 수행됩니다.

#### 가비지 컬렉션의 라이프 사이클
- 새로운 객체가 만들어질 때 파이썬은 객체를 메모리와 0 세대에 할당합니다. 만약, 0 세대의 객체 수가 threshold 0보다 크다면, `collect_generations()`를 실행하게 됩니다.
- `collection_generations()` 메서드가 호출되면 모든 세대(기본적으로 3개의 세대)를 검사하는데, 가장 오래된 세대인 2세대부터 역으로 확인합니다.
- 해당 세대에 객체가 할당된 횟수가 각 세대에 대응되는 threadhold n보다 크면 `collect()`를 호출해 가비지 컬렉션을 수행합니다.
- `collect()` 메서드는 순환 참조 탐지 알고리즘을 수행하고 특정 세대에서 도달할 수 있는 객체와 도달할 수 없는 객체를 구분하여 도달할 수 없는 객체들을 찾습니다.
- 도달할 수 있는 객체 집합은 다음 상위 세대로 합쳐지고, 도달할 수 없는 객체 집합은 콜백을 수행한 후 메모리에서 해제됩니다.

#### 순환참조 되는 경우
```python
class MyClass(object):
		pass
a = MyClass()
a.obj = a
del a
```
- 위와 같은 경우,  클래스의 인스턴스를 생성한 다음 프로퍼티로 자신을 할당했습니다.
- 이 때, 객체를 삭제하더라도 객체의 프로퍼티가 자신이기 때문에 순환참조가 발생하여 객체가 즉시 메모리에서 할당 해제되지 않습니다.
- 따라서, 실제 객체를 삭제하더라도 각 객체 내부에서 서로를 참조하기 때문에 레퍼런스가 지워지지 않아, count가 1이 됩니다.
- 순환참조는 다음과 같은 방법으로 탐지할 수 있습니다.
    1. 각 객체의 gc_refs 필드를 레퍼런스 카운트와 같은 값으로 설정합니다.
    2. 각 객체에서 참조하는 다른 컨테이너 객체를 찾고, 참조되는 컨테이너의 gc_refs를 감소 시킵니다.
    3. 어느 객체의 gc_refs가 0이 되면, 그 객체는 컨테이너 집합 내부에서 순환참조하고 있다는 뜻입니다.
    4. 해당 객체를 unreachable 객체로 구분하고 메모리에서 해제합니다.

### Generational Garbage Collection 모듈
```python
gc.get_count(): 각 세대의 객체 수를 확인합니다.
gc.set_threshold(): 세대별 임계치를 설정합니다.
gc.collect_generations()
: 모든 세대에 대해 2세대부터 0세대까지의 순서로 확인하고, 임계치 초과시 collect() 호출합니다.
gc.collect(): 가비지 컬렉션을 수행하여 순환참조 객체를 메모리에서 해제합니다.
gc.disable(): 가비지 컬렉팅을 비활성화할 수 있습니다.
```

#### GC와 프로그램 성능
- 가비지 컬렉션을 수행하려면 프로그램을 완전히 중지해야하기 때문에, 객체가 많을수록 가비지 컬렉션을 수행하기 위한 시간이 더 많이 필요합니다.
- 가비지 컬렉션 주기가 짧으면 여유 메모리를 확보할 수 있지만, 프로그램이 중단되는 상황이 많이 발생합니다. 주기가 길면 프로그램이 자주 중단되지는 않지만 메모리 공간에 가비지가 많이 쌓입니다.

## 파이썬 글로벌 메모리 관리

### 리눅스에서의 메모리 관리
- 리눅스에서 프로세스가 메모리를 할당 받으면, 프로세스가 종료되기 전까지 할당받은 메모리를 그대로 가지고 있습니다. 따라서, 프로세스에 할당되는 메모리가 시간이 지남에 따라서
- 파이썬의 경우, C언어를 기반으로 사용하는 Cpython에서는 `glibc`를 사용합니다. `glibc`는 동적 메모리 할당에 `malloc()`과 `free()` 함수를 사용합니다.
- 리눅스는 `malloc()`을 호출하면 `brk()`, `nmap()`을 사용하여 메모리를 할당하게 됩니다.
- 128KB보다 작은 경우에는 `brk()`, 보다 큰 메모리의 경우 `nmap()`을 사용합니다.

### 파이썬에서의 메모리 관리
- 파이썬은 PyMalloc을 사용하여 메모리를 할당합니다. PyMalloc은 Arena라는 라이브러리를 기반으로 동작합니다.
- Arena는 힙 메모리 영역에서 256KB를 OS로부터 할당받은 후, 64개의 4KB(x86 아키텍처 페이지 사이즈) 단위의 메모리 풀(pool)로 나뉘게 됩니다. 각 풀은 다시 512B의 블록으로 나뉘어집니다. 블럭은 객체가 저장될 공간이며, 메모리 풀은 블럭을 관리하는데 사용되고, Arena는 메모리 풀을 관리합니다.
- 객체별로 얼로케이터들은 Arena에서 순서대로 풀을 가지고 옵니다. 각 풀을 사용 가능한 블럭들을 이용해 자신의 객체들을 할당합니다. 풀에 사용 가능한 블록이 없다면 다음 풀로 넘어가고 풀이 없다면 새로운 Arena가 할당되어 Arena의 풀에 실제 객체가 할당됩니다.
- `del` 명령어를 사용해 객체를 지우고, reference count가 0이 되거나 순환참조되어 가비지 컬렉션이 진행되더라도 실제로 운영체제로 메모리가 반환되지 않는 이유는 메모리가 Arena 단위로 관리되기 때문입니다.
- Arena는 하위 64개 풀이 모두 할당되지 않은 상태여야만 운영체제로 반환됩니다.

## 참고
#### Garbage Collectiong
[Python의 Garbage Collecting](https://velog.io/@zihs0822/Python의-GC와-GIL)
[파이썬의 가비지 콜렉션에 대해서](https://devbull.xyz/python-garbace-collection/)
[Deeppago님의 블로그](https://deeppago.tistory.com/69)
#### 파이썬 메모리 관리
[파이썬의 메모리 관리](https://changmyeong.tistory.com/52)
