---
title: (Fluter) Flutter MVVM with GetX
categories: Flutter
key: 20221215_01
comments: true
tags: Flutter MVVM GetX 상태관리
---

Flutter의 GetX 모듈을 사용해서 MVVM패턴 방식으로 앱을 설계하고 데이터를 다루는 방법에 대해 알아보도록 하겠습니다.<br/>
Flutter은 .NET의 WPF처럼 처음 구조 설게가 MVVM패턴을 염두해두고 설계되지 않아서 자체적으로 완벽한(?) 데이터 바인딩 기능을 지원하지 않고 있습니다.<br/>
GetIt이나 GetX 모듈 등을 이용해 빠르게 MVVM 구조로 뷰와 비즈니스로직을 분리하도록 구현해볼 수 있습니다.<br/>
MVVM 패턴에 대한 설명은 다음 아티클을 참조 하시면 됩니다. [링크 : MVVM 패턴이란](https://blog.arong.info/wpf/2022/01/21/WPF-WPF-MVVM-%ED%8C%A8%ED%84%B4%EC%97%90-%EB%8C%80%ED%95%B4.html#mvvm-%ED%8C%A8%ED%84%B4%EC%9D%B4%EB%9E%80)<br/>
프로젝트를 다루기전에 앞서 이번 MVVM구조에 사용된 GetX의 주요 기능에 대해 살펴보겠습니다.

<!--more-->

GetX 란?
-

GetX는 Flutter 개발에 있어서 좀더 편리하게 도와주고 생산성을 증가 시켜주는 미니멀 프레임워크 입니다. (미니멀 프레임워크라고 말할 수 있을 정도로 강력한 라이브러리 입니다.)<br/>
애초에 MVP나 MVVM같은 아키텍처 디자인 패턴을 고려해서 만들어지진 않았지만 의존성 주입, 상태관리를 위한 컨트롤러 처리 등 지원되는 기능을 이용해 쉽게 MVVM패턴으로 설계해 볼 수 있습니다.<br/>
그럼 GetX의 주요 기능에 대해 알아보고 해당 기능을 이용해서 MVVM패턴 적용을 알아보도록 하겠습니다.

Route 및 의존성 주입
-
GetX는 내비게이션 처리를 Route로 손쉽게 관리하고 사용할 수 있도록 제공하고 있습니다.<br/>
먼저 페이지들을 라우트 설정 관리로 다음과 같이 등록해주고<br/>
```dart
// GetX 라우트 설정
initialRoute: "/",
getPages: [
        // 메인
        GetPage(name: "/", page: () => AppView()),
        // 상세화면
        GetPage(
            name: "/detail/:id",
            page: () => DetailView(),
            transition: Transition.downToUp)
]
```

Get.toNamed() 함수로 해당 페이지로 쉽게 전환할 수 있습니다.<br/>
```dart
Get.toNamed('detail/1234');
```

GetX 라우트는 동적 URL타입을 지원하기에 DetailView페이지 name으로 명시된 URL형식 처럼 뒤에 고유 id나 특정 값을 같이 전달할 수 있습니다.<br/>
그리고 전환된 페이지에서는 전달된 값을 Get.parameters Map타입으로 받을 수 있습니다.<br/>

```dart
final String _articleId = Get.parameters['id'].toString();
```

위 처럼 DetailView 페이지로 전환시 같이 전달된 id값을 파라메터로 받아 처리할 수 있습니다.

### 의존성 주입
Route 페이지 전환시 의존성 주입을 동시에 같이 처리할 수 있습니다.<br/>
Route 설정시 GetPage에 BindingsBuilder를 통해 필요한 인스턴스를 주입시킬 수 있는데 의존성 주입 방법은 총 4가지의 방법이 있습니다.

- **Get.put() 방법**<br/>
라우트 처리시 binding을 Get.put()을 사용해 의존성 주입을 처리했을때는 해당 페이지로 접근 즉시 객체를 생성하고 인스턴스화 시킵니다.

- **Get.lazyPut&lt;T&gt;() 방법**<br/>
Get.lazyPut&lt;T&gt;() 방법은 지연처리로 객체를 즉시 생성하지 않고 접근해서 호출되는 시점에서 객체를 생성하고 인스턴스화 시켜줍니다.

- **Get.putAsync&lt;T&gt;() 방법**<br/>
Get.putAsync&lt;T&gt;() 방법은 비동기처리 이후 객체를 생성하고 인스턴스화 시켜줍니다.

- **Get.create&lt;T&gt;() 방법**<br/>
Get.create&lt;T&gt;() 방법은 객체를 기본적으로 싱글턴으로 관리하지 않고 의존성 주입한 객체에 접근시 마다 새롭게 객체를 생성하고 인스턴스화 시켜줍니다.

상태관리
-
GetX는 컨트롤러라는 개념을 통해 상태관리 처리를 도와줍니다. 상태괸리는 정적인 상태관리와 반응형 상태관리로 할 수 있는데 두 차이점은<br/>
정적 상태관리는 값 변경시 수동으로 update()처리를 해주어야 해당 컨트롤러를 참조하는 곳에 변경 통보가 되어 데이터를 갱신할 수 있고,<br/>
반응형 상태관리는 Observable처리로 데이터 변경을 감지하고 변경 통보가 이루어 집니다.<br/>
여기서는 반응형 상태관리에 대해 간단히 살펴보겠습니다.<br/>
반응형 상태관리로 처리하기 위해서는 해당 데이터 타입은 GetX에서 사용되는 Rx 리엑트 타입으로 사용되어야 합니다.<br/>
<table>
        <tr>
                <th>Dart</th>
                <th>GetX (Rx)</th>
        </tr>
        <tr>
                <td>int</td>
                <td>RxInt</td>
        </tr>
        <tr>
                <td>bool</td>
                <td>RxBool</td>
        </tr>
        <tr>
                <td>double</td>
                <td>RxDouble</td>
        </tr>
        <tr>
                <td>List&lt;T&gt;</td>
                <td>RxList&lt;T&gt;</td>
        </tr>
        <tr>
                <td>Custom Object</td>
                <td>Rx&lt;T&gt;</td>
        </tr>
</table>

GetX Rx타입으로 변경은 각 타입의 obs확장을 통해 바로 캐스팅할 수 있습니다.<br/>
```dart
RxInt num = 1.obs;
RxList<ArticleModel> _articleList = <ArticleModel>[].obs;
RxBool flag = false.obs;
```

그리고 사용될때는 value get함수를 통해 사용할 수 있습니다. 또한 값 업데이트(변경)시 해당 생성자로 바로 업데이트되는 값을 전달하면 변경 통보까지 자동으로 처리 됩니다.<br/>

```dart
// 데이터 초기화
RxList<ArticleModel> articleList = <ArticleModel>[].obs;
List<ArticleModel> dataList = get from server or get from db
articleList(dataList);
```

```dart
// 데이터 추가
RxList<ArticleModel> articleList = <ArticleModel>[].obs;
articleList.add(...);
```

```dart
// 데이터 제거
RxList<ArticleModel> articleList = <ArticleModel>[].obs;
articleList.remove(...);
```

이렇게 간략하게 GetX의 주요기능을 살펴 보았고 해당 기능을 이용해서 MVVM 간단한 샘플 프로젝트를 만들어 보겠습니다.

MVVM with GetX
-

MVVM패턴을 적용해서 만들어볼 App은 서버로부터 데이터를 받아서 페이지에 데이터의 리스트를 표시하고 받아온 데이터의 값 변경, 라우트를 이용해 화면 전환, 데이터 삭제 처리 기능이 있는
간단한 App입니다. 최종 완성되는 화면은 다음과 같습니다.<br/>
**[완성 App]**<br/>
![image](https://user-images.githubusercontent.com/13028129/207777256-a172d630-7585-47a2-bb93-4f997379e265.png)<br/>

### Model 구성

**Model은 데이터가 포함하는 엔티티의 속성과 데이터를 처리하는 비즈니스 로직을 처리하는 역할을 합니다. 모델에는 비즈니스 로직을 처리하는 함수도 포함될 수 있습니다.**<br/>

먼저 데이터의 모델을 다음과 같이 생성합니다.<br/>
**[article_model.dart]**<br/>
```dart
class ArticleModel {
  final String? id;
  final List<String>? photoList;
  final String title;
  final String content;
  final String town;
  final num price;
  final int? likeCnt;
  final int? readCnt;
  final int? uploadTime;
  final int? updateTime;
  final String category;
  bool favorites;

  ArticleModel(
      {this.id,
      this.photoList,
      required this.title,
      required this.content,
      required this.town,
      required this.price,
      this.likeCnt,
      this.readCnt,
      this.updateTime,
      this.uploadTime,
      required this.category,
      required this.favorites});

  factory ArticleModel.fromJson(Map<String, dynamic> json) => ArticleModel(
      id: json['id'],
      photoList: json['photoList'].cast<String>(),
      title: json['title'],
      content: json['content'],
      town: json['town'],
      price: json['price'],
      likeCnt: json['likeCnt'],
      readCnt: json['readCnt'],
      uploadTime: json['uploadTime'],
      updateTime: json['updateTime'],
      category: json['category'],
      favorites: json['favorites'] == null ? false : json['favorites'] as bool);

  Map<String, dynamic> toJson() => {
        'photoList': photoList,
        'title': title,
        'content': content,
        'town': town,
        'price': price,
        'likeCnt': likeCnt,
        'readCnt': readCnt,
        'category': category,
        'favorites': favorites,
      };
}
```

화면은 총 4개의 화면입니다.<br/>
- AppView : 하단 내비게이션 Scaffold가 구성되어 있는 전체 App 화면
- HomeView : 데이터 목록 화면
- FavoritesView : 즐겨찾기 화면
- DetailView : 데이터 상세 내용 화면

위 페이지의 라우터 구성은 다음과 같이 처리 합니다.<br/>
> 참고로 GetX를 사용하면 기본 MaterialApp은 GetMaterialApp으로 바꿔서 사용됩니다.

**[main.dart]**
```dart
// GetX 라우트 설정
initialRoute: "/",
getPages: [
        // 메인
        GetPage(name: "/", page: () => AppView()),
        // 상세화면
        GetPage(
            name: "/detail/:id",
            page: () => DetailView(),
            binding: BindingsBuilder(
                () => Get.lazyPut<DetailViewModel>(() => DetailViewModel())),
            transition: Transition.downToUp),
],
```
HomeView와 FavoritesView는 AppView안에서 내비게이션으로 보여집니다. 또한 DetailView 라우트 설정시 DetailViewModel(GetX 컨트롤러)을 의존성 주입 처리 합니다.<br/>
그리고 GetMaterialApp의 initialBinding을 통해 App초기화시 바인딩 설정을 할 수 있는데 여기서 분리 바인딩으로 상시적으로 표시되는 AppViewModel과 HomeViewModel<br/>
그리고 데이터 요청 담당인 DataService를 의존성 주입으로 사용할 수 있도록 처리합니다.<br/>

**[main.dart]**
```dart
return GetMaterialApp(
      title: 'Flutter MVVM with GetX Sample',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: const AppView(),
      // 최초 App실행시 컨트롤러 바인딩
      initialBinding: InitialViewModel(),
      
[...기타생략...]
```

**[initial_viewmodel.dart]**
```dart
import 'package:get/get.dart';
import 'package:mvvm_getx_sample/src/services/data_service.dart';
import 'package:mvvm_getx_sample/src/viewmodels/app_viewmodel.dart';
import 'package:mvvm_getx_sample/src/viewmodels/detail_viewmodel.dart';
import 'package:mvvm_getx_sample/src/viewmodels/home_viewmodel.dart';

class InitialViewModel implements Bindings {
  @override
  void dependencies() {
    Get.lazyPut<DataService>(() => DataService());
    Get.lazyPut<AppViewModel>(() => AppViewModel());
    Get.lazyPut<HomeViewModel>(() => HomeViewModel());
  }
}
```

이렇게 main에서 최초 바인딩 설정과 라우트 구성이 되었습니다.<br/>
그러면 각 View에서 ViewModel(GetX 컨트롤러)을 통해 데이터를 표시할 수 있습니다.

### Service 구성

**Service는 서버와 통신해서 실제 데이터를 받아 처리하는 순수 데이터 처리 로직 역할을 하는 부분 입니다. 위에 의존성 주입 처리에서 DataService를 의존성 주입했던 부분 입니다.**<br/>

**[data_service.dart]**
```dart
class DataService {
  // 아티클 리스트 데이터 요청
  Future<List<ArticleModel>?> fetchArticles(String phoneNum) async {
    ...
  }

  // 아티클 리스트 json to list parser
  List<ArticleModel> parseArticles(dynamic jsonMap) {
    ...
  }

  // 아티클 상세 데이터 요청
  Future<ArticleModel?> fetchDetailArticle(String id) async {
    ...
  }

  // 아티클 데이터 삭제
  Future<bool> removeArticle(String id) async {
    ...
  }
}
```

### View 구성

**뷰는 뷰모델을 통해 데이터를 어떻게 표현해 줄 것인지 가시화 처리하는 역할 입니다. 그리고 뷰모델에서 데이터 변경 통보가 되었을때 그에 맞게 다시 표시합니다.**<br/>

GetView&lt;T&gt;를 상속받으면 바로 GetX 컨트롤러에 접근할 수 있습니다. 또한 GetView&lt;T&gt;를 사용하려면 StatelessWidget으로 페이지가 구성되어야 합니다.<br/>
**[home_view.dart]**<br/>
```dart
class HomeView extends GetView<HomeViewModel> {
  const HomeView({super.key});

  // Article 데이터 표시
  Widget _makeArticleWidget(ArticleModel article) {
    return GestureDetector(
      onTap: () async {
        print(article.id);
        final result = await Get.toNamed('/detail/${article.id}');
        if (result) {
          print('데이터 갱신');
          controller.fetchArticles();
        }
      },
      child: Container(
          ...[생략]...
                            GestureDetector(
                              onTap: () {
                                // 즐겨찾기 업데이트
                                controller.updateFavorites(
                                    article.id!, !article.favorites);
                              },
                              child: Container(
                                margin: EdgeInsets.only(right: 10),
                                child: Icon(article.favorites
                                    ? Icons.star
                                    : Icons.star_outline),
                              ),
                            ),
                          ],
                        ),
                        
            )
          ])),
    );
  }

  Widget _bodyWidget() {
    return Container(child:
        // ViewModel(컨트롤러) 통해 데이터 표시
        Obx(
      () {
        // 데이터 요청중
        if (controller.isDataFetching.value == true) {
          return const Center(
              child: CircularProgressIndicator(
                  color: Color.fromARGB(255, 252, 113, 49)));
        }
        // 데이터 요청 오류
        else if (controller.articleList == null) {
          return Center(child: Text("데이터 요청중 오류가 발생되었습니다."));
        }
        // 데이터 없음
        else if (controller.articleList!.length <= 0) {
          return Center(child: Text("데이터가 없습니다."));
        } else {
          // 데이터 요청 완료
          return Container(
            child: ListView.separated(
                padding: EdgeInsets.symmetric(horizontal: 10),
                physics: ClampingScrollPhysics(), // bounce 효과 제거
                itemBuilder: (BuildContext _context, int index) {
                  return _makeArticleWidget(controller.articleList![index]);
                },
                separatorBuilder: (BuildContext _context, int index) {
                  return Container(
                      height: 1, color: Color.fromARGB(150, 163, 155, 155));
                },
                itemCount: controller.articleList!.length),
          );
        }
      },
    ));
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
        appBar: null,
        body: _bodyWidget(),
        floatingActionButton: SpeedDial(
          icon: Icons.add,
          activeIcon: Icons.close,
          spacing: 3,
          children: [
            SpeedDialChild(
                onTap: () {
                  controller.addArticles();
                },
                child: const Icon(Icons.add_box),
                label: '추가'),
            SpeedDialChild(
                onTap: () {
                  controller.fetchArticles();
                },
                child: const Icon(Icons.refresh),
                label: '새로고침')
          ],
        ));
  }
}

```

**HomeView에서는 HomeViewModel을 통해 실제 데이터를 가시화 처리 역할을 담당합니다. 이렇게 실제 데이터 비즈니스 로직과는 전혀 연관이 없게 됩니다.**<br/>
View에서 ViewModel 접근은 GetView&lt;T&gt;의 controller get함수를 통해 접근할 수 있습니다.

### ViewModel 구성

**ViewModel에서는 의존성 주입된 서비스를 받아 실제 데이터를 요청하고 데이터 변경이 되었을때 자신을 참조하고 있는 View에게 변경 통보 역할을 합니다.**<br/>
**[home_viewmodel.dart]**<br/>
```dart
class HomeViewModel extends GetxController {
  final DataService _dataService = Get.find<DataService>();

  // 데이터 요청중
  RxBool isDataFetching = false.obs;

  // ArticleModel 모델 리스트
  RxList<ArticleModel> _articleList = <ArticleModel>[].obs;
  List<ArticleModel>? get articleList => _articleList.value;

  @override
  void onInit() {
    // TODO: implement onInit
    super.onInit();

    this.fetchArticles();
  }

  // 데이터 요청
  void fetchArticles() async {
    // 데이터 요청중
    isDataFetching(true);

    List<ArticleModel>? articleList =
        await _dataService.fetchArticles('01077778888');

    // 데이터 요청 완료
    isDataFetching(false);

    if (articleList != null) {
      _articleList(articleList);
    }
  }

  // 데이터 추가
  void addArticles() {
    _articleList.add(...);
  }

  // 즐겨찾기 목록 필터
  List<ArticleModel> getFavoritesList() {
    if (_articleList.value == null || _articleList.length <= 0) [];

    return _articleList.where((item) => item.favorites).toList();
  }

  void updateFavorites(String id, bool isFavorites) {
    ArticleModel? article =
        _articleList.firstWhereOrNull((element) => element.id == id);

    if (article != null) article.favorites = isFavorites;

    // GetX의 RxList는 Item요소의 변수값이 바뀌었는지 감지할 수 없다. [View에서 BindingStream 사용하지 않음]
    // 수동으로 refresh처리 해준다.
    _articleList.refresh();
  }
}
```

ViewModel은 GetxController를 상속받아서 반응형 상태관리에 의해 View에 변경 통보 처리를 해줄 수 있습니다.<br/>
GetxController는 라이프 사이클이 존재 하는데 그중 처음으로 호출되는 onInit()함수가 있습니다.<br/>
해당 함수에서 서비스를 통해 데이터를 요청해서 View에서 사용되는 데이터에 초기화 해주면 RxList&lt;T&gt; Observable 타입은 GetX 컨트롤러에 의해<br/>
자동으로 해당 View에게 변경 통보가 일어나고 View에서는 해당 부분에 대해서만 다시 렌더링 되어 집니다.<br/>
이렇게 GetX를 사용하면 간단하게 MVVM패턴을 적용해서 View와 데이터 처리 영역을 완전 분리 처리 할 수 있게 됩니다.


위 코드는 다음 Repository에서 확인할 수 있습니다.<br/>
[Flutter_MVVM_GetX](https://github.com/tyeom/Flutter_MVVM_GetX)



{% include content_adsense.html %}
