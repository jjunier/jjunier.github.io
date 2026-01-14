---
title: "[6기] 데브코스 DE WIL 01 | 자료구조/알고리즘"
tags:
    - DevCourse
    - Data Engineering
    - Data Structure
    - Algorithms
date: "2025-04-09"
thumbnail: "/assets/img/thumbnail/devcourse.png"
bookmark: true
---

## 이번 주 학습 목표
---
- 자료구조와 알고리즘의 **필요성과 함께 각각의 특성을 이해**한다.
- 코딩 테스트 대비를 위한 **기본 자료구조/알고리즘 개념을 정리**한다.
- 단순 암기가 아닌 **문제 상황에 맞는 자료구조 선택 기준을 정립**한다.

## 왜 자료구조를 알아야 하는가?
---
먼저, Python 데이터 타입에는 **문자열(str), 리스트(list), 사전(dict), 순서쌍(tuple), 집합(set)** 등이 존재한다. 해당 데이터 타입들은 다음과 같이 사용 가능하다.

- 문자열(str): "This is a String"
- 리스트(list): [5, 9, 2, 7]
- 사전(dict): {'a': 6, 'bc': 4}
- 순서쌍(tuple): (5, 2, 7)
- 집합(set): set([1, 2, 3])

그렇다면 Python에서 기본적으로 제공하는 데이터 타입들을 가지고도 대부분의 문제들을 해결할 수 있을 것 같은데, 왜 자료구조를 알아야 할까? 이 질문에 답을 알기 위해 자료구조와 알고리즘이 무엇인지 우선 짚고 답해본다.

## 자료구조와 알고리즘이란?
---
자료구조의 사전적 정의는 다음과 같다. **어떤 데이터에 대하여 행할 수 있는 연산들이 존재하는 무엇인가의 구조**다. 정의에서 언급된 연산의 종류에는 **최댓값 찾기, 새로운 원소 삽입, 원소 삭제 등**이 있다.

그리고 알고리즘이란 **어떠한 문제들을 해결하기 위한 절차, 방법 그리고 명령어들의 집합을 일컫는 말**이다.

프로그래밍을 진행하다 보면 다방면에서 문제들을 직면하곤 한다. 이에 해결하고자 하는 문제에 따라(응용 종류와 범위에 따라) 최적의 해법은 천차만별이다. 이에 **정답에 근접한 선택을 어떻게 해야 할지 알기 위해서 자료구조를 이해**해야 한다.

## 자료구조 1 | 선형 배열(Linear Array)
---
선형 배열은 원소들을 순서대로 늘어 놓은 것으로, 여기서 말하는 **순서는 인덱스(index)**로 표현하며 0부터 시작한다는 특징이 있다. 

Python에서는 배열처럼 데이터를 줄세우는 데이터 타입인 **리스트(list)**가 존재한다.

![선형 배열](/assets/img/contents/linear_list.png "Linear Array")

![리스트 타입](/assets/img/contents/python_list.png "List")

### 리스트에 활용 가능한 연산들
---
1. 리스트 길이와 관계없이 빠른 실행 결과를 보이는 연산들 
    (**O(1)**: 리스트 길이와 관계없이 무관한 상수 시간)
- `append()`: 원소를 가장 뒤쪽에 덧붙인다.
- `pop()`: 원소를 꺼낸다.

2. 리스트 길이에 비례한 실행 시간이 걸리는 연산들
    (**O(n)**: 리스트 길이에 비례한 실행 시간)
- `insert()`: 특정 위치에 원소 삽입한다.
- `del()`: 특정 위치에 원소 삭제한다.
- `index()`: 특정 원소의 위치를 탐색하여 인덱스 번호를 반환한다.

## 배열 기반의 정렬과 탐색
---
**정렬(sort)**
복수의 원소로 주어진 데이터를 정해진 기준에 따라 새로 줄세우는 작업을 말한다.

파이썬 내장 함수(built-in function): `sorted()` -> 정렬된 새로운 리스트를 얻어낸다.
리스트에 쓸 수 있는 메서드(method): `.sort()` -> 해당 리스트를 정렬한다.

