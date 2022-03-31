---
title: (C#) IAsyncEnumerable<T> 비동기 스트림
categories: C#
key: 20220331_02
comments: true
tags: C# 비동기 비동기스트림 yield IAsyncEnumerable&lt;T&gt;
---

C# 5버전에서 제공되는 async / await을 사용해서 간편하게 비동기 처리에 대해 결과를 작업을 간단하게 구현할 수 있습니다.<br/>
**async／await 사용의 대기 처리** [링크](https://tyeom.github.io/c%23/2022/01/18/C-%EB%8B%B7%EB%84%B7-%EC%8A%A4%EB%A0%88%EB%93%9C-%EB%B9%84%EB%8F%99%EA%B8%B0-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D-TAP-(async-await).html#asyncawait-%EC%82%AC%EC%9A%A9%EC%9D%98-%EB%8C%80%EA%B8%B0-%EC%B2%98%EB%A6%AC)

그런데 연속적인 결과를 비동기로 반환하고 대기 처리 하는 것은 불가능 했습니다.
  
<!--more-->

C# 8버전에서는 **<span style="color: rgb(107, 173, 222);">System.Collections.Generic.IAsyncEnumerable&lt;T&gt;</span>** 인터페이스가 추가 되어
연속적인 결과 스트림에 대해 비동기 처리가 구현이 가능합니다.

그럼 **<span style="color: rgb(107, 173, 222);">System.Collections.Generic.IAsyncEnumerable&lt;T&gt;</span>** 대해 간단한 예제를 통해 어떻게 사용할 수 있는지 보겠습니다.

IAsyncEnumerable&lt;T&gt;
-

**<span style="color: rgb(107, 173, 222);">System.Collections.Generic.IAsyncEnumerable&lt;T&gt;</span>** 인터페이스는
**<span style="color: rgb(107, 173, 222);">System.Collections.Generic.IAsyncEnumerator&lt;T&gt;</span>** 인터페이스를 반환하는 GetAsyncEnumerator() 메서드가 구현 되어 있습니다.

**<span style="color: rgb(107, 173, 222);">System.Collections.Generic.IAsyncEnumerator&lt;T&gt;</span>** 인터페이스는 Enumerator의 기능으로서 컬렉션에 있는 요소를
탐색할 수 있는 기능을 제공해 주는데 MoveNextAsync()메서드를 통해 탐색할 수 있습니다.

MoveNextAsync()메서드는 **<span style="color: rgb(107, 173, 222);">System.Threading.Tasks.ValueTask&lt;TResult&gt;</span>** 타입으로 Awaiter사용이 가능합니다.

즉 다음과 같이 순차적으로 탐색을 할 수 있는 것 입니다.
```cs
IAsyncEnumerable<T>의 GetAsyncEnumerator()메서드를 통해 열거 기능을 받아서
while(await MoveNextAsync()) {
  ...
}
```

위 코드는 foreach의 syntactic sugar로 foreach(...){ } 코드가 저런 형태로 동작 됩니다.

IAsyncEnumerable&lt;T&gt; 사용
-

**<span style="color: rgb(107, 173, 222);">System.Collections.Generic.IAsyncEnumerable&lt;T&gt;</span>**는 네트워크나 IoT 장비를 통해 스트림 데이터와 같은
실시간 연속 데이터를 비동기 적으로 처리하고자 할때 효율적으로 사용할 수 있습니다.

다음 예제 코드는 네트워크상 에서 실시간으로 비동기로 채팅 데이터를 받아 동기적으로 처리 하는 예제 코드 입니다.

우선 동기적 처리로 실시간 채팅 데이터를 받을 수 있는 간단한 ChatStream클래스를 작성해 봅니다.
```cs
public class ChatStream
{
  private CancellationTokenSource _cancel;
  // 채팅 데이터가 수신 되기 까지 대기 처리 용도
  private readonly SemaphoreSlim _sem;
  // 수신 받은 채팅 데이터
  private ConcurrentQueue<String> _chatCollection;
  
  public ChatStream()
  {
    _cancel = new CancellationTokenSource();
    _sem = new SemaphoreSlim(0);
    _chatCollection = new ConcurrentQueue<String>();
  }

  public async IAsyncEnumerable<string> GetChatAsync()
  {
    while (_cancel.IsCancellationRequested == false)
    {
      // 대기
      await _sem.WaitAsync();
      
      string chat;
      // 큐에서 메세지를 꺼내서 return
      if (_chatCollection.TryDequeue(out chat))
      {
          yield return chat;
      }
    }
  }
  
  public void Send(string chat)
  {
    _chatCollection.Enqueue(chat);
    _sem.Release(1);
  }
}
```

채팅 데이터가 전송 되면 큐에 넣어두고 **<span style="color: rgb(107, 173, 222);">System.Threading.SemaphoreSlim</span>** 을 사용해 동기화 시켰습니다.

ChatStream의 채팅 스트림 데이터 처리 코드는 다음과 같습니다.

**[MainWindow.xaml]**
```xml
<Grid>
  <Grid.RowDefinitions>
    <RowDefinition Height="*"/>
    <RowDefinition Height="Auto"/>
  </Grid.RowDefinitions>
  
  <ListView Grid.Row="0"
            ItemsSource="{Binding ChatList}">
    <ListView.ItemTemplate>
      <DataTemplate>
        <TextBlock Text="{Binding .}"/>
      </DataTemplate>
    </ListView.ItemTemplate>
  </ListView>
  
  <StackPanel Grid.Row="1"
              Orientation="Horizontal">
    <TextBox x:Name="xChatTxt" Width="300"/>
      <Button x:Name="xSendBtn" Width="100"
              Content="Send"
              Click="xSendBtn_Click"/>
  </StackPanel>
</Grid>
```

**[MainWindow.xaml.cs]**
```cs
public partial class MainWindow : Window
{
  ChatStream _chatStream = null;
  public MainWindow()
  {
    InitializeComponent();
    
    ChatList = new ObservableCollection<string>();
    this.DataContext = this;
    this.Loaded += MainWindow_Loaded;
  }
  
  public ObservableCollection<String> ChatList  // 샘플이니깐 단순하게 속성 바인딩
  {
    get;set;
  }
  
  private async void MainWindow_Loaded(object sender, RoutedEventArgs e)
  {
    _chatStream = new ChatStream();
    // 실시간 채팅 데이터 열거 await 처리
    // **<span style="color: rgb(107, 173, 222);">System.Collections.Generic.IAsyncEnumerable&lt;String&gt;</span>** 객체를 Awaiter하게 열거한다.
    await foreach (var chat in _chatStream.GetChatAsync())
    {
      ChatList.Add(chat);
    }
  }
  
  private async void xSendBtn_Click(object sender, RoutedEventArgs e)
  {
    // ChatStream의 채팅 데이터 큐에 메세지 보관
    _chatStream.Send(this.xChatTxt.Text);
    this.xChatTxt.Text = String.Empty;
  }
}
```

실행시 결과는 다음과 같습니다.

![IAsyncEnumerable_example](https://user-images.githubusercontent.com/13028129/161003353-7226687d-901a-47d9-a225-10d48a708f99.gif)


{% include content_adsense.html %}
