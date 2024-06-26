---
title: (Dotnet 기본) CS 기본 및 닷넷 CLR의 동기화, 스레드
categories: CS
key: 20240513_02
comments: true
tags: CS기본 동기화 스레드 CLR 컨텍스트 컨텍스트스위칭 Volatile Interlocked CAS
---

CS 기본 및 닷넷 CLR의 동기화 및 스레드와 관련해 '제프리 리처의 CLR via C# 4판' 에서 설명하는 내용을 정리해 봅니다.

<!--more-->

컨텍스트 (Context)
-

프로세스를 실행하기 위한 여러가지 정보들(pointer, process state, program counter, registers) 입니다.<br/>
이러한 정보들은 **<span style="color: rgb(255, 84, 182);">PCB(process control block)에 저장</span>** 됩니다.


컨텍스트 스위칭 (Context Switching)
-

- 실행할 프로세스를 교체하기 위해 위에 언급 했던 컨텍스트 (Context)를 교체 하는 행위를 말합니다.
- 컨텍스트 스위칭은 OS 커널이 실행 합니다.

### 컨텍스트 스위칭 (Context Switching) 동작

스레드A 에서 스레드B 로 전환될때 발생 됩니다. 이때 스레드A, 스레드B가 속해 있는 프로세스가 같으면 스레드 컨텍스트 스위칭 처리가 되고,
프로세스가 다르면 프로세스 컨텍스트 스위칭 처리로 몇가지 작업이 추가로 이루어 집니다.<br/><br/>

1. 현재 실행 중인 프로세스 정보를 PCB에 업데이트 후 메모리에 저장 합니다.
2. 교체할 프로세스 정보를 메모리에서 가져와 PCB에 넣고 실행 합니다.
3. <span style="color: rgb(255, 84, 182);">상황에 따라 CPU 캐시를 비움니다. (flush)&emsp;&emsp;&emsp;&emsp;&emsp;&emsp; -> 프로세스 컨텍스트 스위칭 행위시 추가</span>
4. <span style="color: rgb(255, 84, 182);">TLB (Table Loockaside Buffer)를 비움니다.&emsp;&emsp;&emsp;&emsp;&emsp;&emsp; -> 프로세스 컨텍스트 스위칭 행위시 추가</span>
5. <span style="color: rgb(255, 84, 182);">MMU (Memory Management Unit)를 변경 합니다.&emsp;&emsp; -> 프로세스 컨텍스트 스위칭 행위시 추가</span>


메모리 배리어 (Memory barrier)
-

memory fence 라고도 불리며, 멀티 스레딩 환경에서 데이터의 일관성과 순서를 보장하기 위한 매커니즘 입니다.<br/>
CPU의 성능을 위해 컴파일러는 최적화를 통해 의도한 본질의 코드 순서와 맞지 않게 실행되어 예상치 못한 결과가 나올 수 있습니다.<br/>
이러한 문제를 해결하기 위해 컴파일러에 barrier 명령문 전 후의 **<span style="color: rgb(255, 84, 182);">메모리 연산을 순서에 맞게 실행하도록 강제</span>** 합니다.<br/>
즉 **<span style="color: rgb(255, 84, 182);">barrier 이전에 나온 연산들은 barrier 이후에 나온 연산보다 먼저 실행 되도록 보장</span>** 합니다.


동기화
-

스레드 동기화는 **<span style="color: rgb(255, 84, 182);">다수의 스레드가 공유 데이터에 동시에 접근</span>**하는 경우 데이터가 손상되는 것을 막기 위해서 사용됩니다.<br/>
동기화는 크게 3가지의 문제가 있습니다.<br/>

> **제프리 리처의 CLR via C# 4판, 29장. 단순 스레드 동기화 요소 내용 중**

1. 여러 스레드들이 접근할 가능성이 있는 데이터가 어떤 것이 있는지 확인 후 스레드 동기화 락을 획득하고 해제하는 코드를 이용하여 데이터 접근 부분을 감싸야 합니다.<br/>
이런 처리가 정상으로 동작 되는지 확인 방법은 응용프로그램을 수행하고, 스트레스 테스트를 여러 번에 걸쳐 수행해야 합니다.<br/>
(이런 테스트는 여러 개의 CPU를 가진 컴퓨터에서 수행해야 동시에 리소스에 접근할 가능성이 높아집니다.)

2. 두번째는 성능을 해칩니다. 락을 획득하고 해제하는 과정은 추가적인 함수 호출을 수반하고 CPU가 여러 스레드들 중 어떤 스레드가 처음으로 락을
   획득하도록 할 것인지를 결정해야 하는데 이러는 과정은 상당한 시간을 소요하게 됩니다.