**키를 이용한 정렬 (lambda 기반)**

```python
L = [{'name': 'John', 'score': 83}, 
    {'name': 'Paul', 'score': 92}]

L.sort(key = lambda x: x['score'], reverse = True)
```

**선형 탐색(Linear Search)**
타겟 번호를 주어진 리스트 내의 인덱스 번호 순으로 인자와 비교하여 찾는 방법을 말한다.
-> 리스트 길이에 비례하는 시간(O(n))이 소요된다.

```python
def linear_search(L, x):
    i = 0
    while i < len (L) and L(i) != k:
        i += 1
    if i < len(L):
        return i
    else:
        return -1
```

**이진 탐색(Binary Search)**
탐색하려는 리스트가 이미 크기 순으로 정렬된 경우에만 사용 가능한 탐색 방법을 말한다.
-> O(log n) 시간이 소요된다.

```python
def binary_search(L, x):
    lower = 0
    upper = len(L) - 1
    idx = -1

    while lower <= upper:
        middle = (lower + upper) // 2
        if L[middle] == target:
            ...
        elif L[middle] < target:
            lower = ...
        else:
            upper = ...
```

## 재귀 함수(Recursive Function)
---
하나의 함수에서 자신을 다시 호출하는 작업을 수행하는 함수로, 많은 종류의 문제가 **재귀적(Recursively)**으로 해결이 가능하다.

![재귀 함수](/assets/img/contents/recursive_func.png "Recursive Function Example")

```python
# 재귀적(Recursive) 구조의 자연수 합 구하기
def recursively_sum(n):
    if n <= 1:
        return n
    else:
        return n + recursively_sum(n - 1)

# 반복적(iterative) 구조의 자연수 합 구하기
def iterative_sum(n):
    s = 0
    while n >= 0:
        s += n
        n -= 1
    
    return s
```

## 알고리즘의 복잡도
---
프로그래밍 코드가 얼마나 이해하기 어렵고 복잡한 정도가 아닌, **문제를 해결하는데 얼만큼의 자원을 요구하는가**를 뜻한다. 여기서 말하는 컴퓨팅 자원은 크게 **시간과 공간**으로 나뉜다.

**시간 복잡도(Time Complexity)**: 문제(일반적으로 주어지는 데이터의 집합)의 크기에 따라 걸리는 시간으로, 데이터의 크기가 커지면 해결하기까지 소요되는 시간과의 상관관계를 나타낸다.

**공간 복잡도(Space Complexity)**: 문제 해결을 위해 필요한 물리적인 메모리 공간의 크기로, 데이터를 저장하는 자체 메모리 뿐만 아니라 문제 해결을 위한 부가적인 메모리 요구량가지 포함한다.

### 시간 복잡도
---
- **평균 시간 복잡도(Average Time Complexity)**: 임의의 입력 패턴 해결에 소요되는 시간의 평균을 말한다.
- **최악 시간 복잡도(Worst Time Complexity)**: 가장 긴 시간을 소요하게 만드는 입력 패턴 해결에 소요되는 시간을 말한다.

**빅오 표기법(Big-O Notation)**
점근 표기법(Asymptotic Notation) 중 하나로, 어떤 함수의 증가 양상을 다른 함수와의 비교로 표현한다.

- 입력의 크기: n
    - O(1): 입력된 크기와 상관없이 한 번의 실행 시간이 소요된다.
    - O(log n): 입력된 크기의 비례하는 시간이 소요된다.
    - O(n): 입력된 크기에 비례하는 시간이 소요된다.
    - 그 외: O(n^2), O(2^n)


## 자료구조 2 | 연결 리스트(Linked List)
---
연결 리스트는 **추상적 자료구조(Abstract Data Structures)** 중 하나로, 자료 구조의 내부 구현은 숨겨두고 밖에서 보이는 것들인 **데이터와 연산을 제공하는 구조**이다.

여기서 말하는 데이터는 **정수, 문자열, 레코드 등의 타입**을 의미하며, 연산은 삽입, 삭제, 순회, 정렬, 탐색 등이 있다.

