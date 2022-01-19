---
title: (C#) ThreadLocal&lt;T&gt;와 AsyncLocal&lt;T&gt;의 차이점
categories: C#
key: 20220114_01
comments: true
tags: C# TLS ThreadLocal AsyncLocal
---

멀티 스레드 환경에서 각 스레드별로 하나의 공통 데이터에 접근하여 처리 될때 스레드 마다 원자성을 보장하면서<br/>
데이터를 다뤄야 하는 상황이 발생할 수 있습니다.

이때 제공 되는것이 Thread Local Storage(TLS) 변수라는 것이 있습니다.

쉽게 말해 tls변수는 스레드별 고유한 데이터를 저장(?)할 수 있는 공간이라고 이해 하면 됩니다. 또한 이 tls변수는(이하 거론되는 **<span style="color: rgb(107, 173, 222);">ThreadLocal&lt;T&gt;</span>** **<span style="color: rgb(107, 173, 222);">AsyncLocal&lt;T&gt;</span>** 동일) 자체적으로 데이터 원자성이 보장되기에 lock없이 접근하여 사용이 가능합니다.

<!--more-->

닷넷 C#에서는 이것을 **<span style="color: rgb(107, 173, 222);">System.ThreadStaticAttribute</span>** 어트리뷰트 클래스로 제공 하고 있으며<br/>
**<span style="color: rgb(107, 173, 222);">System.Threading.ThreadLocal&lt;T&gt;</span>** 그리고 **<span style="color: rgb(107, 173, 222);">System.Threading.AsyncLocal&lt;T&gt;</span>** 클래스로 제공하고 있고, C++에서는 __declspec(thread)로 사용 가능합니다.<br/>

**<span style="color: rgb(107, 173, 222);">System.Threading.ThreadLocal&lt;T&gt;</span>** 같은 경우 **<span style="color: rgb(107, 173, 222);">System.ThreadStaticAttribute</span>** 보다 변수 초기 방법을 제공하고 있어 **<span style="color: rgb(107, 173, 222);">[ThreadStatic]</span>** 어트리뷰트 보다 좀 더 최신 기능을 제공합니다.

이번 포스트는 **<span style="color: rgb(107, 173, 222);">System.Threading.ThreadLocal&lt;T&gt;</span>** 와 **<span style="color: rgb(107, 173, 222);">System.Threading.AsyncLocal&lt;T&gt;</span>** 가 서로 어떤 부분에 차이점이 있는지에 대한 내용 입니다.

자 그럼 어떻게 다른지 살펴보겠습니다.

**<span style="color: rgb(107, 173, 222);">ThreadLocal&lt;T&gt;</span>** **<span style="color: rgb(107, 173, 222);">AsyncLocal&lt;T&gt;</span>** 차이점
-

위 tls역할을 하는 두 클래스의 큰 차이점은 스레드풀을 사용하는 스레드 환경에서 차이점이 있습니다.

우선 **<span style="color: rgb(107, 173, 222);">System.Threading.ThreadLocal&lt;T&gt;</span>** 은 해당 스레드가 스레드풀에 반환되어도 해당 데이터는 항상 유지 되어 집니다.

```cs
private static ThreadLocal<int> _tl = new ThreadLocal<int>();

private static async Task WorkAsync(int i)
{
  if(_tl.Value == 0)
  {
    _tl.Value = i;
    Console.WriteLine($"[새로운 값] Thread Id : {Environment.CurrentManagedThreadId} - tl Value : {_tl.Value}");
  }
  else
  {
    Console.WriteLine($"[기존 값] Thread Id : {Environment.CurrentManagedThreadId} - tl Value : {_tl.Value}");
  }
  Console.WriteLine("-----------------------------------------");
}
```

위 처럼 **<span style="color: rgb(107, 173, 222);">System.Threading.ThreadLocal&lt;T&gt;</span>** 을 사용하는 메서드를 스레드로 호출해 봅니다.
그리고 차이점을 확실히 보기 위해 스레드풀의 스레드 개수 제한을 4개로 설정해 보았습니다.

```cs
// ThreadPool의 개수를 4개로 임의 제한
ThreadPool.SetMinThreads(4, 4);
ThreadPool.SetMaxThreads(4, 4);

for (int i = 1; i < 11; i++)
{
  await Task.Run(() => WorkAsync(i));
}
```

스레드를 사용하여 WorkAsync() 메서드를 10번 호출하였고 파라메터로 tls값을 동시에 넘겨 주었습니다.

결과는 어떻게 나올까요?

![image](https://user-images.githubusercontent.com/13028129/149457378-d0d88799-b015-49d6-a382-bf15e5a8b7f4.png)

위 처럼 처음 스레드가 사용되어서 tls값이 설정 되고 해당 스레드가 반환된 이후 다시 같은 스레드가 사용 되었을때<br/>
tls값이 처음 설정 된 값으로 유지 되고 있는 걸 확인 할 수 있습니다.
  
반면,

**<span style="color: rgb(107, 173, 222);">AsyncLocal&lt;T&gt;</span>** 는 어떻게 처리 되는지 위 코드에서 **<span style="color: rgb(107, 173, 222);">System.Threading.ThreadLocal&lt;T&gt;</span>** 부분만 바꿔서 실행 시켜 보겠습니다.<br/>

![image](https://user-images.githubusercontent.com/13028129/149457495-e2ec922b-988b-4e1a-a929-f879533ad172.png)

결과를 보니 위 결과와는 다르게 사용된 스레드가 스레드풀에 반환될때는 값이 초기화가 되는 걸 볼 수 있습니다.

그럼 **<span style="color: rgb(107, 173, 222);">ThreadLocal&lt;T&gt;</span>** 와 **<span style="color: rgb(107, 173, 222);">AsyncLocal&lt;T&gt;</span>** 의 차이점을 알았으니 어떤 상황에서 제대로 사용해야 하는지 보겠습니다.

  
데이터 복사
-

위 예제에서 **<span style="color: rgb(107, 173, 222);">ThreadLocal&lt;T&gt;</span>** 은 사용된 스레드가 스레드풀에 반환되도 값이 항상 유지 되는 것을 확인 했습니다.<br/>
이 뜻은 **<span style="color: rgb(107, 173, 222);">ThreadLocal&lt;T&gt;</span>** 에 값이 설정된 스레드 내에서 await으로 어떤 동기 작업을 대기시킨 이후 이어서 작업을 할때<br/>
새로운 스레드로 처리가 된다면 **<span style="color: rgb(107, 173, 222);">ThreadLocal&lt;T&gt;</span>** 값은 엉뚱한 값으로 처리될 문제가 있습니다.

> 이 상황은 Task 스레드 안에서 await 대기 사용시 또는 SynchronizationContext가 없는 콘솔 환경 같은 곳에서 해당 됩니다.<br/>
> SynchronizationContext가 있는 UI 환경 (Winform, WPF 등에서 메인 스레드(UI 스레드) 에서 호출된 비동기 메서드 await 이후 구문은 메인 스레드로 처리 됩니다.)

```cs
private static async Task WorkAsync(int i)
{
  if(_tl.Value == 0)
  {
    _tl.Value = i;
    Console.WriteLine($"[새로운 값] Thread Id : {Environment.CurrentManagedThreadId} - tl Value : {_tl.Value}");
  }
  else
  {
    Console.WriteLine($"[기존 값] Thread Id : {Environment.CurrentManagedThreadId} - tl Value : {_tl.Value}");
  }
  
  await Task.Delay(1000);
  
  Console.WriteLine($"[await 이후 값] Thread Id : {Environment.CurrentManagedThreadId} - tl Value : {_tl.Value}");
  Console.WriteLine("-----------------------------------------");
}
```

위 코드에 await Task.Delay(1000); 구문을 추가해 또 다른 작업을 대기 시킨 이후 계속해서 로직이 처리 되는 것으로 간주한 후 결과를 보겠습니다.

![image](https://user-images.githubusercontent.com/13028129/149459931-8a15e08f-9ded-44aa-b89c-3dacb5a70e07.png)

처음 빨간색 표시의 데이터는 await이후 같은 스레드로 처리가 되어 동일한 값으로 표시 되었지만<br/>
분홍색 표시의 데이터는 await이후 처음 사용되는 스레드로 처리되어 초기값인 0으로 표시 되고 있습니다.<br>
파란색 표시는 await이후 스레드는 이전에 사용 되었던 5번 스레드가 다시 재 사용 되어 기존 값 2로 표시 되고 있습니다.

이 처럼 중간 비동기 작업 이후 연속적인 참조 로직 처리가 있는 경우 기대 값과 다른 결과가 나올 수 있습니다.

이런 상황에서 사용할 수 있는 것이 **<span style="color: rgb(107, 173, 222);">AsyncLocal&lt;T&gt;</span>** 입니다.

바로 코드로 확인해 보겠습니다.

```cs
private static AsyncLocal<int> _al = new AsyncLocal<int>();

private static async Task WorkAsync(int i)
{
  if(_al.Value == 0)
  {
    _al.Value = i;
    Console.WriteLine($"[새로운 값] Thread Id : {Environment.CurrentManagedThreadId} - al Value : {_al.Value}");
  }
  else
  {
    Console.WriteLine($"[기존 값] Thread Id : {Environment.CurrentManagedThreadId} - al Value : {_al.Value}");
  }
  
  await Task.Delay(1000);
  
  Console.WriteLine($"[await 이후 값] Thread Id : {Environment.CurrentManagedThreadId} - al Value : {_al.Value}");
  Console.WriteLine("-----------------------------------------");
}
```

![image](https://user-images.githubusercontent.com/13028129/149460912-1a85253d-e190-49b4-b57c-8ca96231cebc.png)

결과는 위와 같이 await이후 다른 스레드가 사용 되어도 이전 스레드에서 사용 되었던 값이 그대로 복사되어 동일하게 표시 되고 있습니다.


의도치 않은 데이터 복사 방지
-

이런 동작은 **<span style="color: rgb(107, 173, 222);">System.Runtime.Remoting.Messaging.CallContext</span>** 의 LogicalData와 같습니다.<br/>
스레드간 데이터 복사는 의도치 않은 오버헤드를 일으킬 수 있습니다. 만약 **<span style="color: rgb(107, 173, 222);">AsyncLocal&lt;T&gt;</span>** 사용에서 스레드풀에서 새로 사용 되는 스레드간 데이터 복사를 막아야 하는 경우라면 <br/>

**<span style="color: rgb(107, 173, 222);">System.Runtime.Remoting.Messaging.CallContext</span>** 스레드간 데이터 이관 처리를 막는 것 처럼<br/>

**<span style="color: rgb(107, 173, 222);">System.Threading.ExecutionContext</span>** 클래스의 SuppressFlow()와 RestoreFlow()의 정적 메서드를 사용하면 됩니다.

```cs
ExecutionContext.SuppressFlow();
... [스레드 작업] ...
ExecutionContext.RestoreFlow();
```

위 처럼 스레드간 데이터 복사시 오버헤드를 방지 할 수 있습니다.
