---
title: (C#) 미세한 분기(if, else if)리펙토링
categories: C#
key: 20230116_01
comments: true
tags: 리펙토링 refactoring design_pattern 분기 if elseif 분기문줄이기
---

코드 리펙토링은 유지보수 하는데에 있어 중요한 작업입니다.<br/>
이번 글 에서는 리펙토링 기법중 가장 빈번하게 만나고 누구나 한번쯤은 고민해 보았을 분기문에 대해 살펴보겠습니다.

<!--more-->

코드를 작성하는데 분기 처리(if else / else if)는 아주 흔하게 사용되고 있습니다. 그런데 분기문이 길어지고 복잡해 진다면 그만큼 가독성이 떨어지고<br/>
더 나아가 미세하지만 성능 저하까지 나타날 수 있습니다. 그래서 어떻게 분기문을 줄일 수 있는지 각 상황별로 알아보겠습니다.<br/><br/>


Early Return
-

Early Return은 핵심 로직 부분을 뒤로 미루고 명확한 유효성 체크 검사를 먼저 체크해서 조건에 맞지 않다면 루틴을 즉시 벗어나도록 처리하는 것입니다.<br/>
early exit 또는 first check, fast return 등 불리우는 명칭은 다양한데 모두 같은 의미를 가집니다.<br/><br/>

예제를 통해 알아보겠습니다.<br/><br/>

```cs
bool? result = null;
if( objInstance == null ) {
  result = null;
}
else if(objInstance.A > 100) {
  result = true;
}
else if(objInstance.B > 200) {
  result = false;
}
else {
  ...
}

return result;
```

위 코드를 Early Return로 바꾸어 보면 다음과 같이 바꿔볼 수 있습니다.<br/><br/>

```cs
if( objInstance == null ) {
  return null;
}

if(objInstance.A > 100) {
  ...
  return true;
}
if(objInstance.B > 200) {
  ...
  return false;
}
```

하지만 Early Return 몇가지의 비판점이 있습니다.<br/>
- 함수는 오직 하나의 exit point를 가져야 하는 단일 엔트리, 단일 종료(Single Entry, Single Exit, SESE) 규칙에 어긋난다.
- 하나의 함수에 많은 return 구문은 가독성을 오히려 떨어트린다.
- 빠른 return 이후 상황에 따른 경우의 수 처리 부분에 간과할 여지가 있다.
- 제 3자의 동료가 코드를 분석할때 모든 경우의 수를 한번에 이해하기 어렵다.

이러한 비판 이유가 있습니다. 그렇기 때문에 이러한 고려를 생각해보고 상황에 맞게 적절히 쓰는 것이 좋습니다.


Dictionary 이용
-

때로는 분기처리를 Dictionary 자료구조를 사용하여 분기문을 대체 처리할 수 있습니다.<br/>
다음 예시는 MediaType의 enum 상수 값에 따라 wav파일을 재생하기 위해 파일 경로를 분기처리 하는 코드 입니다.<br/><br/>

```cs
EMediaType mediaType = this.ResponseMediaType();
switch(mediaType)
{
    case EMediaType.AMedia:
        this.PlayMedia("Resources/A.wav");
        break;
    case EMediaType.BMedia:
        this.PlayMedia("Resources/B.wav");
        break;
    case EMediaType.CMedia:
        this.PlayMedia("Resources/C.wav");
        break;
}

public enum EMediaType
{
    AMedia,
    BMedia,
    CMedia
}
```

위 코드는 다음과 같이 바꿔볼 수 있습니다.<br/><br/>

```cs
Dictionary<EMediaType, string> mediaMap = new()
{
    [EMediaType.AMedia] = "Resources/A.wav",
    [EMediaType.BMedia] = "Resources/B.wav",
    [EMediaType.CMedia] = "Resources/C.wav"
};

EMediaType mediaType = this.ResponseMediaType();
this.PlayMedia(mediaMap[mediaType]);
```

이 처럼 switch 구문 없이 바로 PlayMedia메서드 호출이 가능합니다.


범위 패턴 사용
-

다음 상황은 범위 조건에 따른 분기문을 간단하게 바꿔보도록 해보겠습니다.<br/>
다음과 같은 예시 코드가 있습니다.<br/><br/>

```cs
/// 0 ~ 10 : F학점
/// 11 ~ 20 : D학점
/// 21 ~ 30 : C학점
/// 31 ~ 40 : B학점
/// 41 ~ : A학점
var getScore = (int score) =>
{
    if (score <= 10) return "F학점";
    if (score <= 20) return "D학점";
    if (score <= 30) return "C학점";
    if (score <= 40) return "B학점";
    else return "A학점";
};
```

이 처럼 각 점수 범위 별로 학점을 반환하는 메서드 인데 범위 조건을 보면 공통 패턴이 있는데 바로 10씩 늘어나는 조건을 찾을 수 있습니다.<br/>
이 조건을 공식화 하면 0 ~ 4 총 5개의 각각 값으로 바꾸어 처리 할 수 있습니다.<br/>
(입력받은 파라메터값에 -1) 을 하고 이 값을 다시 10으로 나누면 0, 1, 2, 3, 4 로 처리할 수 있으며 
4 이상의 값과 0 이하의 값은 **<span style="color: rgb(107, 173, 222);">System.Math</span>** 클래스의 Min() 메서드와 Max() 메서드를 이용해 
4보다 큰 값인 경우 4로 반환하도록 처리해 줄 수 있습니다.<br/>
코드로 작성해 보면 다음과 같은 공식입니다.<br/><br/>

```cs
var inputScore = Math.Min(Math.Max(Math.Floor((decimal)(score - 1) / 10), 0), 4);
```

이 것을 기반으로 위 코드를 리펙토링 해본다면 다음과 같이 바꿀 수 있습니다.<br/><br/>

```cs
var getScore2 = (int score) =>
{
    var inputScore = Math.Min(Math.Max(Math.Floor((decimal)(score - 1) / 10), 0), 4);

    switch (inputScore)
    {
        case 0:
            return "F학점";
        case 1:
            return "D학점";
        case 2:
            return "C학점";
        case 3:
            return "B학점";
        case 4:
            return "A학점";
        default:
            return "Error";
    }
};
```

그리고 다시 위 코드는 0부터 4의 순차적인 사용을 볼 수 있듯이 배열로 바꿔볼 수 있습니다.<br/><br/>

```cs
var getScore2 = (int score) =>
{
    var scores = new string[]
    { "F학점", "D학점", "C학점", "B학점", "A학점" };

    var inputScore = Math.Min(Math.Max(Math.Floor((decimal)(score - 1) / 10), 0), 4);

    return scores[(int)inputScore];
};

getScore2(41);  // A확점
```


map 이용
-

위 처럼 범위 조건에 공통된 규칙이 있다면 일관된 공식을 통해 배열 인덱스로 처리할 수 있지만 공통 패턴의 규칙이 없다면 위에서 설명한 Dictionary와 비슷하게<br/>
map에 지정된 조건을 미리 등록해 놓고 처리하는 할 수 있습니다.<br/><br/>

다음과 같은 조건이 있습니다.<br/>
```
/// 미세먼지
/// 0 ~ 30 : 좋음
/// 31 ~ 80 : 보통
/// 81 ~ 150 : 나쁨
/// 151 ~ : 매우나쁨
```

이 미세먼지 수치 범위별로 현재 미세먼지 상태를 나타내기 위해 if else if 범위로 처리해야 하는 코드를<br/><br/>

```cs
public enum EFine_dust
{
    good,
    usually,
    bad,
    very_bad
}

// 미세먼지 수치를 받아옴
var fine_dust_level = this.GetFineDustLevel();
if (fine_dust_level >= 0 && fine_dust_level  <= 30)
{
    return EFine_dust.good;
}
else if (fine_dust_level >= 31 && fine_dust_level  <= 80)
{
    return EFine_dust.usually;
}
else if ...
.
.
.
```

다음과 같이 리펙토링 가능합니다.<br/><br/>

```cs
var fine_dustRule = new[]
{
    new { Rule = (Func<int, bool>) (p => p >= 0 && p <= 30 ), Value = EFine_dust.good },
    new { Rule = (Func<int, bool>) (p => p >= 31 && p <= 80 ), Value = EFine_dust.usually },
    new { Rule = (Func<int, bool>) (p => p >= 81 && p <= 150 ), Value = EFine_dust.bad },
    new { Rule = (Func<int, bool>) (p => p >= 151 ), Value = EFine_dust.very_bad }
};

var fine_dust = fine_dustRule.First(p => p.Rule(31));
Console.WriteLine(fine_dust);
```

**[출력 결과]**<br/>
```
{ Rule = System.Func`2[System.Int32,System.Boolean], Value = usually }
```

State Design Pattern
-

위 상황 외 분기문을 줄이기 위해 다양한 디자인 패턴들이 존재 합니다.<br/>
Factory Pattern, Strategy Pattern, State Pattern 등 상황에 맞는 디자인 패턴 설계로 분기문을 최소화 할 수 있습니다.<br/>
이 중 State Pattern에 대해 간단히 살펴 보겠습니다.<br/><br/>

ATM 기계의  상태에 맞게 로직을 구현해야 하는 상황인 경우 다음과 같이 각각 상태에 따른 분기 코드를 구현할 수 있습니다.<br/><br/>

```cs
internal class Atm
{
    public enum ECardState
    {
        None,
        Inserted,
        Tradeable,
        ErrorCard
    }

