---
title: (Flutter) BLoC 패턴 ＃1
categories: Flutter
key: 20230110_01
comments: true
tags: Flutter bloc 상태관리 로직분리 pattern bloc_parrern
---

Flutter 개발에 있어서 상태관리는 반드시 필요한 부분이고 상태관리에 도움되는 패키지는 여러가지가 존재합니다.<br/>
대표적으로 [Provider](https://pub.dev/packages/provider), [GetX](https://pub.dev/packages/get), [BloC](https://pub.dev/packages/flutter_bloc) 패키지들이 존재하는데
이 패키지들은 모두 비즈니스 로직분리를<br/>
목표로 BLoC패턴 기반으로 되어 있습니다. 이번 글에서는 BLoC패턴이 무엇이고,<br/>
왜 사용되는지 그리고 BLoC패턴을 구현하기 위해선 어떻게 구현해야 하는지 알아보겠습니다.

<!--more-->


> 이 글에서 다루는 코드는 다음 Repository에서 확인할 수 있습니다.<br/>
> [Code_check - flutter_bloc_example](https://github.com/tyeom/flutter_bloc_example)


BLoC(Business Logic Component) 패턴이란?
-

한마디로 정의하면 BLoC(Business Logic Component) 패턴은 상태 변화에 있어 UI(View)와 Busuness Logic을 분리하는데 사용되는 디자인 패턴 입니다.<br/>
[bloclibrary](https://bloclibrary.dev/#/whybloc)의 문서를 보면 보다 자세한 내용을 볼 수 있는데 BLoC은 'Simple', 'Powerful', 'Testable' 이 3가지 핵심 가치를 중심으로 개발되었다고 설명하고 있습니다.<br/>

위에서 거론된 패키지들 모두 이 BLoC 패턴 기반으로 클래스를 설계할 수 있도록 해주는 패키지들 입니다.<br/>
자 그럼 Dart에서 BLoC 패턴은 어떤 방식으로 처리 되어 있는지 알아보겠습니다.

왜 BLoC을 사용해야 할까?
-

**첫번째로 위에서 말했듯 UI(View)와 Busuness Logic을 분리함으로서 유지보수하기 좋은 구조로 설계할 수 있습니다.**<br/>
UI의 입장에서는 데이터가 처리되는 과정을 알필요가 없습니다. 단지 필요한 데이터를 요청하고 그 결과만을 가지고서 화면에 맞게 표시해 주는 역할에만 충실하면 됩니다.<br/>
하지만 UI처리 코드와 Busuness Logic이 같이 섞여 있는 코드라면 복잡성은 높아지고 유지보수도 힘들어 집니다. 추가로 사이드 이펙트 오류가 발생할 확율도 높아 집니다.<br/><br/>

**두번째로 이렇게 분리된 코드는 상태관리되기 때문에 UI에 있어 변경된 부분만 다시 그려주거나 상태만 변경해줌으로써 앱은 빠른 성능이 보장됩니다.**<br/>
이렇게 상태관리가 되어 있지 않은 앱은 불필요한 UI 다시 그리기 처리, 상태체크 등 작업이 발생됨으로써 그만큼 좋은 성능을 낼 순 없습니다.<br/><br/>

**세번째로는 테스트 하기 쉽고 재사용을 가능하게 해줍니다.**<br/>
![image](https://user-images.githubusercontent.com/13028129/211696252-b3803724-2c93-43af-bf79-4593b741f7fe.png)<br/>
TDD 개발 방법론 적용시 BLoC적용은 특정 비즈니스 로직 부분만 유닛 테스트 -> 리펙토링 반복 개발을 수월하게 해줍니다.<br/><br/>

> MVVM 아키텍처 패턴과 BLoC 패턴의 차이점은 ?<br/>
> MVVM 과 BLoC는 모두 큰 목표로 Busuness Logic을 분리하는데 있어 유사하지만 다음과 같은 개념적인 차이점이 있습니다.<br/>
> MVVM은 View와 ViewModel간 처리를 바인딩을 통해 **'View의 추상적인 View관련'** View Logic을 분리하자는데 목표가 있고,<br/>
> BLoC는 Stream을 통해 **'꼭 View가 아니더라도'** Busuness Logic을 분리하자는데 목표가 있습니다.


BLoC의 구조
-
![image](https://user-images.githubusercontent.com/13028129/211697658-5bee7701-8b73-4e0f-845a-da73f570cc18.png)<br/>
BLoC 패턴의 구조를 간단히 살펴보면 다음과 같습니다. UI는 특정 이벤트에 대한 스트림을 구독하고 언제든지 데이터를 받을 수 있도록 준비가 되어 있습니다.<br/>
이 상태에서 비즈니스 로직과 상태관련 처리는 bloc 안에서 처리가 되는 것입니다. bloc에서는 데이터를 처리하고 스트림으로 전송역할만 충실하면 됩니다.


Stream 이란?
-

위에서 BLoC의 구조 설명에 나왔듯 BLoC에서 Stream(스트림)은 중요한 개념 입니다.<br/>
Stream은 데이터 및 이벤트가 수신되는 하나의 통로 이면서 그 데이터는 언제 수신될지 모르는 상황 일때 사용됩니다.<br/>
이 처럼 Stream은 데이터 처리를 비동기로 처리할때 사용됩니다.<br/>
> Dart **Future&lt;T&gt; class**와 **Stream&lt;T&gt; class**의 차이점은<br/>
> **Future&lt;T&gt;** 는 작업이 아직 완료되지 않는 상태를 반환하고 작업이 끝나기를 '대기' 할 수 있도록 처리해 주고<br/>
> **Stream&lt;T&gt;** 은 구독자 패턴처럼 구독자에게 변화가 발생되면 알려주는 방식 입니다.<br/><br/>

Stream 구독의 종류는 단일 구독과 브로드캐스트가 있습니다.<br/>
단일 구독 Stream은 구독자 단일에게 주어지는 Stream으로 보통 File I/O 처리를 위한 Stream이 있습니다.<br/>
Broadcast stream(브로드캐스트 스트림)은 구독하고 있는 여러 구독자에게 발생할 수 있으며 이벤트 통보 처리에 적합 합니다.<br/><br/>

Dart에서 Stream 처리는 **<span style="color: rgb(107, 173, 222);">StreamController&lt;T&gt;</span>** 클래스로 전달되는 데이터를 제공하고,<br/>
**<span style="color: rgb(107, 173, 222);">Stream&lt;T&gt;</span>** 클래스로 제공된 데이터를 전달하거나 브로드캐스트가 처리 됩니다.<br/>
데이터 전달은 방법은 **<span style="color: rgb(107, 173, 222);">StreamSink&lt;T&gt;</span>** 클래스에서 Add() 함수로 제공 되는데 **<span style="color: rgb(107, 173, 222);">StreamController&lt;T&gt;</span>** 에서 sink 속성으로 접근할 수 있습니다.<br/>
**<span style="color: rgb(107, 173, 222);">StreamSink&lt;T&gt;</span>** 로 데이터 추가가 된다면 **<span style="color: rgb(107, 173, 222);">Stream&lt;T&gt;</span>** 를 구독하고 있는 구독자에게 
전송 처리 됩니다. 전송 처리는 단일 구독 방식인지 브로드캐스트 방식인지에 따라 여러 구독자에게 전파 시킬 수 도 있습니다.<br/>
 **<span style="color: rgb(107, 173, 222);">StreamController&lt;T&gt;</span>** 클래스는 기본 생성자로는 단일 구독으로 처리되고  **<span style="color: rgb(107, 173, 222);">StreamController&lt;T&gt;.broadcast</span>** 생성자로 브로드캐스트 할 수 있습니다.


Flutter로 살펴보는 예제
-

직접 코드로 BLoC 패턴을 구현해보고 BLoC 사용 없이 구현했을때 단점을 확인해 보겠습니다.<br/>
예제는 간단히 가상의 서버로 데이터 요청을을 처리 한다는 가정하에 Todo List를 표시하고 데이터를 조작하는 앱을 만들어 보면서 확인해 보겠습니다.

### BLoC 패턴 미 적용의 단점

Todo List의 데이터를 표시하고 내용을 추가할 수 있는 위젯을 별도의 위젯으로 구현해서 사용하고 데이터의 변화가 발생 했을때 해당 위젯이 매번 다시 갱신 되는 상황입니다.<br/>
우선 예제에 필요한 모델과 데이터 요청 역할을 하는 Repository를 다음과 같이 구현 합니다. 참고로 이 부분은 추후 예제에서 공통적으로 사용되는 부분 입니다.<br/><br/>
**[todo_model.dart]**<br/>
```dart
class TodoModel {
  final String? uuid;
  final String title;
  final String? createDT;

  TodoModel({
    this.uuid,
    required this.title,
    this.createDT,
  });

  factory TodoModel.fromJson(Map<String, dynamic> json) => TodoModel(
      uuid: json['uuid'] as String,
      title: json['title'] as String,
      createDT: json['createDT'] as String);

  Map<String, dynamic> toJson() => {
        'uuid': uuid,
        'title': title,
        'createDT': createDT,
      };
}
```

데이터 모델은 간단하게 고유값의 uuid와 title, 생성날짜 이렇게 세가지 정보가 있습니다. 그리고 json 직렬화/역직렬화 처리에 필요한 함수도 정의했습니다.<br/>
그리고 Repository는 다음과 같이 구현합니다.<br/><br/>

**[todo_repository.dart]**<br/>
```dart
// 실제 서버 요청 부분
import 'package:bloc_example/models/todo_model.dart';

class TodoRepository {
  Future<List<Map<String, dynamic>>> getListTodo() async {
    print('TodoRepository - getListTodo');

    // 서버 요청 딜레이 5sec
    await Future.delayed(Duration(seconds: 5));

    return [
      {
        'uuid': 'aa-bb-cc-dd',
        'title': '할일 1',
        'createDT': DateTime.now().toString(),
      },
    ];
  }

  Future<Map<String, dynamic>> createTodo(TodoModel todo) async {
    print('TodoRepository - createTodo');

    // 서버 요청 딜레이 5sec
    await Future.delayed(Duration(seconds: 5));

    return todo.toJson();
  }

  Future<bool> deleteTodo(String uuid) async {
    print('TodoRepository - deleteTodo');

    // 서버 요청 딜레이 5sec
    await Future.delayed(Duration(seconds: 5));

    return false;
  }
}
```

실제 서버에 요청된것으로 간주해서 모든 함수에 약 5초의 딜레이 처리를 해주었고 데이터 가져오기(getListTodo), 데이터 추가(createTodo), 데이터 삭제(deleteTodo) 함수가 구현되어 있습니다.<br/>
데이터 삭제(deleteTodo)는 삭제 요청 성공 유무를 반환하는데 **항상 실패(false)로 반환 되도록 했습니다.** (이유는 잠시후 다른 예제코드 설명에서)<br/>
그리고 View작업은 다음과 같이 구현합니다.<br/><br/>

**[listView_add_widget2.dart]**

```dart
final TextEditingController _titleController = TextEditingController();
final TodoRepository _repository = TodoRepository();
final List<TodoModel> _todoList = [];

// 데이터 로드
Future<List<TodoModel>> _listTodo() async {
    try {
      if (_isFirstLoaded) return _todoList;

      final todoJson = await _repository.getListTodo();

      _todoList.addAll(todoJson
          .map<TodoModel>(
            (p) => TodoModel.fromJson(p),
          )
          .toList());
      _isFirstLoaded = true;
    } catch (ex) {
      print(ex.toString());
    }

    return _todoList;
}

// 데이터 추가
Future<void> _addTodo(String title) async {
    try {
      final todoJson = await _repository.createTodo(TodoModel(
          uuid: 'a-b-c-d', title: title, createDT: DateTime.now().toString()));
      setState(() {
        _todoList.add(TodoModel.fromJson(todoJson));
      });
    } catch (ex) {
      print(ex.toString());
    }
}

@override
Widget build(BuildContext context) {
    print('ListView_AddWidget2 - build !!!');

    return Container(
      child: Padding(
        padding: const EdgeInsets.symmetric(horizontal: 15),
        child: Column(children: [
          Row(
            mainAxisAlignment: MainAxisAlignment.start,
            children: [
              Expanded(
                child: TextField(
                  controller: _titleController,
                ),
              ),
              IconButton(
                onPressed: () => _addTodo(_titleController.text),
                icon: const Icon(Icons.add_task),
              )
            ],
          ),
          const SizedBox(
            height: 20,
          ),
          Expanded(
              child: FutureBuilder(
            future: _listTodo(),
            builder: (_, snapshot) {
              if (snapshot.hasData == false) return Container();

              return ListView.separated(
                  itemBuilder: (_, index) {
                    final item = snapshot.data![index] as TodoModel;

                    return Column(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      children: [
                        Row(
                          mainAxisAlignment: MainAxisAlignment.start,
                          children: [
                            Expanded(
                                child: Text(
                              item.title,
                              style: const TextStyle(
                                  fontSize: 20, fontWeight: FontWeight.bold),
                            )),
                            GestureDetector(
                              onTap: () {
                                setState(() {
                                  _todoList
                                      .removeWhere((p) => p.uuid == item.uuid);
                                });
                              },
                              child: const Icon(Icons.delete),
                            )
                          ],
                        ),
                        Text(
                          item.createDT.toString(),
                          style:
                              const TextStyle(fontSize: 16, color: Colors.grey),
                        )
                      ],
                    );
                  },
                  separatorBuilder: (_, index) => const Divider(),
                  itemCount: snapshot.data!.length);
            },
          )),
        ]),
      ),
    );
}
```

Todo List를 ListView 위젯으로 표시하고 데이터를 추가할 수 있는 화면 입니다.<br/>
이렇게 구현한 listView_add_widget2 위젯을 붙여서 확인해 보면 'ListView_AddWidget2 - build !!!' 메세지가 출력되는 것 을 확인할 수 있습니다.<br/>
이후 데이터를 추가하거나 삭제할때 마다 매번 'ListView_AddWidget2 - build !!!' 메세지 출력이 확인 됩니다.<br/>
![그림1](https://user-images.githubusercontent.com/13028129/211708498-4522a82f-661d-4ea0-a8ad-600d5396d779.png)<br/>
이렇게 상태변화가 발생될때 마다 전체적으로 위젯이 계속 새로 갱신되는 것을 알 수 있습니다. 이러한 상황은 위젯 트리가 다소 복잡한 구조인 경우 상위에서 하위 또는 하위에서 상위로 데이터 전파가 
발생된다면 그 영향받는 모든 위젯들은 전체적으로 갱신되는 현상이 생기게 됩니다. 이렇게 되면 앱의 전체적으로 성능이 떨어질 수 밖에 없습니다.


### BLoC 구현

위 예제를 BLoC를 적용하기 위해 BLoC을 구현해서 적용해 보겠습니다.<br/>
데이터 로드/추가/삭제 관련 비즈니스 로직을 모두 BLoC으로 분리하고 UI는 스트림을 구독해서 데이터 표시 되도록 처리 합니다. 우선 BLoC를 다음과 같이 구현합니다.<br/><br/>

**[normal_todo_bloc.dart]**<br/>
```dart
import 'dart:async';
import 'package:bloc_example/models/todo_model.dart';
import 'package:bloc_example/repository/todo_repository.dart';
import 'package:uuid/uuid.dart';

/// [BLoC 패턴 직접 구현]
/// 비즈니스 로직 처리
class NormalTodoBloc {
  late List<TodoModel> _todoList = [];

  // todo List 상태
  final StreamController<List<TodoModel>> _todoListSubject =
      StreamController<List<TodoModel>>.broadcast();
  Stream<List<TodoModel>> get todoList => _todoListSubject.stream;

  // 에러 상태
  final StreamController<String> _errorMessageSubject =
      StreamController<String>.broadcast();
  Stream<String> get errorMessage => _errorMessageSubject.stream;

  // Repository [이렇게 직접 정의 보다 DI처리로 생성자 의존성 주입으로 받는 것이 좋다.]
  final TodoRepository _repository = TodoRepository();

  // 데이터 로드
  void listTodo() async {
    try {
      final todoJson = await _repository.getListTodo();

      _todoList = todoJson
          .map<TodoModel>(
            (p) => TodoModel.fromJson(p),
          )
          .toList();

      _todoListSubject.sink.add(_todoList);
    } catch (ex) {
      final String errorMessage = ex.toString();
      _errorMessageSubject.sink.add(errorMessage);
    }
  }

  // 데이터 추가
  void createTodo(String title) async {
    try {
      /// fake 데이터 [실제 서버에 요청전 결과 데이터를 빠르게 보여주기 위해]
      /// 임시로 새로 만들어진 TodoModel데이터를 바로 추가해서 스트림 전송
      final fakeTodoList = [
        ..._todoList,
        TodoModel(title: title),
      ];
      _todoListSubject.sink.add(fakeTodoList);

      /// 실제 서버 요청 처리
      /// id, createDT 정보는 서버에서 처리됨을 간주
      const uuid = Uuid();
      final idv4 = uuid.v4();
      TodoModel newTodo = TodoModel(
          uuid: idv4, title: title, createDT: DateTime.now().toString());
      final newTodoJson = await _repository.createTodo(newTodo);

      _todoList.add(TodoModel.fromJson(newTodoJson));
      _todoListSubject.sink.add(_todoList);
    } catch (ex) {
      final String errorMessage = ex.toString();
      _errorMessageSubject.sink.add(errorMessage);
    }
  }

  // 데이터 삭제
  deleteTodo(String uuid) async {
    try {
      /// fake 데이터 [실제 서버에 요청전 결과 데이터를 빠르게 보여주기 위해]
      /// 로컬에서 먼저 삭제처리 후 스트림 전송
      final currentTodoList = [
        ..._todoList,
      ];
      final fakeTodoList =
          currentTodoList.where((item) => item.uuid != uuid).toList();

      _todoListSubject.sink.add(fakeTodoList);

      // 실제 서버에 삭제 요청 처리
      final result = await _repository.deleteTodo(uuid);
      // 삭제 요청 실패시
      if (result == false) {
        _todoListSubject.sink.add(_todoList);
        _errorMessageSubject.sink.add('삭제 요청중 오류 발생');
      }
      else {
        // 로컬 데이터 실제 삭제 처리
        _todoList.removeWhere((item) => item.uuid == uuid);
      }
    } catch (ex) {
      final String errorMessage = ex.toString();
      _errorMessageSubject.sink.add(errorMessage);
    }
  }

  void dispose() {
    _todoListSubject.close();
    _errorMessageSubject.close();
  }
}
```

코드를 보면 repository를 통해 데이터 요청을 하고 관련된 비즈니스 로직 처리 후 **<span style="color: rgb(107, 173, 222);">StreamController&lt;T&gt;</span>** 로 데이터를 제공합니다.<br/>
UI는 **<span style="color: rgb(107, 173, 222);">StreamController&lt;T&gt;</span>** 로 제공되는 **<span style="color: rgb(107, 173, 222);">Stream&lt;T&gt;</span>** 를 구독하고 
변경 통보를 받고 데이터를 표시합니다.<br/><br/>

여기서 한가지 살펴볼 부분이 데이터 추가/삭제시 fake처리가 되어 있는 부분 입니다.<br/>
데이터 추가 요청시 실제 서버에 요청하기전 사용자에게 입력받은 데이터로 추가될 데이터를 만들어서 미리 UI에 표시 하고 있습니다.<br/>
이후에 서버 요청이 정상적으로 응답이 되었다면 실제 서버에서 받아온 고유 uuid나 생성날짜 등 정보를 업데이트 시켜주고 있습니다.<br/>
이러한 처리 방법은 UI에 반영 되는 데이터를 빠르게 미리 표시 함으로써 사용자의 반응을 빠릿하게 보이도록 속임수 기법(?)으로 활용할 수 있습니다.<br/>
데이터 삭제 처리도 똑같은 방식으로 fake처리 되서 항상 실패(false)를 반환하기 때문에 데이터를 삭제해도 다시 보여질 것 입니다.<br/>
다만 예제에서는 서버에서 받아온 데이터를 업데이트하고 전체 데이터를 다시 갱신하는데 실제로는 백그라운드상에서 해당 데이터의 속성만 업데이트 할 수 있도록 처리하는 것이 좋습니다.<br/><br/>

이렇게 만들어진 BLoC은 UI에서 스트림 구독 처리를 해주고 데이터 요청 또한 BLoC을 통해 요청하고 구독하고 있는 스트림 데이터만 받아서 표시해주도록 하면 됩니다.<br/>
다음은 UI부분 입니다.<br/><br/>

**[listView_add_widget.dart]**<br/>
```dart
import 'package:bloc_example/bloc/normal_todo_bloc.dart';
import 'package:bloc_example/models/todo_model.dart';
import 'package:bloc_example/views/app_view_with_normal_bloc.dart';
import 'package:flutter/material.dart';

/// Normal Bloc 패턴 용도 위젯
/// 상태관리가 제대로 되면서 위젯 렌더링이 다시 발생되는지 확인용도로
/// 임의로 별도 위젯으로 처리
class ListView_AddWidget extends StatelessWidget {
  final NormalTodoBloc _todoBloc = AppViewWithNormalBloc.todoBloc;
  final TextEditingController _titleController = TextEditingController();

  @override
  Widget build(BuildContext context) {
    print('ListView_AddWidget - build !!!');

    return Container(
      child: Padding(
        padding: const EdgeInsets.symmetric(horizontal: 15),
        child: Column(children: [
          Row(
            mainAxisAlignment: MainAxisAlignment.start,
            children: [
              Expanded(
                child: TextField(
                  controller: _titleController,
                ),
              ),
              IconButton(
                onPressed: () => _todoBloc.createTodo(_titleController.text),
                icon: const Icon(Icons.add_task),
              )
            ],
          ),
          const SizedBox(
            height: 20,
          ),
          Expanded(
              child: StreamBuilder(
            stream: _todoBloc.todoList,
            initialData: [],
            builder: (_, snapshot) {
              return ListView.separated(
                  itemBuilder: (_, index) {
                    final item = snapshot.data![index] as TodoModel;

                    return Column(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      children: [
                        Row(
                          mainAxisAlignment: MainAxisAlignment.start,
                          children: [
                            Expanded(
                                child: Text(
                              item.title,
                              style: const TextStyle(
                                  fontSize: 20, fontWeight: FontWeight.bold),
                            )),
                            GestureDetector(
                              onTap: () => _todoBloc.deleteTodo(item.uuid!),
                              child: const Icon(Icons.delete),
                            )
                          ],
                        ),
                        Text(
                          item.createDT.toString(),
                          style:
                              const TextStyle(fontSize: 16, color: Colors.grey),
                        )
                      ],
                    );
                  },
                  separatorBuilder: (_, index) => const Divider(),
                  itemCount: snapshot.data!.length);
            },
          )),
        ]),
      ),
    );
  }
}
```

코드는 listView_add_widget2 와 동일한데 StreamBuilder 위젯을 사용해서 bloc의 List&lt;TodoModel&gt; Stream을 구독하는 부분만 달라졌습니다.<br/>
이 위젯을 사용하는 부분은 다음과 같습니다.<br/><br/>

**[app_view_with_normal_bloc.dart]**<br/>
```dart
import 'package:bloc_example/bloc/normal_todo_bloc.dart';
import 'package:bloc_example/components/listView_add_widget.dart';
import 'package:bloc_example/models/todo_model.dart';
import 'package:bloc_example/views/add_view.dart';
import 'package:flutter/material.dart';

class AppViewWithNormalBloc extends StatefulWidget {
  const AppViewWithNormalBloc({super.key});

  static final NormalTodoBloc todoBloc = NormalTodoBloc();

  @override
  State<AppViewWithNormalBloc> createState() => _AppViewWithNormalBlocState();
}

class _AppViewWithNormalBlocState extends State<AppViewWithNormalBloc> {
  final NormalTodoBloc _todoBloc = AppViewWithNormalBloc.todoBloc;

  @override
  void initState() {
    super.initState();

    _todoBloc.listTodo();
  }

  @override
  void dispose() {
    super.dispose();
    _todoBloc.dispose();
  }

  Widget _bodyWidget() {
    return Container(
      child: ListView_AddWidget(),
    );
  }

  @override
  Widget build(BuildContext context) {
    print('AppViewWithNormalBloc - build');

    return Scaffold(
      appBar: AppBar(title: Text('BLoC Example')),
      body: _bodyWidget(),
    );
  }
}
```
**[결과 화면]**<br/>
![bloc1](https://user-images.githubusercontent.com/13028129/211712227-d32d4dde-75e4-449f-8414-b8edb988ba6c.gif)<br/>
실행해서 확인해 보면 데이터 추가시 서버에서 요청전에 미리 데이터 표시하고 서버 응답이 성공되었을때 id및 생성날짜를 갱신하는걸 확인할 수 있고,<br/>
데이터 삭제 또한 서버 요청전 미리 삭제 했지만 응답이 실패(false)로 반환되어 정상 삭제 되지 않고 다시 원복 되는걸 확인할 수 있습니다.<br/><br/>

이렇게 BLoC 패턴이 무엇인지, 왜 사용해야 하는지, BLoC 구조 개념과 간단한 예제를 통해 구현 방법까지 알아 보았습니다.<br/>
다음은 flutter_bloc 패키지를 사용해서 BLoC 패턴을 적용하는 방법에 대해 알아보겠습니다.<br/><br/><br/>



위 코드는 다음 Repository에서 확인할 수 있습니다.<br/>
[Code_check - flutter_bloc_example](https://github.com/tyeom/flutter_bloc_example)



{% include content_adsense.html %}
