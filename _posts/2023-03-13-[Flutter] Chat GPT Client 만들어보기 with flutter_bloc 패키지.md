---
title: (Flutter) Chat GPT Client 만들어보기 with flutter_bloc 패키지
categories: Flutter
key: 20230313_01
comments: true
tags: Flutter ChatGPT flutter_bloc
---

인공지능 ChatGPT 모델을 사용하는 채팅 서비스를 Open API를 사용하여 간단한 클라이언트 앱을 만들어 보는 과정을 설명해 보고자 합니다.

<!--more-->

> 이 글에서 다루는 코드는 다음 Repository에서 확인할 수 있습니다.<br/>
> [ChatGPT_Flutter](https://github.com/tyeom/ChatGPT_Flutter)

사용되는 패키지는 모두 다음과 같습니다.
- dio (http 요청)
- popup_card (팝업창 표시)
- flutter_bloc (BLoC를 이용한 상태관리)
- number_inc_dec (숫자 입력 Numberic Text Feild Widget)
- shared_preferences (환경설정 데이터 내부 저장)
- flutter_markdown (Markdown 데이터 렌더링)

이제 보니 간단한 앱인데도 사용되는 패키지들이 많이 있습니다.

UI 구성
-

앱의 전체 UI구성은 이미 Avalonia 라는 멀티 플랫폼 기술로 만들어진 ChatGPT Client 앱을 그대로 똑같이 따라해 보려고 합니다.<br/>
GitHub : [ChatGPT_Avalonia](https://github.com/wieslawsoltes/ChatGPT)<br/><br/>

해당 앱의 UI를 보면 직관적이면서 단순 합니다.<br/>
전체적으로 대화 목록과 답변 대화 안에 메세지를 입력 할 수 있는 메세지 입력창이 있고, 하단에는 환경설정과 라이트/다크 모드 변경 및 기타 기능에 해당되는 버튼들이 나열 되어 있습니다.<br/>
우선 간단하게 이 화면의 기본적 레이아웃을 구성해 봅니다.<br/><br/>

**[src/views/app_view.dart]**<br/>
```dart
import 'package:flutter/material.dart';

class AppView extends StatefulWidget {
  const AppView({super.key});

  @override
  State<AppView> createState() => _AppViewState();
}

class _AppViewState extends State<AppView> {
  ScrollController _listScrollController = ScrollController();

  Widget _bodyWidget() {
    return Container(
      margin: const EdgeInsets.all(15),
      child: ListView.separated(
        controller: _listScrollController,
        // 임시
        itemCount: 7,
        itemBuilder: (_, index) {
          return Container();
        },
        separatorBuilder: (_, index) => const SizedBox(
          height: 10,
        ),
      ),
    );

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      // 채팅 대화 목록 및 대화 입력 위젯 영역
      body: _bodyWidget(),
      
      // 하단 환경설정 버튼 및 기타 기능 버튼 들
      bottomNavigationBar: Row(
        mainAxisAlignment: MainAxisAlignment.start,
        crossAxisAlignment: CrossAxisAlignment.end,
        children: [
          // 라이트/다크 모드 변경
          IconButton(
              padding: const EdgeInsets.only(bottom: 33),
              onPressed: () {
                // 테마 토글 버튼
              },
              icon: const Icon(Icons.brightness_6)),
          const SizedBox(
            width: 50,
          ),
          // 하단 우측 이미지
          Padding(
              padding: const EdgeInsets.only(bottom: 20),
              child: Image.asset('assets/images/clipart855284.png',
                  height: 100, fit: BoxFit.fill)),
        ],
      ),
    );
  }
}
```

이렇게 하면 기본적인 레이아웃이 다음과 같이 잡혀 집니다.<br/>
![image](https://user-images.githubusercontent.com/13028129/224640614-0465e4fe-eada-45f1-83bd-b76ebadd506a.png)


라이트/다크 모드 변경 처리
-

제일 간단한것 부터 하나씩 구현해 보도록 합니다.<br/>
라이트/다크 모드 변경 처리는 모드 변경 토글 버튼에 반응하는 것으로 간단하게 상태관리로 처리 할 수 있습니다.<br/>
그 처리를 위해 테마 변경 관련 BLoC을 구현합니다.<br/><br/>

**[src/bloc/theme_cubit.dart]**<br/>
```dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';

/// 테마 변경 BLoC
class ThemeCubit extends Cubit<ThemeData> {
  /// {@macro brightness_cubit}
  ThemeCubit() : super(_lightTheme);

  static final _lightTheme = ThemeData(
    primarySwatch: Colors.blue,
    brightness: Brightness.light,
  );

  static final _darkTheme = ThemeData.dark(useMaterial3: false);

  /// Toggles the current brightness between light and dark.
  void toggleTheme() {
    emit(state.brightness == Brightness.dark ? _lightTheme : _darkTheme);
  }
}
```

Flutter SDK의 **<span style="color: rgb(107, 173, 222);">ThemeData</span>** 클래스 를 상태로 사용하고 lightTheme, dartTheme 이렇게 각각의 테마를 정의 합니다.<br/>
Dart Theme 역시 기본적으로 **<span style="color: rgb(107, 173, 222);">ThemeData</span>** 클래스에 포함 되어 있습니다.<br/>
그리고 현재 설정된 Theme의 구분은 **<span style="color: rgb(107, 173, 222);">AccessibilityFeatures</span>** 클래스의 **<span style="color: rgb(107, 173, 222);">Brightness</span>** enum 값으로
구분 할 수 있습니다.<br/><br/>

그럼 이 BLoC을 연결해서 테마를 적용해 보겠습니다. Flutter에서 테마는 기본 **<span style="color: rgb(107, 173, 222);">MaterialApp</span>** 클래스를 사용해서 App 위젯으로
사용한다면 **<span style="color: rgb(107, 173, 222);">MaterialApp</span>** 클래스의 theme 파라메터로 간단하게 설정 할 수 있습니다.<br/>
해당 부분에 **<span style="color: rgb(107, 173, 222);">BlocBuilder</span>** 를 사용해서 테마 상태가 변경 되도록 처리해 줍니다.<br/><br/>

**[src/main.dart]**<br/>
```dart
class MyApp extends StatelessWidget {
  const MyApp({super.key});

  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    return BlocBuilder<ThemeCubit, ThemeData>(
      builder: (_, theme) {
        return MaterialApp(
          debugShowCheckedModeBanner: false,
          title: 'ChatGPT',
          theme: theme,
          home: const AppView(),
        );
      },
    );
  }
}
```

그리고 AppView 위젯의 테마 변경 토글 버튼에서 테마 상태 변경 메서드를 호출 되도록 처리 합니다.<br/><br/>

**[src/views/app_view.dart]**<br/>
```dart
// 라이트/다크 모드 변경
IconButton(
  padding: const EdgeInsets.only(bottom: 33),
  onPressed: () {
    // 테마 토글 버튼
  },
  icon: const Icon(Icons.brightness_6)),
```

해당 부분에<br/>
```dart
context.read<ThemeCubit>().toggleTheme();
```

이렇게 테마 변경 메서드가 호출되도록 해주면 라이트/다크 모드 테마 변경 처리가 됩니다.


UI 구성 - 환경설정
-

제일 먼저 환경설정 화면을 구현 해야 하는데 환경설정 화면은 어떻게 하면 좋을까 pub.dev 에서 팝업 관련 패키지를 찾아 보다가 [popup_card](https://pub.dev/packages/popup_card) 패키지를 사용해서
환경설정 팝업 화면을 구현해 보자 생각 했습니다.<br/>
일단 환경설정 팝업 화면은 popup_card 패키지를 사용해서 구현하도록 하고, 입력된 환경설정 데이터는 json 데이터 포맷으로 시리얼라이즈 해서 그대로 로컬 SharedPreferences 에 저장하고 사용 할 수 있도록 했습니다.<br/>
우선 환경설정 데이터에 대한 모델을 다음과 같이 생성 합니다.<br/><br/>

**[src/models/settings_model.dart]**<br/>
```dart
class SettingsModel {
  final double temperature;
  final int maxTokens;
  final String? apiKey;
  final String model;
  final String directions;

  SettingsModel({
    this.temperature = 0.7,
    this.maxTokens = 256,
    this.apiKey,
    this.model = 'gpt-3.5-turbo',
    this.directions =
        'You are a helpful assistant named Clippy. Write answers in Markdown blocks. For code blocks always define used language.',
  });

  factory SettingsModel.fromJson(Map<String, dynamic> json) => SettingsModel(
      temperature: json['temperature'] as double,
      maxTokens: json['maxTokens'] as int,
      apiKey: json['apiKey'] as String,
      model: json['model'] as String,
      directions: json['directions'] as String);

  Map<String, dynamic> toJson() => {
        'temperature': temperature,
        'maxTokens': maxTokens,
        'apiKey': apiKey,
        'model': model,
        'directions': directions,
      };
}
```

각 필드에 대한 정보는 저도 정확히는 모르겠습니다. [ChatGPT_Avalonia](https://github.com/wieslawsoltes/ChatGPT) 에서 사용 되고 있는 설정 그대로 작성 하였습니다.<br/>
다만 apiKey가 사용자에게 받는 필수 정보임을 짐작해 볼 수 있습니다.<br/>
이제 이렇게 만든 모델을 사용해서 SharedPreferences에 저장 하고 사용 할 수 있도록 구현해 봅니다.<br/>
데이터 처리 개념으로 생각해서 Repository로 별도 분리해서 처리해 볼까 생각해 보았습니다.<br/><br/>

**[src/repository/settings_repository.dart]**<br/>
```dart
import 'dart:convert';

import 'package:chat_gpt/src/models/settings_model.dart';
import 'package:shared_preferences/shared_preferences.dart';

class SettingsRepository {
  Future<void> saveSettings(double temperature, int maxTokens, String? apiKey,
      String model, String directions) async {
    SettingsModel settingsModel = SettingsModel(
        temperature: temperature,
        maxTokens: maxTokens,
        apiKey: apiKey,
        model: model,
        directions: directions);
    var settingsJson = settingsModel.toJson();

    final SharedPreferences prefs = await SharedPreferences.getInstance();
    prefs.setString('settings', json.encode(settingsJson));
  }

  Future<SettingsModel> getSettings() async {
    final SharedPreferences prefs = await SharedPreferences.getInstance();
    var settingsJson = prefs.getString('settings');
    if (settingsJson == null) {
      return SettingsModel();
    } else {
      return SettingsModel.fromJson(jsonDecode(settingsJson));
    }
  }
}
```

이렇게 saveSettings() 메서드와 getSettings() 메서드를 노출해서 외부에서 사용 할 수 있도록 구현 하였습니다.<br/>
환경설정 값이 변경 되었을때 실시간으로 상태 변화를 관리 하기 위해 BLoC 으로 상태 처리를 구현해 봅니다.<br/><br/>

**[src/bloc/settings_cubit.dart]**<br/>
```dart
import 'package:chat_gpt/src/bloc/settings_state.dart';
import 'package:chat_gpt/src/models/settings_model.dart';
import 'package:chat_gpt/src/repository/settings_repository.dart';
import 'package:flutter_bloc/flutter_bloc.dart';

class SettingsCubit extends Cubit<SettingsState> {
  final SettingsRepository settingsRepository;

  // Repository 생성자 의존성 주입 처리
  SettingsCubit({
    required this.settingsRepository,
  }) : super(LoadedSettingsState(settingsModel: SettingsModel()));

  Future<void> updateSettings(
      double temperature, int maxTokens, String? apiKey, String model, String directions) async {
    try {
      await settingsRepository.saveSettings(temperature, maxTokens, apiKey, model, directions);
      final settingsModel = await settingsRepository.getSettings();

      emit(LoadedSettingsState(settingsModel: settingsModel));
    } catch (ex) {
      print(ex.toString());
    }
  }

  Future<void> getSettings() async {
    try {
      final settingsModel = await settingsRepository.getSettings();
      emit(LoadedSettingsState(settingsModel: settingsModel));
    } catch (ex) {
      print(ex.toString());
    }
  }
}
```

**[src/bloc/settings_state.dart]**<br/>
```dart
import 'package:chat_gpt/src/models/settings_model.dart';

abstract class SettingsState {}

class LoadedSettingsState extends SettingsState {
  final SettingsModel settingsModel;

  LoadedSettingsState({
    required this.settingsModel,
  });
}
```

이렇게 처리 함으로써 환경설정 값이 변경 될때 SettingsState 상태 반환으로 변경된 환경설정 값을 감지하여 사용 할 수 있습니다.<br/>
그러면 이제 이렇게 만들어진 bloc(Cubit)을 사용해서 환경설정 UI를 구현해 봅니다.<br/>
숫자를 입력하는 항목은 [number_inc_dec](https://pub.dev/packages/number_inc_dec) 패키지를 사용해서 구현해 보았습니다.<br/>
[popup_card](https://pub.dev/packages/popup_card) 패키지를 설치 하고 환경설정 화면의 위젯을 다음과 같이 구현 했습니다.<br/><br/>

**[src/components/popup_item_settings.dart]**<br/>
```dart
import 'package:chat_gpt/src/bloc/settings_cubit.dart';
import 'package:chat_gpt/src/bloc/settings_state.dart';
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:number_inc_dec/number_inc_dec.dart';

class PopupItemSettings extends StatelessWidget {
  final TextEditingController _temperatureText = TextEditingController();
  final TextEditingController _maxtokensText = TextEditingController();
  final TextEditingController _directionsText = TextEditingController();
  final TextEditingController _modelText = TextEditingController();
  final TextEditingController _apiKeyText = TextEditingController();

  PopupItemSettings({super.key});

  void _updateSettings(BuildContext context) {
    context.read<SettingsCubit>().updateSettings(
        double.parse(_temperatureText.text),
        int.parse(_maxtokensText.text),
        _apiKeyText.text,
        _modelText.text,
        _directionsText.text);
  }

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: const EdgeInsets.all(10),
      child: SingleChildScrollView(
        child: BlocBuilder<SettingsCubit, SettingsState>(builder: (_, state) {
          LoadedSettingsState settingsState = state as LoadedSettingsState;
          _apiKeyText.text = (settingsState.settingsModel.apiKey == null)
              ? ""
              : settingsState.settingsModel.apiKey!;
          _modelText.text = settingsState.settingsModel.model;
          _directionsText.text = settingsState.settingsModel.directions;

          return Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              const Text('Temperature:'),
              const SizedBox(
                height: 10,
              ),
              NumberInputWithIncrementDecrement(
                controller: _temperatureText,
                isInt: false,
                incDecFactor: 0.1,
                initialValue: settingsState.settingsModel.temperature,
                min: 0,
                onDecrement: (value) => _updateSettings(context),
                onIncrement: (value) => _updateSettings(context),
                onChanged: (value) => _updateSettings(context),
              ),
              const SizedBox(
                height: 10,
              ),
              const Text('Max tokens:'),
              const SizedBox(
                height: 10,
              ),
              NumberInputWithIncrementDecrement(
                controller: _maxtokensText,
                isInt: true,
                incDecFactor: 1,
                initialValue: settingsState.settingsModel.maxTokens,
                min: 0,
                onDecrement: (value) => _updateSettings(context),
                onIncrement: (value) => _updateSettings(context),
                onChanged: (value) => _updateSettings(context),
              ),
              const SizedBox(
                height: 10,
              ),
              const Text('Directions:'),
              const SizedBox(
                height: 10,
              ),
              TextField(
                controller: _directionsText,
                keyboardType: TextInputType.text,
                maxLines: 3,
                onChanged: (value) => _updateSettings(context),
              ),
              const SizedBox(
                height: 10,
              ),
              const Text('Model:'),
              const SizedBox(
                height: 10,
              ),
              TextField(
                controller: _modelText,
                keyboardType: TextInputType.text,
                onChanged: (value) => _updateSettings(context),
              ),
              const SizedBox(
                height: 10,
              ),
              const Text('Api Key:'),
              const SizedBox(
                height: 10,
              ),
              TextField(
                controller: _apiKeyText,
                keyboardType: TextInputType.text,
                onChanged: (value) => _updateSettings(context),
              ),
              const SizedBox(
                height: 10,
              ),
            ],
          );
        }),
      ),
    );
  }
}
```

onChanged의 ValueCallBack 에서 값 변경시 즉시 BLoC을 통해 환경설정 값이 업데이트 되도록 처리 되었고 동시에 해당 BLoC를 사용하는 위젯에서는 바로 반영 되어 사용 할 수 있습니다.<br/><br/>

**[결과 화면]**<br/>
![Flutter_ChatGPT2](https://user-images.githubusercontent.com/13028129/224866866-756c09e0-02c8-476e-bbb0-2320951db0ed.gif)<br/><br/>

이렇게 환경설정 팝업 화면 및 환경설정 데이터 저장/로드 처리 까지 완성 입니다.


ChatGPT API 요청
-

다음으로 실제 ChatGPT API를 사용해서 채팅 요청 처리를 구현해 보겠습니다.<br/>
이 부분 역시 ChatGPT_Avalonia 코드를 참고해서 사용되는 모델을 똑같이 사용 했습니다.<br/>
코드를 보고 필요한 모델을 모두 생성합니다.<br/>
역시 하나하나 속성은 어떤 역할인지 정확히 파악 하진 않았습니다. API 요청에 필요한 데이터가 이렇게 구성 되어 있고,
메세지 데이터 규칙은 'role'과 'content'의 포맷 형식이며, 지난 대화 이력을 배열 데이터로 포함해서 요청해야 하고, 요청에 대한 응답 데이터중 메세지는 여러 메세지로 반환 되는 구나 정도로 간략히 파악 했습니다.<br/>
그래서 API 요청 및 응답에 관련된 모델들은 다음과 같이 구현 됩니다.<br/><br/>

**[src/models/chat_choice_model.dart]**<br/>
```dart
import 'package:chat_gpt/src/models/chat_message_model.dart';

class ChatChoiceModel {
  ChatMessageModel message;
  int index;
  Object? logprobs;
  String? finishReason;

  ChatChoiceModel(
    this.message,
    this.index,
    this.logprobs,
    this.finishReason,
  );

  factory ChatChoiceModel.fromJson(Map<String, dynamic> json) =>
      ChatChoiceModel(
        ChatMessageModel.fromJson(json['message']),
        json['index'] as int,
        json['logprobs'] as Object?,
        json['finish_reason'] as String?,
      );

  Map<String, dynamic> toJson() => {
        'message': message,
        'index': index,
        'logprobs': logprobs,
        'finish_reason': finishReason,
      };
}
```

**[src/models/chat_message_model.dart]**<br/>
```dart
class ChatMessageModel {
  final String role;
  String content;

  ChatMessageModel(
    this.role,
    this.content,
  );

  factory ChatMessageModel.fromJson(Map<String, dynamic> json) =>
      ChatMessageModel(
        json['role'] as String,
        json['content'] as String,
      );

  Map<String, dynamic> toJson() => {
        'role': role,
        'content': content,
      };
}
```

**[src/models/chat_request_body_model.dart]**<br/>
```dart
import 'package:chat_gpt/src/models/chat_message_model.dart';

class ChatRequestBodyModel {
  String? model;
  List<ChatMessageModel>? messages;
  double temperature;
  double topP;
  int n;
  bool stream;
  String? stop;
  int maxTokens;
  double presencePenalty;
  double frequencyPenalty;
  Map<String, double>? logitBias;
  String? user;

  ChatRequestBodyModel(
    this.model,
    this.messages,
    this.temperature,
    this.topP,
    this.n,
    this.stream,
    this.stop,
    this.maxTokens,
    this.presencePenalty,
    this.frequencyPenalty,
    this.logitBias,
    this.user,
  );

  factory ChatRequestBodyModel.fromJson(Map<String, dynamic> json) =>
      ChatRequestBodyModel(
        json['model'] as String,
        (json['messages'] as List).map((p) => ChatMessageModel.fromJson(p)).toList(),
        json['temperature'] as double,
        json['top_p'] as double,
        json['n'] as int,
        json['stream'] as bool,
        json['stop'] as String,
        json['max_tokens'] as int,
        json['presence_penalty'] as double,
        json['frequency_penalty'] as double,
        json['logitBias'] as Map<String, double>,
        json['user'] as String,
      );

  Map<String, dynamic> toJson() => {
        'model': model,
        'messages': messages,
        'temperature': temperature,
        'top_p': topP,
        'n': n,
        'stream': stream,
        'stop': stop,
        'max_tokens': maxTokens,
        'presence_penalty': presencePenalty,
        'frequency_penalty': frequencyPenalty,
        'logitBias': logitBias,
        'user': user,
      };
}
```

**[src/models/chat_response_model.dart]**<br/>
```dart
import 'package:chat_gpt/src/models/chat_choice_model.dart';
import 'package:chat_gpt/src/models/chat_usage_model.dart';

abstract class ChatResponseModel {}

class ChatResponseErrorModel extends ChatResponseModel {
  String? message;
  String? type;
  Object? param;
  String? code;

  ChatResponseErrorModel(
    this.message,
    this.type,
    this.param,
    this.code,
  );

  factory ChatResponseErrorModel.fromJson(Map<String, dynamic> json) =>
      ChatResponseErrorModel(
        json['message'] as String?,
        json['type'] as String?,
        json['param'] as Object?,
        json['code'] as String?,
      );

  Map<String, dynamic> toJson() => {
        'message': message,
        'type': type,
        'param': param,
        'code': code,
      };
}

class ChatResponseSuccessModel extends ChatResponseModel {
  String? id;
  Object? object;
  int created;
  String? model;
  List<ChatChoiceModel> choices;
  ChatUsageModel? usage;

  ChatResponseSuccessModel(
    this.id,
    this.object,
    this.created,
    this.model,
    this.choices,
    this.usage,
  );

  factory ChatResponseSuccessModel.fromJson(Map<String, dynamic> json) =>
      ChatResponseSuccessModel(
        json['id'] as String?,
        json['object'] as Object?,
        json['created'] as int,
        json['model'] as String?,
        (json['choices'] as List)
            .map((p) => ChatChoiceModel.fromJson(p))
            .toList(),
        ChatUsageModel.fromJson(json['usage']),
      );

  Map<String, dynamic> toJson() => {
        'id': id,
        'object': object,
        'created': created,
        'model': model,
        'choices': choices,
        'usage': usage,
      };
}
```

**[src/models/chat_service_settings_model.dart]**<br/>
```dart
import 'package:chat_gpt/src/models/chat_message_model.dart';

class ChatServiceSettingsModel {
  final String? model;
  final List<ChatMessageModel>? messages;
  final String? suffix;
  final double temperature;
  final int maxTokens;
  final double topP;
  final String? stop;

  ChatServiceSettingsModel(
    this.model,
    this.messages,
    this.suffix,
    this.temperature,
    this.maxTokens,
    this.topP,
    this.stop,
  );

  factory ChatServiceSettingsModel.fromJson(Map<String, dynamic> json) =>
      ChatServiceSettingsModel(
        json['model'] as String,
        (json['messages'] as List).map((p) => ChatMessageModel.fromJson(p)).toList(),
        json['suffix'] as String,
        json['temperature'] as double,
        json['maxTokens'] as int,
        json['topP'] as double,
        json['stop'] as String,
      );

  Map<String, dynamic> toJson() => {
        'model': model,
        'messages': messages,
        'suffix': suffix,
        'temperature': temperature,
        'maxTokens': maxTokens,
        'topP': topP,
        'stop': stop,
      };
}
```

**[src/models/chat_usage_model.dart]**<br/>
```dart
class ChatUsageModel {
  int promptTokens;
  int completionTokens;
  int totalTokens;

  ChatUsageModel(
    this.promptTokens,
    this.completionTokens,
    this.totalTokens,
  );

  factory ChatUsageModel.fromJson(Map<String, dynamic> json) =>
      ChatUsageModel(
        json['prompt_tokens'] as int,
        json['completion_tokens'] as int,
        json['total_tokens'] as int,
      );

  Map<String, dynamic> toJson() => {
        'prompt_tokens': promptTokens,
        'completion_tokens': completionTokens,
        'total_tokens': totalTokens,
      };
}
```

이렇게 만들어진 모델을 이용해서 실제 http로 API 요청 부분을 서비스로 분리하여 구현해 보았습니다.<br/><br/>

**[src/services/chat_gpt_service.dart]**<br/>
```dart
import 'dart:convert';
import 'dart:io';

import 'package:chat_gpt/src/models/chat_response_model.dart';
import 'package:chat_gpt/src/models/chat_service_settings_model.dart';
import 'package:dio/dio.dart';

import '../models/chat_request_body_model.dart';

class ChatGptService {
  Future<ChatResponseModel?> getResponseDataAsync(
      ChatServiceSettingsModel settings, String? apiKey) async {
    // Set up the API URL and API key
    var apiUrl = "https://api.openai.com/v1/chat/completions";

    if (apiKey == null) {
      return null;
    }

    // Get the request body JSON
    var requestBodyJson = _getRequestBodyJson(settings);

    // Send the API request and get the response data
    return await sendApiRequestAsync(apiUrl, apiKey, requestBodyJson);
  }

  String _getRequestBodyJson(ChatServiceSettingsModel settings) {
    ChatRequestBodyModel requestBody = ChatRequestBodyModel(
        settings.model,
        settings.messages,
        settings.temperature,
        settings.topP,
        1,
        false,
        settings.stop,
        settings.maxTokens,
        0.0,
        0.0,
        null,
        null);

    var requestMap = requestBody.toJson();
    requestMap.removeWhere((key, value) => value == null);

    return json.encode(requestMap);
  }

  Future<ChatResponseModel> sendApiRequestAsync(
      String apiUrl, String apiKey, String requestBodyJson) async {
    var dio = Dio(BaseOptions(
      responseType: ResponseType.json,
      contentType: ContentType.json.toString(),
    ));
    dio.options.headers["Authorization"] = "Bearer $apiKey";
    Response<Map<String, dynamic>> resposne =
        await dio.post(apiUrl, data: requestBodyJson, options: Options(
          followRedirects: false,
          // Status code 501 미만 까지만 유효성 검증 이후 코드는 throw 된다.
          validateStatus: (status) => status! < 501,
        ));
    if (resposne.statusCode == 200) {
      return ChatResponseSuccessModel.fromJson(resposne.data!);
    }

    switch (resposne.statusCode) {
      case 401:
      case 429:
      case 500:
        return ChatResponseErrorModel.fromJson(resposne.data!);
    }
    return ChatResponseErrorModel('statusCode : ${resposne.statusCode}', 'An unknown error occurred', null, null);
  }
}
```

getResponseDataAsync() 메서드를 통해 ChatGPT API를 요청 할 수 있습니다.<br/>
해당 부분을 잠시 살펴 보면 **<span style="color: rgb(107, 173, 222);">ChatServiceSettingsModel</span>** 클래스를 json으로 시리얼라이즈 해서 요청하고 있는데 해당 모델에는
지난 대화 메세지(**<span style="color: rgb(107, 173, 222);">ChatMessageModel</span>**)가 배열 형태로 포함되어 있고 ChatGPT의 사용 모델 이름과 GPT 인공지능 대화 반응의 온도(**온도가 높을 수록 예측하기 어려운 답변과 창의성이 올라감**) 최대 토큰수 등
정를 포함 하고 있습니다.<br/><br/>

이렇게 요청된 응답은 성공시 **<span style="color: rgb(107, 173, 222);">ChatResponseSuccessModel</span>** 실패시 **<span style="color: rgb(107, 173, 222);">ChatResponseErrorModel</span>** 모델 정보로
반환 되어 집니다.<br/>


채팅 UI 구현
-

채팅 화면의 구조를 보면 대화 메세지 부분이 나오고 답변 메세지에는 사용자가 다시 메세지를 입력 할 수 있는 TextField가 포함되어 있는걸 볼 수 있습니다.<br/>
또한 메세지 전송시에는 요청중인 ProgressIndicator가 표시 되고 있습니다.<br/>
추가로 각 대화 메세지 마다 수정, 삭제를 할 수 있는데 수정은 사용자가 입력한 메세지 한해서 수정 가능하도록 보여 지고 있습니다.<br/><br/>

**[사용자 입력 메세지와 답변 메세지(사용자 메세지 입력 박스 포함)]**<br/>
![image](https://user-images.githubusercontent.com/13028129/224870544-9e7944e3-fd1c-455f-b8d9-219ebdaa052c.png)<br/><br/>

**[메세지 전송 요청중 표시]**<br/>
![image](https://user-images.githubusercontent.com/13028129/224870601-db67e736-7973-4039-a74b-7b705aaf508e.png)<br/><br/>

이렇게 화면에 보여지는 기능에 대해 필요한 정보인 뷰모델을 다음과 같이 구현해 볼 수 있습니다.<br/><br/>

**[src/viewmodels/chat_message_viewmodel.dart]**<br/>
```dart
class ChatMessageViewModel {
  String? prompt;
  String message;
  bool isSent;
  bool isAwaiting;
  bool isError;
  bool canRemove;
  bool isEditing;
  ChatMessageViewModel? result;

  ChatMessageViewModel(this.prompt, this.message,
      {this.isSent = false,
      this.isAwaiting = false,
      this.isError = false,
      this.canRemove = false,
      this.isEditing = false,
      this.result});
}
```

ViewModel은 json 시리얼라이즈가 필요 없어 fromJson() / toJson() 구현은 필요 없습니다.<br/>
이렇게 만들어진 ViewModel은 화면에서 표시 되는 채팅 정보 표시에 사용 되는 정보로 BLoC을 통해 상태 통보될때 사용 됩니다.<br/>

이제 해당 서비스를 사용하는 대화 상태의 BLoC를 구현해 보겠습니다.<br/>
대화 상태는 간단하게 두개의 상태 변경으로 처리 하였습니다.<br/><br/>

채팅을 할때 마다 늘어 나거나 삭제할때 줄어 드는 대화 리스트에 대한 상태 변화<br/>
그리고 메세지 요청중 또는 해당 대화 메세지 변경(메세지 수정 및 응답 메세지 변화, 사용자 메세지 입력 필드 표시 여부 변경) 상태 변화<br/><br/>

이렇게 상태 변경 클래스를 각각 구현해 봅니다.<br/><br/>

**[src/bloc/chat_message_state.dart]**<br/>
```dart
import 'package:chat_gpt/src/viewmodels/chat_message_viewmodel.dart';

abstract class ChatMessageState {}

/// 대화 리스트 상태
class ChatListMessageState extends ChatMessageState {
  final List<ChatMessageViewModel> chatMessageList;

  ChatListMessageState(this.chatMessageList);
}

/// 대화 메세지 상태 변경
class ChatMessageChangeState extends ChatMessageState {
  final ChatMessageViewModel chatMessage;

  ChatMessageChangeState(this.chatMessage);
}
```

그리고 해당 **<span style="color: rgb(107, 173, 222);">ChatMessageState</span>** 상태를 사용하는 BLoC도 다음과 같이 구현 합니다.<br/><br/>

**[src/bloc/chat_message_cubit.dart]**<br/>
```dart
import 'package:chat_gpt/src/bloc/chat_message_state.dart';
import 'package:chat_gpt/src/models/chat_message_model.dart';
import 'package:chat_gpt/src/models/chat_response_model.dart';
import 'package:chat_gpt/src/models/chat_service_settings_model.dart';
import 'package:chat_gpt/src/repository/settings_repository.dart';
import 'package:chat_gpt/src/services/chat_gpt_service.dart';
import 'package:chat_gpt/src/viewmodels/chat_message_viewmodel.dart';
import 'package:flutter_bloc/flutter_bloc.dart';

class ChatMessageCubit extends Cubit<ChatMessageState> {
  final ChatGptService chatGptService;
  final SettingsRepository settingsRepository;
  final List<ChatMessageViewModel> _chatMessageList = [];

  ChatMessageCubit({
    required this.chatGptService,
    required this.settingsRepository,
  }) : super(ChatListMessageState([]));

  void getChatMessageData() {
    if (_chatMessageList.isEmpty) {
      // 최초 시작시 보여지는 고정 메세지
      _chatMessageList.add(ChatMessageViewModel(
        null,
        "Hi! I'm Clippy, your Windows Assistant. Would you like to get some assistance?",
      ));
    }
    emit(ChatListMessageState(_chatMessageList));
  }

  /// 메세지 삭제
  removeChatMessage(ChatMessageViewModel chatMessage) {
    _chatMessageList.remove(chatMessage);
    emit(ChatListMessageState(_chatMessageList));

    _chatMessageList.last.isSent = false;
    emit(ChatMessageChangeState(_chatMessageList.last));
  }

  // 메세지 수정
  editChatMessage(ChatMessageViewModel chatMessage) {
    chatMessage.isEditing = !chatMessage.isEditing;
    emit(ChatMessageChangeState(chatMessage));
  }

  // 메세지 전송
  Future<void> sendChatMessage(ChatMessageViewModel sendMessage) async {
    try {
      // load 환경설정 데이터
      var userSettings = await settingsRepository.getSettings();

      // ChatGPT API - Send 요청시 지난 대화 기록 포함하여 요청
      List<ChatMessageModel> chatMessageList = [
        ChatMessageModel('system', userSettings.directions),
      ];

      for (var chatMessage in _chatMessageList) {
        if (chatMessage.message.isNotEmpty && chatMessage.result != null) {
          chatMessageList.add(ChatMessageModel('user', chatMessage.message));
          chatMessageList
              .add(ChatMessageModel('assistant', chatMessage.result!.message));
        }
      }

      // sendChatMessage 메서드 호출시는 chatMessage.prompt 속성이 null이 될 수 없다.
      chatMessageList.add(ChatMessageModel('user', sendMessage.prompt!));

      bool isUpdate = false;
      var last = _chatMessageList.last;
      // 중간 대화 메세지 수정 해서 전송 한 경우
      if (last.hashCode != sendMessage.hashCode) {
        sendMessage.isAwaiting = true;
        isUpdate = true;
        sendMessage.isEditing = false;
        // 중간 대화 메세지 수정 후 전송처리 상태 변경
        emit(ChatMessageChangeState(sendMessage));
      }

      // user 입력 메세지
      ChatMessageViewModel promptMessage;
      // 전송한 메세지에 대한 결과
      ChatMessageViewModel resultMessage;

      sendMessage.isSent = true;

      if (isUpdate == false) {
        // user 입력 메세지
        promptMessage = ChatMessageViewModel(
          null,
          sendMessage.prompt!,
          canRemove: true,
          isSent: true,
          // 전송 요청 대기
          isAwaiting: true,
        );
        // 전송한 메세지에 대한 결과
        resultMessage = ChatMessageViewModel(
          null,
          "",
          canRemove: true,
        );

        promptMessage.result = resultMessage;
      } else {
        var prompt = sendMessage.prompt!;
        promptMessage = sendMessage;
        promptMessage.message = prompt;
        resultMessage = sendMessage.result!;
      }

      if (isUpdate == false) {
        // user 입력 메세지 기록 추가
        _chatMessageList.add(promptMessage);
        emit(ChatListMessageState(_chatMessageList));
      }

      // ChatGPT Send API 요청에 필요한 데이터 모델
      ChatServiceSettingsModel chatServiceSettings = ChatServiceSettingsModel(
          userSettings.model,
          chatMessageList,
          null,
          userSettings.temperature,
          userSettings.maxTokens,
          1.0,
          null);

      // 실제 ChatGPT 대화 Send API 요청
      var responseData = await chatGptService.getResponseDataAsync(
          chatServiceSettings, userSettings.apiKey);

      // 전송 완료
      promptMessage.isAwaiting = false;

      // 요청 완료 이벤트 발생 - isAwaiting = false 상태 변경
      emit(ChatMessageChangeState(promptMessage));

      // 응답 오류 발생시
      if (responseData == null) {
        resultMessage.isError = true;
        resultMessage.message = 'Requires apiKey setting.';
      }
      else if (responseData is ChatResponseErrorModel) {
        var error = responseData;

        resultMessage.isError = true;
        // ChatGPT에서 오류 메세지 반환
        if (error.message != null) {
          resultMessage.message = '${error.message!} - ${error.code}';
        }
        // ChatGPT에서 오류 메세지 반환 없음
        else {
          resultMessage.message = 'An unknown error occurred, try again!';
        }
      }
      // 정상 응답
      else {
        var responseSuccess = responseData as ChatResponseSuccessModel;

        // 전송한 메세지에 대한 결과
        resultMessage.message =
            responseSuccess.choices.first.message.content.trim();
      }

      if (isUpdate == false) {
        // 메세지 결과 기록 추가
        _chatMessageList.add(resultMessage);
        // 요청 결과 완료 이벤트 발생
        emit(ChatListMessageState(_chatMessageList));
      } else {
        // 요청 결과 완료 이벤트 발생
        emit(ChatMessageChangeState(resultMessage));
      }
    } catch (ex) {
      print(ex.toString());
    }
  }
}
```

이렇게 **BLoC에서**(여기선 간단하게 Cubit을 사용 합니다.) 대략 메세지 삭제, 메세지 수정, 메세지 전송 처리에 대한 **View 관련 비즈니스 로직을 위와 같이 구현**하고, 각 기능에 대한 **상태를 이벤트로 통보**하여
**BLoC를 구독하고 있는 위젯(View)에서 상태에 따라 화면이 반영** 될 수 있도록 처리해 주었습니다.<br/>
**(MVVM 패턴과 비교하자면 ViewModel 과 같은 역할)**<br/><br/>

그럼 이제 View에 관한 위젯만 구현해 주면 될 것 같습니다. 먼저 채팅 메세지 관련 부분에 해당 되는 위젯을 구현 합니다.<br/><br/>

**[src/components/chat_item.dart]**<br/>
```dart
import 'package:chat_gpt/src/bloc/chat_message_cubit.dart';
import 'package:chat_gpt/src/bloc/chat_message_state.dart';
import 'package:chat_gpt/src/bloc/theme_cubit.dart';
import 'package:chat_gpt/src/syntax_highlighter.dart';
import 'package:chat_gpt/src/viewmodels/chat_message_viewmodel.dart';
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:flutter_markdown/flutter_markdown.dart';

class ChatItem extends StatefulWidget {
  ChatMessageViewModel chatMessageViewModel;
  ChatItem(this.chatMessageViewModel, {super.key});

  @override
  State<ChatItem> createState() => _ChatItemState();
}

class _ChatItemState extends State<ChatItem> {
  late TextEditingController _chatTextEditingController;
  final TextEditingController _sendTextEditingController =
      TextEditingController();

  /// Alt + Enter : 줄바꿈 처리, Enter : Send Key Event 처리
  late final _focusNode = FocusNode(
    onKey: (FocusNode node, RawKeyEvent evt) {
      if (!evt.isAltPressed && evt.logicalKey.keyLabel == 'Enter') {
        if (evt is RawKeyDownEvent) {
          _sendChatMessage();
        }
        return KeyEventResult.handled;
      } else {
        return KeyEventResult.ignored;
      }
    },
  );

  /// 우측 복사, 편집, 대화삭제 컨트롤 표시
  List<Widget> _controlWidget() {
    if (widget.chatMessageViewModel.canRemove == false) {
      return [
        IconButton(
          onPressed: () {
            Clipboard.setData(
                ClipboardData(text: widget.chatMessageViewModel.message));
            ScaffoldMessenger.of(context).showSnackBar(
              const SnackBar(
                content: Text("Copied."),
                duration: Duration(milliseconds: 1000),
              ),
            );
          },
          iconSize: 17,
          icon: const Icon(Icons.copy),
          tooltip: 'Copy',
          hoverColor: Colors.transparent,
          highlightColor: Colors.transparent,
        ),
      ];
    }

    if (widget.chatMessageViewModel.message.isNotEmpty &&
        widget.chatMessageViewModel.result != null) {
      return [
        IconButton(
          onPressed: () {
            Clipboard.setData(
                ClipboardData(text: widget.chatMessageViewModel.message));
            ScaffoldMessenger.of(context).showSnackBar(
              const SnackBar(
                content: Text("Copied."),
                duration: Duration(milliseconds: 1000),
              ),
            );
          },
          iconSize: 17,
          icon: const Icon(Icons.copy),
          tooltip: 'Copy',
          hoverColor: Colors.transparent,
          highlightColor: Colors.transparent,
        ),
        IconButton(
          onPressed: () {
            context
                .read<ChatMessageCubit>()
                .editChatMessage(widget.chatMessageViewModel);
          },
          iconSize: 17,
          icon: const Icon(Icons.edit),
          tooltip: 'Edit',
          hoverColor: Colors.transparent,
          highlightColor: Colors.transparent,
        ),
        IconButton(
          onPressed: () {
            context
                .read<ChatMessageCubit>()
                .removeChatMessage(widget.chatMessageViewModel);
          },
          iconSize: 17,
          icon: const Icon(Icons.delete),
          tooltip: 'Remove',
          hoverColor: Colors.transparent,
          highlightColor: Colors.transparent,
        ),
      ];
    }

    if (widget.chatMessageViewModel.message.isNotEmpty &&
        widget.chatMessageViewModel.result == null) {
      return [
        IconButton(
          onPressed: () {
            Clipboard.setData(
                ClipboardData(text: widget.chatMessageViewModel.message));
            ScaffoldMessenger.of(context).showSnackBar(
              const SnackBar(
                content: Text("Copied."),
                duration: Duration(milliseconds: 1000),
              ),
            );
          },
          iconSize: 17,
          icon: const Icon(Icons.copy),
          tooltip: 'Copy',
          hoverColor: Colors.transparent,
          highlightColor: Colors.transparent,
        ),
        IconButton(
          onPressed: () {
            context
                .read<ChatMessageCubit>()
                .removeChatMessage(widget.chatMessageViewModel);
          },
          iconSize: 17,
          icon: const Icon(Icons.delete),
          tooltip: 'Remove',
          hoverColor: Colors.transparent,
          highlightColor: Colors.transparent,
        ),
      ];
    }

    return [];
  }

  /// 메세지 입력 TextField 표시
  Widget _displayMessageInputWidget(ChatMessageViewModel chatMessage) {
    // Editing 모드가 아니고, 지난 대화기록은 user 메세지 Input TextField 표시 하지 않는다.
    if (chatMessage.isEditing == false && chatMessage.isSent == true) {
      return Container();
    }

    // Editing 모드
    if (chatMessage.isEditing =
        true && chatMessage.result != null && chatMessage.message.isNotEmpty) {
      _sendTextEditingController.text = chatMessage.message;
    }

    // ChatGPT 답변 후 user 메세지 Input TextField 표시
    return Row(children: [
      Expanded(
        child: Container(
            padding: const EdgeInsets.fromLTRB(10, 0, 10, 0),
            decoration: BoxDecoration(
              border: Border.all(
                  color: const Color.fromARGB(255, 0, 0, 0), width: 1),
              borderRadius: BorderRadius.circular(5),
            ),
            child: RawKeyboardListener(
              focusNode: _focusNode,
              child: TextField(
                controller: _sendTextEditingController,
                maxLength: null,
                maxLines: null,
                keyboardType: TextInputType.multiline,
                decoration: const InputDecoration(
                    border: InputBorder.none,
                    hintText: 'Ask me anyting',
                    hintStyle:
                        TextStyle(fontSize: 15, overflow: TextOverflow.clip)),
              ),
            )),
      ),

      const SizedBox(
        width: 15,
      ),

      // 전송 버튼
      Container(
          height: 50,
          padding: const EdgeInsets.all(5),
          decoration: BoxDecoration(
            border:
                Border.all(color: const Color.fromARGB(255, 0, 0, 0), width: 1),
            borderRadius: BorderRadius.circular(5),
          ),
          child: IconButton(
            onPressed: () => _sendChatMessage(),
            icon: const Icon(Icons.send),
            tooltip: 'Send',
            hoverColor: Colors.transparent,
          )),
    ]);
  }

  /// 채팅 메세지 및 메세지 전송 TextField 표시
  List<Widget> _displayChatMessage(ChatMessageViewModel chatMessage) {
    // system 메세지
    if (chatMessage.canRemove == false) {
      _chatTextEditingController =
          TextEditingController(text: chatMessage.message);
      return [
        // system 메세지 TextField
        TextField(
          readOnly: true,
          controller: _chatTextEditingController,
          maxLength: null,
          maxLines: null,
          keyboardType: TextInputType.multiline,
          decoration: const InputDecoration(border: InputBorder.none),
        ),
      ];
    }

    // user의 메세지
    if (chatMessage.message.isNotEmpty && chatMessage.result != null) {
      _chatTextEditingController =
          TextEditingController(text: chatMessage.message);
      return [
        // user 입력 TestField
        TextField(
          readOnly: true,
          controller: _chatTextEditingController,
          maxLength: null,
          maxLines: null,
          keyboardType: TextInputType.multiline,
          decoration: const InputDecoration(border: InputBorder.none),
        )
      ];
    }

    // ChatGPT의 답변 메세지
    if (chatMessage.message.isNotEmpty && chatMessage.result == null) {
      return [
        BlocBuilder<ThemeCubit, ThemeData>(
          builder: (_, theme) {
            // ChatGPT 답변 Markdown
            return MarkdownBody(
                selectable: true,
                syntaxHighlighter: DartSyntaxHighlighter(
                    theme.brightness == Brightness.dark
                        ? SyntaxHighlighterStyle.darkThemeStyle()
                        : SyntaxHighlighterStyle.lightThemeStyle()),
                data: chatMessage.message,
                styleSheet: MarkdownStyleSheet.fromTheme(Theme.of(context))
                    .copyWith(textAlign: WrapAlignment.start)
                    .copyWith(
                        p: Theme.of(context)
                            .textTheme
                            .bodyText1
                            ?.copyWith(fontSize: 15.0)));
          },
        )
      ];
    }

    return [];
  }

  void _sendChatMessage() {
    if (widget.chatMessageViewModel.isEditing) {
      widget.chatMessageViewModel.message = '';
    }

    widget.chatMessageViewModel.prompt = _sendTextEditingController.text;
    context
        .read<ChatMessageCubit>()
        .sendChatMessage(widget.chatMessageViewModel);
  }

  Widget _awaitingWidget(ChatMessageViewModel chatMessage) {
    if (chatMessage.isAwaiting) {
      return const LinearProgressIndicator();
    } else {
      return Container();
    }
  }

  @override
  Widget build(BuildContext context) {
    return Container(
      margin: const EdgeInsets.fromLTRB(10, 0, 10, 0),
      child: Row(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Expanded(
            child: Container(
              padding: const EdgeInsets.symmetric(horizontal: 15, vertical: 15),
              decoration: BoxDecoration(
                border: Border.all(
                    color: const Color.fromARGB(255, 181, 181, 181), width: 1),
                borderRadius: BorderRadius.circular(5),
              ),
              child: BlocBuilder<ChatMessageCubit, ChatMessageState>(
                builder: (_, state) {
                  if (state is ChatMessageChangeState &&
                      state.chatMessage.hashCode ==
                          widget.chatMessageViewModel.hashCode) {
                    return Column(
                      children: _displayChatMessage(state.chatMessage)
                        ..add(_displayMessageInputWidget(state.chatMessage))
                        ..add(const SizedBox(height: 10))
                        ..add(_awaitingWidget(state.chatMessage)),
                    );
                  }

                  return Column(
                    children: _displayChatMessage(widget.chatMessageViewModel)
                      ..add(_displayMessageInputWidget(
                          widget.chatMessageViewModel))
                      ..add(const SizedBox(height: 10))
                      ..add(_awaitingWidget(widget.chatMessageViewModel)),
                  );
                },
              ),
            ),
          ),
          Column(
            children: _controlWidget(),
          )
        ],
      ),
    );
  }
}
```

**<span style="color: rgb(107, 173, 222);">BlocBuilder</span>** 를 사용해서 상태에 따라 메세지 요청중, 사용자 메세지 입력 TextField 표시 처리를 하고 있습니다.<br/>
그리고 ChatGPT의 답변 메세지 부분은 프로그래밍 코드 관련 답변 메세지는 Markdown 포맷 형태로 응답 되도록 방향을 정할 수 있기 때문에 Markdown 데이터를 렌더링 표현 할 수 있는
[flutter_markdown](https://pub.dev/packages/flutter_markdown) 패키지를 사용하였고,
코드 하이라이트 표시 처리를 위해 **<span style="color: rgb(107, 173, 222);">SyntaxHighlighter</span>** 클래스를 상속받아 별도 구현 하여 code syntax highlighter 처리를 하였습니다.<br/><br/>

이렇게 구현된 **<span style="color: rgb(107, 173, 222);">ChatItem</span>** 위젯은 메인 화면에서 리스트 형태로 표시 하면 비로소 채팅 부분 UI가 완성 됩니다.<br/>
다음은 리스트로 표시 되는 채팅 부분 UI 구현 부분 입니다.<br/><br/>

**[src/views/app_view.dart]**<br/>
```dart
Widget _bodyWidget() {
    return Container(
      margin: const EdgeInsets.all(15),
      child: BlocConsumer<ChatMessageCubit, ChatMessageState>(
        listenWhen: (previous, current) => current is ChatListMessageState,
        listener: (context, state) {
          // ChatListMessageState 상태 변경시 리스트뷰 스크롤 제일 하단으로 이동
          final postion = _listScrollController.position.maxScrollExtent;
          _listScrollController.jumpTo(postion);
        },
        buildWhen: (previous, current) => current is ChatListMessageState,
        builder: (_, state) {
          final chatMessageList =
              (state as ChatListMessageState).chatMessageList;

          return ListView.separated(
            controller: _listScrollController,
            itemCount: chatMessageList.length,
            itemBuilder: (_, index) {
              final chatMessageViewModel = chatMessageList[index];

              return ChatItem(chatMessageViewModel);
            },
            separatorBuilder: (_, index) => const SizedBox(
              height: 10,
            ),
          );
        },
      ),
    );
}
```

앞에서 채팅 메세지 표시 부분의 위젯을 구현해 놓았기 때문에 그것을 리스트 형식으로 표현 하는 것은 간단 합니다.<br/>
그리고 채팅이 진행되는 상태(**<span style="color: rgb(107, 173, 222);">ChatListMessageState</span>**) 수신시 리스트뷰의 스크롤이 제일 하단으로 이동 되도록 추가 하였습니다.<br/>
**<span style="color: rgb(107, 173, 222);">ChatMessageCubit</span>** 에서 사용 되는 **<span style="color: rgb(107, 173, 222);">ChatMessageState</span>** 상태는 하나가 아니고
대화 메세지 변경에 대한 상태(**<span style="color: rgb(107, 173, 222);">ChatMessageStatet</span>**)도 관리 되고 있기 때문에 listenWhen과 buildWhen 속성으로 조건을 사용 하였습니다.<br/>
또한 상태 수신과 상태에 따른 위젯 변화를 동시에 처리 하기 위해 **<span style="color: rgb(107, 173, 222);">BlocConsumer&lt;B extends StateStreamable&lt;S&gt;, S&gt;</span>** 위젯을 사용해서
BLoC을 구독하도록 하였습니다.<br/><br/>

***

**[결과 화면]**<br/>
![image](https://user-images.githubusercontent.com/13028129/224889604-f78d219d-21a9-4845-b74f-779b1e7b8396.png)<br/><br/>


이렇게 flutter_bloc 패키지를 사용한 BLoC 구조인 View와 비즈니스 로직을 분리 설계하고 ChatGPT Open API를 사용해 ChatGPT GUI Client App을 구현해 보았습니다.

***


위 코드는 다음 Repository에서 확인할 수 있습니다.<br/>
> [ChatGPT_Flutter](https://github.com/tyeom/ChatGPT_Flutter)



{% include content_adsense.html %}
