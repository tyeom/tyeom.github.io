---
title: (C#) TPL Dataflow 라이브러리
categories: C#
key: 20221223_01
comments: true
tags: .NET C# TPL Dataflow 데이터흐름 작업병렬처리 병렬처리 BroadcastBlock ActionBlock TransformBlock
---

.NET 공식(?) 라이브러리중 데이터 병렬 처리를 다룰 수 있도록 제공하는 Dataflow Task 라이브러리가 있습니다. (TPL Dataflow 라이브러리)<br/>
TPL Dataflow는 .NET 기본 API에 포함되지 않고 NuGet 패키지로 설치해서 사용할 수 있습니다.<br/>
[NuGet - System.Threading.Tasks.Dataflow](https://www.nuget.org/packages/System.Threading.Tasks.Dataflow)<br/>
[TPL Dataflow MS Doc](https://learn.microsoft.com/ko-kr/dotnet/standard/parallel-programming/dataflow-task-parallel-library)<br/>
이번 글에선 TPL Dataflow에서 제공하는 **<span style="color: rgb(107, 173, 222);">System.Threading.Tasks.Dataflow.BroadcastBlock&lt;T&gt;</span>** 클래스에 대해 알아보고
간단한 샘플 예제를 작성해 보겠습니다.

<!--more-->

> 이 글에서 다루는 코드는 다음 Repository에서 확인할 수 있습니다.<br/>
> [Code_check - TPL_DataFlow](https://github.com/tyeom/code_check/tree/main/TestSample/csharp/TPL_DataFlow)

TPL Dataflow는 스트림형태의 데이터 메세지를 쉽게 구현할 수 있습니다. TPL Dataflow에서 메세지(데이터)는 Block단위로 처리되고 이 Block을 조작할 수 있는 기능들이 여러가지로 제공되고 있습니다.<br/>
**<span style="color: rgb(107, 173, 222);">System.Threading.Tasks.Dataflow.Post&lt;TInput&gt;()</span>** 메서드 및 **<span style="color: rgb(107, 173, 222);">System.Threading.Tasks.Dataflow.SendAsync&lt;TInput&gt;()</span>** 메서드로 
동기 또는 비동기 형식으로 메세지를 Block에 추가할 수 있습니다.<br/>
TPL Dataflow에서 사용되는 Block은 여러 종류가 있는데 <br/>
**<span style="color: rgb(107, 173, 222);">System.Threading.Tasks.Dataflow.ActionBlock&lt;TInput&gt;</span>** 클래스<br/>
**<span style="color: rgb(107, 173, 222);">System.Threading.Tasks.Dataflow.BufferBlock&lt;T&gt;</span>** 클래스<br/>
**<span style="color: rgb(107, 173, 222);">System.Threading.Tasks.Dataflow.TransformBlock&lt;TInput, TOutput&gt;</span>** 클래스<br/>
**<span style="color: rgb(107, 173, 222);">System.Threading.Tasks.Dataflow.BroadcastBlock&lt;TInput, TOutput&gt;</span>** 클래스 등이 있습니다.<br/>
그리고 각 Block들은 서로 **<span style="color: rgb(107, 173, 222);">System.Threading.Tasks.Dataflow.DataflowBlock.LinkTo()</span>** 메서드로 메세지 흐름을 서로 연결할 수 있습니다.

Synchronization Blocks
-

기본적으로 각 Block들은 다른 Block들과 독립적으로 처리됩니다. 즉 각 개별 Block에서 다른 처리를 할 수 있고 이렇게 Block은 병렬 처리를 제공합니다.<br/>
동기화가 필요한 처리인 경우 Block을 생성할때 **<span style="color: rgb(107, 173, 222);">System.Threading.Tasks.Dataflow.DataflowBlockOptions.TaskScheduler</span>** 속성을 지정할 수 있습니다.
이 속성으로 어떤 컨텍스트에서 실행될지 설정할 수 있습니다.<br/>
만약 **<span style="color: rgb(107, 173, 222);">System.Threading.Tasks.TaskScheduler.FromCurrentSynchronizationContext()</span>** 메서드로 현재의 스레드 컨텍스트에서
처리되도록 한다면 동기화 처리가 가능합니다.

ActionBlock
-

**<span style="color: rgb(107, 173, 222);">System.Threading.Tasks.Dataflow.ActionBlock&lt;TInput&gt;</span>** 클래스는 수신되는 각 메세지에 대응할 수 있는 콜백을 지원합니다.<br/>
메세지들을 foreach로 순차적 처리하는 것 처럼 처리할 수 있습니다.

TransformBlock
-

**<span style="color: rgb(107, 173, 222);">System.Threading.Tasks.Dataflow.TransformBlock&lt;TInput, TOutput&gt;</span>** 클래스는 **<span style="color: rgb(107, 173, 222);">System.Threading.Tasks.Dataflow.ActionBlock&lt;TInput&gt;()</span>** 클래스와 역할이 동일하지만<br/>
데이터 입력과 데이터 출력 부분을 동시에 받아서 입력 데이터 기준으로 출력 데이터를 가공해서 새로운 데이터로 출력 할 수 있습니다.<br/>
즉 **<span style="color: rgb(107, 173, 222);">System.Threading.Tasks.Dataflow.ActionBlock&lt;TInput&gt;</span>** 클래스는 수신된 데이터를 그대로 사용해야 하지만 **<span style="color: rgb(107, 173, 222);">System.Threading.Tasks.Dataflow.TransformBlock&lt;TInput, TOutput&gt;()</span>** 클래스는 수신된 데이터를 가공해서
새로운 데이터 타입으로 출력이 가능합니다.

BroadcastBlock
-

**<span style="color: rgb(107, 173, 222);">System.Threading.Tasks.Dataflow.BroadcastBlock&lt;TInput, TOutput&gt;</span>** 클래스는 수신된 데이터를 복사하고 자신의 Block에 연결된 모든 Block에게 전달 합니다.<br/>
만약 **<span style="color: rgb(107, 173, 222);">System.Threading.Tasks.Dataflow.BroadcastBlock&lt;TInput, TOutput&gt;</span>** 에 **<span style="color: rgb(107, 173, 222);">System.Threading.Tasks.Dataflow.TransformBlock&lt;TInput, TOutput&gt;()</span>** 와 **<span style="color: rgb(107, 173, 222);">System.Threading.Tasks.Dataflow.ActionBlock&lt;TInput&gt;()</span>** 가 연결되어 있다면<br/>
모든 Block에게 복사된 메세지를 전달할 수 있습니다.

Completing Blocks
-

온전히 Dataflow 처리가 완료 되었다면 **<span style="color: rgb(107, 173, 222);">System.Threading.Tasks.Dataflow.IDataflowBlock.Complete()</span>** 메서드로 완료 처리 할 수 있습니다.<br/>
Dataflow가 오류 없이 완료 되었는지 확인은 **<span style="color: rgb(107, 173, 222);">System.Threading.Tasks.Dataflow.IDataflowBlock.Completion</span>** 속성으로 확인 가능합니다.<br/>
해당 속성은 **<span style="color: rgb(107, 173, 222);">System.Threading.Tasks.Task</span>** 클래스를 반환하는데 **<span style="color: rgb(107, 173, 222);">System.Threading.Tasks.Wait()</span>** 메서드로<br/>
작업을 완료를 대기할때 오류가 발생되는 경우 try - catch 예외 처리로 확인할 수 있습니다.<br/>
```cs
_stockInfoBroadCaster.Complete();
try
{
    _stockInfoBroadCaster.Completion.Wait();
}
catch (Exception ex)
{
    // 데이터흐름 완료 오류
}
```

Example [WPF - StockInfo]
-

다음 예제는 TPL Dataflow 라이브러리를 사용해서 간단한 주식 정보 데이터를 받아 화면에 출력하는 예제 입니다.<br/>
**<span style="color: rgb(107, 173, 222);">System.Threading.Tasks.Dataflow.BroadcastBlock&lt;TInput, TOutput&gt;</span>** 에 **<span style="color: rgb(107, 173, 222);">System.Threading.Tasks.Dataflow.ActionBlock&lt;TInput&gt;</span>** 을 연결하고<br/>
**<span style="color: rgb(107, 173, 222);">System.Threading.Tasks.Dataflow.TransformBlock&lt;TInput, TOutput&gt;</span>** 으로 메세지를 가공해서 또 다른 **<span style="color: rgb(107, 173, 222);">System.Threading.Tasks.Dataflow.ActionBlock&lt;TInput&gt;</span>** 으로 연결시켜서 
최종적으로 화면에 출력하게 되는 예제 입니다.<br/>

**[Block 생성 및 연결 부분]**<br/>
```cs
private BroadcastBlock<List<StockInfoModel>> _broadcast;

public MainViewModel()
{
        // 브로드캐스트 생성
        _broadcast = new BroadcastBlock<List<StockInfoModel>>(null);
}

private async Task MonitoringExecute(object param)
{
        // 브로드캐스팅된 블록(주식정보)을 수신받아
        // 정보 표시 ViewModel에 데이터 전달 역할을 하는 블록 생성
        ActionBlock<List<StockInfoModel>> updateStockInfoStart = this.UpdateStockInfoStart();
        // 브로드캐스트에 ActionBlock 연결
        _broadcast.LinkTo(updateStockInfoStart);

        // 브로드캐스트에 추가 블록을 연결할 수 있다.
        ActionBlock<List<StockInfoModel>> foo = Foo();
        _broadcast.LinkTo(foo);

        // 브로드캐스팅으로 데이터를 수신받고
        // 가공된 데이터를 ViewModel에 전달 역할을 하는 블록 생성
        ActionBlock<Tuple<double, double>> updateChangeCalc = this.UpdateChangeCalc();
        var changeCalc = new TransformBlock<List<StockInfoModel>, Tuple<double, double>>(p =>
        {
            var stockInfo1_currPrice = p[0].CurrentPrice;
            var stockInfo2_currPrice = p[1].CurrentPrice;

            double percentChangeByItem1 = (stockInfo1_currPrice - StockInfo1VM.PreviousPrice) * 100 / StockInfo1VM.PreviousPrice;
            double percentChangeByItem2 = (stockInfo2_currPrice - StockInfo2VM.PreviousPrice) * 100 / StockInfo2VM.PreviousPrice;

            StockInfo1VM.PreviousPrice = stockInfo1_currPrice;
            StockInfo2VM.PreviousPrice = stockInfo2_currPrice;

            return new Tuple<double, double>(percentChangeByItem1, percentChangeByItem2);
        });

        // 브로드캐스트에 TransformBlock 연결
        _broadcast.LinkTo(changeCalc);
        // TransformBlock에 ActionBlock 연결
        changeCalc.LinkTo(updateChangeCalc);

        // 브로드캐스팅 시작
        StockInfoBroadCast stockInfoBroadCast = new(_broadcast);
        await stockInfoBroadCast.RealTimeData();
}

private ActionBlock<List<StockInfoModel>> UpdateStockInfoStart()
{
        return new ActionBlock<List<StockInfoModel>>(p => {
            StockInfo1VM.UpdateStockInfo(p[0]);
            StockInfo2VM.UpdateStockInfo(p[1]);
        },
        new ExecutionDataflowBlockOptions()
        {
            TaskScheduler =
            TaskScheduler.FromCurrentSynchronizationContext()
        });
}

private ActionBlock<List<StockInfoModel>> Foo()
{
        return new ActionBlock<List<StockInfoModel>>(p => { /* do something */ },
        new ExecutionDataflowBlockOptions()
            {
                TaskScheduler =
                TaskScheduler.FromCurrentSynchronizationContext()
            });
}

private ActionBlock<Tuple<double, double>> UpdateChangeCalc()
{
        return new ActionBlock<Tuple<double, double>>(p =>
        {
            StockInfo1VM.PercentChange = p.Item1;
            StockInfo2VM.PercentChange = p.Item2;
        },
        new ExecutionDataflowBlockOptions()
        {
            TaskScheduler =
            TaskScheduler.FromCurrentSynchronizationContext()
        });
}
```

**[StockInfoBroadCast.cs]**
```cs
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Threading.Tasks.Dataflow;
using TPL_DataFlowBroadCasting_Example.Models;

namespace TPL_DataFlowBroadCasting_Example;

public class StockInfoBroadCast
{
    private readonly BroadcastBlock<List<StockInfoModel>> _stockInfoBroadCaster;

    public StockInfoBroadCast(BroadcastBlock<List<StockInfoModel>> broadcaster)
    {
        _stockInfoBroadCaster = broadcaster;
    }

    public async Task RealTimeData()
    {
        var random1 = new Random();
        var random2 = new Random();

        StockInfoModel stockInfo1 = new()
        {
            StockName = "삼성전자",
            StartPrice = random1.Next(1000, 10000)
        };

        StockInfoModel stockInfo2 = new()
        {
            StockName = "삼성전자우",
            StartPrice = random2.Next(1000, 10000)
        };

        for (int i = 0; i < 100; i++)
        {
            await Task.Delay(1200);
            stockInfo1.CurrentPrice = random1.Next(1000, 10000);
            stockInfo2.CurrentPrice = random2.Next(1000, 10000);
            await _stockInfoBroadCaster.SendAsync(new() { stockInfo1, stockInfo2 });
        }
        _stockInfoBroadCaster.Complete();
        try
        {
            _stockInfoBroadCaster.Completion.Wait();
        }
        catch (Exception ex)
        {
            // 데이터흐름 완료 오류
        }
    }
}
```

**[예제 프로그램 화면]**<br/>
![TPL_DataFlowBroadCasting](https://user-images.githubusercontent.com/13028129/209281202-f38c2190-c578-4b75-8d5c-2b737034f437.gif)<br/><br/>

***

위 코드는 다음 Repository에서 확인할 수 있습니다.<br/>
[Code_check - TPL_DataFlow](https://github.com/tyeom/code_check/tree/main/TestSample/csharp/TPL_DataFlow)



{% include content_adsense.html %}
