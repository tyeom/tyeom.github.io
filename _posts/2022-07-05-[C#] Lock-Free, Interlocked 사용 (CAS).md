---
title: (C#) Lock-Free, Interlocked 사용 (CAS)
categories: C#
key: 20220705_02
comments: true
tags: C# 동기화 Lock-Free 원자적 유저모드 Interlocked CAS
---

이전 포스팅에서 Volatile을 통한 원자적 동기 처리를 알아 보았습니다. [링크](https://tyeom.github.io/c%23/2022/07/05/C-%EB%8F%99%EA%B8%B0%ED%99%94-%EC%84%A4%EB%AA%85-%EB%B0%8F-Volatile-%EB%8F%99%EA%B8%B0%ED%99%94.html)<br/>
이렇게 원자적 처리를 이용해서 읽기, 쓰기, 비교 처리를 사용해서 명시적인 Lock 사용 없이 Non-Blocking 으로 동기화 처리 할 수 있는데 이러한 방법을 Lock-Free 동기화 라고 부릅니다.<br/>
이번 글은 Lock-Free가 왜 필요한지 또 어떻게 구현되고 사용할 수 있는지에 대해 알아보겠습니다.

<!--more-->

Lock-Free 동기화 처리는 왜 하는 걸까요? 비동기 환경에서 동기화 처리를 하는데 있어 Lock을 이용한 블로킹 처리는 다음과 같은 잠재적인 문제 발생의 여지가 있습니다.<br/>
- 성능 하락 : 여러 스레드가 하나의 공유자원을 획득하기 위해 경쟁하는데 CPU는 어떤 스레드에 자원을 허락해야 할지 연산에 있어 성능 저하가 발생 됩니다.
- 라이브락(livelock) : 어떤 스레드가 락을 획득하고 해제되지 않으면 영원히 블로킹 될 수 있습니다. (* 커널 모드에서 동기화 처리시 영원히 블로킹되는 현상은 **데드락(deadlock)** 이라 부릅니다.)

이러한 이유로 Non-Blocking 적으로 동기화 처리 방법에 있어 Lock-Free 방식이 많이 사용 됩니다.

Compare And Exchange (CAS)
-

Lock-Free를 구현하는 알고리즘은 여러가지가 있는데 선형 자료구조인 Queue또는 Stack, 이진트리 구조를 사용합니다. 이런 자료구조를한 처리 연산 방법에 해당 되는 것이 CAS 입니다. <br/>
CAS는 특정 값을 비교해서 같다면 X값으로 교체 하는 연산 처리를 말합니다.<br/>
이런 알고리즘을 이용해서 동기화 처리를 Blocking(차단) 하지 않고 Non-Blocking 하게 처리 되도록 하여 성능을 유지하고 라이브락 또는 데드락 되는 현상을 방지할 수 있습니다.<br/>

Interlocked
-

C# 에서는 Lock-Free를 비교적 쉽게 구현할 수 있습니다. **<span style="color: rgb(107, 173, 222);">System.Threading.Interlocked</span>** 클래스에서는 CAS를 사용할 수 있는<br/>
CompareExchange() 메서드가 제공됩니다.<br/>

바로 코드로 구현해 보자면 다음과 같이 구현할 수 있습니다.<br/>
```cs
// CAS를 이용한 Non-Blocking 동기 처리 클래스
public class CAS_Lock
{
        // 0 = false
        // 1 = true
        private volatile int _lock = 0;

        public void Lock()
        {
            while(true)
            {
                if(Interlocked.CompareExchange(ref _lock, 1, 0) == 0)
                {
                    return;
                }
            }
        }

        public void Free()
        {
            _lock = 0;
        }
}

CAS_Lock _cas = new CAS_Lock();
private async void StartWorker()
{
  Enumerable.Range(0, 10).ToList().ForEach(item =>
  {
    Task.Run(this.Worker);
  });
}

private void Worker()
{
  _cas.Lock();

  Console.WriteLine($"Lock 획득! - id : {System.Threading.Thread.CurrentThread.ManagedThreadId} / {DateTime.Now.Second}s");
  Thread.Sleep(1000);

  _cas.Free();
}
```

**<span style="color: rgb(107, 173, 222);">System.Threading.Interlocked</span>** 를 사용해서 Non-Blocking 동기 처리를 수행하는 CAS_Lock 클래스를 구현하고<br/>
스레드개가 동시에 위 CAS_Lock 클래스를 사용하여 동기화 처리를 한 코드 입니다.

위 코드를 실행해보면 1초마다 각 스레드가 순차적으로 Lock 상태에 따라 Worker() 메서드의 내부코드를 처리 하는걸 확인할 수 있습니다.<br/>
**출력결과**
```
Lock 획득! - id : 10 / 58s
Lock 획득! - id : 6 / 0s
Lock 획득! - id : 14 / 1s
Lock 획득! - id : 8 / 2s
Lock 획득! - id : 12 / 3s
Lock 획득! - id : 7 / 4s
Lock 획득! - id : 3 / 5s
Lock 획득! - id : 11 / 6s
Lock 획득! - id : 9 / 7s
Lock 획득! - id : 5 / 8s
```

이 처리 과정을 단순히 그려보면<br/>
![그림1](https://user-images.githubusercontent.com/13028129/177270160-64bf7673-3534-4f8f-8b3d-2773395e074f.png)<br/>

＊그림1<br/>

그림1과 같습니다.

{% include content_adsense.html %}
