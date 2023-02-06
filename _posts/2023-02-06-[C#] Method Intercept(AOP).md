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

**<span style="color: rgb(107, 173, 222);">RealProxy</span>** 클래스는 WCF와 .NET Remoting에서 사용 되는 오래전 부터 있었던 클래스인데 Proxy를 통해서 특정 메서드를 호출할 수 있도록 제공해 줍니다.<br/>
예전 기술인 만큼(?) .NET Core에서는 사용 불가능합니다.<br/>
Proxy는 클라이언트와 서버간 서로 약속된 메세지 를 통해 메서드 호출을 할 수 있는데 메세지 요청의 Invoke() 구현 처리에서 해당 메서드의 호출을 가로챌 수 있습니다.<br/>
이때 Aspect 모듈을 호출해서 AOP 적용 처리가 가능합니다.<br/>
다음은 **<span style="color: rgb(107, 173, 222);">RealProxy</span>** 클래스를 사용한 AOP 적용 방법의 간단 예제 코드 입니다.<br/>
**[출처 : https://stackoverflow.com/questions/13659185/intercepting-method-calls-in-c-sharp-using-proxies]**<br/><br/>

**[RuntimeProxy.cs]**<br/>
```cs
namespace Intercepting_Method_RealProxy
{
    using System;
    using System.Runtime.Remoting.Proxies;
    using System.Runtime.Remoting.Messaging;
    using System.Reflection;

    public abstract class RuntimeProxy
    {
        public static readonly object Default = new object();

        public static Target Create<Target>(Target instance, Func<RuntimeProxyInvoker, object> factory) where Target : class
        {
            return (Target)new InternalProxy<Target>(instance, new InternalRuntimeProxyInterceptor(factory)).GetTransparentProxy();
        }


        class InternalProxy<Target> : RealProxy where Target : class
        {
            readonly object Instance;
            readonly RuntimeProxyInterceptor Interceptor;

            public InternalProxy(Target instance, RuntimeProxyInterceptor interceptor)
                : base(typeof(Target))
            {
                Instance = instance;
                Interceptor = interceptor;
            }

            public override IMessage Invoke(IMessage msg)
            {
                var methodCall = (IMethodCallMessage)msg;
                var method = (MethodInfo)methodCall.MethodBase;

                try
                {
                    // Proxy를 통해 메세지 요청이 오면 Interceptor Func 호출
                    var result = Interceptor.Invoke(new InternalRuntimeProxyInterceptorInvoker(Instance, method, methodCall.InArgs));

                    if (result == RuntimeProxy.Default)
                        result = method.ReturnType.IsPrimitive ? Activator.CreateInstance(method.ReturnType) : null;

                    return new ReturnMessage(result, null, 0, methodCall.LogicalCallContext, methodCall);
                }
                catch (Exception ex)
                {
                    if (ex is TargetInvocationException && ex.InnerException != null)
                        return new ReturnMessage(ex.InnerException, msg as IMethodCallMessage);

                    return new ReturnMessage(ex, msg as IMethodCallMessage);
                }
            }
        }

        class InternalRuntimeProxyInterceptor : RuntimeProxyInterceptor
        {
            readonly Func<RuntimeProxyInvoker, object> Factory;

            public InternalRuntimeProxyInterceptor(Func<RuntimeProxyInvoker, object> factory)
            {
                this.Factory = factory;
            }

            public override object Invoke(RuntimeProxyInvoker invoker)
            {
                return Factory(invoker);
            }
        }

        class InternalRuntimeProxyInterceptorInvoker : RuntimeProxyInvoker
        {
            public InternalRuntimeProxyInterceptorInvoker(object target, MethodInfo method, object[] args)
                : base(target, method, args)
            { }
        }
    }
}
```

**[RuntimeProxyInterceptor.cs]**<br/>
```cs
namespace Intercepting_Method_RealProxy
{
    public abstract class RuntimeProxyInterceptor
    {
        public virtual object Invoke(RuntimeProxyInvoker invoker)
        {
            return invoker.Invoke();
        }
    }
}
```

**[RuntimeProxyInvoker.cs]**<br/>
```cs
namespace Intercepting_Method_RealProxy
{
    using System;
    using System.Collections.ObjectModel;
    using System.Linq;
    using System.Reflection;

    public abstract class RuntimeProxyInvoker
    {
        public readonly object Target;
        public readonly MethodInfo Method;
        public readonly ReadOnlyCollection<object> Arguments;

        public RuntimeProxyInvoker(object target, MethodInfo method, object[] args)
        {
            this.Target = target;
            this.Method = method;
            this.Arguments = new ReadOnlyCollection<object>(args);
        }

        public object Invoke()
        {
            return Invoke(this.Target);
        }

        public object Invoke(object target)
        {
            if (target == null)
                throw new ArgumentNullException("target");

            try
            {
                return this.Method.Invoke(target, this.Arguments.ToArray());
            }
            catch (TargetInvocationException ex)
            {
                throw ex.InnerException;
            }
        }
    }
}
```

**[Program.cs]**<br/>
```cs
internal class Program
{
    private int Add(int a, int b) => a + b;
    public int Val { get; set; }

    static void Main(string[] args)
    {
        var test = new TestClass();
        var proxy = RuntimeProxy.Create<TestClass>(test,
            // 실제 클래스 메서드 호출 전 가로채기 처리 Func
            t =>
            {
              try
              {
                Console.WriteLine("Executing...!");
                return t.Invoke();
              }
              finally
              {
                Console.WriteLine("Executed!");
              }
            });

        var res = proxy.Add(3, 4);
        Console.WriteLine(res);
        proxy.Val= 5;
        Console.WriteLine(proxy.Val);

        Console.ReadLine();
    }
}

class TestClass : MarshalByRefObject
{
    public int Add(int a, int b) => a + b;
    public int Val { get; set; }
}
```

**[출력 결과]**<br/>
```
Executing...!
Executed!
7
Executing...!
Executed!
Executing...!
Executed!
5
```

**Add()호출**시 Executing...! Executed!<br/>
**Val속성의 Set Method()호출**시 Executing...! Executed!<br/>
**Val속성의 Get Method()호출**시 Executing...! Executed!<br/>
각 메서드 호출이 차단되고 위와 같이 공통 처리되는 것을 확인해 볼 수 있습니다.<br/><br/>

하지만 이런 Proxy 사용은 사용 방법이 다소 복잡합니다.<br/>
Method Intercept 처리를 위해 Proxy를 통해서 객체 생성이 이루어 져야 하고 메서드 호출이 사용 되야 하기 때문에 코드 자체도 한눈에 들어 오지 않습니다.


Cauldron.Intercept 라이브러리
-

Fody 플러그인의 Cauldron.Intercept 라이브러리는 Proxy 방식으로 처리되지 않고 IL-weavers 사용으로 런타임시 Method Intercept 처리를 깔끔한 방식으로 제공 합니다.<br/>
우선 Fody 플러그인의 Cauldron.Intercept 라이브러리는 NuGet으로 다운받아 설치 할 수 있으며, .NET Framework 4.x 와 .NET Standart / UWP를 지원합니다.<br/>
다음 3가지의 패키지를 설치합니다.<br/>
> Cauldron.Interception.Fody<br/>
> Costura.Fody<br/>
> Fody<br/><br/>

***

> ※ 참고로<br/>
> Costura.Fody 및 Cauldron.Interception.Fody 최신 버전에서<br/>
> IL Weavers 정보 xml파일(FodyWeavers.xml) 내용중 <Cauldron.Interception /><br/>
> dll을 찾을 수 없다는 오류가 발생되어 이전 버전으로 테스트 하였습니다.

***

위 패키지 설치후 빌드시 IL-weavers 처리를 위해 프로젝트 경로 루트에 FodyWeavers.xml 파일이 존재 해야 합니다.<br/><br/>

**[FodyWeavers.xml]**<br/>
```xml
<?xml version="1.0" encoding="utf-8"?>
<Weavers>
  <Cauldron.Interception />
  <Costura />
</Weavers>
```

Cauldron.Intercept 라이브러리의 Method Intercept는 메서드 진입과 메서드 처리 후 해당 메서드 블록 종료시, 그리고 메서드 오류 발생시에 대한 Intercept를 지원합니다.<br/>
Method Intercept 기능 관련의 **<span style="color: rgb(107, 173, 222);">IMethodInterceptor</span>**, Property value change intercept 관련의 **<span style="color: rgb(107, 173, 222);">IPropertySetterInterceptor</span>**, 
Constructor interceptor 관련 **<span style="color: rgb(107, 173, 222);">IConstructorInterceptor</span>** 인터페이스를 제공합니다.<br/>
이렇게 제공되는 인터페이스는 어트리뷰트로 구현하여 필요한 클래스에서만 사용할 수 있게 쉽게 사용 가능합니다.<br/><br/>

**[LoggerAttribute.cs]**<br/>
```cs
namespace Intercepting_Method_Cauldron.Interception
{
    using Cauldron.Interception;
    using System;
    using System.Reflection;

    [AttributeUsage(AttributeTargets.Method | AttributeTargets.Class, AllowMultiple = false, Inherited = false)]
    public class LoggerAttribute : Attribute, IMethodInterceptor
    {
        private string _methodName;

        public void OnEnter(Type declaringType, object instance, MethodBase methodbase, object[] values)
        {
            _methodName = methodbase.Name;
            this.AppendToFile($"Enter -> {declaringType.Name} {methodbase.Name} {string.Join(" ", values)}");
        }

        public void OnException(Exception e) => this.AppendToFile($"Exception -> {e.Message}");

        public void OnExit() => this.AppendToFile($"Exit -> {_methodName}");

        private void AppendToFile(string line)
        {
            Console.WriteLine("[Log] >> " + line);
        }
    }
}
```

**[OnPropertySetAttribute.cs]**<br/>
```cs
namespace Intercepting_Method_Cauldron.Interception
{
    using Cauldron.Interception;
    using System;

    [AttributeUsage(AttributeTargets.Property | AttributeTargets.Class, AllowMultiple = false, Inherited = false)]
    public sealed class OnPropertySetAttribute : Attribute, IPropertySetterInterceptor
    {
        [AssignMethod("{CtorArgument:0}")]
        public Action<string, object> _onSetMethod = null;

        public OnPropertySetAttribute(string methodName)
        {
            
        }

        public void OnException(Exception e)
        {
        }

        public void OnExit()
        {
        }

        public bool OnSet(PropertyInterceptionInfo propertyInterceptionInfo, object oldValue, object newValue)
        {
            this._onSetMethod?.Invoke(propertyInterceptionInfo.PropertyName, newValue);
            return false;
        }
    }
}
```

**[Program.cs]**<br/>
```cs
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace Intercepting_Method_Cauldron.Interception
{
    internal class Program
    {
        static void Main(string[] args)
        {
            var test = new TestClass();
            var res = test.Add(3, 4);
            Console.WriteLine(res);
            test.Val = 5;
            Console.WriteLine(test.Val);

            Console.ReadLine();
        }
    }


    [Logger]
    [OnPropertySet(nameof(ExecuteMe))]
    class TestClass
    {
        public int Add(int a, int b) => a + b;
        public int Val { get; set; }

        private void ExecuteMe(string propertyName, object newValue) =>
            Console.WriteLine($"Name : '{propertyName}' / Value : {newValue}");
    }
}
```

이렇게 Logger 어트리뷰트와 OnPropertySet 어트리뷰트 사용으로 공통적인 Aspect 처리가 간단하게 처리 가능 합니다.

***


위 코드는 다음 Repository에서 확인할 수 있습니다.<br/>
> [code_check - Method_Interception](https://github.com/tyeom/code_check/tree/main/TestSample/csharp/Method_Interception)



{% include content_adsense.html %}
