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
메서드의 시그니처를 보면 Action<Task<TResult>>의 델리게이트를 파라메터로 받고 있습니다. 바로 이 부분이 비동기 작업이 완료된 이후 콜백으로 호출되는 부분입니다.<br/>
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
**[**<span style="color: rgb(107, 173, 222);">System.Threading.Tasks.Task</span>** 사용]**
```cs
Task.Run(this.Work);
```

**[**<span style="color: rgb(107, 173, 222);">System.Threading.Tasks.TaskFactory</span>** 사용]**
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

이런 상황에서 async／await 예약어 사용으로 훨신 간결한 코드를 만들 수 있습니다.


