---
title: (C#) 동기화 설명 및 Volatile 동기화
categories: C#
key: 20220419_01
comments: true
tags: C# 동기화 Volatile 원자적 유저모드 커널모드
---

멀티 스레드 환경에서 공유 자원에 대해 동시에 서로 읽고/쓰기 처리를 하는데 있어 동기화 처리 고려를 하지 않을 수 없습니다.<br/>
스레드 동기화 처리 방법은 크게 **유저 모드**에서 처리 되는 기법과 **커널 모드**에서 처리 되는 기법 두가지로 나눌 수 있는데<br/>
이번 글에서 유저 모드 동기 처리 방식은 Volatile 동기화에 대해 알아보도록 하겠습니다.

<!--more-->

<u>Volatile 설명 전</u>에 동기화 처리에 대해 먼저 간략하게 정리해 보려고 합니다.<br/>
동기화 처리는 반드시 필요한 상황에서 처리해야 좋은 결과를 기대할 수 있지만 잘못 처리 된다면 기대치 못한 성능이 나올 수 있습니다.
아무래도 동기화 처리를 하지 않는 것이 제일 좋은 선택지 입니다. 동기화 처리는 여러가지 많은 이유들<u>(여러 스레드간 락 획득 경쟁, CPU 스레드 스케줄링 관리 등)</u><br/>
로 인해 동기화 하지 않고 처리 되는 것 보단 좋은 성능면에선 느릴 수 밖에 없습니다.<br/>

그럼 어떤 경우에서 동기화가 불필요 한지 간단히 알아보겠습니다.

동기화 처리 불필요 조건
-

위에서 말했듯 공유 자원에 대해 동시에 읽고/쓰기 수행을 할때 필요하다고 했습니다. 여기서 핵심은 <u>읽기 전용</u>인 경우 스레드가 동시에 접근해도<br/>
문제가 없기 때문에 이런 경우는 동기화 처리를 하지 않아도 됩니다. immutable 타입인 String 객체는 변경되지 않기에 여러 스레드가 동시 접근되어도 안전합니다.

여러 스레드에서 어떤 객체를 생성하였다 해도 해당 객체의 참조 값은 객체를 생성한 스레드만이 갖고 있기 때문에<br/>
여러 스레드 서로간의 데이터 접근 자체가 불필요 하다면 동기화 처리는 하지 않아도 됩니다.

Volatile 동기화
-

Volatile 동기화 요소는 유저 모드에서 처리 할 수 있는 동기화 기법입니다. Volatile외에도 **<span style="color: rgb(107, 173, 222);">System.Threading.Interlocked</span>** 동기화 방법이 있습니다.<br/>
Volatile 동기화 처리는 **<span style="color: rgb(107, 173, 222);">System.Threading.Volatile</span>** 클래스로 제공되고<br/>
Volatile.Write() / Volatile.Read() 두개의 정적 메서드를 이용해서 순서 보장을 처리 할 수 있습니다.

닷넷의 C# 컴파일러, JIT 컴파일러는 CPU 플랫폼 타겟별로 코드를 자체적으로 최적화 시키는데 최적화 결과가 의도치 않은 오류를 만들어 낼 수 있습니다.<br/>
이런 상황은 <b>**코드가 순차적으로 실행되지 않거**</b>나 혹은 <b>**컴파일러 최적화 과정에 의해서 새로운 코드가 만들어져 영향**</b>받는 문제로<br/>
**<span style="color: rgb(107, 173, 222);">System.Threading.Volatile</span>** 을 사용해서 동기화 처리가 필요합니다.

이런 상황의 단순한 예제 코드가 아래 코드와 같습니다.<br/>
코드는 별도 스레드로 어떠한 작업을 처리한 후 약 0.01s 후 결과를 출력하는 단순 예제 입니다.<br/>
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

스레드로 Worker()가 호출되고 약 0.01s 이후 loop를 탈출시키고 결과를 출력하는 의도인 코드 입니다.<br/>
컴파일 후 DEBUG 모드로 실행해보면 기대하는 의도로 정상 동작 됨을 확인할 수 있습니다.<br/>
하지만 RELEASE 모드로 컴파일 후 실행해 보거나 DEBUG에서 코드 최적화 옵션으로 컴파일 후 확인해 보면 무한 루프로 동작되고 있는걸로 확인됩니다.

이유는 C# 컴파일러, JIT 컴파일러가 코드를 최적화 과정에 _isBusy필드 값은 Worker() 메서드 내부에서 while 루프의 조건 구문 말고는 접근하거나 변경하는 코드가 없기 때문에 컴파일러는 
다음과 같이 새로운 코드를 추가로 생성합니다.

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

결국 처음 Worker() 메서드 호출 시점엔 _isBusy값은 true이기 때문에 무한 루프로 빠지게 됩니다.

이를 해결 하기 위해서 **<span style="color: rgb(107, 173, 222);">System.Threading.Volatile</span>** 의 Write()와 Read()로 무조건 _isBusy 값을 메모리에서 직접적으로 읽고 쓰도록<br/>
처리하여 해결할 수 있습니다.

다음은 수정된 코드 입니다.<br/>
```cs
private bool _isBusy = true;

private async void StartWorker()
{
  Task.Run(this.Worker);
  await Task.Delay(10);
  // 스레드 종료
  Volatile.Write(ref _isBusy, false);
}

private void Worker()
{
  int count = 0;
  while (Volatile.Read(ref _isBusy) == true)
  {
    count++;
  }
  // 결과 출력
  Console.WriteLine($"count : {count}");
}
```

Volatile 필드
-

**<span style="color: rgb(107, 173, 222);">System.Threading.Volatile</span>** 의 Write()와 Read() 메서드는 해당 순서를 파악하면서 적절히 사용해야 하는데<br/>
이를 단순화 하기 위해서 C# 에서 volatile 키워드로 간단히 처리 할 수 있습니다.

위의 예제 코드에서는 Write() / Read() 사용 대신 다음과 같이 _isBusy를 volatile 필드로 처리 하면 컴파일러 최척화시 해당 부분은 제외 됩니다.<br/>
```cs
private volatile bool _isBusy = true;
```

단 volatile 필드는 CLS(Common Language Specification)에 포함되지 않기 때문에 C#이 아닌 VB.NET등 다른 닷넷 플랫폼 지원 언어에서는 사용할 수 없어 호환되지 않습니다.

{% include content_adsense.html %}
