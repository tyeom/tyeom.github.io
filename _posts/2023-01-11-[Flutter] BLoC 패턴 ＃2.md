---
title: (Flutter) BLoC 패턴 ＃2
categories: Flutter
key: 20230111_01
comments: true
tags: Flutter bloc flutter_bloc 상태관리 로직분리 pattern bloc_parrern
---

지난글에서 BLoC 패턴에 대해 알아보았고 직접 구현하는 방법에 대해 알아보았습니다.<br/>
[BLoC 패턴 ＃1](https://blog.arong.info/flutter/2023/01/10/Flutter-BLoC-%ED%8C%A8%ED%84%B4-1.html)<br/>
이번글에서는 좀더 많은 기능을 제공해 주고 일일히 BLoC동작을 구현할 필요 없이 자체적으로 제공해 주는 BLoC 패키지인 [flutter_bloc](https://pub.dev/packages/flutter_bloc) 패키지 사용에 대해 
알아보겠습니다.

<!--more-->


> 이 글에서 다루는 코드는 다음 Repository에서 확인할 수 있습니다.<br/>
> [Code_check - flutter_bloc_example](https://github.com/tyeom/flutter_bloc_example)

<br/><br/>

우선 flutter_bloc 패키지를 사용하면 다음과 같은 이점이 있습니다.<br/>
일일히 BLoC 동작을 직접 구현할 필요 없이 보다 쉬운 방식으로 제공되는걸 사용할 수 있습니다.<br/>
그리고 이벤트 상태 변경 추적이라던지 **BlocObserver** 를 통해 통합적으로 모든 상태에 대한 변경 트래킹 기능등 좀 더 많은 기능들이 지원됩니다.


flutter_bloc 패키지
-

현재 시점 flutter_bloc은 8.1.1 버전까지 꾸준하게 업데이트 되고 있고, 4950개 이상 좋아요 수치 처럼 전세계 Flutter개발자에게 많은 관심을 받으면서 사용되고 있습니다.<br/>
GetX는 상태관리 로직분리 외에 통합 미니멀 프레임워크의 성격이기 때문에 BLoC패턴 설계가 필요한 프로젝트에서는 flutter_bloc으로 단독 사용하는 추세 인것 같습니다.<br/><br/>

Event
-

flutter_bloc은 Event와 State로 구성되는데 Event는 UI에서 요청될 단위 입니다.<br/>
지난글에서 사용된 예제 내용 기준으로 보면<br/>
* 데이터 로드 이벤트 - GetTodoListEvent
* 데이터 추가 이벤트 - CreateTodoEvent
* 데이터 삭제 이벤트 - DeleteTodoEvent

이렇게 구현될 수 있습니다.<br/><br/>

실제 Event를 코드로 구현해 보면 다음과 같습니다.<br/>
**[todo_event.dart]**<br/>
```dart
// event base class
abstract class TodoEvent {}

// 데이터 로드 이벤트
class GetTodoListEvent extends TodoEvent {}

// 데이터 추가 이벤트
class CreateTodoEvent extends TodoEvent {
  final String title;

  CreateTodoEvent({
    required this.title,
  });
}

// 데이터 삭제 이벤트
class DeleteTodoEvent extends TodoEvent {
  final String uuid;

  DeleteTodoEvent({
    required this.uuid,
  });
}
```

데이터 추가 부분은 사용자에게 입력받은 title값을 이용하기 때문에 title을 매개변수로 받고, 데이터 삭제 또한 uuid기준으로 삭제되기 때문에 uuid를 매개변수로 받습니다.<br/>
이렇게 되면 Event부분은 구현이 다 되었습니다.


State
-

State는 UI에 표시될 상태들 입니다. State또한 지난글 사용예제로 보면 데이터를 요청중인 상태(로드중), 데이터 요청시 오류 발생 상태, 정상적으로 데이터가 로드된 상태, 데이터가 없는 상태 
이렇게 상태별로 나뉘어 볼 수 있을 것 입니다.<br/>
* 데이터가 없는 상태 - EmptyDataState
* 데이터 로드중 상태 - LoadingState
* 오류 상태 - ErrorState
* 데이터 로드 완료 상태 - LoadedState

그래서 State도 다음과 같이 구현할 수 있습니다.<br/><br/>
**[todo_state.dart]**<br/>
```dart
// state base class
import 'package:bloc_example/models/todo_model.dart';

abstract class TodoState {}

// 데이터가 없는 상태
class EmptyDataState extends TodoState {}

// 데이터 로드 요청중 상태
class LoadingState extends TodoState {}

// 오류발생 상태
class ErrorState extends TodoState {
  final String message;

  ErrorState({
    required this.message,
  });
}

// 데이터 로드 완료 상태
class LoadedState extends TodoState {
  // 로드 결과 데이터 리스트
  final List<TodoModel> todoList;

  LoadedState({
    required this.todoList,
  });
}
```

Bloc
-

이렇게 Event와 State가 정의되었다면 이제 Bloc를 구현해서 사용할 수 있습니다.<br/>
Bloc구현은 **<span style="color: rgb(107, 173, 222);">Bloc&lt;Event, State&gt;</span>** 추상 클래스를 확장해서 구현할 수 있고 각각 Event타입과 State타입을 제네릭으로 받습니다.<br/>
이렇게 정의한 Bloc 클래스에서는 각 이벤트에 맞는 비즈니스 로직을 처리하고 **<span style="color: rgb(107, 173, 222);">BlocBase&lt;State&gt;</span>** 의 emit() 함수로 구독자 UI에 상태변경을 전달합니다.<br/>
이렇게 상태 정보를 받은 UI는 해당 상태에 맞게 화면처리를 할 수 있게 됩니다.<br/>
그리고 이벤트에 맞는 비즈니스 로직 매핑은  **<span style="color: rgb(107, 173, 222);">Bloc&lt;Event, State&gt;</span>** 클래스의 on&lt;E&gt;() 함수로 매핑 정의를 해야 합니다.<br/><br/>

Bloc 구현 코드는 다음과 같습니다.<br/><br/>

**[todo_bloc.dart]**<br/>
```dart
import 'package:bloc_example/bloc/todo_event.dart';
import 'package:bloc_example/bloc/todo_state.dart';
import 'package:bloc_example/models/todo_model.dart';
import 'package:bloc_example/repository/todo_repository.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:uuid/uuid.dart';

/// [flutter_bloc 패키지 사용해서 BLoC 패턴 적용]
/// 비즈니스 로직 처리
class TodoBloc extends Bloc<TodoEvent, TodoState> {
  final TodoRepository repository;

  // Repository 생성자 의존성 주입 처리
  TodoBloc({
    required this.repository,
  }) : super(EmptyDataState()) {
    on<GetTodoListEvent>((_, emit) => _listTodoEvent());
    on<CreateTodoEvent>((event, _) => _createTodoEvent(event.title));
    on<DeleteTodoEvent>((event, _) => _deleteTodoEvent(event.uuid));
  }

  Future<void> _listTodoEvent() async {
    try {
      // 로딩 상태 반환 [스트림 처리]
      emit(LoadingState());

      final todoJson = await repository.getListTodo();

      final todoList = todoJson
          .map<TodoModel>(
            (p) => TodoModel.fromJson(p),
          )
          .toList();

      emit(LoadedState(todoList: todoList));
    } catch (ex) {
      emit(ErrorState(message: ex.toString()));
    }
  }

  Future<void> _createTodoEvent(String title) async {
    try {
      // 현재 상태가 로드 완료 상태 일때만 데이터 추가
      if (state is LoadedState == false) return;

      // 현재 상태
      final loadedState = (state as LoadedState);

      /// fake 데이터 [실제 서버에 요청전 결과 데이터를 빠르게 보여주기 위해]
      /// 임시로 새로 만들어진 TodoModel데이터를 바로 추가해서 스트림 전송
      final fakeTodoList = [
        ...loadedState.todoList,
        TodoModel(title: title),
      ];

      emit(LoadedState(todoList: fakeTodoList));

      /// 실제 서버 요청 처리
      /// id, createDT 정보는 서버에서 처리됨을 간주
      const uuid = Uuid();
      final idv4 = uuid.v4();
      TodoModel newTodo = TodoModel(
          uuid: idv4, title: title, createDT: DateTime.now().toString());
      final newTodoJson = await repository.createTodo(newTodo);
      emit(LoadedState(
        todoList: [
          ...loadedState.todoList,
          newTodo,
        ],
      ));
    } catch (ex) {
      emit(ErrorState(message: ex.toString()));
    }
  }

  Future<void> _deleteTodoEvent(String uuid) async {
    try {
      // 현재 상태가 로드 완료 상태 일때만 데이터 삭제
      if (state is LoadedState == false) return;

      // 현재 상태
      final loadedState = (state as LoadedState);

      /// fake 데이터 [실제 서버에 요청전 결과 데이터를 빠르게 보여주기 위해]
      /// 로컬에서 먼저 삭제처리 후 스트림 전송
      final currentTodoList = [
        ...loadedState.todoList,
      ];
      final fakeTodoList =
          currentTodoList.where((item) => item.uuid != uuid).toList();
      emit(LoadedState(todoList: fakeTodoList));

      // 실제 서버에 삭제 요청 처리
      final result = await repository.deleteTodo(uuid);
      // 삭제 요청 실패시
      if (result == false) {
        // 원래 데이터로 다시 표시 (원복)
        emit(loadedState);
      } else {
        // 로컬 데이터 실제 삭제 처리
        loadedState.todoList.removeWhere((item) => item.uuid == uuid);
      }
    } catch (ex) {
      emit(ErrorState(message: ex.toString()));
    }
  }
}
```

이렇게 구현된 Bloc은 **<span style="color: rgb(107, 173, 222);">BlocProvider&lt;T&gt;</span>** 를 통해 관리되면서 생성되어야 사용할 수 있습니다.<br/>
**<span style="color: rgb(107, 173, 222);">BlocProvider&lt;T&gt;</span>** child로 생성되는 위젯의 하위 위젯은 **<span style="color: rgb(107, 173, 222);">BlocProvider&lt;T&gt;</span>** 또는 **<span style="color: rgb(107, 173, 222);">StatefulWidget</span>** dml 
context 속성으로 특정 Bloc을 접근해서 사용 할 수 있습니다.<br/><br/>

**[main.dart]**<br/>
```dart
void main() {
  // flutter_bloc 패키지 사용
  unApp(BlocProvider(
   // TodoRepository DI 처리
   create: (_) => TodoBloc(repository: TodoRepository()),
   child: const MyApp(),
  ));
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'BLoC Example',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      
      home: const AppView(),
    );
  }
}
```

이렇게 처리하면 MyApp위젯 하위로 생성되는 위젯은 다음과 같이 Bloc에 접근할 수 있습니다.<br/><br/>

```dart
context.read<TodoBloc>().add(someEvent());
```
또는<br/>

```dart
BlocProvider.of<TodoBloc>(context).add(someEvent());
```

그리고 Bloc의 State를 구독하기 위해서 **<span style="color: rgb(107, 173, 222);">BlocBuilder&lt;B&gt;</span>** 위젯을 사용할 수 있습니다.<br/><br/>

```dart
BlocBuilder<TodoCubit, TodoState>(
    builder: (_, state) {
        if (state is EmptyDataState) {
            // 데이터가 없는 상태에 대한 UI 처리
        }
        else if (state is LoadingState) {
            // 데이터 로딩중 상태 UI 처리
        }
        else if (state is ErrorState) {
            // 오류 발생 상태에 대한 UI 처리
        }
        else if (state is LoadedState) {
            // 데이터 로드 완료 상태에 대한 UI 처리
        }
    }
)
```

Cubit
-

**<span style="color: rgb(107, 173, 222);">Cubit&lt;State&gt;</span>** 은 **<span style="color: rgb(107, 173, 222);">BlocBase&lt;State&gt;</span>** 클래스의 베이스 입니다.<br/>
**<span style="color: rgb(107, 173, 222);">BlocBase&lt;State&gt;</span>** 클래스와 차이점은 **<span style="color: rgb(107, 173, 222);">Bloc&lt;Event, State&gt;</span>** 클래스는 
이벤트를 정의하고 해당 이벤트를 통해서 처리할 수 있지만 **<span style="color: rgb(107, 173, 222);">Cubit&lt;State&gt;</span>** 클래스는 좀 더 간편하게 이벤트 정의 없이 직접 
상태 변경에 해당되는 함수를 정의해서 호출하는 방식 입니다.<br/><br/>

**[todo_cubit.dart]**<br/>
```dart
import 'package:bloc_example/bloc/todo_state.dart';
import 'package:bloc_example/models/todo_model.dart';
import 'package:bloc_example/repository/todo_repository.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:uuid/uuid.dart';

/// [flutter_bloc 패키지 사용해서 BLoC 패턴 적용 - Cubit 사용]
/// 비즈니스 로직 처리
class TodoCubit extends Cubit<TodoState> {
  final TodoRepository repository;

  // Repository 생성자 의존성 주입 처리
  TodoCubit({
    required this.repository,
  }) : super(EmptyDataState());

  Future<void> listTodoEvent() async {
    try {
      // 로딩 상태 반환 [스트림 처리]
      emit(LoadingState());

      final todoJson = await repository.getListTodo();

      final todoList = todoJson
          .map<TodoModel>(
            (p) => TodoModel.fromJson(p),
          )
          .toList();

      emit(LoadedState(todoList: todoList));
    } catch (ex) {
      emit(ErrorState(message: ex.toString()));
    }
  }

  Future<void> createTodoEvent(String title) async {
    try {
      // 현재 상태가 로드 완료 상태 일때만 데이터 추가
      if (state is LoadedState == false) return;

      // 현재 상태
      final loadedState = (state as LoadedState);

      /// fake 데이터 [실제 서버에 요청전 결과 데이터를 빠르게 보여주기 위해]
      /// 임시로 새로 만들어진 TodoModel데이터를 바로 추가해서 스트림 전송
      final fakeTodoList = [
        ...loadedState.todoList,
        TodoModel(title: title),
      ];

      emit(LoadedState(todoList: fakeTodoList));

      /// 실제 서버 요청 처리
      /// id, createDT 정보는 서버에서 처리됨을 간주
      const uuid = Uuid();
      final idv4 = uuid.v4();
      TodoModel newTodo = TodoModel(
          uuid: idv4, title: title, createDT: DateTime.now().toString());
      final newTodoJson = await repository.createTodo(newTodo);
      emit(LoadedState(
        todoList: [
          ...loadedState.todoList,
          newTodo,
        ],
      ));
    } catch (ex) {
      emit(ErrorState(message: ex.toString()));
    }
  }

  Future<void> deleteTodoEvent(String uuid) async {
    try {
      // 현재 상태가 로드 완료 상태 일때만 데이터 삭제
      if (state is LoadedState == false) return;

      // 현재 상태
      final loadedState = (state as LoadedState);

      /// fake 데이터 [실제 서버에 요청전 결과 데이터를 빠르게 보여주기 위해]
      /// 로컬에서 먼저 삭제처리 후 스트림 전송
      final currentTodoList = [
        ...loadedState.todoList,
      ];
      final fakeTodoList =
          currentTodoList.where((item) => item.uuid != uuid).toList();
      emit(LoadedState(todoList: fakeTodoList));

      // 실제 서버에 삭제 요청 처리
      final result = await repository.deleteTodo(uuid);
      // 삭제 요청 실패시
      if (result == false) {
        // 원래 데이터로 다시 표시 (원복)
        emit(loadedState);
      } else {
        // 로컬 데이터 실제 삭제 처리
        loadedState.todoList.removeWhere((item) => item.uuid == uuid);
      }
    } catch (ex) {
      emit(ErrorState(message: ex.toString()));
    }
  }
}
```

**<span style="color: rgb(107, 173, 222);">Bloc&lt;Event, State&gt;</span>** 과 마찬가지로 **<span style="color: rgb(107, 173, 222);">Cubit&lt;State&gt;</span>** 도 UI에서 다음과 같이 접근할 수 있습니다.<br/><br/>

```dart
context.read<TodoCubit>().someEvent();
```
또는<br/>

```dart
BlocProvider.of<TodoCubit>(context).someEvent();
```

BlocObserver
-

flutter_bloc 패키지는 통합 이벤트 관찰 기능을 제공합니다. 이 기능을 이용해서 특정 상태변경에 대한 후 처리나 기타 작업을 처리할 수 있습니다.<br/>
통합 이벤트 관찰 처리는 **<span style="color: rgb(107, 173, 222);">Bloc&lt;Event, State&gt;</span>** 클래스의 observer 정적 속성에 **<span style="color: rgb(107, 173, 222);">BlocObserver</span>** 클래스를 
등록함으로써 통합적으로 이벤트 트래킹 처리를 할 수 있습니다.<br/><br/>

**[events_observer.dart]**<br/>
```dart
import 'package:bloc_example/bloc/todo_cubit.dart';
import 'package:flutter_bloc/flutter_bloc.dart';

class EventsObserver extends BlocObserver {
  @override
  void onCreate(BlocBase bloc) {
    super.onCreate(bloc);

    print('onCreate -- cubit: ${bloc.runtimeType}');
  }

  @override
  void onChange(BlocBase bloc, Change change) {
    super.onChange(bloc, change);

    print('onChange -- cubit: ${bloc.runtimeType}, change: $change');
  }
}
```

위 함수 외 onError(), onClose() 함수들이 있습니다.<br/>
이렇게 정의된 **<span style="color: rgb(107, 173, 222);">BlocObserver</span>** 클래스는 **<span style="color: rgb(107, 173, 222);">Bloc&lt;Event, State&gt;</span>** 클래스의 observer 정적 속성에 할당해 주어야 합니다.<br/><br/>

**[main.dart]**
```dart
void main() {
  ...[생략]...

  Bloc.observer = EventsObserver();
}
```

이렇게 flutter_bloc 패키지에 사용에 대해 알아보았습니다.<br/><br/><br/>



위 코드는 다음 Repository에서 확인할 수 있습니다.<br/>
[Code_check - flutter_bloc_example](https://github.com/tyeom/flutter_bloc_example)



{% include content_adsense.html %}
