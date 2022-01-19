---
title: (C#) 닷넷 스레드 비동기 프로그래밍 [TAP] (async／await)
categories: C#
key: 20220117_01
comments: true
tags: C# 스레드 TAP thread 비동기 비동기프로그래밍 async await
---

닷넷에서는 비동기 프로그래밍 처리를 지원하는 방식이 여러가지 있습니다. 이를 닷넷에서는 '비동기 프로그래밍 패턴'이라고 정하고 있습니다.<br/>
비동기 프로그래밍 패턴은 세 가지의 패턴이 있습니다.

- IAsyncResult 형태의 콜백을 사용하는 **APM 패턴**(IAsyncResult 패턴) [링크](https://tyeom.github.io/c%23/2022/01/17/C-%EB%8B%B7%EB%84%B7-%EC%8A%A4%EB%A0%88%EB%93%9C-%EB%B9%84%EB%8F%99%EA%B8%B0-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D-APM.html)
- 이벤트 기반의 **EAP 패턴** [링크](https://tyeom.github.io/c%23/2022/01/17/C-%EB%8B%B7%EB%84%B7-%EC%8A%A4%EB%A0%88%EB%93%9C-%EB%B9%84%EB%8F%99%EA%B8%B0-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D-EAP.html)
- 작업 기반의 **TAP 패턴** 이 방식은 .NET Framework 4에서 도입되었으며, 비동기 프로그래밍에 권장되는 방식 입니다. [링크](https://tyeom.github.io/c%23/2022/01/18/C-%EB%8B%B7%EB%84%B7-%EC%8A%A4%EB%A0%88%EB%93%9C-%EB%B9%84%EB%8F%99%EA%B8%B0-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D-TAP-(async-await).html)

<!--more-->

그럼 위 비동기 프로그래밍 패턴이 어떻게 사용되고 어떤 특징이 있는지 하나씩 살펴 보겠습니다.

작업 기반 TAP 패턴
-

APM 패턴 방식이나 EAP 패턴 방식을 사용하여 비동기 작업 처리를 하는 것은 많은 코드 작업이 필요하고 구현 방식에 따라 복잡해질 수 있는 단점이 있습니다.<br/>
TAP 패턴 방식은 이러한 단점을 보안하면서 기존 패턴에서 사용 되는 콜백, 작업 완료 통보 처리 등 모든 것을 간결하게 지원하고 있습니다.
APM 패턴은 **<span style="color: rgb(107, 173, 222);">System.IAsyncResult</span>** 를 통해 '콜백' 방식을 구현하여 결과를 통보 받고<br/>
EAP 패턴 방식은 **<span style="color: rgb(107, 173, 222);">System.ComponentModel.AsyncOperation</span>** 를 통해 이벤트로 결과를 통보 받았습니다.<br/>
그리고 결과를 받기 위해 위 방식 모두 직접 구현을 해주어야 했습니다.<br/>
TAP에서는 **<span style="color: rgb(107, 173, 222);">System.Threading.Tasks.Task</span>** 클래스에서 비동기 결과 처리를 받을 수 있도록 제공 됩니다.

기본적인 **<span style="color: rgb(107, 173, 222);">System.Threading.Tasks.Task</span>** 사용은 다음과 같이 사용할 수 있습니다.

```cs
Task.Run(비동기 대상)
```

실제 코드로 작성해보면<br/>
```cs
Task.Run(() =>
{
  for(int i = 0; i < 10; i++)
  {
    System.Threading.Thread.Sleep(1000);
    Console.WriteLine($"{i}");
  }
});
```

단순히  **<span style="color: rgb(107, 173, 222);">System.Threading.Tasks.Task</span>** 의 Run&lt;TResult&gt;() 정적 메서드 사용으로만 비동기 작업을 실행 할 수 있습니다.<br/>
그럼 비동기 작업의 결과를 받으려면 어떻게 하면 될까요?<br/>
작업이 끝날때 까지 기다렸다가 그 결과를 반환 받는데 사용할 수 있는 Result속성이 제공 됩니다.

```cs
var task = new Task<int>(() =>
{
  System.Threading.Thread.Sleep(5000);
  return 10;
});

task.Start();
Console.WriteLine("Task 비동기 시작!");

var result = task.Result;
Console.WriteLine($"결과는 : {result}");
```

위 코드 실행시 'Task 비동기 시작!' 먼저 출력 된 이후 스레드가 블로킹 되고 약 5초뒤 블로킹이 해제 되면서 결과 10이 출력 되는걸 볼 수 있습니다.<br/>
이 처럼 Result속성은 자동으로 동기 처리를 해서 명시적으로 Wait()메서드를 호출하는 것과 동일한 결과를 가져 옵니다.

여러 작업을 비동기 병렬 처리또한 지원합니다. WaitAll()메서드를 사용해서 여러개의 Task작업이 모두 끝날때 까지 대기한 후 결과를 받아 올 수 있습니다.<br/>
```cs
// 5개의 Task&lt;int&gt; 메서드 델리게이트 생성
var tasks = Enumerable.Range(0, 5).Select(p => Output($"작업 {p}", p)).ToArray();

// Task모두 실행
foreach (var task in tasks)
{
  task.Start();
}

Console.WriteLine("Task 비동기 시작!");
// 모든 Task작업이 완료 될때 까지 블로킹
Task.WaitAll(tasks);
Console.WriteLine("Task 비동기 완료");

// 결과 출력
foreach (var task in tasks)
{
  Console.WriteLine($"결과는 : {task.Result}");
}
```

> 참고로 이러한 반복 적인 병렬로  병렬 처리는 **<span style="color: rgb(107, 173, 222);">System.Threading.Tasks.Parallel</span>** 클래스에도 제공 되고 있습니다.

비동기 작업이 모두 끝날때 까지 스레드를 블로킹 하지 않고 콜백을 사용하여 결과를 받아 처리 할 수 있습니다. <br/>
이때 사용할 수 있는 메서드가 ContinueWith() 입니다.<br/>
메서드의 시그니처를 보면 Action<Task&lt;TResult&gt;>의 델리게이트를 파라메터로 받고 있습니다. 바로 이 부분이 비동기 작업이 완료된 이후 콜백으로 호출되는 부분입니다.<br/>
```cs
Task.Run<bool>(() =>
{
  for (int i = 0; i < 10; i++)
  {
    System.Threading.Thread.Sleep(1000);
    Console.WriteLine(i);
  }
  
  return true;
}).ContinueWith(t =>
{
  Console.WriteLine($"비동기 작업 결과 : {t.Result}");
});
Console.WriteLine("Task 비동기 시작!");
```

출력 결과는<br/>
```
Task 비동기 시작!
0
1
2
3
4
5
6
7
8
9
비동기 작업 결과 : True
```

비동기 작업 이후 '작업 결과' 메세지가 출력되는걸 확인할 수 있습니다.

System.Threading.Tasks.TaskFactory
-

**<span style="color: rgb(107, 173, 222);">System.Threading.Tasks.Task</span>** 를 사용한 비동기 처리는 기본적으로 스레드 풀을 사용해서 처리 됩니다.<br/>
하지만 상황에 따라 스레드 풀을 사용하지 않거나 Task의 스케줄 옵션을 처리해야 하는 경우 비동기 작업 처리의 상세한 옵션이 제공되는 **<span style="color: rgb(107, 173, 222);">System.Threading.Tasks.TaskFactory</span>** 를 사용할 수 있습니다.<br/>
**<span style="color: rgb(107, 173, 222);">System.Threading.Tasks.Task</span>** 에 Factory 정적 속성으로 싱글턴 형식으로 바로 사용할 수 있습니다.<br/>
하지만 특별한 상황이 아닌 경우 스레드 풀을 사용하는 것이 성능에도 더 도움이 되고 짧은 코드로 간편하게 사용할 수 있는 **<span style="color: rgb(107, 173, 222);">System.Threading.Tasks.Task</span>** 를 사용하는 것이 가독성 측면에서 더 좋습니다.<br/>
**[<span style="color: rgb(107, 173, 222);">System.Threading.Tasks.Task</span> 사용]**
```cs
Task.Run(this.Work);
```

**[<span style="color: rgb(107, 173, 222);">System.Threading.Tasks.TaskFactory</span> 사용]**
```cs
Task.Factory.StartNew(this.Work,
    TaskCreationOptions.DenyChildAttach,
    TaskScheduler.Default);
```

위 두 코드의 동작은 동일 합니다.

async／await 사용의 대기 처리
-

위에서 **<span style="color: rgb(107, 173, 222);">System.Threading.Tasks.Task</span>** 비동기 처리 이후 ContinueWith() 메서드를 통한 작업 결과를 콜백 받는 방법을 설명했습니다.<br/>
만약 ContinueWith() 메서드안에서 또 다른 Task가 시작되고 그에 따른 또 콜백을 받아 처리 해야 하는 경우라면 그것은 콜백지옥이 될 것 입니다.<br/>
그리고 ContinueWith() 로 인한 콜백 메서드 영역은 UI스레드가 아니므로 해당 구문에서 UI에 접근시 크로스스레드에 대한 처리가 별도 필요 합니다.

이런 상황에서 async／await 예약어 사용으로 훨신 간결한 코드를 만들 수 있습니다. 위에서 예를 들었던 코드를 async await 으로 바꾸면 다음과 같이 변경 할 수 있습니다.<br/>
```cs
Task.Run<bool>(() =>
{
  for (int i = 0; i < 10; i++)
  {
    System.Threading.Thread.Sleep(1000);
    Console.WriteLine(i);
  }
  
  return true;
}).ContinueWith(t =>
{
  Console.WriteLine($"비동기 작업 결과 : {t.Result}");
});
Console.WriteLine("Task 비동기 시작!");
```

위 코드를 async await 예약어를 사용해서 처리 한다면<br/>
```cs
private Task<bool> WorkAsync()
{
  return Task.Run<bool>(() =>
  {
    for (int i = 0; i < 10; i++)
      {
        System.Threading.Thread.Sleep(1000);
        Console.WriteLine(i);
      }
                           
    return true;
  });
}

private async Task<bool> GetWorkResult()
{
  bool result = await this.WorkAsync();  // UI 블로킹 없이 10초 후 true반환
}
```

이 처럼 await 예약어를 통해 비동기 작업에 대한 결과를 처리 할 수 있습니다.

그리고 async 예약어를 사용하려면 반환 타입이 **<span style="color: rgb(107, 173, 222);">System.Threading.Tasks.Task&lt;TResult&gt;</span>** 이거나<br/>
결과가 없는 비동기 작업이라면 **<span style="color: rgb(107, 173, 222);">System.Threading.Tasks.Task</span>** 타입이거나<br/>
void 반환 메서드에서 사용할 수 있습니다.<br/>
사실 위 3가지 반환 메서드 외 '일반화된 비동기 반환 형식'의 **<span style="color: rgb(107, 173, 222);">System.Threading.Tasks.ValueTask&lt;T&gt;<span>** 타입도 사용이 가능합니다.<br/>
이 부분은 다음 포스트를 참조하면 됩니다. [링크](https://tyeom.github.io/c%23/2022/01/13/C-%EC%BB%A4%EC%8A%A4%ED%85%80-%EB%B9%84%EB%8F%99%EA%B8%B0-Task.html)

await 예약어에 대해 계속해서 설명을 하면 메서드 내에서 동기로 코드를 수행하다가 await 구문을 만나면 해당 메서드에서 내에서 비동기 수행 스레드는 현재 스레드의 컨텍스트를 캡쳐하고 아직 작업이 완료 되지 않은 **<span style="color: rgb(107, 173, 222);">System.Threading.Tasks.Task</span>** 타입 또는 **<span style="color: rgb(107, 173, 222);">System.Threading.Tasks.Task&lt;TResult&gt;</span>** 를 즉시 반환 합니다. (위 예제에서는 WorkAsync() 메서드가 됩니다.)<br/>
그리고선 async메서드를 호출한 메서드에서 그 다음 구문을 계속해서 수행해 나갑니다. 그러다가 비동기 코드 처리가 완료 되면 다시 await 이후 구문을 수행하게 됩니다.<br/>
이런 내부 처리 동작으로 스레드 블로킹이 되지 않고 자체 콜백 처리로 깔끔한 코드로 표현이 가능 합니다.

이런 처리 과정을 순서대로 정리해본다면 (winform)<br/>
```cs
private Task<bool> WorkAsync()
{
  return Task.Run<bool>(() =>
  {
    for (int i = 0; i < 10; i++)
    {
      System.Threading.Thread.Sleep(1000);
      Console.WriteLine(i);
    }
  
    return true;
  });
}

private async Task GetWorkResult()
{
  bool result = await this.WorkAsync();
  Console.WriteLine(result);
}
```

```cs
private void button_Click(object sender, EventArgs e)
{
  Console.WriteLine("Logic1");
  Console.WriteLine("Logic2");
  this.GetWorkResult();
  Console.WriteLine("Logic3");
```

출력 결과는<br/>
```
Logic1
Logic2
Logic3
0
1
2
3
4
5
6
7
8
9
True
```

위 코드의 처리 순서는 다음과 같이 설명할 수 있습니다.<br/>
```
1. 메인 UI스레드에서 'Logic1', 'Logic2' 출력
2. GetWorkResult() 메서드 진입   [메인 스레드에서 처리]
  2-1. bool result = await this.WorkAsync(); 구문 수행    [메인 스레드에서 처리]
  2-2. WorkAsync() 메서드가 작업이 끝나지 않은 System.Threading.Tasks.Task&lt;TResult&gt; 를 반환하면 메인 스레드는 바로 리턴    [메인 스레드에서 처리]
3. 메인 스레드는 계속해서 'Logic3' 출력
4. 3번 작업과 동시에 WorkAsync() 메서드내의 System.Threading.Tasks.Task 로 처리 되는 비동기 작업을 스레드 풀에서 새로운 스레드를 가져와 별도의 스레드로 처리 합니다.    [작업 스레드에서 처리]
5. 4번 비동기 작업이 완료 되면 await이후 구문 처리   [메인 스레드에서 처리 (＊ConfigureAwait() 메서드 호출에 따라 항상 그렇진 않습니다.)]
```

사실 위와 같은 내부적 동작은 async／await 예약어를 사용하고 컴파일 할때 컴파일러에 의해 새로운 IL코드가 생성되어져서 가능한 것 입니다.<br/>
이렇게 컴파일러에 의해 자동 생성된 코드로 인해 개발자 입장에서 편리하게 사용할 수 있는 것을 Syntatic sugar 라고 표현 합니다.

그럼 위 코드가 컴파일 되었을때 어떻게 변경되는지 간단히 한번 살펴보겠습니다. (위의 처리 순서만 이해가 된다면 이 부분은 자세하게 살펴볼 필요는 없습니다.)<br/>
await 예약어가 사용된 async GetWorkResult() 메서드는 아래와 같이 컴파일러에 의해 변경 되었습니다.<br/>
```cs
[AsyncStateMachine(typeof (Form2.<GetWorkResult>d__13))]
[DebuggerStepThrough]
private Task GetWorkResult()
{
  Form2.<GetWorkResult>d__13 stateMachine = new Form2.<GetWorkResult>d__13();
  __builder = AsyncTaskMethodBuilder.Create();
  __this = this;
  __state = -1;
  _builder.Start<Form2.<GetWorkResult>d__13>(ref stateMachine);
  return __builder.Task;
}
```

살펴보면 StateMachine 로직이 생성된 **<span style="color: rgb(107, 173, 222);">System.Runtime.CompilerServices.IAsyncStateMachine</span>** 인터페이스가 상속된 GetWorkResult 클래스 생성하고 상태정보를 -1로 초기화 하고 **<span style="color: rgb(107, 173, 222);">System.Runtime.CompilerServices.AsyncTaskMethodBuilder</span>** 구조체를 생성해서 실행 합니다.

그리고 Create() 메서드 호출로 **<span style="color: rgb(107, 173, 222);">System.Runtime.CompilerServices.AsyncTaskMethodBuilder</span>** 구조체가 생성되고 그 안에 정의 되어 있는<br/>
**<span style="color: rgb(107, 173, 222);">System.Runtime.CompilerServices.AsyncTaskMethodBuilder&lt;TResult&gt;</span>** 구조체도 같이 초기화 시켜 줍니다. 그리고는 비동기 작업 Task를 바로 반환 합니다.

그럼 자동 생성된 StateMachine이 포함되어 있는 클래스는 어떻게 되어 있는지 보겠습니다. 위에서 실행하고 있는 자동 생성된 GetWorkResult 클래스 입니다.<br/>
```cs
[CompilerGenerated]
private sealed class <GetWorkResult>d__13 : IAsyncStateMachine
{
  public int <>1__state;
  public AsyncTaskMethodBuilder <>t__builder;
  public Form2 <>4__this;
  private bool <result>5__1;
  private bool <>s__2;
  private TaskAwaiter<bool> <>u__1;
  
  public <GetWorkResult>d__13()
  {
    base..ctor();
  }
  
  void IAsyncStateMachine.MoveNext()
  {
    int num1 = this.<>1__state;
    try
    {
      TaskAwaiter<bool> awaiter;
      int num2;
      if (num1 != 0)
      {
        /////////// await 이전 구문 [메인 스레드에서 처리] ///////////
  
        awaiter = this.<>4__this.WorkAsync().GetAwaiter();  // 비동기 작업의 System.Runtime.CompilerServices.TaskAwaiter&lt;TResult&gt; 구조체를 가져 옵니다.
        if (!awaiter.IsCompleted)
        {
          this.<>1__state = num2 = 0;
          this.<>u__1 = awaiter;
          Form2.<GetWorkResult>d__13 stateMachine = this;
          this.<>t__builder.AwaitUnsafeOnCompleted<TaskAwaiter<bool>, Form2.<GetWorkResult>d__13>(ref awaiter, ref stateMachine);
          return;
        }
      }
      else
      {
        awaiter = this.<>u__1;
        this.<>u__1 = new TaskAwaiter<bool>();
        this.<>1__state = num2 = -1;
      }
      this.<>s__2 = awaiter.GetResult();
  
      /////////// await 이후 구문 ///////////
      
      this.<result>5__1 = this.<>s__2;
      Console.WriteLine(this.<result>5__1);
    }
    catch (Exception ex)
    {
      this.<>1__state = -2;
      this.<>t__builder.SetException(ex);
      return;
    }
  
    this.<>1__state = -2;
    this.<>t__builder.SetResult();
  }
  
  [DebuggerHidden]
  void IAsyncStateMachine.SetStateMachine(IAsyncStateMachine stateMachine)
  {
  }
}
```

코드를 보면 알 수 있듯이 await 예약어 기준으로 이전 로직과 이후 로직을 분리하고 작업 전, 완료의 상태를 구분하고 있습니다.<br/>
비동기 작업의 **<span style="color: rgb(107, 173, 222);">System.Runtime.CompilerServices.TaskAwaiter&lt;TResult&gt;</span>** 구조체를 가져와서 Task가 끝났는지 체크를 합니다.<br/>
아직 완료 되지 않았다면 **<span style="color: rgb(107, 173, 222);">System.Runtime.CompilerServices.AsyncTaskMethodBuilder</span>** 의 AwaitUnsafeOnCompleted<TAwaiter, TStateMachine>() 메서드를 통해 작업 완료 이후 콜백 처리 델리게이트를 등록합니다. 이 처리는 해당 Task완료 후 .ContinueWith(t => { MoveNext() }); 로 하는 것과 동일합니다.

결국 비동기 작업이 완료 되면 MoveNext()메서드가 호출되고 this.<>1__state 값이 -1로 바뀌고 나서 await 이후 구문이 수행 됩니다.<br/>
또한 이 StateMachine 코드는 반복문 안에서 await을 하는지 여부에 따라 goto 문으로 처리되기도 합니다.

await과 SynchronizationContext 의 관계
-

그럼 어떻게 await이후 메인 스레드로 다시 처리가 되는걸까요? 정확히는 비동기 작업 스레드를 호출한 스레드를 어떻게 찾을 수 있는 것일까요?<br/>
그 처리는 **<span style="color: rgb(107, 173, 222);">System.Threading.SynchronizationContext</span>** 클래스에 있습니다<br/>
닷넷의 모든 스레드는 하나 이상의 **<span style="color: rgb(107, 173, 222);">System.Threading.SynchronizationContext</span>** 를 가지고 있습니다.<br/>
await 으로 스레드 풀에서 새로운 작업 스레드로 비동기 작업이 호출 될때 **<span style="color: rgb(107, 173, 222);">System.Threading.SynchronizationContext</span>** 클래스의<br/>
Current 속성으로 현재 컨텍스트를 캡쳐해둔 후 await이후 작업을 처리하게 됩니다.<br/>
하지만 UI가 없는 콘솔 앱 등의 경우는 UI 처리를 위한 동기화 처리가 필요 없으므로 **<span style="color: rgb(107, 173, 222);">System.Threading.SynchronizationContext</span>** 의 Current가 없습니다.

따라서 아래 코드는
```cs
private async Task<string> GetString()
{
  string result = await Task.Run<string>(() =>
  {
    System.Threading.Thread.Sleep(2000);
    return "test";
  });
  
  return result;
}

string result = await this.GetString();
Console.WriteLine(result);
```

이렇게 처리 되는 것과 동일합니다.
```cs
SynchronizationContext sc = SynchronizationContext.Current;
Task.Run<string>(() =>
{
  System.Threading.Thread.Sleep(2000);
  return "test";
})
.ContinueWith(t =>
{
  sc.Post((p) =>
  {
    string result = t.Result;
    Console.WriteLine(result);
  }, null);
});
```