기본적인 연결 리스트의 구성은 **노드(Node)**라고 칭하며, **Data와 Link(Next)**로 이루어져 있다. Node 내의 데이터는 정수형뿐만 아니라 다른 구조로 이루어질 수 있는데 문자열, 레코드, 또 다른 연결 리스트 등이 담길 수 있다.

```python
# 첫 번째 노드 정의
class Node:
    def __init__(self, item):
        self.data = item
        self.next = None

# 연결 리스트 정의: 노드 내 시작 부분(Head)과 끝 부분(Tail)
class LinkedList:
    def __init__(self):
        self.nodeCount = 0
        self.head = None
        self.tail = None
```
**배열 vs 연결 리스트**

||배열|연결 리스트|
|---|---|---|
|저장 공간|연속한 위치|임의의 위치|
|특정 원소 지칭|매우 간편함|선형 탐색과 유사함|

## 연결 리스트 연산 네 가지
---
**연산 1. k번째 특정 원소 참조**
첫 번째 연결 리스트를 이용한 연산으로는 **k(pos)번째에 위치한 Node 내 Data 자체를 리턴하는 연산**이다.

```python
def getAt(self, pos):
    if pos <= 0 and pos > self.nodeCount:
        return None
    i = 1
    curr = self.head
    
    while i < pos:
        curr = curr.next
        i += 1
    
    return curr
```

**연산 2. 원소 삽입**
pos가 가리키는 위치(1 <= pos <= nodeCount+1)에 **newNode를 삽입하는 연산**으로, 성공/실패 여부에 따라 True/False를 반환한다.

```python
def insertAt(self, pos, newNode):
    if pos < 1 or self.nodeCount + 1:
        return False
    
    if pos == 1:
        newNode.next = self.head
        self.head = newNode
    
    else:
        prev = self.getAt(pos-1)
        newNode.next = prev.next
        prev.next = newNode

    if pos == self.nodeCount + 1:
        self.tail = newNode
    
    self.nodeCount += 1
    return True
```

- **연결 리스트 원소 삽입의 시간 복잡도**
    - 맨 앞에 삽입하는 경우: O(1)
    - 중간에 삽입하는 경우: O(n)
    - 끝에 삽입하는 경우: O(1)

**연산 3. 원소 삭제**
pos가 가리키는 위치(1 <= ps <= nodeCount) **node를 삭제하는 연산**으로, node의 데이터를 반환한다.

```python
def popAt(self, pos):
    if pos < 1 or pos > self.nodeCount:
        raise IndexError
    
    if pos == 1:
        curr = self.head
        self.head = curr.next

        if self.nodeCout == 1:
            self.tail = None
    
    else:
        prev = self.getAt(pos-1)
        curr = prev.next
        prev.next = curr.next

        if pos == self.nodeCount:
            self.tail = prev
    
    self.nodeCount -= 1
    return curr.data
```

- **연결 리스트 원소 삭제의 시간 복잡도**
    - 맨 앞에 삽입하는 경우: O(1)
    - 중간에 삽입하는 경우: O(n)
    - 끝에 삽입하는 경우: O(n)

**연산 4. 두 리스트의 연결**
두 연결 리스트 L1과 L2를 이어 붙이기 위해서 연결 리스트 self 뒤에 또 다른 연결 리스트를 이어 붙이는 연산이다.

![두 연결 리스트 연결](/assets/img/contents/linkedlist_concat.png "Linked List Concatenation")

```python
def concat(self, L):
    self.tail.next = L.head
    # L.tail이 None인 경우를 방지
    if L.tail:
        self.L.tail
    self.nodeCount += L.nodeCount
```

## 자료구조 3 | 확장된 연결 리스트(Extended Linked List)
---
연결 리스트는 삽입과 삭제가 유연하다는 가장 큰 장점을 가지고 있다. 하지만, 어느 위치에 노드를 삽입하는 괒에서 삽입 위치까지 따라 가야하는 부담이 존재한다. 

이에 기존의 연결 리스트의 맨 앞에 dummy node를 추가하여 기존의 부담을 해소한다.