3. 세번째는 블로킹 될때 새로운 스레드가 생성되고 다시 블로킹 되었던 스레드가 수행을 재개하는 경우 새로 생성된 스레드와 함께 스케줄링되어야 하는데
   이런 상황은 컨텍스트 스위칭이 발생 되어 성능에 나쁜 영향을 주게 됩니다.



스레드 안정성
-

> **제프리 리처의 CLR via C# 4판, 29장. 단순 스레드 동기화 요소**<br/>
> **# 클래스 라이브러리와 스레드 안정성 내용 중**

닷넷프레임워크, 닷넷 코어 이상의 **<span style="color: rgb(255, 84, 182);">FCL(Framework Class Libaray)는 모든 정적 메서드에 대해서 스레드-안정적임을 보장</span>** 합니다.<br/>
메서드를 스레드-안정적으로 만든다는 것의 의미는 반드시 동기화 락을 사용함을 의미하지는 않습니다.<br/><br/>

```cs
public static Int32 Max(Int32 val1, Int32 val2)
{
    return (val1 < val2) ? val2 : val1;
}
```

의 Max(Int32 val1, Int32 val2) 메서드는 어떠한 락도 사용하지 않지만 스레드-안정적인 메서드 입니다.<br/>
이유는 메서드 파라메터 두 개의 Int32는 값 타입이므로 복사본이 전달되기 때문에 여러 스레드가 동시에 호출해도 각 스레드에서 별개로 고유의 데이터를 가지고 작업을 수행합니다.<br/>
하지만 **<span style="color: rgb(255, 84, 182);">인스턴스 메서드의 경우 여러 스레드가 동시에 호출하는 경우 스레드에 안전하지 않습니다.</span>**


동기화 요소
-

CLR에서 동기화의 종류는 크게 **<span style="color: rgb(255, 84, 182);">'단순 동기화'</span>** 와 **<span style="color: rgb(255, 84, 182);">'복합 스레드 동기화'</span>** 가 있습니다.<br/>
- 단순 동기화 : **<span style="color: rgb(255, 84, 182);">유저 모드 동기화</span>**, **<span style="color: rgb(255, 84, 182);">커널 모드 동기화</span>**가 있습니다.
- 복합 스레드 동기화 : 단순 동기화 요소에 해당 하는 **<span style="color: rgb(255, 84, 182);">'유저 모드 동기화, 커널 모드 동기화'</span>** 를 결합하여 만드는 형식 입니다.


유저 모드 동기화
-

> **제프리 리처의 CLR via C# 4판, 29장. 단순 스레드 동기화 요소**<br/>
> **# 단순 유저 모드 동기화 요소와 커널 모드 동기화 요소 내용 중**

유저 모드 동기화는 특수한 CPU 명령을 사용해서 하드웨어적으로 동기화를 사용하는 것 입니다.<br/>
그렇기 때문에 **<span style="color: rgb(255, 84, 182);">커널 모드 동기화에 비해 훨씬 빠르게 수행</span>**되고 커널에서 스레드가 블로킹 되었는지 알지 못하기에 새로 스레드를 생성하지 않습니다.<br/><br/>

C#에서 유저 모드 동기화의 종류는 다음과 같습니다.<br/><br/>

