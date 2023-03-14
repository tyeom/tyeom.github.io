---
title: (Flutter) Chat GPT Client 만들어보기 wirh 음성인식 추가
categories: Flutter
key: 20230314_01
comments: true
tags: Flutter ChatGPT flutter_speech 음성인식 SpeechRecognizer voiceRecognition
---

지난 글에서 만들어 보았던 [ChatGPT GUI Client App](https://blog.arong.info/flutter/2023/03/13/Flutter-Chat-GPT-Client-%EB%A7%8C%EB%93%A4%EC%96%B4%EB%B3%B4%EA%B8%B0-wirh-flutter_bloc-%ED%8C%A8%ED%82%A4%EC%A7%80.html)을
단순히 따라해 보기만 하고 끝내기는 아쉬워 음성인식 기능을 추가해 보도록 하겠습니다.

<!--more-->

> 이 글에서 다루는 코드는 다음 Repository에서 확인할 수 있습니다.<br/>
> [ChatGPT_Flutter](https://github.com/tyeom/ChatGPT_Flutter)

음성 인식 처리
-

Flutter의 음성 인식 관련 패키지를 찾아 보니 [flutter_speech](https://pub.dev/packages/flutter_speech) 패키지를 발견 했습니다.<br/>
사용 방법도 단순하고 Android와 iOS, MacOS를 지원해서 단순 음성 인식 처리에는 적절한 패키지인 것 같습니다.<br/>
해당 패키지를 추가 하고 대화 메세지 작성 부분 위젯인 **<span style="color: rgb(107, 173, 222);">ChatItem</span>** 클래스에 음성 인식 기능을 추가해 봅니다.<br/><br/>

음성 인식 기능 추가에 필요한 정보는 총 5가지 입니다.<br/>
1. 음성 인식 언어 코드 (영어권, 한국어권, 등)
2. flutter_speech 패키지 클래스
3. 음성 인식 사용 가능 여부
4. 음성 인식 중 여부
5. 음성 인식 결과 Text

위에 대한 정보를 처리 하기 위해 맴버 변수를 다음과 같이 정의 및 초기화 합니다.<br/><br/>

**[src/components/chat_item.dart]**<br/>
```dart
class _ChatItemState extends State<ChatItem> {
  ...[중간 생략]...

  // 음성 인식 관련
  // 영어권인 경우 'en_US'
  final String speechRecognition_locale = 'ko_KR';
  late SpeechRecognition _speech;
  bool _speechRecognitionAvailable = false;
  bool _isListening = false;
  String _transcription = '';
  
  ...[중간 생략]...
}
```

그리고 지연 초기화 대상인 음성 인식 패키지의 **<span style="color: rgb(107, 173, 222);">SpeechRecognition</span>** 인스턴스 생성은 initState 주기에 생성해서 초기화 하고,
관련 이벤트 콜백 메서드를 정의해 줍니다.<br/><br/>

**[src/components/chat_item.dart]**<br/>
```dart
...[중간 생략]...

@override
void initState() {
  super.initState();
  // Android, iOS, macOS 플랫폼만 지원
  if (foundation.defaultTargetPlatform == foundation.TargetPlatform.iOS ||
    foundation.defaultTargetPlatform == foundation.TargetPlatform.android ||
    foundation.defaultTargetPlatform == foundation.TargetPlatform.macOS) {
    _activateSpeechRecognizer();
  }
}

void _activateSpeechRecognizer() {
  print('_MyAppState.activateSpeechRecognizer... ');
  _speech = SpeechRecognition();
  _speech.setAvailabilityHandler(_onSpeechAvailability);
  _speech.setRecognitionStartedHandler(_onRecognitionStarted);
  _speech.setRecognitionResultHandler(_onRecognitionResult);
  _speech.setRecognitionCompleteHandler(_onRecognitionComplete);
  _speech.setErrorHandler(_errorHandler);
  _speech.activate(speechRecognition_locale).then((res) {
    setState(() {
      _speechRecognitionAvailable = res;
      _isListening = false;
    });
  });
}

/// 음성 인식 유효성 검증
void _onSpeechAvailability(bool result) => setState(() => _speechRecognitionAvailable = result);

/// 음성 인식 시작
void _onRecognitionStarted() {
  setState(() => _isListening = true);
}

/// 음성 인식 결과
void _onRecognitionResult(String text) {
  print('_MyAppState.onRecognitionResult... $text');
  setState(() {
    _sendTextEditingController.text = text;
    _transcription = text;
  });
}

/// 음성 인식 완료
void _onRecognitionComplete(String text) {
  print('_MyAppState.onRecognitionComplete... $text');
  setState(() => _isListening = false);
}

/// 음성 인식 오류 발생
void _errorHandler() => _activateSpeechRecognizer();

...[중간 생략]...
```

음성 인식 시작은 **<span style="color: rgb(107, 173, 222);">SpeechRecognition</span>** 클래스의 activate() 메서드로 음성 인식 활성화 후 listen() 메서드 호출로 시작할 수 있습니다.<br/>
listen() 호출시 별다른 오류발생이 없다면 RecognitionStartedHandler의 **<span style="color: rgb(107, 173, 222);">VoidCallback</span>** 콜백 메서드가 호출 됩니다.<br/><br/>

**[src/components/chat_item.dart]**<br/>
```dart
...[중간 생략]...

/// 음성 인식 시작
void _start() {
  ScaffoldMessenger.of(context).showSnackBar(const SnackBar(
    content: Text("Listening to your voice..."),
    duration: Duration(milliseconds: 2000),
  ));

  _speech.activate(speechRecognition_locale).then((_) {
    return _speech.listen().then((result) {
      print('SpeechRecognition start => result : $result');
      setState(() {
        _isListening = result;
      });
    });
  });
}

...[중간 생략]...
```

이제 음성 인식 기능 추가 준비가 되었습니다.<br/>
버튼을 추가하고 해당 버튼으로 음성 인식을 하고 인식 결과를 입력 메세지 Input TextField 컨트롤러인 <span>_sendTextEditingController</span>에 설정만 해주면 됩니다.<br/><br/>

**[src/components/chat_item.dart]**<br/>
```dart
...[중간 생략]...

/// 음성 인식 버튼 위짓
List<Widget> _displaySpeechRecognitionWidget() {
  // Android, iOS, macOS 플랫폼만 지원
  if (foundation.defaultTargetPlatform == foundation.TargetPlatform.iOS ||
    foundation.defaultTargetPlatform == foundation.TargetPlatform.android ||
    foundation.defaultTargetPlatform == foundation.TargetPlatform.macOS) {
      return [
        const SizedBox(width: 15,),

        // 음성 인식 버튼
        Container(
          height: 50,
          padding: const EdgeInsets.all(5),
          decoration: BoxDecoration(
            border: Border.all(
              color: const Color.fromARGB(255, 0, 0, 0), width: 1),
              borderRadius: BorderRadius.circular(5),
            ),
            child: IconButton(
              onPressed: () {
                if (_speechRecognitionAvailable && !_isListening) {
                  _start();
                }
              },
            icon: _isListening ? const Icon(Icons.voice_chat) : const Icon(Icons.mic),
            tooltip: 'Voice recognition',
            hoverColor: Colors.transparent,
          )),
      ];
  }
  else {
    return [];
  }
}

...[중간 생략]...
```

이렇게 위젯 배열을 반환하는 <span>_displaySpeechRecognitionWidget</span> 메서드는 기존에 전송 버튼 위젯을 감싸고 있는 Row 위젯의 children 부분에 추가 합니다.<br/><br/>

**[src/components/chat_item.dart]**<br/>
```dart
/// 메세지 입력 TextField 표시
Widget _displayMessageInputWidget(ChatMessageViewModel chatMessage) {
...[중간 생략]...

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
  ..._displaySpeechRecognitionWidget(),  // 음성 인식 버튼 추가

...[중간 생략]...
}
```

***

**[결과 화면]**<br/>

![Flutter_ChatGPT3](https://user-images.githubusercontent.com/13028129/224908964-d85ef777-35d4-4895-b20c-b0013632c001.gif)<br/>
![KakaoTalk_20230314_145155855](https://user-images.githubusercontent.com/13028129/224909157-9ac77909-7a70-45fa-a331-7da43e2e99ca.png)<br/><br/>

flutter_speech 패키지 사용으로 간단하게 음성 인식 기능을 추가해 보았습니다.

***


위 코드는 다음 Repository에서 확인할 수 있습니다.<br/>
> [ChatGPT_Flutter](https://github.com/tyeom/ChatGPT_Flutter)



{% include content_adsense.html %}