![확장된 연결 리스트](/assets/img/contents/extended_linkedlist.png "Extended Linked List")


```python
class LinkedList:
    def __init__(self):
        self.nodeCount = 0
        self.head = None
        self.tail = None
        self.head.next = self.tail
```

## 연결 리스트 연산 네 가지
---
**연산 1. 리스트 순회**

```python
def traverse(self):
    result = []
    curr = self.head
    while curr.next:
        curr = curr.next
        result.append(curr.data)
        
    return result
```

**연산 2. 원소의 삽입**
prev가 가리키는 node 다음에 newNode를 삽입하고 성공/실패 여부에 따라 True/False를 리턴하는 연산이다.

```python
def insertAfter(self, prev, newNode):
    newNode.next = prev.next
    if prev.next is None:
        self.tail = newNode
    prev.next = newNode
    self.nodeCount += 1
    
    return True
```

**연산 3. k번째 원소 얻어내기**

```python
def getAt(self, pos):
    if pos < 1 or pos > self.nodeCount:
        return None
    i = 0
    curr = self.head
    while i < pos:
        curr = curr.next
        i += 1
    
    return curr

def insertAt(self, pos, newNode):
    if pos < 1 or pos > self.nodeCount + 1:
        return False
    if pos != 1 and pos == self.nodeCount + 1:
        prev = self.tail
    else:
        prev = self.getAt(pos-1)
    
    return self.insertAfter(prev, newNode)
```

**연산 4. 원소의 삭제**
prev의 다음 node를 삭제하고 그 node의 데이터를 리턴하는 연산이다.

```python
def popAfter(self, prev):
    curr = prev.next
    if curr is None:
        return None
    prev.next = curr.next
    if curr.next is None:
        self.tail = prev
    self.nodeCount -= 1

    return curr.data

def popAt(self, pos):
    if pos < 1 or pos > self.nodeCount:
        raise IndexError
    prev = self.getAt(pos-1)

    return self.popAfter(prev)
```


**두 리스트의 연결**

```python
def concat(self, L):
    self.tail.next = L.head.next
    if L.tail:
        self.tail = L.tail
    self.nodeCount += L.nodeCount
```
## 자료구조 4 | 양방향 연결 리스트(Doubly Linked List)
---
기존의 연결 리스트는 한 쪽으로만 링크를 연결하였지만, **앞(다음 node)과 뒤(이전 node)로 각각 양쪽으로 연결**하여 진행 및 연산이 가능한 구조를 말한다.

![양방향 연결 리스트](/assets/img/contents/doubly_linkedlist.png "Doubly Linked List")

```python
class DoublyLinkedList:
    def __init__(self, item):
        self.nodeCount = 0
        self.head = None
        self.tail = None
        self.head.prev = None
        self.head.next = self.tail
        self.tail.prev = self.head
        self.tail.next = None
```

## 자료구조 5 | 스택(Stack)
---
스택은 자료(element)를 보관할 수 있는 선형 구조로써, **데이터 삽입(push 연산)** 시에는 안 쪽 끝에서 밀어 넣어야 하는 제약사항이 있는 자료구조이다. **데이터 삭제(pop 연산)** 시에는 같은 쪽을 뽑아 꺼내야만 한다. 이러한 구조를 **후입 선출(LIFO; Last-In First-Out)**이라고 칭한다.

**스택에서 발생하는 오류**
1. 비어 있는 스택에서 데이터 원소를 꺼내는 경우
    -> `스택 언더플로우(Stack Underflow)`
2. 꽉 찬 스택에 데이터 원소를 넣으려는 경우
    -> `스택 오버플로우(Stack Overflow)`

### 스택의 연산 정의
---
- `size()`: 현재 스택에 담긴 데이터 원소 갯수를 반환한다.
- `isEmpty()`: 현재 스택이 비어 있는가에 대한 참/거짓을 반환한다.
- `push(x)`: 데이터 원소 x를 스택의 맨 위에 추가한다.
- `pop()`: 스택의 맨 위에 저장된 데이터 원소를 제거한다. (혹은 반환한다.)
- `peek()`:스택의 맨 위에 저장된 데이터 원소를 반환한다. (제거하지 않는다.)

