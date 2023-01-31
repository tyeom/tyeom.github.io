---
title: (Flutter) 플러터로 게임 만들어보기 01 (with flame)
categories: Flutter
key: 20230119_01
comments: true
tags: Flutter flame game 게임 2D게임 2D 게임엔진 GameEngine
---

이번 포스팅은 Flutter로 간단한 게임 개발 목표로 시리즈 형태로 작성해볼까 합니다.<br/>
Flutter에서 게임 개발에 필요한 기능을 제공해 주는 패키지들이 몇가지가 있는데 그중 [flame](https://pub.dev/packages/flame) 패키지를 이용해서<br/>
동적인 2D 게임 개발을 목표로 잡고 만들어 보도록 하겠습니다.

<!--more-->

> 이 글에서 다루는 코드는 다음 Repository에서 확인할 수 있습니다.<br/>
> [Flutter_flame_game](https://github.com/tyeom/Flutter_flame_game)

[flame](https://pub.dev/packages/flame) 패키지는 Flutter 에서 게임 제작에 필요한 기능을 제공해 주는 미니멀리스트의 게임 엔진 입니다.<br/>
flame 패키지를 이용하면 Flutter 에서 보다 쉽고 빠르게 동적인 2D 게임 제작이 가능합니다.<br/>
카메라, 사운드, 이펙트, 화면 라우팅, 충돌 감지 등 필수 기능을 제공하고 컴포넌트 자체가 플러터의 위젯과 동일하게 제공되기 때문에 기존 위젯들과 같이 호환되어 사용 가능합니다.


Game Loop
-

게임 루프는 게임이 진행되는 전반적인 라이프 사이클의 구조 입니다.<br/>
![image](https://user-images.githubusercontent.com/13028129/213368744-7b1f9a0c-6573-4186-84d3-60429c3b1922.png)<br/>
FlameGame 위젯이 추가 되면 처음 최초 한번 onLoad() 메서드가 호출되고 이후에는 Update() 메서드 <--> Render() 메서드가 서로 반복되서 진행됩니다.<br/><br/>

**OnLoad() 메서드**에서는 게임 컨텐츠에 필요한 Assets 처리 밎 컴포넌트가 생성되는 단계입니다.<br/>
컴포넌트란 게임 컨텐츠에 필요한 하나의 오브젝트라고 이해하면 됩니다. 플레이어의 캐릭터, 배경, 바닥, 구조물, 아이템 등등 각각이 컴포넌트 단위 입니다.<br/>
이렇게 컴포넌트가 준비 되면 **Update() 메서드**에서는 컴포넌트의 상태 처리가 되고 Render() 메서드에서 현재 상태를 그리기 위해 캔버스를 사용해서 화면에 표시 하게 됩니다.<br/>
Update() 메서드는 Render에서 마지막 업데이트 이후 델타 시간을 MS단위로 계속 반복 호출이 되어 집니다.


Component
-

![image](https://user-images.githubusercontent.com/13028129/213626643-bb9eddd5-d5f6-46cc-942e-26d16e8f7e6b.png)<br/><br/>
컴포넌트는 Effects, SpriteComponent, SpriteAnimationComponent 등 게임 컨텐츠에 필요한 구성요소를 뜻합니다.<br/>
예를 들어 이미지가 필요한 경우 SpriteComponent 클래스를 이용해서 구성요소를 생성할 수 있는데 모든 컴포넌트 클래스는 **<span style="color: rgb(107, 173, 222);">Component</span>** 클래스를 상속받고 있어 
add() 메서드를 통해 화면에 추가 할 수 있습니다.<br/>
또한 각 컴포넌트 또한 render() 메서드에서 어떻게 화면에 표시할지 동작을 제어할 수 있습니다.


GameWidget
-

**<span style="color: rgb(107, 173, 222);">GameWidget&lt;T&gt;</span>** 클래스는 flame 게임 컨텐츠를 표현할 수 있는 위젯입니다.<br/>
그래서 게임이 시작되는 화면에 가장 배치되어 집니다.
또한 기본적으로 Flutter 프레임워크의 **<span style="color: rgb(107, 173, 222);">StatefulWidget</span>** 추상 클래스를 상속받고 있어서 Flutter 위젯중 자식으로 받을 수 있는 위젯에 포함해서 사용 가능합니다.<br/>
기본적으로 **<span style="color: rgb(107, 173, 222);">Scaffold</span>** 위젯의 Body에 추가되서 기본 앱에 게임을 삽입하고, 기본적인 구조는 다음과 같이 처리할 수 있습니다.<br/>

**[main.dart]**<br/>
```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();

  // 이미지 로드
  var backgroundSprite = await Flame.images.loadAll(["Backgrounds.png"]);

  runApp(MaterialApp(
    title: 'Flame Game',
    home: GestureDetector(
      child: Scaffold(
        body: GameWrapper(MyGame(backgroundSprite[0])),
      ),
    ),
  ));
}

class GameWrapper extends StatelessWidget {
  final MyGame myGame;
  const GameWrapper(this.myGame, {super.key});

  @override
  Widget build(BuildContext context) {
    return GameWidget(game: myGame);
  }
}
```

**[game/my_game.dart]**<br/>
```dart
class MyGame extends FlameGame {
  final Image backgroundSprite;
  MyGame(this.backgroundSprite) {}
  
  @override
  onLoad() async {
    await super.onLoad();
  }

  @override
  void update(double dt) {
    super.update(dt);
  }
}
```


Sprite
-

게임에서 이미지는 단독으로 개별적인 이미지 파일을 로드해서 사용될 수 도 있지만 하나의 이미지에 여러개의 컨텐츠 이미지를 모두 포함시켜<br/>
필요한 영역의 이미지만 로드해서 렌더링 할 수 있습니다.<br/>
Sprites된 이미지를 사용하면 연속적인 이미지를 표현할때 애니메이션 효과로 나타내기가 용이합니다.<br/>
다음과 같은 이미지가 있습니다.<br/>
![Backgrounds](https://user-images.githubusercontent.com/13028129/213628182-efe99197-61b0-454a-b514-bf856baba746.png)<br/><br/>
위 이미지는 3개 영역의 배경 이미지로 되어 있는데 **<span style="color: rgb(107, 173, 222);">SpriteComponent</span>** 로 개별 영역의 이미지로 표시할 수 있습니다.<br/>
먼저 이미지 로드는 다음과 같이 여러개의 이미지 파일을 로드할 수 있습니다.<br/>

**[main.dart]**<br/>
```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  // 이미지 로드
  var backgroundSprite = await Flame.images.loadAll(["Backgrounds.png"]);
}
```

**<span style="color: rgb(107, 173, 222);">Images</span>** 의 loadAll() 메서드는 한번에 여러 이미지 경로를 받아 여러개의 이미지를 로드 처리할 수 있고, 
배열의 인덱스로 접근할 수 있습니다.<br/>
이미지 로드는 비동기로 처리되서 async / await으로 대기를 해야 하며, **main() 메서드에서 비동기 처리 수행을 하기 위해서는 Flutter 엔진이 바인딩 상태가 준비**되어야 하기 때문에 **<span style="color: rgb(107, 173, 222);">WidgetsFlutterBinding</span>** 클래스의 정적 ensureInitialized() 메서드를 호출해줍니다.<br/>
이후 이미지 로드는 다음과 같이 처리합니다.<br/>

**GameWidget 부분**
```dart
// main에서 로드한 backgroundSprite에 접근
var background = SpriteComponent.fromImage(backgroundSprite, Vector2(290, 0), Vector2(144, 280));
add(background);
```

이렇게 X, Y 좌표를 좌측 상단(0, 0) 기준으로 X 축 290 부터 144 x 280 사이즈의 영역 만큼 이미지로 사용할 수 있습니다.<br/>
> 참고고 이미지 영역의 좌표 위치 정보는<br/>
> [spritecow](http://www.spritecow.com/) 사이트에서 해당 이미지를 로드하여 쉽게 알아낼 수 있습니다.<br/>

**[결과 화면]**<br/>
![image](https://user-images.githubusercontent.com/13028129/213628766-b74cf9dc-d6c7-4c10-838d-4dbf4341e53b.png)<br/><br/>

이렇게 해당 이미지의 위치(좌표)와 사이즈를 이용해서 마지막 영역 부분만 표시할 수 있습니다.<br/>
또한 컴포넌트는 우선순위 특성을 가지고 있어 순차적으로 add 해서 Z-Index효과를 나타낼 수 있습니다.<br/>
```dart
// 이미지 로드
var backgroundSprite = await Flame.images.loadAll(["Backgrounds.png"]);
var background01 = SpriteComponent.fromImage(backgroundSprite, Vector2(290, 0), Vector2(144, 280));
var background02 = SpriteComponent.fromImage(backgroundSprite, Vector2(144, 0), Vector2(144, 280));
var background03 = SpriteComponent.fromImage(backgroundSprite, Vector2(0, 0), Vector2(144, 280));

add(background01);
add(background02);
add(background03);
```

**[결과 화면]**<br/>
![image](https://user-images.githubusercontent.com/13028129/213629187-75999e50-4797-46a0-9581-e440e84164a4.png)


SpriteAnimation
-

**<span style="color: rgb(107, 173, 222);">SpriteAnimationComponent</span>** 클래스는 Sprite 처럼 특정 Sprites된 이미지의 일부를 연속으로 표현하여 애니메이션 효과를 나타낼 수 있습니다.<br/>
![Player](https://user-images.githubusercontent.com/13028129/213638147-36def200-b7c6-4de8-859d-425f976ff8ec.png)<br/><br/>

위와 같은 연속으로 애니메이션을 표현할 수 있는 이미지 한장이 있는 경우 새로로 각각 영역을 4등분 하여 다음과 같이 표현할 수 있습니다.<br/>
```dart
// 이미지 로드
var spriteImag = await Flame.images.load("Player.png");
List<Sprite> spritesGo = [
      Sprite(spriteImag, srcPosition: Vector2(0, 0), srcSize: Vector2(24, 24)),
      Sprite(spriteImag, srcPosition: Vector2(25, 0), srcSize: Vector2(24, 24)),
      Sprite(spriteImag, srcPosition: Vector2(50, 0), srcSize: Vector2(24, 24)),
      Sprite(spriteImag, srcPosition: Vector2(75, 0), srcSize: Vector2(24, 24)),
    ];

// 위 4개의 이미지를 0.15ms 시간 단위로 표현
var animatedPlayer = SpriteAnimation.spriteList(spritesGo, stepTime: 0.15);
// 40 x 40 사이즈 이미지로 SpriteAnimationComponent 생성
SpriteAnimationComponent animation = SpriteAnimationComponent(
        animation: animatedPlayer, size: Vector2(40, 40));
add(animation);
```

**[결과 화면]**<br/>
![player_ani](https://user-images.githubusercontent.com/13028129/213640293-1c9f5cfa-7791-4b8d-b67a-8318d61fb729.gif)<br/><br/>

이렇게 이미지를 연속 표현하여 애니메이션으로 표현할 수 있습니다.

***

지금까지 기본적인 flame의 주요 Game loop와 이미지를 표현하는 구성요소를 알아보았습니다.<br/>
다음에는 드래그 액션에 따른 플레이어 상태 애니메이션 처리와 배경 애니매이션 처리를 살펴보겠습니다.<br/><br/>


***

위 코드는 다음 Repository에서 확인할 수 있습니다.<br/>
> [Flutter_flame_game](https://github.com/tyeom/Flutter_flame_game)



{% include content_adsense.html %}
