---
title: (C#) Method Intercept(AOP)
categories: C#
key: 20230206_01
comments: true
tags: C# Method_Intercept Intercept 가로채기 AOP Fody Cauldron Cauldron.Intercept
---

Method Intercept는 Method 호출을 차단하고 추가적인 작업 처리를 위해 사용할 수 있는 기술 입니다.<br/>
이러한 처리는 보통 공통으로 사용되는 작업에 대해 Method 호출을 차단하고 공통 작업 수행을 처리할때 많이 사용 됩니다.<br/>
가령 공통으로 메서드 호출시 로그 기록을 처리하거나 예외 발생 처리를 할 때 사용될 수 있습니다.

<!--more-->

이렇게 주 처리 작업 이외의 부가적인 공통된 작업에 대해 각각 모듈화하여 처리하는 방법론은 **AOP** 라고 불리우는데 **Aspect Oriented Programming** 의 약자로 관점 지향 프로그래밍 이라고 불립니다.<br/>
코드상에서 자주 반복되는 코드들을 흩어져 있는 관심사로 표현하는데 이 공통된 특징들을 공통된 모듈로 처리하여 하나의 작업으로 묶는 것을 말합니다.<br/>
![image](https://user-images.githubusercontent.com/13028129/216884207-d0cb47ef-a615-4b65-8413-14db32308869.png)<br/>
위 그림과 같이 공통으로 사용되는 흩어진 관심사를 Aspect로 묶고 주요 핵심 로직과 분리해서 Proxy 처리 등으로 대체해서 사용될 수 있습니다.<br/><br/>

AOP를 적용하는 방법은 여러가지가 있습니다.<br/>
디자인 패턴중 Decorator pattern을 적용해서 구현할 수 있고, 닷넷이나 자바 같이 관리 언어에서는 컴파일되어 만들어진 IL코드(중간언어)에 새로운 Aspect 코드를 삽입해서 처리하는 방법, 
Proxy를 사용하여 메서드 호출을 위임하고 Aspect 처리 하는 방법 등이 있는데 여기서는 .NET Framework에서 제공하는 Proxy와 **(.NET Core 이상에서는 Dynamic Object를 이용해서 구현할 수 있습니다.)** Fody의 Cauldron.Intercept 라이브러리를 사용하여 
Method Intercept 구현에 대해 알아보겠습니다.<br/><br/>


> 이 글에서 다루는 코드는 다음 Repository에서 확인할 수 있습니다.<br/>
> [code_check - Method_Interception](https://github.com/tyeom/code_check/tree/main/TestSample/csharp/Method_Interception)

RealProxy
-

**<span style="color: rgb(107, 173, 222);">RealProxy</span>** 클래스는 WCF와 .NET Remoting에서 사용 되는 오래전 부터 있었던 클래스인데 Proxy를 통해서 




{% include content_adsense.html %}
