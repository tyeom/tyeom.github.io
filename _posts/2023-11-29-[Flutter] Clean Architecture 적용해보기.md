---
title: (Flutter) Clean Architecture 적용해보기
categories: Flutter
key: 20231129_01
comments: true
tags: Flutter CA 예외처리 CleanArchitecture Architecture bloc, 관심사분리, 분리
---

프로젝트 개발을 시작할때 고민하게 되는 것들 중 하나는 Architecture(아키텍처) 입니다.<br/>
아키텍처 패턴중 하나인 MVVM 패턴은 보통 아래 그림과 같은 설계로 이루어 집니다<br/><br/>
![image](https://github-production-user-asset-6210df.s3.amazonaws.com/13028129/286452519-c57492d2-fadd-4dc4-a298-c09b53b62dcc.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20231130%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20231130T002836Z&X-Amz-Expires=300&X-Amz-Signature=27a5b5098683b46020a3ca542ce93041da308ce16c47da88b0b62e518d7b3337&X-Amz-SignedHeaders=host&actor_id=13028129&key_id=0&repo_id=445749826)

규모가 작은 복잡하지 않은 프로젝트에서는 좋긴 하겠지만 계속해서 규모가 커지고, 요구사항이 늘어나 페이지와 기능들이 점차 커지는 경우
UI 관련 비즈니스 로직등이 포함되어 있는 ViewModel이 복잡해지면서 결국 유지보수에도 어려움이 발생 합니다.<br/>
이러한 문제의 해결 설계중 하나인 Clean Architecture를 도입해 보고 Flutter에서 어떻게 적용 시킬 수 있을지 방법을 알아 봅니다.

<!--more-->

> 이 글에서 다루는 코드는 다음 Repository에서 확인할 수 있습니다.<br/>
> [flutter_basic_architecture](https://github.com/tyeom/flutter_basic_architecture)

우선 Clean Architecture에 대해 알아 봅시다
-

![image](https://github-production-user-asset-6210df.s3.amazonaws.com/13028129/286454240-f8a9020d-4244-4eb6-8bfa-843b5ea4bbec.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20231130%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20231130T002914Z&X-Amz-Expires=300&X-Amz-Signature=ffb8b6f79cf75ec0c7b5c56e8e2e34978da54e5b92ace6e315c29847ed7cbaac&X-Amz-SignedHeaders=host&actor_id=13028129&key_id=0&repo_id=445749826)
Clean Architecture하면 너무나도 많이 나오고 유명한(?) 다이어그램 그림 입니다.<br/>

화살표 방향이 의존성을 나타내고 있는 방향 입니다. 따라서 가장 안쪽에 있는 영역 일수록 정책의 기준을 가지고 있고 핵심들이 추상적으로 표현 됩니다.<br/>
그리고 바깥쪽 영역 일수록 정책에 따른 역할을 직접 수행하는 매커니즘 책임을 가지고 있습니다.<br/>
이 처럼 **클린 아키텍처**는 각 계층의 경계를 중요시 하게 여기고 의존성 규칙을 따르는 것이 핵심 입니다.<br/>
이것을 쉽게 말하면 **〃모듈의 의존성은 반드시 단방향으로 이어져야 한다.〃** 입니다.<br/><br/>
어떻게 하면 의존성 규칙을 지킬 수 있을지 고민하고, 그 고민의 결과가 의존성을 지키고 있다면 클린 아키텍처 설계와 부합 하다고도 볼 수 있습니다.<br/>
> **[Clean Architecture(로버트C.마틴, 엉클밥)]**<br/>
> There’s no rule that says you must always have just these four.<br/>
> **However, The Dependency Rule always applies.**

그렇게 고민해서 제시 되는 방법중 하나가 인터넷상 많이 거론 되고 있는 클린 아키텍처 구현체들 중에서 Repository의 추상적 구현을 통한 모듈 분리 방법 입니다.<br/><br/>

안드로이드 진영에서 많이 설명 되고 있는 클린 아키텍처 설계의 구조는 다음과 같습니다.<br/>
![image](https://github-production-user-asset-6210df.s3.amazonaws.com/13028129/286473572-51fd18a6-ea4f-47cc-b313-6e7a825431d9.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20231130%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20231130T002935Z&X-Amz-Expires=300&X-Amz-Signature=d7563dedf329f27baaa2a9c0297a7ded2f2d0dea95095a15edf6bc94d5498b99&X-Amz-SignedHeaders=host&actor_id=13028129&key_id=0&repo_id=445749826)

Presentation 계층과 Data 계층은 Domain 계층을 의존하고 있으며, 각 계층의 세부 영역 역할은 다음과 같습니다.<br/>
- **Presentation**
  - **UI :** 현재 개발 환경에 의존적으로 사용자에게 표시 되는 UI 구현을 담당 합니다.
  - **ViewModel :** UI와 관련된 비즈니스 로직이 구현 되고 데이터 흐름에 따라 상황 판단을 담당 합니다. (이 예제에서는 BLoC를 사용 합니다.)
- **Domain**
  - **Repositories :** 비즈니스 로직의 추상 구현체 입니다.
  - **Use cases :** Repository를 통해 입력에 대한 결과물을 받아 처리 하는 비즈니스 로직을 담당 합니다. (하지만 실제 구현체는 Data 계층에 있습니다.)
  - **Entities :** 앱 내에서 사용 되는 데이터 정의를 담당 합니다. (실제 데이터)
- **Data**
  - **Repositories :** Use Case에서 사용 되는 데이터 요청 처리(Save, Load, Update, Delete)가 구현되고 Data Source를 통해 서버와 통신을 합니다.
  - **Data Sources :** 실제 데이터의 입력과 출력을 처리 담당 합니다. (API 통신, DB 통신 등)
  > Data Source는 경우에 따라 서버 API 요청인 경우 remote data source, 로컬 처리인 경우 catch data source로 구분 될 수 있습니다.

여기서 데이터의 흐름은 Data 계층에서 부터 시작 되는데 데이터 결과를 Domain이 가지고 있는 엔티티로 변환을 해주어야 합니다.<br/>
그래서 Data 계층에서는 Domain의 엔티티와 매핑 할 수 있는 트랜스레이터(Mapper)가 존재 합니다.<br/>
이 트랜스레이터로 Domain 계층에서는 Data 계층의 엔티티를 자신이 알고 있는 엔티티로 변환되어 사용 됩니다.<br/><br/>

위 도표에서 추가 적으로 궁금한 부분이 있을 수 있는데 Domain 계층과 Data 계층에 모두 Repositories를 가지고 있다는 것 입니다.<br/>
각각의 사용자 요구사항은 Domain에서 Use cases로 정의 되어 있습니다.<br/>
이러한 요구사항을 처리 하려면 외부로 부터 통신이 원할하게 되어야 하는데, Domain계층은 바깥 영역인 Data계층에 대해 알 수 없습니다.<br/><br/>

이때 해결 할 수 있는 방법 중 하나는 로버트 마틴이 제시한 **DIP 원칙**을 사용 할 수 있습니다.<br/>
객체지향 설계 원칙 SOLID중 'D'에 해당 되는 DIP(Dependency Inversion Principle) 원칙은 특정 객체의 참조가 필요할때 직접 참조 하지 않고 해당 객체의 추상 클래스나
인터페이스로 대체 하여 참조 하라는 원칙 입니다.<br/>
이렇게 처리 하면 직접적인 의존 관계가 형성 되지 않고 분리 시킬 수 있습니다.<br/><br/>

즉, Domain 계층에서는 Data 계층의 Repositories를 추상적으로만 가지고 있고, 실제 구현은 Data 계층에서 구현하면 자연스럽게 Domain 계층에서 외부 통신을 처리 할 수 있게 됩니다.

Flutter에 도입시켜 보기
-

Clean Architecture가 추구 하는 방향과 안드로이드 진영에서 Clean Architecture 설게 구조를 알아 보았으니 그대로 Flutter에 도입시켜 보겠습니다.<br/>
우선 각 계층의 의존성이 필요한 부분은 Dependency Injection(의존성 주입)을 사용하고 이에 따라 get_it 패키지와 injectable 패키지를 사용하겠습니다.<br/>
해당 패키지의 기본적인 기능을 알아 봅니다.

### get_it 패키지
get_it 패키지는 의존성 주입 패키지로 자체적으로 IoC Container 등록을 통한 객체의 수명관리를 하고 의존성 주입 기능을 제공 합니다.<br/>
IoC Container 등록 종류는 대표적으로 3가지가 있는데 아래 표로 설명 할 수 있습니다.<br/>
<table>
        <tr>
                <th>메서드</th>
                <th>설명</th>
        </tr>
        <tr>
                <td>registerFactory</td>
                <td>새로운 instance를 생성하여 반환 합니다.</td>
        </tr>
        <tr>
                <td>registerSingleton</td>
                <td>생성된 instance를 반환 합니다.</td>
        </tr>
        <tr>
                <td>registerLazySingleton</td>
                <td>바로 생성하지 않고 해당 객체가 사용 될때 생성되며 이후에는 최초 생성한 instance를 반환 합니다.</td>
        </tr>
</table>

이 밖에도 비동기 방식을 지원하는 registerFactoryAsync(), registerSingletonAsync(), registerLazySingletonAsync() 등이 제공 되고,<br/>
사용은 **<span style="color: rgb(107, 173, 222);">GetIt</span>** 클래스의 정적 속성인 instance 속성을 통해 **<span style="color: rgb(107, 173, 222);">GetIt</span>** 인스턴스를 반환 받아 사용 할 수 있습니다.<br/><br/>

```dart
final getIt = GetIt.instance;

// Dio 등록
getIt.registerSingleton<Dio>(() => Dio());
/// Dio 인스턴스 의존성 생성자 주입을 통해
/// ApiService 객체 관리
getIt.registerFactory<ApiService>(() => ApiService(getIt<Dio>()));
```

### injectable 패키지

injectable 패키지는 get_it 패키지를 활용해서 의존성 주입 처리를 쉽게 도와주는 패키지 입니다.<br/>
이에 따라 몇가지 어노테이션을 제공하고 있으며, 어노테이션 설정에 맞게 get_it을 활용한 IoC Container 등록 코드가 자동 생성 됩니다.<br/>
injectable 패키지를 사용하기 위해서는 get_it 패키지와 injectable 패키지 그리고 코드 제너레이터를 위한 injectable_generator, build_runner 설치가 필요 합니다.<br/><br/>

```yaml
dependencies:
  flutter:
    sdk: flutter

  get_it: ^7.6.4  # DI(의존성 주입) 패키지
  injectable: ^2.3.2  # get_it DI code generator 패키지

dev_dependencies:
  flutter_test:
    sdk: flutter
  
  injectable_generator:  # injectable code generator
  build_runner:
```

이렇게 패키지 설치가 되면 다음과 같이 초기 설정을 할 수 있습니다.<br/><br/>

**[configurations.dart]**
```dart
import 'package:get_it/get_it.dart';
import 'package:injectable/injectable.dart';
import 'configurations.config.dart';

final getIt = GetIt.instance;

@injectableInit
void configureDependencies() =>  getIt.init();
```

그리고 App 시작시 다음과 같이 호출해서 getIt을 통해 객체 등록을 처리 합니다.<br/><br/>

**[main.dart]**<br/>
```dart
void main() {
  configureDependencies();
  runApp(const MyApp());
}
```

그리고 build_runner를 이용해서 code generating을 실행합니다.<br/>
**프로젝트의 root 경로에서 다음 명령으로 실행 합니다.**
```
flutter pub run build_runner build - delete-conflicting-outputs
```

잠시후 실행 결과로 config.dart의 같은 위치에 다음과 같은 파일이 생성된 것을 확인 할 수 있습니다.<br/><br/>

**[configurations.config.dart]**
```dart
// GENERATED CODE - DO NOT MODIFY BY HAND

// **************************************************************************
// InjectableConfigGenerator
// **************************************************************************

// ignore_for_file: type=lint
// coverage:ignore-file

// ignore_for_file: no_leading_underscores_for_library_prefixes
import 'package:get_it/get_it.dart' as _i1;
import 'package:injectable/injectable.dart' as _i2;

[... 중간 import 생략 ...]

// initializes the registration of main-scope dependencies inside of GetIt
_i1.GetIt $initGetIt(
  _i1.GetIt getIt, {
  String? environment,
  _i2.EnvironmentFilter? environmentFilter,
}) {
  final gh = _i2.GetItHelper(
    getIt,
    environment,
    environmentFilter,
  );
  
  return getIt;
}
```

Dio 패키지 같이 외부 모듈이 최초 수동으로 IoC에 등록되서 해당 모듈이 의존성 주입이 필요하다면 module로 등록하는 방법(아래에서 별도 어노테이션 설명)이 있고,<br/>
다음과 같이 최초 등록을 코드로 처리하고 자동 생성 코드를 호출하여 객체 등록을 처리 할 수 있습니다.<br/><br/>

**[configurations.dart]**
```dart
import 'package:basic_architecture/App/app_preferences.dart';
import 'package:basic_architecture/Data/data_source/remote_data_source.dart';
import 'package:basic_architecture/Data/data_source/remote_data_source_imp.dart';
import 'package:basic_architecture/Data/network/api_client.dart';
import 'package:basic_architecture/Data/network/api_provider.dart';
import 'package:basic_architecture/Data/network/api_sanple_data_service.dart';
import 'package:basic_architecture/Data/network/api_service.dart';
import 'package:dio/dio.dart';
import 'package:get_it/get_it.dart';
import 'package:injectable/injectable.dart';
import 'package:shared_preferences/shared_preferences.dart';

import 'configurations.config.dart' as config;

final getIt = GetIt.instance;

@InjectableInit(
  initializerName: r'$initGetIt',
  preferRelativeImports: true,
  asExtension: false,
)
Future<void> configureDependencies() => $initGetIt(getIt);

Future<void> $initGetIt(
  GetIt getIt, {
  String? environment,
  EnvironmentFilter? environmentFilter,
}) async {
  final gh = GetItHelper(getIt, environment.toString());
  final sharedPreferences = await SharedPreferences.getInstance();

  // IoC 등록
  gh.lazySingleton<AppPreferences>(() => AppPreferences(sharedPreferences));

  // Http 요청 처리 등록
  var baseApiClient = ApiClient(ApiType.base, enableLogging: true);

  // Dio 등록 [ApiClient에서 생성한 apiProvider가 가지고 있는 Dio]
  gh.factory<Dio>(() => baseApiClient.apiProvider.getDio);

  // 실제 RestFul API 처리 서비스 등록
  gh.factory<ApiService>(() => ApiService(getIt<Dio>()));

  config.$initGetIt(getIt);
}
```
<br/><br/>

**[main.dart]**<br/>
```dart
import 'package:basic_architecture/App/app.dart';
import 'package:basic_architecture/Injectable/configurations.dart';
import 'package:flutter/material.dart';

main() => App.run();

class App {
  App();

  static Future<void> run() async {
    WidgetsFlutterBinding.ensureInitialized();
    await configureDependencies();
    runApp(const MyApp());
  }
}
```

IoC Container 등록 방법은 위 get_it 패키지의 IoC Container 등록 종류와 같은 기능을 어노테이션으로 제공 합니다.
- Injectable
- Singleton
- LazySingleton

<br/><br/>

**[@injectable 어노테이션 적용 예]**<br/>
```dart
@injectable
class MemberInfoUseCase implements BaseUseCase<void, MemberInfo?> {
  final MemberRepository _memberRepository;

  MemberInfoUseCase(this._memberRepository);

  @override
  Future<Either<String, MemberInfo?>> execute(void input) async {
    return await _memberRepository
        .getMemberInfo();
  }
}
```

그 외에 다른 외부 모듈이 의존성으로 필요할때 사용 되는 @module 어노테이션과 Scope 설정을 통해 객체 인스턴스를 별도 설정 하거나,<br/>
의존성 순서를 지정하는 방법등 다양한 기능들이 제공 됩니다.<br/>
- [Using a register module](https://github.com/Milad-Akarie/injectable#using-a-register-module-for-third-party-dependencies)
- [Using Scopes](https://github.com/Milad-Akarie/injectable#using-scopes)
- [Manual order](https://github.com/Milad-Akarie/injectable#manual-order)


프로젝트 구조
-

get_it 패키지 사용 방법과 get_it을 이용한 DI code generator 를 제공하는 injectable 패키지까지 알아 보았으니, 전체 적인 프로젝트 구조를 설계해 봅니다.<br/>
전체 프로젝트 아키텍처 구조는 아래 그림과 같은 모습으로 목표를 정했습니다.<br/>
![플러터_클린아키텍처_구조](https://github-production-user-asset-6210df.s3.amazonaws.com/13028129/286791496-3723f684-157a-4504-a5d3-34365b6c34e9.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20231130%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20231130T015139Z&X-Amz-Expires=300&X-Amz-Signature=e4b01f5969fe3220068604ea51f9967f56384b8620c1d4b6a9fc6af09c25cb38&X-Amz-SignedHeaders=host&actor_id=13028129&key_id=0&repo_id=445749826)<br/>

그리고 위 설계대로 솔루션 구조는 다음과 같이 만들 수 있습니다.<br/>
프로젝트 솔루션 폴더 구조는 크게 다음과 같이 정의했습니다.<br/>

├─**App** - App의 초기 설정<br/>
├─**Data**<br/>
│  ├─**data_source** - API 요청 처리<br/>
│  ├─**network** - http 요청 모듈<br/>
│  ├─**repository** - 실제 데이터 요청 처리 구현<br/>
│  ├─**responses** - 데이터 entity<br/>
│  └─**translator** - Domain 계층 Model Mapper<br/>
├─**Domain**<br/>
│  ├─**models** - 데이터 모델<br/>
│  ├─**repository** - 비즈니스 로직 추상체<br/>
│  ├─**request** - 데이터 요청 정보<br/>
│  └─**usecase** - UseCase 정의<br/>
├─**Injectable** - injectable 패키지 초기화<br/>
├─**l10n** - 다국어 처리<br/>
│  └─arb<br/>
└─**Presentation**<br/>
    ├─**authentication**<br/>
    │  └─**bloc** - 인증 관련 UI 비즈니스 로직<br/>
    ├─**component** - 공통 위젯<br/>
    ├─**home**<br/>
    │  ├─**bloc** - 메인 홈 UI 비즈니스 로직<br/>
    │  └─**view** - 메인 홈 UI<br/>
    ├─**login**<br/>
    │  ├─**bloc** - 로그인 UI 비즈니스 로직<br/>
    │  ├─**forms** - 로그인 사용자 입력 forms (forms 패키지에서 사용)<br/>
    │  └─**view** - 로그인 UI<br/>
    ├─**splash** - splash UI<br/>
    └─**subscription_info**<br/>
        ├─**bloc** - 구독키 정보 UI 비즈니스 로직<br/>
        └─**view** - 구독키 정보 UI<br/>

<br/>
전체 완성된 프로젝트 솔루션 구조입니다.

### UseCase 정의하기

UseCase는 어떤 서비스를 사용하기 위한 행동의 최소 단위라고 볼 수 있습니다.<br/>
**〃인증〃** 이라는 서비스를 사용할때 사용자가 수행 할 수 있는 행동을 구분해 보면<br/>
- 아이디와 패스워드로 로그인을 수행한다.
- 인증한 사용자 정보를 알아본다.
- 로그아웃을 수행한다.

이렇게 각각 정의해 볼 수 있습니다. 그리고 Clean Architecture 에서 UseCase는 몇가지 규칙을 가집니다.<br/>
- UseCase는 입력 값 으로 결과 값을 반환할 수 있는 단일 메서드만 외부에 제공.
- UseCase가 사용되는 Repository는 추상 클래스나 인터페이스로 주입 받습니다.
- 이렇게 정의한 Use Cases는 다른 프로젝트 환경에서도 동일하게 적용 될 수 있어야 합니다. (Domain 계층의 전반적인 특징)


### Useless Use Cases

이렇게 Use Cases를 정의해 놓고 보면 단지 Repository를 가지고 있는 래핑 역할만 해서 한번 더 감싼 느낌만 들 수 있습니다.<br/>
하지만 세분화로 Use Cases를 정의하면 해당 Domain이 어떤 요구사항을 처리 할 수 있는지 한눈에 파악할 수 있는 이점이 더 크고,<br/><br/>

Use Case 없이 ViewModel 또는 UI관련 비즈니스 로직을 담당하고 있는 곳에서 바로 Repository를 사용하는 경우 처음 MVVM 구조의 단점 설명의 ViewModel이 복잡해 집니다.<br/>
해당 ViewModel에서는 필요하지 않는 기능들이 불필요하게 Repository를 통해 노출 되고 있기 때문 입니다.<br/>
또한 해당 Repository의 메서드 시그니처 등이 수정 될 경우 Presentation 계층도 영향을 받기 때문에 OCP 원칙에 위배라고도 볼 수 있습니다.<br/>
이렇기 때문에 **일관성**을 위해서라도 Use Case는 반드시 사용 되는 것이 좋습니다.


API 요청 처리 (Data 계층 구현)
-

기본적으로 App에서 사용자 인증 요청 부분을 구현해 봅니다.<br/>
실제 API가 요청되고 처리 되는 영역은 Data 계층 입니다.<br/>
Data 계층 부터 구현해 보겠습니다.<br/>
API를 통해 인증 요청을 구현해 볼텐데 우선 dio 패키지로 http 요청을 사용하고, retrofit 패키지로 http 요청 처리 부분을 code generator로 사용하겠습니다.<br/><br/>

```yaml
dependencies:
  flutter:
    sdk: flutter

  dio: ^5.3.3  # 클라이언트 http 요청 패키지
  pretty_dio_logger: ^1.3.1  # dio 로그 패키지
  retrofit: ^4.0.3  # dio 패키지를 사용한 http 요청 code generator 패키지

dev_dependencies:
  flutter_test:
    sdk: flutter
  
  retrofit_generator:  # retrofit code generator
  build_runner:
```

위 처럼 패키지 설치후 인증 요청 부분을 다음과 같이 구현 합니다.<br/>
인증에 대한 RestFul API 명세서는 다음과 같이 정의 되어 있습니다.<br/><br/>

```
curl -X 'POST' \
  'https://api.namicro.co.kr/public/Login' \
  -H 'accept: */*' \
  -H 'Content-Type: application/json' \
  -d '{
  "Id": "",
  "Password": ""
}'
```

**[Data\network\api_endpoint.dart]**<br/>
```dart
class ApiEndpoint {
  static const String baseUrl = 'https://api.namicro.co.kr';
  static const String publicApi = '/public';
  static const String privateApi = '/private';
}
```

**[Data\network\api_service.dart]**
```dart
import 'package:basic_architecture/Data/network/api_endpoint.dart';
import 'package:basic_architecture/Data/responses/authentication_response.dart';
import 'package:dio/dio.dart';
import 'package:retrofit/http.dart' as http;
import 'package:retrofit/retrofit.dart';

part 'api_service.g.dart';

@RestApi()
abstract class ApiService {
  factory ApiService(final Dio dio) = _ApiService;

  /// 로그인 요청
  @POST("${ApiEndpoint.publicApi}/login")
  Future<AuthenticationResponse> login(
      @Field("Id") String id, @Field("Password") String password);
}
```

위 처럼 ApiService 추상 클래스를 만들고 build_runner로 code generator를 처리하게 되면 @RestApi 어노테이션으로 해당 메서드 시그니처에 맞게 Dio를 사용해 http 요청 코드가 자동 생성 됩니다.<br/><br/>

**프로젝트의 root 경로에서 다음 명령으로 실행 합니다.**
```
flutter pub run build_runner build - delete-conflicting-outputs
```

**[Data\network\api_service.g.dart]**
```dart
// GENERATED CODE - DO NOT MODIFY BY HAND

part of 'api_service.dart';

// **************************************************************************
// RetrofitGenerator
// **************************************************************************

// ignore_for_file: unnecessary_brace_in_string_interps,no_leading_underscores_for_local_identifiers

class _ApiService implements ApiService {
  _ApiService(
    this._dio, {
    this.baseUrl,
  });

  final Dio _dio;

  String? baseUrl;

  @override
  Future<AuthenticationResponse> login(
    String id,
    String password,
  ) async {
    const _extra = <String, dynamic>{};
    final queryParameters = <String, dynamic>{};
    final _headers = <String, dynamic>{};
    final _data = {
      'Id': id,
      'Password': password,
    };
    final _result = await _dio.fetch<Map<String, dynamic>>(
        _setStreamType<AuthenticationResponse>(Options(
      method: 'POST',
      headers: _headers,
      extra: _extra,
    )
            .compose(
              _dio.options,
              '/public/login',
              queryParameters: queryParameters,
              data: _data,
            )
            .copyWith(
                baseUrl: _combineBaseUrls(
              _dio.options.baseUrl,
              baseUrl,
            ))));
    final value = AuthenticationResponse.fromJson(_result.data!);
    return value;
  }

  RequestOptions _setStreamType<T>(RequestOptions requestOptions) {
    if (T != dynamic &&
        !(requestOptions.responseType == ResponseType.bytes ||
            requestOptions.responseType == ResponseType.stream)) {
      if (T == String) {
        requestOptions.responseType = ResponseType.plain;
      } else {
        requestOptions.responseType = ResponseType.json;
      }
    }
    return requestOptions;
  }

  String _combineBaseUrls(
    String dioBaseUrl,
    String? baseUrl,
  ) {
    if (baseUrl == null || baseUrl.trim().isEmpty) {
      return dioBaseUrl;
    }

    final url = Uri.parse(baseUrl);

    if (url.isAbsolute) {
      return url.toString();
    }

    return Uri.parse(dioBaseUrl).resolveUri(url).toString();
  }
}

```

로그인 요청에 대한 응답 정의 입니다. 이 객체가 Domain 계층에 있는 실제 Model로 변환 되어 사용 되어 집니다.<br/><br/>

**[Data\responses\authentication_response.dart]**<br/>
```dart
import 'package:basic_architecture/Data/responses/base_response.dart';
import 'package:json_annotation/json_annotation.dart';

part 'authentication_response.g.dart';

/// 로그인 요청 응답
@JsonSerializable()
class AuthenticationResponse extends BaseResponse {
  @JsonKey(name: "token")
  String token;
  @JsonKey(name: "expiration")
  DateTime expiration;

  AuthenticationResponse(this.token, this.expiration);
  factory AuthenticationResponse.fromJson(Map<String, dynamic> json) =>
      _$AuthenticationResponseFromJson(json);
  Map<String, dynamic> toMap(Map<String, dynamic> json) =>
      _$AuthenticationResponseToJson(this);
}
```

그리고 실제 인증 처리 요청의 구현부 입니다.<br/><br/>

**[Data\repository\authentication_repository_imp.dart]**<br/>
```dart
import 'package:basic_architecture/Data/data_source/remote_data_source.dart';
import 'package:basic_architecture/Data/network/auth_token_dio_interceptor.dart';
import 'package:basic_architecture/Data/responses/authentication_response.dart';
import 'package:basic_architecture/Domain/models/authentication.dart';
import 'package:basic_architecture/Domain/repository/authentication_repository.dart';
import 'package:basic_architecture/Domain/request/login_request.dart';
import 'package:basic_architecture/Data/translator/translator.dart';
import 'package:dartz/dartz.dart';
import 'package:dio/dio.dart';
import 'package:flutter_secure_storage/flutter_secure_storage.dart';
import 'package:injectable/injectable.dart';

@LazySingleton(as: AuthenticationRepository)
class AuthenticationRepositoryImp implements AuthenticationRepository {
  final RemoteDataSource _remote;

  AuthenticationRepositoryImp(this._remote);

  @override
  Future<Either<String, Authentication>> login(
      LoginRequest loginRequest) async {
    try {
      final AuthenticationResponse response = await _remote.login(loginRequest);
      var authentication = response.toDomain();

      const storage = FlutterSecureStorage();
      await storage.write(key: ACCESS_TOKEN_KEY, value: authentication.token);

      return Right(authentication);
    } on DioException catch (ex) {
      if (ex.response != null) {
        if (ex.response!.data != null) {
          return Left(ex.response!.data.toString());
        } else {
          return Left(ex.response!.statusMessage ?? '인증 요청 - 서버 응답 오류 01');
        }
      } else {
        return Left(ex.message ?? '인증 요청 - 서버 응답 오류 02');
      }
    } catch (ex) {
      return Left(ex.toString());
    }
  }
}
```

인증에 성공하면 Access Token을 Secure Storage에 보관 처리 합니다. (flutter_secure_storage 패키지 사용)<br/>


Domain 계층 구현
-

실제 App에서 사용되는 데이터 모델을 다음과 같이 정의 합니다.<br/><br/>

**[Domain\models\authentication.dart]**<br/>
```dart
class Authentication {
  final String token;
  final DateTime expiration;

  const Authentication(this.token, this.expiration);
}
```

그리고 인증 처리의 비즈니스 로직의 추상체를 다음과 같이 정의 합니다.<br/><br/>

**[Domain\repository\authentication_repository.dart]**<br/>
```dart
import 'package:basic_architecture/Domain/models/authentication.dart';
import 'package:basic_architecture/Domain/request/login_request.dart';
import 'package:basic_architecture/Domain/request/register_request.dart';
import 'package:dartz/dartz.dart';

abstract class AuthenticationRepository{
  Future<Either<String,Authentication>> login(LoginRequest loginRequest);
  Future<Either<String,String>> logout();
  Future<Either<String,String>> forgotPassword(String id);
  Future<Either<String,Authentication>> register(RegisterRequest registerRequest);
}
```

로그인 요청에 필요한 정보도 다음과 같이 정의 합니다.<br/><br/>

**[Domain\request\login_request.dart]**<br/>
```dart
class LoginRequest {
  String id;
  String password;

  LoginRequest(this.id, this.password);
}
```

다음은 인증 처리에 대한 Use Case를 정의하는데 로그인, 로그아웃 각 개별적인 행동으로 각각 정의 합니다.<br/>
그리고 UseCase의 규칙중 하나인 '입력 값 으로 결과 값을 반환할 수 있는 단일 메서드만 외부에 제공' 규칙을 따라 BaseUseCase 클래스를 만들어 구현 합니다.<br/><br/>

**[Domain\usecase\base_usecase.dart]**<br/>
```dart
import 'package:dartz/dartz.dart';

/// Base UseCase
/// clean-architecture의 UseCase 규칙 : 한 개의 행동을 담당, Input과 Output의 단일 실행 메서드만 외부에 제공한다.
abstract class BaseUseCase<In, Out> {
  Future<Either<String, Out>> execute(In input);
}
```

**[Domain\usecase\login_usecase.dart]**<br/>
```dart
import 'package:basic_architecture/Domain/models/authentication.dart';
import 'package:basic_architecture/Domain/repository/authentication_repository.dart';
import 'package:basic_architecture/Domain/request/login_request.dart';
import 'package:basic_architecture/Domain/usecase/base_usecase.dart';
import 'package:dartz/dartz.dart';
import 'package:injectable/injectable.dart';

@injectable
class LoginUseCase implements BaseUseCase<LoginUseCaseInput, Authentication> {
  final AuthenticationRepository _authenticationRepository;

  LoginUseCase(this._authenticationRepository);

  @override
  Future<Either<String, Authentication>> execute(
      LoginUseCaseInput input) async {
    return await _authenticationRepository
        .login(LoginRequest(input.id, input.password));
  }
}

class LoginUseCaseInput {
  String id;
  String password;
  LoginUseCaseInput(this.id, this.password);
}
```

위에서 사용되는 AuthenticationRepository는 Domain계층의 추상 구현체이며, 실제 구현은 Data 계층에서 구현 되어 DIP(Dependency Inversion Principle) 되고 있다는 것이 중요 합니다.<br/>
이렇게 구현된 LoginUseCase는 Presentation 계층의 ViewModel UI 비즈니스 로직 담당측에서 의존성 주입등으로 사용되어 직접적인 Repository 접근 없이 단일 사용자 행동 관점으로 처리 할 수 있습니다.<br/>


Clean Architecture 도입 이후
-

사실 전체적으로 프로젝트 구조를 놓고 보았을때 Clean Architecture는 구조가 다소 복잡해 보일 수 있습니다.
각 계층간의 의존성을 엄격히 지키기 위해 그런것들을 해소하기 위해 몇가지 코드가 더 추가 되서 처음 설계 할때는 다소 많은 시간 투자가 필요할 수 도 있습니다.<br/><br/>

하지만 추후 유지보수 관점에서 생각해보면 장점이 훨씬 더 부각 될 것 입니다.<br/>
특히 이렇게 각 계층 구간을 분리 해놓으면 테스트가 간편하다는 장점이 크게 와닿을 수 있습니다.<br/>
이미 구현체는 추상 클래스나 인터페이스로 구현이 되어 있기 때문에 테스트 코드 작성시 해당 추상 구현체만 상속 받아 간편히 테스트할 수 있습니다.


***

**※ 참고로 이 프로젝트는 기본적인 앱 구현에 있어 필요한 다양한 기능들이 구현 되어 있습니다.**<br/>
(반응형 UI처리, skeleton loading 효과, 다국어 처리, BLoC를 통한 상태 관리, Dio Interceptor를 통한 API Access token 인증 처리, API Access token 만료시 재발급 로직 설명 등)

***


위 코드는 다음 Repository에서 확인할 수 있습니다.<br/>
> [flutter_basic_architecture](https://github.com/tyeom/flutter_basic_architecture)



{% include content_adsense.html %}
