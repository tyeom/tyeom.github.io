---
title: (C#) False sharing(거짓 공유) - 병렬처리가 더 느릴 수 있다!
categories: C#
key: 20220706_01
comments: true
tags: C# Parallel 병렬처리 거짓공유 FalseSharing 
---

이번 포스팅은 False sharing(거짓 공유)에 대해 알아보고 이런 현상 이유와 해결하는 방법에 대해 알아보겠습니다.<br/>
우리는 어떤 작업에 대해서 오래 걸리는 작업인 경우 다수의 스레드를 이용해서 병렬처리를 하면 단일 처리를 하는 것 보다 더 빠르게 처리할 수 있다는것을 잘 알고 있습니다.<br/>
그런데 이렇게 빠르게 처리하기 위한 병렬처리 부분이 하드웨어(CPU) 구조적인 문제로 인해 정상 속도가 나오지 않는 문제가 있습니다. 심지어는 단일 스레드로 처리하는 속도 보다
더 느린 결과가 나올 수도 있습니다.

<!--more-->

False sharing(거짓 공유)
-

이런 문제를 결과적으로는 False sharing(거짓 공유) 이라고 불르는데 먼저 CPU의 데이터 처리 방식을 알고 있어야 합니다.<br/>
보통 CPU는 여러개 코어를 가지고 있고 각각 코어에는 데이터 처리를 빠르게 수행하기 위한 L1, L2캐시라는 것이 존재 합니다.<br/>
병렬처리를 하게 되면 2개 이상의 코어에서 처리를 하게 되는데 CPU는 메모리(RAM)에서 한번 로드된 데이터는 이후 빠르게 처리하기 위해서 해당 데이터를 캐시화 하게 됩니다.<br/>
그리고 각 코어는 캐시 되는 데이터는 같은 메모리 주소 영역의 데이터를 로드 한 것에서 데이터가 서로 다르다면 필수적으로 서로 동기화 하려고 합니다<br/>
그런데 동기화 작업이 문제가 되어 오히려 성능을 떨어뜨리게 되는 것입니다.<br/>
예제 코드를 통해 확인해 보겠습니다.<br/>
```cs
public struct SomeData
{
  public int num1;  // size : 4
  public int num2;  // size : 4
}

SomeData _data;
private void FalseSharingEx()
{
  int size = Marshal.SizeOf(typeof(SomeData));  // 4
  
  Stopwatch sw = new Stopwatch();
  sw.Start();
  
  Parallel.Invoke(
      () => this.DoWork1(),
      () => this.DoWork2());

  sw.Stop();

  Console.WriteLine($"SomeData - num1: {_data.num1}");
  Console.WriteLine($"SomeData - num2: {_data.num2}");
  Console.WriteLine($"elapsed time : {sw.Elapsed}");
}

private void DoWork1()
{
  for (int i = 0; i < Int32.MaxValue; i++)
  {
    _data.num1 += i;
  }
}

private void DoWork2()
{
  for (int i = 0; i < Int32.MaxValue; i++)
  {
    _data.num2 += i;
  }
}
```

병렬처리로 SomeData구조체의 num1과 num2의 값을 단순히 증가 시키는 샘플 코드 입니다. 이때 SomeData구조체의 사이즈는 8 Byte(단순 논리상 계산으로 봤을때)로 잡힙니다.<br/>
실행해보면 결과는 다음과 같이 나옵니다.<br/>
```
SomeData - num1: 1073741825
SomeData - num2: 1073741825
elapsed time : 00:00:17.8297557
```

num1과 num2 보다 같은 값으로 증가 되었고 시간은 약 17초정도 걸렸네요<br/>
혹시 병렬처리가 아닌 싱글 스레드에서는 얼마나 걸리는지 확인해 볼까요? 병렬처리 부분 코드를 단순히<br/>
```cs
this.DoWork1();
this.DoWork2();

//Parallel.Invoke(
//    () => this.DoWork1(),
//    () => this.DoWork2());
```
이렇게 바꿔보고 실행해 보면

```
SomeData - num1: 1073741825
SomeData - num2: 1073741825
elapsed time : 00:00:10.6069792
```

싱글 스레드가 더 빠른 결과가 나옵니다! 왜 이런상황이 나오는 걸까요?<br/>
우선 원인을 파악해 보려면 CPU의 캐시 구조와 특성에 대해서 알아보아야 합니다 위 코드내용을 기반으로 보면 num1과 num2는 대략적으로 메모리에 다음과 같이 할당 됩니다.<br/>
![그림3](https://user-images.githubusercontent.com/13028129/177661960-a51ac313-e091-4f04-a373-58133b14c799.png)
<br/>


그리고 병렬 처리되는 PC의 CPU 코어가 4개라고 가정한다면 CPU구조는 다음과 같습니다.<br/>
![그림4](https://user-images.githubusercontent.com/13028129/177659298-4c3c0ec5-5f5e-4df7-94a6-3f75f70546b3.png)

<br/>

프로세스가 실행되면 런타임시에 데이터가 메모리에 할당되고 병렬처리로 CPU의 각 코어에서 연산처리를 하게 됩니다.<br/>
이때 CPU는 빠른 연산처리를 위해 각각의 코어가 가지고 있는 캐시에 데이터를 캐시해서 처리를 하게 되는데 문제는 데이터를 캐시하는 사이즈 입니다.<br/><br/>
CPU에 캐시라인(Cache Line)이 있는데 바로 이 캐시라인의 사이즈 만큼 한번에 데이터를 캐시한다는 것입니다. 캐시라인 사이즈는 모던 X86, X64 플랫폼에서는 보통 64kb 사이즈로
잡혀 있습니다.<br/><br/>
그런데 위에 메모리에 할당된 데이터의 사이즈는 총 8Byte로 캐시라인 사이즈보다 작기 때문에 병렬처리되는 각각 코어 캐시에는 모든 데이터가 한번에 캐시 되서 서로 다른 연산 수행을 하게 됩니다.<br/>

이런 형태에서 병렬처리로 각 코어들은 연산 처리를 진행 합니다.<br/>
```
Core1 : num1 = num1 + 1
Core2 : num2 = num2 + 1
```

![그림5](https://user-images.githubusercontent.com/13028129/177659972-3f523aeb-cc5d-4e94-908d-8dc25cded969.png)
<br/>

이렇게 연산처리가 되면 서로 캐시된 데이터는 물리적으로 다른 값을 가지게 됩니다. 실제 메모리에 있는 하나의 데이터가 각 코어에선 서로 다른 값을 가지고 있는 걸로 착각하고 있는것이죠.<br/>
이렇게 서로 값이 다르다고 판단되어서 캐시 데이터를 서로 동기화 시키는 수행을 하게 됩니다.<br/>
![그림6](https://user-images.githubusercontent.com/13028129/177660729-154112d6-bc91-4200-ad16-92a69980b951.png)
<br/>


위 그림과 같이 동기화를 위해 캐시라인을 이용해서 메모리에서 다시 데이터를 로드하고 캐시 하게 됩니다.<br/>
병렬처리로 수행되는 동안 이런 처리가 계속해서 반복되어지고 있는 것입니다.<br/>
데이터를 게속 로드하고 캐시하고 처리 과정때문에 싱글 스레드 처리 보다도 속도가 오히려 느리게 나오고 있는 것이죠<br/>

그럼 어떻게 해결할 수 있는지 알아볼까요?<br/>
False sharing을 해결하는 방법은 여러가지가 있는데 흔히 두가지 방법으로 처리합니다.

Padding Solution
-

False sharing의 문제는 캐시라인 사이즈 만큼 한번에 로드되어 연산되는 것이 원인 입니다.<br/>
이런 현상을 해결하기 위해 메모리에 값을 할당할때 연산 대상의 값을 근접하게 할당 하지 않고 캐시라인 사이즈 보다 멀리 떨어지도록 할당 시키는 방법이 있습니다.<br/>
이를 Padding solution이라고도 불르는데 이렇게 할당하면 각 코어에 데이터가 로드될때 서로 각각 데이터가 로드되기 때문에 동기화 처리 필요 없이 각각의 코어는 방해받지 않고 연산할 수 있습니다.<br/>
이것을 메모리 할당 형태로 보면 이렇게 됩니다.<br/>
![image](https://user-images.githubusercontent.com/13028129/177661703-90952760-a541-4905-82d2-a25a731c7160.png)
<br/>

해결방법의 코드는 다음과 같습니다.<br/>
```cs
[StructLayout(LayoutKind.Explicit)]
public struct SomeData
{
  [FieldOffset(0)]
  public int num1;  // size : 4
  [FieldOffset(512)]
  public int num2;  // size : 4
}
```

구조체에 num2 할당 위치를 512만큼 거리를 두고 할당 시켜줍니다. 이제 구조체의 사이즈는 516 입니다.<br/>
이렇게 하고 실행을 해보면 결과는 다음과 같습니다.<br/>
```
SomeData - num1: 1073741825
SomeData - num2: 1073741825
elapsed time : 00:00:05.7889446
```

num1, num2모두 동일한 값으로 증가됬음을 확인할 수 있고 처리 시간을 보면 False sharing 현상이 있었을때 보다 눈에 띄게 2배 이상 빨라졌습니다.<br/>
```
False sharing : 약 17초
Single thread : 약 10초
Padding Solution : 약 5초
```

Solve with local variables
-

두번째 해결 방법은 연산 대상을 공유 되는 메모리 값에 직접 접근해서 처리하는 하지 않고 각 병렬처리 수행 스레드에서 지역 변수를 사용해서 연산 되도록 하여 해결할 수 있습니다.<br/>
SomeData _data 데이터는 서로 스레드에서 공유되는 공유 객체입니다. 각 스레드에서 처리되는 연산을 처음부터 공유 데이터를 쓰지 않으면 False sharing 문제는 발생하지 않습니다.<br/>
지역 변수를 사용한 처리는 다음과 같이 수정해볼 수 있습니다.<br/>
```cs
[... FalseSharingEx() 부분 생략 ...]

private void DoWork1()
{
  int tmpNum1 = 0;  // False sharing 해결방법 2 [로컬 변수 사용]
  for (int i = 0; i < Int32.MaxValue; i++)
  {
      tmpNum1 = _data.num1 + i;
  }
  _data.num1 = tmpNum1;
}

private void DoWork2()
{
  int tmpNum2 = 0;  // False sharing 해결방법 2 [로컬 변수 사용]
  for (int i = 0; i < Int32.MaxValue; i++)
  {
      tmpNum2 = _data.num2 + i;
  }
  _data.num2 = tmpNum2;
}
```

결과는 다음과 같습니다.<br/>
```
SomeData - num1: 2147483646
SomeData - num2: 2147483646
elapsed time : 00:00:05.3625496
```

이렇게 False sharing의 원인과 해결방법에 대해 알아보았는데 닷넷에서는 C나 C++ 네이티브 언어와는 다르게 변수 선언시 메모리 할당 처리를 디테일적으로 할 수 없고<br/>
런타임시 메모리에 할당되기 때문에 병렬처리 설계를 할때 이런 부분을 잘 파악하면서 설계해야 합니다.<br/>
또한 False sharing은 코드로만 봤을땐 오류가 아니기 때문에 개발 시점에서도 눈에 띄지 않기 때문에 모르고 그냥 지나치게 될 수 있고 성능이 안좋은 상태로 서비스 될 수 있습니다.

메모리 접근 패턴
-

관련해서 같은 문제로 행렬 구조의 배열을 탐색할때 열(Col)에서 행(Row)을 탐색할때 **근접해 있는 행의 앞.뒤도 같이 읽기 때문에 속도가 느린** 반면 행(Row)에서 각 열(Col)을 탐색하는 것이 더 빠른 속도 결과를 볼 수 있습니다.<br/>
```cs
using System;
using System.Diagnostics;

class Program {
  public static void Main() {
    const int SIZE = 10000;
    int[,] matrix = new int[SIZE, SIZE];

    while (true) {
      // Faster
      Stopwatch sw = Stopwatch.StartNew();
      for (int row = 0; row < SIZE; row++) {
        for (int column = 0; column < SIZE; column++) {
          matrix[row, column] = (row * SIZE) + column;
        }
      }
      Console.WriteLine(sw.Elapsed);

      // Slower
      sw = Stopwatch.StartNew();
      for (int column = 0; column < SIZE; column++) {
        for (int row = 0; row < SIZE; row++) {
          matrix[row, column] = (row * SIZE) + column;
        }
      }
      Console.WriteLine(sw.Elapsed);

      Console.WriteLine("=================");
      Console.ReadLine();
    }
  }
}
```

[.NET Matters(False Sharing) 참고 링크](https://docs.microsoft.com/en-us/archive/msdn-magazine/2008/october/net-matters-false-sharing)

{% include content_adsense.html %}