- Monitor 클래스 (lock 키워드) [[링크]](https://learn.microsoft.com/ko-kr/dotnet/api/system.threading.monitor?view=net-7.0)
- volatile 키워드 [[링크]](https://learn.microsoft.com/ko-kr/dotnet/csharp/language-reference/keywords/volatile), [[volatile 설명 링크]](https://blog.arong.info/c%23/2022/07/05/C-%EB%8F%99%EA%B8%B0%ED%99%94-%EC%84%A4%EB%AA%85-%EB%B0%8F-Volatile-%EB%8F%99%EA%B8%B0%ED%99%94.html)
- Interlocked 클래스 [[링크]](https://learn.microsoft.com/ko-kr/dotnet/api/system.threading.interlocked?view=net-7.0), [[Lock-Free, Interlocked 설명 링크]](https://blog.arong.info/c%23/2022/07/05/C-Lock-Free,-Interlocked-%EC%82%AC%EC%9A%A9-(CAS).html)

**<span style="color: rgb(255, 84, 182);">유저 모드 동기화는 커널에서 알 수 없기 때문에</span>** 스레드가 락 획득을 위해 리소스를 반복적으로 요구할 경우 **<span style="color: rgb(255, 84, 182);">CPU 시간을 낭비</span>**할 수 있습니다.<br/>
**<span style="color: rgb(255, 84, 182);">반면 커널 모드 동기화는 OS가 자체적으로 제공</span>**하는 기능으로 블로킹의 스레드는 CPU 시간을 더이상 낭비 하지 않도록 스레드를 중단 시킬 수 있습니다.<br/><br/>

따라서 **<span style="color: rgb(255, 84, 182);">커널 모드 동기화는 응용프로그램의 스레드가 운영체제의 커널이 구현하고 있는 함수를 호출</span>**해서 동기화 요소를 사용 하고, **<span style="color: rgb(255, 84, 182);">유저 모드에서 커널 모드로의 전환이 이루어져 상당한 성능 저하</span>**를 일으킵니다.<br/>
이러한 이유로 유저 모드 동기화 사용을 권장 합니다.<br/><br/>


유저 모드 동기화 요소
-

> **제프리 리처의 CLR via C# 4판, 29장. 단순 스레드 동기화 요소**<br/>
> **# 유저 모드 동기화 요소 내용 중**

CLR은 Boolean, Char, (S)Byte, (U)Int16, (U)Int32, (U)IntPtr, Single, 참조 타입의 변수에 대해서는 **<span style="color: rgb(255, 84, 182);">원자적(atomic)</span>** 으로 값을 읽고 쓸 수 있음을 보장 합니다.<br/><br/>

```cs
int x = 0;
x = 0x-1234567;
```

위 코드에서 x 변수가 0x00000000에서 0x01234567로 한 번에 변경되기 때문에 다른 스레드가 동일한 변수의 값을 조회 하였을 때, 변경 중인 상태의 값을 얻어올 가능성은 없습니다.<br/>
하지만 x 변수가 Int64로 정의 되어 있을때는 0x0123456700000000 이나 0x0000000089abcdef와 같은 값을 얻을 가능성이 있습니다. (double 타입도 동일 합니다.)
이러한 현상을 **<span style="color: rgb(255, 84, 182);">쪼개어진 읽기(torn read)</span>** 라고 합니다.


### Volatile 동기화 요소

> **제프리 리처의 CLR via C# 4판, 29장. 단순 스레드 동기화 요소**<br/>
> **# Volatile 동기화 요소 내용 중**

> [[volatile 설명 참고]](https://blog.arong.info/c%23/2022/07/05/C-%EB%8F%99%EA%B8%B0%ED%99%94-%EC%84%A4%EB%AA%85-%EB%B0%8F-Volatile-%EB%8F%99%EA%B8%B0%ED%99%94.html)

Volatile 동기화 요소는 **<span style="color: rgb(255, 84, 182);">원자적으로 변수의 값을 읽고 쓸 수 있게</span>** 해줍니다.<br/>
이런 상황이 필요한 이유는 앞서 위에서 메모리 배리어 (Memory barrier)에 대해 설명한 컴파일러에 의한 최적화 때문 입니다.<br/>
다음과 같은 코드가 있습니다.<br/><br/>

```cs
private bool _isBusy = true;

private async void StartWorker()
{
  Task.Run(this.Worker);
  await Task.Delay(10);
  // 스레드 종료
  _isBusy = false;
}

private void Worker()
{
  int count = 0;
  while (_isBusy)
  {
    count++;
  }
  // 결과 출력
  Console.WriteLine($"count : {count}");
}
```

코드는 별도 스레드로 어떠한 작업을 처리한 후 약 0.01s 후 결과를 출력하는 단순 예제 입니다.<br/>
하지만 최적화 옵션을 키거나 RELEASE 모드로 컴파일 하면 다음과 같이 코드가 최적화 됩니다.<br/><br/>

```cs
private void Worker()
{
  if(_isBusy == false)
  {
    Console.WriteLine($"count : {count}");
  }
  else
  {
    int count = 0;
    while(true)
    {
      ...
    }
  }
}
```

이유는 C# 컴파일러, JIT 컴파일러가 코드를 최적화 과정에 _isBusy필드 값은 Worker() 메서드 내부에서 while 루프의 조건 구문 말고는 접근하거나 변경하는 코드가 없기 때문에
최적화를 거쳐 무한루프 코드를 만들어 내게 됩니다.<br/><br/>

이런 상황을 수정하려면 **<span style="color: rgb(107, 173, 222);">System.Threading.Volatile</span>** 정적 클래스를 사용할 수 있습니다.<br/>
그리고 클래스에는 Write()와 Read() 두 개의 정적 메서드를 제공 함을 알 수 있습니다.<br/><br/>

- **<span style="color: rgb(107, 173, 222);">System.Threading.Volatile.Write()</span>** : 이 메서드를 호출한 위치에서 그 값이 반드시 쓰여질 것임을 보장 합니다.<br/>
코드의 순서상 이 코드를 호출한 위치보다 앞쪽에서 수행된 로드(load) / 스토어(store) 과정은 반드시 이 코드보다 앞서 수행될 것임을 보장합니다.

- **<span style="color: rgb(107, 173, 222);">System.Threading.Volatile.Read()</span>** : 이 메서드를 호출한 위치에서 그 값이 읽혀일 것임을 보장 합니다.<br/>
코드의 순서상 이 코드 이후에 위치한 로드(load) / 스토어(store) 과정은 반드시 이 메서드가 수행된 이후에 수행될 것임을 보장합니다.

> 로드(Load) : 메모리에서 CPU로 값을 가져오는 작업<br/>
> 스토어(Store) CPU에서 메모리로 값을 저장하는 작업

```cs
private bool _isBusy = true;

private async void StartWorker()
{
  Task.Run(this.Worker);
  await Task.Delay(10);
  // 스레드 종료
  // _isBusy 값이 false로 바뀌기 전에 스레드 실행 보장 (Worker)
  Volatile.Write(ref _isBusy, false);
}

private void Worker()
{
  int count = 0;
  // _isBusy 값을 먼저 가져온 다음에 비교하는 것을 보장
  while (Volatile.Read(ref _isBusy) == true)
  {
    count++;
  }
  // 결과 출력
  Console.WriteLine($"count : {count}");
}
```

### Interlocked 요소

> **제프리 리처의 CLR via C# 4판, 29장. 단순 스레드 동기화 요소**<br/>
> **# Interlocked 요소 내용 중**

> [[Lock-Free, Interlocked 설명 참고]](https://blog.arong.info/c%23/2022/07/05/C-Lock-Free,-Interlocked-%EC%82%AC%EC%9A%A9-(CAS).html)

**<span style="color: rgb(107, 173, 222);">System.Threading.Interlocked</span>** 클래스도 **<span style="color: rgb(107, 173, 222);">System.Threading.Volatile</span>** 정적 클래스와 동일하게
원자적 읽기와 쓰기를 수행할 수 있고 메모리 배리어 (Memory barrier) 기능을 제공 합니다.<br/>
하지만 **<span style="color: rgb(107, 173, 222);">System.Threading.Volatile</span>** 정적 클래스와 다른점은 **<span style="color: rgb(255, 84, 182);">여러 스레드에 의한 동시 수정을 방지하지 않기에</span>** 스레드 경합 상태를 방지할 수 없습니다.<br/>
추가 설명으로 원자적 읽기와 쓰기 기능을 제공 하는 것은 두개의 값을 비교 및 교체 하는 처리를 원자적(Atomic)으로 비교(read) 하여 교체(write) 그리고 Increment / Decrement 연산 하는 것을 말합니다.<br/><br/>

```cs
static void Work1()
{
     for (int i = 0; i < 1000000; i++)
        number++;
}
```

위 코드는 아래와 같은 순서로 연산 됩니다.<br/><br/>

```cs
static void Work1()
{
    for (int i = 0; i < 1000000; i++)
    {
    	int temp = number;
      temp += 1;
      number = temp;
    }
        
}
```

이렇게 3단계로 나눠서 연산이 진행 되는것은 멀티 스레드 환경에서는 문제가 발생되는데 **<span style="color: rgb(107, 173, 222);">System.Threading.Interlocked</span>** 클래스 사용으로 원자적 처리를 해서 해결 할 수 있습니다.<br/><br/>

**[멀티 스레드 환경에서 문제 되는 코드]**
```cs
static int number = 0;

static void Thread_1()
{
    for (int i = 0; i < 1000000; i++)
        number += 1;
}

static void Thread_2()
{
    for (int i = 0; i < 1000000; i++)
        number -= 1;
}
```

**[안전한 코드]** <br/>
```cs
static int number = 0;

static void Thread_1()
{
    for (int i = 0; i < 1000000; i++)
        Interlocked.Increment(ref number);
}

static void Thread_2()
{
    for (int i = 0; i < 1000000; i++)
        Interlocked.Decrement(ref number);
}
```

커널 모드 동기화 요소
-

> **제프리 리처의 CLR via C# 4판, 29장. 단순 스레드 동기화 요소**<br/>
> **# 커널 모드 동기화 요소 내용 중**

윈도우 운영체제는 스레드 동기화를 위한 몇 가지 요소를 제공하고 있습니다.<br/>
커널 모드 동기화 요소는 유저 모드 동기화 요소에 비해서 상당히 느린 편 입니다.<br/>
그 이유는 **<span style="color: rgb(255, 84, 182);">커널 모드 동기화 요소들이 운영체제에게 스레드 간의 동기화를 요청</span>**하는 것이기 때문입니다. 그리고 **<span style="color: rgb(255, 84, 182);">각각의 메서드들은 커널 객체를 이용</span>**하고 이로 인해 스레드가
**<span style="color: rgb(255, 84, 182);">관리 코드에서 네이티브 사용자 코드를 거쳐 네이티브 커널 모드 코드로까지 전환</span>**되었다가 다시 역순으로의 전환을 반복할 수 밖에 없기 때문입니다.<br/><br/>

C#에서 커널 모드 동기화의 종류는 다음과 같습니다.<br/><br/>

- Mutex 클래스 [[링크]](https://learn.microsoft.com/ko-kr/dotnet/api/system.threading.mutex?view=net-7.0)
- Semaphore 클래스 [[링크]](https://learn.microsoft.com/ko-kr/dotnet/api/system.threading.semaphore?view=net-7.0)
- 이벤트
  - AutoResetEvent 클래스 [[링크]](https://learn.microsoft.com/ko-kr/dotnet/api/system.threading.autoresetevent?view=net-7.0)
  - ManualResetEvent 클래스 [[링크]](https://learn.microsoft.com/ko-kr/dotnet/api/system.threading.manualresetevent?view=net-7.0)
  - CountdownEvent 클래스 [[링크]](https://learn.microsoft.com/ko-kr/dotnet/api/system.threading.countdownevent?view=net-7.0)


CAS(Compare And Swap) Algorithm
-

CAS 알고리즘은 **<span style="color: rgb(255, 84, 182);">락 기능을 구현</span>**하기 위해 만든 **<span style="color: rgb(255, 84, 182);">비교와 교환 연산이 합처진 연산 알고리즘</span>** 입니다.<br/><br/>

```cs
int CompareAndSwap(ref int var, int newVal, int expectedVal)
{
    int curVar = var;
    if (var == expectedVal)
    {
        var = newVal;
    }
    return curVar;
}

```

이때 핵심은 값을 변경하는 시점에 변경할 변수가 기대하는 값과 같으면 새로운 값으로 교체하는 것입니다.<br/>
또한 **<span style="color: rgb(255, 84, 182);">lock 처리로 인한 Blocking이 없다는 것</span>**입니다. 이를 활용하여 **<span style="color: rgb(255, 84, 182);">다른 스레드를 Block 시키지 않고 동기화를 구현</span>**할 수 있습니다.<br/>
C# 에서는 값을 비교하고 교환 연산을 원자적으로 처리하는 **<span style="color: rgb(107, 173, 222);">System.Threading.Interlocked</span>** 클래스를 사용해서 쉽게 구현할 수 있습니다.<br/>


ABA Problem
-

추가적으로 CAS를 사용하는 Lock-free 알고리즘이 가지고 있는 고질적인 문제가 있습니다.<br/>
공유 객체의 변경을 감지하지 못하는 현상을 말하는데 **<span style="color: rgb(255, 84, 182);">CAS 연산에 메모리 주소 혹은 레퍼런스를 사용할때 메모리가 재사용</span>** 되는 경우에 발생하게 됩니다.

<br/>


Reference
-

- [https://velog.io/@yarogono/C-유저-모드와-커널-모드-동기화-C-코드로-알아보기](https://velog.io/@yarogono/C-%EC%9C%A0%EC%A0%80-%EB%AA%A8%EB%93%9C%EC%99%80-%EC%BB%A4%EB%84%90-%EB%AA%A8%EB%93%9C-%EB%8F%99%EA%B8%B0%ED%99%94-C-%EC%BD%94%EB%93%9C%EB%A1%9C-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0)
- [https://velog.io/@cedongne/C-Interlocked.Exchange%EC%99%80-CAS](https://velog.io/@cedongne/C-Interlocked.Exchange%EC%99%80-CAS)
- [https://velog.io/@yarogono/C-Interlocked에-대해-알아보자](https://velog.io/@yarogono/C-Interlocked%EC%97%90-%EB%8C%80%ED%95%B4-%EC%95%8C%EC%95%84%EB%B3%B4%EC%9E%90)
- [[C#] Interlocked.Exchange와 CAS algorithm, ABA problem](https://velog.io/@cedongne/C-Interlocked.Exchange%EC%99%80-CAS#%C2%B7-aba-problem)


***



{% include content_adsense.html %}