**스택의 추상적 자료구조 구현 01 - 배열**
Python 리스트와 메서드들을 이용하여 스택을 구현할 수 있다.

```python
class ArrayStack:
    # 빈 스택 초기화하는 함수
    def __init__(self):
        self.data = []
    # 스택의 크기를 반환하는 함수
    def size(self):
        return len(self.data)
    # 스택이 비어 있는지 판단
    def isEmpty(self):
        return self.size() == 0
    # 데이터 원소 추가
    def push(self.item):
        self.data.append(item)
    # 데이터 원소 삭제 후 반환
    def pop(self):
        return self.data.pop()
    # 스택의 꼭대기 원소 반환
    def peek(self):
        return self.data[-1]
```

**스택의 추상적 자료구조 구현 02 - 연결 리스트**
양방향 연결 리스트를 기반으로 하여 스택을 구현할 수 있다.

```python
class LinkedListStack:
    def __init__:
        self.data = DoublyLinkedList()
    def size(self):
        return self.data.getLength()
    def isEmpty(self):
        return self.size() == 0
    def push(self.item):
        node = Node(item)
        self.data.insertAt(self.size() + 1, node)
    def pop(self):
        return self.data.popAt(self.size())
    def peek(self):
        return self.data.getAt(self.size()).data
```

## 자료구조 6 | 큐(Queue)
---
스택은 자료(element)를 보관할 수 있는 선형 구조로써, **데이터 삽입(push 연산)** 시에는 안 쪽 끝에서 밀어 넣어야 하는 제약사항이 있는 자료구조이다. **데이터 삭제(pop 연산)** 시에는 반대 쪽을 뽑아 꺼내야만 한다. 이러한 구조를 **선입 선출(FIFO; First-In First-Out)**이라고 칭한다.

### 큐의 연산 정의
---
- `size()`: 현재 스택에 담긴 데이터 원소 갯수를 반환한다.
- `isEmpty()`: 현재 스택이 비어 있는가에 대한 참/거짓을 반환한다.
- `enqueue(x)`: 데이터 원소 x를 스택의 맨 위에 추가한다.
- `dequeue()`: 스택의 맨 위에 저장된 데이터 원소를 제거한다. (혹은 반환한다.)
- `peek()`:스택의 맨 위에 저장된 데이터 원소를 반환한다. (제거하지 않는다.)

**큐의 추상적 자료구조 구현 01 - 배열**
Python 리스트와 메서드들을 이용하여 스택을 구현할 수 있다.

```python
class ArrayStack:
    # 빈 큐를 초기화
    def __init__(self):
        self.data = []
    # 큐의 크기를 리턴
    def size(self):
        return len(self.data)
    # 큐가 비었는 지 판단
    def isEmpty(self):
        return self.size() == 0
    # 데이터 원소를 추가
    def enqueue(self):
        self.data.append(item)
    # 데이터 원소를 삭제(리턴)
    def dequeue(self):
        return self.pop(0)
    # 큐 맨 앞의 원소를 반환
    def peek(self):
        return self.data[0]
```

**배열로 구현한 큐의 연산 복잡도**

|연산|복잡도|
|---|---|
|`size()`|O(1)|
|`isEmpty()`|O(1)|
|`enqueue()`|O(1)|
|`dequeue()`|O(n)|
|`peek()`|O(1)|

**큐의 활용**
자료를 생성하는 작업과 그 자료를 이용하는 작업이 비동기적(asynchronously)으로 일어나는 경우 
자료를 생성하는 작업이 한 곳이 아닌 여러 곳에서 일어나는 경우 (producer)
자료를 이용하는 작업이 여러 곳에서 일어나는 경우(consumer)
자료를 생성하는 작업과 그 자료를 이용하는 작업이 양쪽 다 여러 곳에서 일어나는 경우
자료를 처리하여 새로운 자료를 생성하고, 나중에 그 자료를 또 처리해야 하는 작업의 경우