    public ECardState CardState { get; private set; } = ECardState.None;

    public void Initial()
    {
        // 카드 삽입 기능 Init
        // 키패드 입력 기능 Disable
        // 영수증 프린트 포트 init
    }

    public void Paymenmt()
    {
        if (CardState == ECardState.None)
        {
            Console.WriteLine("거래를 위해 카드를 넣어주세요.");
            // 카드 삽입 기능 Open
        }
        else if (CardState == ECardState.Inserted)
        {
            Console.WriteLine("카드가 삽입되었습니다.");
            Console.WriteLine("카드 비밀번호를 입력하세요.");
            // 카드 삽입 기능 Disable
            // 키패드 입력 기능 Enable

        }
        else if (CardState == ECardState.ErrorCard)
        {
            Console.WriteLine("카드를 다시 넣어주세요.");
            // 카드 삽입 기능 Open
        }
        else if (CardState == ECardState.Tradeable)
        {
            Console.WriteLine("출금할 금액을 입력하세요.");
            // 카드 삽입 기능 Disable
            // 영수증 프린트 포트 Open
        }
    }
}
```

현재 카드의 상태에 따라 if else if 분기 처리가 되어 있는데 **모든 상태에 따른 로직이 분리 되어 있지 않고 한곳에서 모두 처리되고 있습니다.**<br/>
**상태가 추가 되거나** **일부 로직이 변경** 되는 경우 해당 클래스에 의존할 수 밖에 없는 구조 입니다. 코드가 복잡하다면 유지보수가 까다롭고 사이드 이펙트 오류 발생 여지도 충분히 보여집니다.<br/>
이 부분은 각 상태에 따른 로직만을 처리할 수 있도록 분리하고 사용하는 부분에선 상황에 맞게 상태변경만 해주면 각 상태에 맞는 기능을 수행할 수 있도록 리펙토링 해주는 것이 좋습니다.<br/><br/>

**[State Pattern 처리]**<br/>
```cs
internal interface IATMState
{
    void InsertedCard();
    void Tradeable();
    void ErrorCard();
}

