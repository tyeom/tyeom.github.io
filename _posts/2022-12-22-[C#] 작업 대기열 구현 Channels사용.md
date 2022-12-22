---
title: (C#) 작업 대기열 구현 Channels사용
categories: C#
key: 20221222_01
comments: true
tags: .NET C# Thread 대기열 Channel Producer Consumer
---

.NET Core 3.0부터 포함 API로 제공되는 **<span style="color: rgb(107, 173, 222);">System.Threading.Channels.Channel&lt;T&gt;</span>** 클래스에 대해 알아보도록 하겠습니다.<br/>
**<span style="color: rgb(107, 173, 222);">System.Threading.Channels.Channel&lt;T&gt;</span>** 클래스는 비동기 작업에서 메세지 처리를 스레드로 부터 안전하게 처리할 수 있도록<br/>
간편한 방식으로 작업 대기열을 구성할 수 있도록 제공합니다.

<!--more-->

**<span style="color: rgb(107, 173, 222);">System.Threading.Channels.Channel&lt;T&gt;</span>** 클래스는 **<span style="color: rgb(107, 173, 222);">System.Collections.Concurrent.ConcurrentQueue&lt;T&gt;</span>** 클래스와 동일하게 큐에 항목을 추가하고 꺼내는 제공을 하지만
보다 비동기적으로 좀 더 많은 기능을 제공하고 있습니다. 또한 내부적으로 **<span style="color: rgb(107, 173, 222);">System.Threading.Tasks.ValueTask </span>** 구조체를 사용하여 성능과 메모리의 효율성도 뛰어납니다.
그럼 간단한 예제를 통해 알아보겠습니다.

이 글에서 다루는 코드는 다음 Repository에서 확인할 수 있습니다.<br/>
[Code_check - ChannelExample](https://github.com/tyeom/code_check/tree/main/TestSample/csharp/ChannelExample)


기본적인 ConcurrentQueue 처리
-

**<span style="color: rgb(107, 173, 222);">System.Collections.Concurrent.ConcurrentQueue&lt;T&gt;</span>** 클래스를 이용하여 기본적인 생산자와 소비자 패턴을 다음과 같이 작성해볼 수 있습니다.<br/>
```cs
using System.Collections.Concurrent;

ConcurrentQueueEx ex1 = new();
ex1.Start();

public class ConcurrentQueueEx
{
    ConcurrentQueue<int> _workingCollection = new();

    public async void Start()
    {
        Task.Run( async () =>
        {
            for (int i = 0; i < 10; i++)
            {
                _workingCollection.Enqueue(i);
                await Task.Delay(1000);
            }
        });

        while (true)
        {
            int num;
            if (_workingCollection.TryDequeue(out num))
                Console.WriteLine($"> {num}");
        }
    }
}
```

**[출력결과]**<br/>
```
> 0
> 1
> 2
> 3
> 4
> 5
> 6
> 7
> 8
> 9
```

0에서 9까지의 항목을 대기열로 사용하는 큐에 넣고 추가된 항목은 약 1초 주기로 출력되는 간단한 코드 입니다.<br/>
보는 것 처럼 **<span style="color: rgb(107, 173, 222);">System.Collections.Concurrent.ConcurrentQueue&lt;T&gt;</span>** 클래스는 기본적으로 항목을 가져올때 '대기'(스레드 차단이 아닌) 처리를 지원하는 기본 메서드가 없습니다.
그렇기 때문에 지속적으로 loop안에서 항목 체크를 반복하게 됩니다. 이런 부분에서 비효율적으로 처리되고 있으며, 물론 추가적으로 직접 대기하는 기능을 추가하거나  **<span style="color: rgb(107, 173, 222);">
System.Threading.SemaphoreSlim</span>** 클래스 등을 사용해서 동기화 구현을 추가하면 되지만 여러가지 상황을 고려해야 합니다.<br/>

Channel&lt;T&gt; 사용
-

반면 **<span style="color: rgb(107, 173, 222);">System.Threading.Channels.Channel&lt;T&gt;</span>** 클래스는 비동기로 대기 처리를 지원합니다.<br/>
위와 동일한 코드를 **<span style="color: rgb(107, 173, 222);">System.Threading.Channels.Channel&lt;T&gt;</span>** 클래스로 작성해보면 다음과 같습니다.<br/>
```cs
// Channel<T> 사용
// 항목 무한 추가 채널 생성
var channel = Channel.CreateUnbounded<int>();

// 외부로 제공하는 경우 이렇게 쓰기 전용과 읽기 전용 각 개별로 만들어서 제공하는 것이 좋다.
var producer = new Producer<int>(channel.Writer);
var consumer = new Consumer<int>(channel.Reader);

// 생산자
Task.Run(async () =>
{
    for (int i = 0; i < 10; i++)
    {
        bool result = producer.TryWrite(i);
    }
    // 채널 닫음 [* 추후 Channel Close 부분에서 설명]
    producer.Complete();
});

var reader = async () =>
{
    try
    {
        while (true)
        {
            var item = await consumer.ReadAsync();
            Console.WriteLine($"> {item}");
        }
    }
    catch (ChannelClosedException ex)
    {
        Console.WriteLine($"Channel was closed!");
    }
};

reader();
```

**[Producer.cs]**
```cs
/// <summary>
/// 생산 클래스 - Writer 전용
/// </summary>
/// <typeparam name="T"></typeparam>
public class Producer<T>
{
    private readonly ChannelWriter<T> _channelWriter;

    public Producer(ChannelWriter<T> channelWriter)
    {
        _channelWriter = channelWriter;
    }

    public bool TryWrite(T item)
    {
        return _channelWriter.TryWrite(item);
    }

    public ValueTask WriteAsync(T item)
    {
        return _channelWriter.WriteAsync(item);
    }

    public ValueTask<bool> WaitToWriteAsync()
    {
        return _channelWriter.WaitToWriteAsync();
    }

    public void Complete()
    {
        _channelWriter.Complete();
    }
}
```

**[Consumer.cs]**
```cs
/// <summary>
/// 소비 클래스 - Reader 전용
/// </summary>
/// <typeparam name="T"></typeparam>
public class Consumer<T>
{
    private readonly ChannelReader<T> _channelReader;

    public Consumer(ChannelReader<T> channelReader)
    {
        _channelReader = channelReader;
    }

    public ValueTask<T> ReadAsync()
    {
        return _channelReader.ReadAsync();
    }

    public IAsyncEnumerable<T> ReadAllAsync()
    {
        return _channelReader.ReadAllAsync();
    }
}
```

**[출력결과]**<br/>
```
> 0
> 1
> 2
> 3
> 4
> 5
> 6
> 7
> 8
> 9
```

위에서 구현했던 동일한 생산자와 소비자 패턴 형태 입니다. 코드가 다소 길어 졌는데 **<span style="color: rgb(107, 173, 222);">System.Threading.Channels.Channel&lt;T&gt;</span>** 클래스를 이용해서 
Writer와 Reader를 각각 분리해서 별도로 래핑 클래스를 사용했습니다.<br/>
여기서 살펴 볼 부분은 **<span style="color: rgb(107, 173, 222);">System.Threading.Channels.ChannelReader&lt;T&gt;</span>** 클래스의 **<span style="color: rgb(107, 173, 222);">System.Threading.Channels.ChannelReader&lt;T&gt;.ReadAsync()</span>** 메서드를 
이용해서 항목을 읽고 있는 점 입니다. **<span style="color: rgb(107, 173, 222);">System.Threading.Channels.ChannelReader&lt;T&gt;.ReadAsync()</span>** 메서드는 이름에서 처럼 비동기 대기를 지원하기 때문에
항목이 추가될 때까지 스레드 차단 없이 대기할 수 있는 이점이 있습니다.<br/><br/>

또한 항목의 Read처리는 멀티 스레드 환경에서도 내부적으로 동기화 처리로 되어 있어 스레드 안전 하므로 별도 동기화 처리 없이 사용 가능합니다.<br/>
다음은 멀티 스레드에서 다중으로 Read하는 코드 입니다.<br/>
```cs
...[생략]...

// reader 메서드에 스레드Id 출력 추가
var reader = async () =>
{
    try
    {
        while (true)
        {
            var item = await consumer.ReadAsync();
            Console.WriteLine($"> {item} / {Thread.CurrentThread.ManagedThreadId}");
        }
    }
    catch (ChannelClosedException ex)
    {
        Console.WriteLine($"Channel was closed!");
    }
};

// 소비자 - 멀티 스레드
Parallel.For(0, 2, (_) => reader());
```

**[출력결과]**<br/>
```
> 1 / 1
> 0 / 5
> 3 / 5
> 4 / 5
> 5 / 5
> 6 / 5
> 7 / 5
> 8 / 5
> 2 / 1
> 9 / 5
```

중복 항목 없이 모든 항목을 잘 읽어오는 걸 볼 수 있습니다.

Channel Close
-

더 이상의 대기열이 사용되지 않은 경우 해당 Channel을 Close해서 종료 시킬 수 있습니다. Channel이 Close 되었다고 바로 종료되는 것은 아닙니다.<br/>
구독자측에서 대기열의 모든 메세지를 다 읽기 전 까지는 Close되지 않고 마지막 메세지를 읽은 직후 Close 됬다고 예외처리로 알 수 있습니다.<br/>
다음 예제는 위와 동일한 코드에서 메세지를 Read하는 부분에서 지연처리를 하여 이미 Close 되었어도 계속해서 메세지를 읽을 수 있음을 확인해볼 수 있습니다.<br/>
```cs
var reader = async () =>
{
    try
    {
        while (true)
        {
            var item = await consumer.ReadAsync();
            Console.WriteLine($"> {item}");
            // 약 1초 지연
            await Task.Delay(1000);
        }
    }
    catch (ChannelClosedException ex)
    {
        Console.WriteLine($"Channel was closed!");
    }
};
```

마지막 메세지를 읽은 후 'Channel was closed!' 가 출력됨을 확인해 볼 수 있습니다. 그리고 **<span style="color: rgb(107, 173, 222);">System.Threading.Channels.ChannelReader&lt;T&gt;</span>** 클래스는 비동기 스트림을 지원합니다.
관련해서 **<span style="color: rgb(107, 173, 222);">System.Threading.Channels.ChannelReader&lt;T&gt;.ReadAllAsync()</span>** 메서드를 제공하는데 해당 메서드의 반환은 **<span style="color: rgb(107, 173, 222);">System.Collections.Generic.IAsyncEnumerable&lt;T&gt;</span>** 타입으로
다음과 같이 읽어볼 수 도 있습니다.<br/>
```cs
...[중간 생략]...

var reader2 = async () =>
{
    await foreach (var item in consumer.ReadAllAsync())
    {
        Console.WriteLine(item);
        await Task.Delay(1000);
    }
};

// 소비자
reader2();
```

비동기스트림의 await foreach 처리로 channel이 close 되었다면 더 이상의 읽을 메세지가 없으므로 예외 없이 바로 foreach처리 종료로 끝나게 됩니다.<br/>
여기서 producer.Complete() 처리 없이 모든 메세지를 읽은 후 추가로 항목을 Write하면 비동기스트림에서 계속 항목을 읽을 수 있습니다.<br/>
```cs
// 채널 닫음 [* 추후 Channel Close 부분에서 설명]
//producer.Complete();  <- 주석 처리

...[중간 생략]...

// 소비자
reader2();

// ReadAllAsync 이후 새로운 항목 추가
Console.ReadLine();
producer.TryWrite(100);
Console.ReadLine();
```

**[출력결과]**<br/>
```
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

100
```

Back Pressure 설정
-

**<span style="color: rgb(107, 173, 222);">System.Threading.Channels.Channel&lt;T&gt;</span>** 을 생성할때 항목 요소의 조건과 모드 옵션을 지정해서 생성할 수 있습니다.<br/>
예를 들어 대기열의 메세지 갯수를 5개로 제한하고 5개 꽉차 있을 경우 Write를 대기처리 할 수 있도록 옵션으로 제한 설정이 가능합니다. 다음 예제는 메세지 갯수를 5개로 제한하고 꽉 차있는 경우 대기모드로 사용되는 예제 입니다.<br/>
```cs
// Channel<T> 사용
// 채널에 배합(Back Pressure)조건 옵션 설정
var channelOptions = new BoundedChannelOptions(5)
{
    FullMode = BoundedChannelFullMode.Wait
};

var channel = Channel.CreateBounded<int>(channelOptions);

// 외부로 제공하는 경우 이렇게 쓰기 전용과 읽기 전용 각 개별로 만들어서 제공하는 것이 좋다.
var producer = new Producer<int>(channel.Writer);
var consumer = new Consumer<int>(channel.Reader);

// 생산자
Task.Run(async () =>
{
    for (int i = 0; i < 10; i++)
    {
        bool result = producer.TryWrite(i);
        if (result == false)
        {
            Console.WriteLine("대기열 Full - 대기");
            await producer.WaitToWriteAsync();
        }
        else
        {
            Console.WriteLine($"{i} 추가");
        }
        await Task.Delay(1000);
    }
});

var reader2 = async () =>
{
    await Task.Delay(6000);
    await foreach (var item in consumer.ReadAllAsync())
    {
        Console.WriteLine($"소비 - {item}");
    }
};

// 소비자
reader2();
```

**[출력결과]**
```
0 추가
1 추가
2 추가
3 추가
4 추가
대기열 Full - 대기
소비 - 0
소비 - 1
소비 - 2
소비 - 3
소비 - 4
6 추가
소비 - 6
7 추가
소비 - 7
8 추가
소비 - 8
9 추가
소비 - 9
```

항목을 Read하는 곳에선 약 6초 후에 Read처리 되도록 딜레이를 주었습니다. 그리고 출력결과를 보면 약 1초 간격으로 항목이 추가 되고, 0 ~ 4까지 추가 이후 대기열이 꽉 찼기 때문에<br/>
**<span style="color: rgb(107, 173, 222);">System.Threading.Channels.ChannelWriter&lt;T&gt;.WaitToWriteAsync()</span>** 메서드에 의해 대기 되었다가 6초 후 모든 메세지가 읽어진후 
하나씩 항목이 Write될때마다 바로 Read되고 있는 결과를 확인할 수 있습니다.<br/>
**<span style="color: rgb(107, 173, 222);">System.Threading.Channels.ChannelWriter&lt;T&gt;.WaitToWriteAsync()</span>** 메서드외 **<span style="color: rgb(107, 173, 222);">System.Threading.Channels.ChannelWriter&lt;T&gt;.WriteAsync()</span>** 메서드로 대기와 동시에 Write할 수 있는 상황이 되었을때
Write할 수 있는 메서드도 있습니다. 또한 **<span style="color: rgb(107, 173, 222);">System.Threading.Channels.BoundedChannelOptions</span>** 의 FullMode는 Wait옵션 외 'DropNewest', 'DropOldest', 'DropWrite' 옵션이 있으니 상황에 맞게
사용하면 됩니다.<br/>
이렇게 **<span style="color: rgb(107, 173, 222);">System.Threading.Channels.Channel&lt;T&gt;</span>** 클래스를 이용해서 비동기로 메세지 대기열 처리에 대해 구현하고 사용하는 방법을 알아보았습니다.<br/><br/>

***

위 코드는 다음 Repository에서 확인할 수 있습니다.<br/>
[Code_check - ChannelExample](https://github.com/tyeom/code_check/tree/main/TestSample/csharp/ChannelExample)



{% include content_adsense.html %}