internal class EmptyCardState : IATMState
{
    public void ErrorCard()
    {
        //
    }

    public void InsertedCard()
    {
        Console.WriteLine("카드가 삽입 되었습니다.");
        Console.WriteLine("카드 비밀번호를 입력하세요.");
    }

    public void Tradeable()
    {
        Console.WriteLine("카드가 없어 거래할 수 없습니다.");
    }
}

internal class InsertedCardState : IATMState
{
    public void ErrorCard()
    {
        
    }

    public void InsertedCard()
    {
        Console.WriteLine("카드가 이미 삽입 되었습니다.");
        Console.WriteLine("카드 비밀번호를 입력하세요.");
    }

    public void Tradeable()
    {
        Console.WriteLine("카드 조회중 입니다.");
    }
}

internal class TradeableState : IATMState
{
    public void ErrorCard()
    {

    }

    public void InsertedCard()
    {
        Console.WriteLine("카드가 이미 삽입 되었습니다.");
    }

    public void Tradeable()
    {
        Console.WriteLine("출금할 금액을 입력하세요.");
    }
}

internal class ErrorCardState : IATMState
{
    public void ErrorCard()
    {
        Console.WriteLine("카드를 다시 넣어주세요.");
    }

    public void InsertedCard()
    {
        Console.WriteLine("카드가 이미 삽입 되었습니다.");
        Console.WriteLine("카드 조회에 오류가 발생했습니다.");
    }

    public void Tradeable()
    {
        Console.WriteLine("카드 조회에 오류가 발생했습니다.");
    }
}

/// <summary>
/// context
/// </summary>
internal class ATMMachine
{
    public IATMState _atmState;
    
    public ATMMachine(IATMState atmState)
    {
        _atmState = atmState;
    }

    public void InsertedCard() => _atmState.InsertedCard();
    public void Tradeable() => _atmState.Tradeable();
    public void ErrorCard() => _atmState.ErrorCard();

    public void ChangeState(IATMState state) => _atmState= state;
}
```

**[Client 측 코드]**<br/>
```cs
ATMMachine atm = new(new EmptyCardState());
// 카드 삽입 전 거래 시도
atm.Tradeable();

// 카드 삽입
atm.InsertedCard();
// 카드 삽입된 상태로 변경
atm.ChangeState(new InsertedCardState());
// 카드가 아직 조회되지 않은 상태에서 거래시도 했을 경우
atm.Tradeable();

// 조회성공시 거래가능 상태로 변경
atm.ChangeState(new TradeableState());
// 거래 시작
atm.Tradeable();
```

**[출력 결과]**<br/>
```
카드가 없어 거래할 수 없습니다.
카드가 삽입 되었습니다.
카드 비밀번호를 입력하세요.
카드 조회중 입니다.
출금할 금액을 입력하세요.
```

지금까지 일부 상황에 따른 분기 처리(if else / else if) 리펙토링에 대해 살펴 보았습니다.<br/>
정리해보면 Early Return 사용으로 if분기를 보다 깔끔하게 처리하고, map자료 구조 등을 이용하거나 공통 패턴의 모습이 보이는 분기는 수식화를 if else를 대체 할 수 있고, <br/>
상황에 맞는 디자인 패턴 설계로 복잡한 분기 처리를 간결하게 처리해 볼 수 있습니다.


{% include content_adsense.html %}
