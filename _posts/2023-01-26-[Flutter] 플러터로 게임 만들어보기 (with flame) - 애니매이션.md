---
title: (Flutter) 플러터로 게임 만들어보기 02 - 애니매이션 처리 (with flame)
categories: Flutter
key: 20230126_01
comments: true
tags: Flutter flame game 게임 2D게임 2D 게임엔진 GameEngine 애니매이션처리 SpriteAnimation SpriteAnimationGroupComponent
---

지난 글 - [플러터로 게임 만들어보기 01](https://blog.arong.info/flutter/2023/01/19/Flutter-%ED%94%8C%EB%9F%AC%ED%84%B0%EB%A1%9C-%EA%B2%8C%EC%9E%84-%EB%A7%8C%EB%93%A4%EC%96%B4%EB%B3%B4%EA%B8%B0-(with-flame).html)에 이서서 
이번에는 배경 이미지의 움직임 효과와 드래그 액션에 따른 각 상태에 맞는 플레이어 컴포넌트 애니매이션 처리 방법에 대해 알아보겠습니다.

<!--more-->

> 이 글에서 다루는 코드는 다음 Repository에서 확인할 수 있습니다.<br/>
> [Flutter_flame_game](https://github.com/tyeom/Flutter_flame_game)

일단 지난글에서 main.dart에서 GameWidget을 삽입하는 부분을 Scaffold를 GestureDetector위젯으로 감싸서 처리했는데 Scaffold 구조형식의 앱이 아니기 때문에 다음과 같이 변경했습니다.<br/><br/>

**[main.dart]**
```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();

  runApp(MaterialApp(
    title: 'Flame Game',
    home: GameWrapper(MyGame()),
  ));
}
```

***

컴포넌트 움직임 처리
-

지난 글에 배경 Sprite이미지를 가지고 **<span style="color: rgb(107, 173, 222);">SpriteComponent</span>** 를 사용해 3개의 영역을 겹쳐서 배경 처리를 해보았는데 
이렇게 만든 배경에 움직임을 더해 이동되는 효과를 처리할 수 있습니다.<br/><br/>

**<span style="color: rgb(107, 173, 222);">SpriteComponent</span>** 로 각 영역의 컴포넌트 객체에 position을 위에서 아래로 이동되도록 하여 배경의 움직임 효과 처리를 할 수 있는데 
이 처리는 **<span style="color: rgb(107, 173, 222);">SpriteComponent</span>** 의 update(double dt) 메서드에서 구현할 수 있습니다.<br/>
지난 글 에서 설명했듯이 Flame의 Game Loop 구조는 onLoad() 이후 Update() 와 Render()가 반복 호출처리 되는데 Update() 메서드는 delta ms 단위로 반복 호출 됩니다.<br/>
그렇기 때문에 update(double dt) 메서드에서 컴포넌트의 position 값 변경으로 움직임 처리를 쉽게 구현할 수 있습니다.<br/>
이 부분을 코드로 구현한다면 다음과 같이 처리할 수 있습니다.<br/>
아래 그림과 같이 position이 Vector2(0, 0) 위치에 배경 컴포넌트가 있을때<br/>
![image](https://user-images.githubusercontent.com/13028129/214759609-a0ec6ba9-3787-40b9-b6dd-7178245773bc.png)<br/><br/>

```dart
@override
void update(double dt) {
  super.update(dt);
  component.position.x = 0;
  component.position.y += (dt * 속력);
}
```

이렇게 position의 y값에 변화를 주어 움직임 처리가 가능하고 중력 기준으로 가속력 처리또한 다음 처럼 처리할 수 있습니다.<br/><br/>

```dart
// 중력값
final double gravity = 700;
// 시간
double time = 0;

@override
void update(double dt) {
  time += dt;
  component.position.x = 0;
  component.position.y += (gravity * time * time) / 2;
}
```

### 무한 움직임 처리

배경 이미지를 무한으로 움직임 처리 하는 방법은 여러가지가 있는데 그중 하나는 배경 이미지 사이즈를 스크린 사이즈의 두배 크기로 설정하고,<br/>
이미지가 스크린의 끝 부분 까지 도달 했을때 위치를 초기화 시켜서 무한으로 움직임 처리를 나타낼 수 있습니다.<br/>
![image](https://user-images.githubusercontent.com/13028129/214760229-bdd59a27-b9c7-4bf2-86d7-1a544268a356.png)<br/>
위 그림과 같은 위치에서 컴포넌트의 y값을 특정 속력만큼 계속 증가 시키고 Vector2(0, 0) 위치가 된다면 다시 초기화 하고 이를 반복합니다.<br/>
![image](https://user-images.githubusercontent.com/13028129/214760375-130e95b9-bbb2-4f8e-a24d-e3a3ef3eec73.png)<br/>
**위와 같은 조건일때 위치 초기화 (position: Vector2(0, 스크린.y)**<br/><br/>

지금까지 설명한 내용대로 겹쳐있는 3개의 배경 컴포넌트에 대해 각각 위치 설정이 가능하도록 하나의 배경 컴포넌트를 만들어 보겠습니다.<br/><br/>

**[components/background.dart]**<br/>
```dart
import 'package:flame/components.dart';
import 'package:flame/image_composition.dart';
import 'package:flame_game/main.dart';

class Background extends PositionComponent {
  late final BackgroundComponent _background01;
  late final BackgroundComponent _background02;
  late final BackgroundComponent _background03;

  double get background01X => _background01.x;
  double get background01Y => _background01.y;

  double get background02X => _background02.x;
  double get background02Y => _background02.y;

  double get background03X => _background03.x;
  double get background03Y => _background03.y;

  Background(Image spriteImg, Vector2 postion) {
    _background01 = BackgroundComponent(
        spriteImg, Vector2(290, 0), Vector2(144, 280), postion);
    _background02 = BackgroundComponent(
        spriteImg, Vector2(144, 0), Vector2(144, 280), postion);
    _background03 = BackgroundComponent(
        spriteImg, Vector2(0, 0), Vector2(144, 280), postion);

    add(_background01);
    add(_background02);
    add(_background03);
  }

  @override
  void update(double dt) {
    super.update(dt);
  }

  void setPositionBGImg01(double x, double y) {
    _background01.position.x = x;
    _background01.position.y = y;
  }

  void setPositionBGImg02(double x, double y) {
    _background02.position.x = x;
    _background02.position.y = y;
  }

  void setPositionBGImg03(double x, double y) {
    _background03.position.x = x;
    _background03.position.y = y;
  }
}

class BackgroundComponent extends SpriteComponent {
  BackgroundComponent(Image backgroundImag, Vector2 srcPosition,
      Vector2 srcSize, Vector2 postion)
      : super.fromImage(backgroundImag,
            srcPosition: srcPosition,
            srcSize: srcSize,
            position: postion,
            // 배경 이미지 사이즈를 전체 화면 세로 사이즈의 두배로 설정
            size: Vector2(
                Singleton().screenSize!.x, Singleton().screenSize!.y * 2));

  @override
  void render(Canvas canvas) {
    super.render(canvas);
  }
}
```

이렇게 만든 Background 컴포넌트를 사용해서 position 설정으로 움직임 처리를 다음과 같이 처리할 수 있습니다.<br/><br/>

**[components/my_world.dart]**<br/>
```dart
import 'package:flame/components.dart';
import 'package:flame_game/components/background.dart';
import 'package:flame_game/game/my_game.dart';
import 'package:flame_game/main.dart';

class MyWorld extends Component with HasGameRef<MyGame> {
  late Background _background;

  @override
  Future<void> onLoad() async {
    var backgroundSprites = gameRef.images.fromCache("Backgrounds.png");
    _background = Background(backgroundSprites, Vector2(0, -gameRef.size.y));
    add(_background);
  }

  @override
  void update(double dt) {
    super.update(dt);

    _background.setPositionBGImg01(
        0, _background.background01Y + (dt * 속도));

    if (_background.background01Y >= 0) {
      _background.setPositionBGImg01(0, -gameRef.size.y);
    }

    _background.setPositionBGImg02(
        0, _background.background02Y + (dt * 속도));

    if (_background.background02Y >= 0) {
      _background.setPositionBGImg02(0, -gameRef.size.y);
    }

    _background.setPositionBGImg03(
        0, _background.background03Y + (dt * 속도));

    if (_background.background03Y >= 0) {
      _background.setPositionBGImg03(0, -gameRef.size.y);
    }
  }
}
```

**[game/my_game.dart]**<br/>
```dart
late MyWorld world;

@override
onLoad() async {
  await super.onLoad();
  
  world = MyWorld();
  add(world);
}
```

**[결과 화면]**<br/>
![back_ani](https://user-images.githubusercontent.com/13028129/214761528-9474a0af-cf0d-45a8-96e8-e1f425182469.gif)


SpriteAnimationGroupComponent
-

**<span style="color: rgb(107, 173, 222);">SpriteAnimationGroupComponent</span>** 은 여러개의 **<span style="color: rgb(107, 173, 222);">SpriteAnimationComponent</span>** 를 각 상태별로 그룹 관리 할 수 있는 
컴포넌트 입니다.<br/>
앞서 구현해 보았던 플레이어의 Sprite 이미지는 직진, 좌, 우 새개의 동작 상태로 애니매이션을 표현할 수 있습니다.<br/>
![Player](https://user-images.githubusercontent.com/13028129/214761939-0bc13faf-75f0-49a2-9a18-fcee4bd69f65.png)<br/>
지난번 구현에서 **<span style="color: rgb(107, 173, 222);">SpriteAnimationComponent</span>** 를 사용해 직진 상태에 해당되는 이미지를 연속적으로 표현하여 애니매이션 처리를 하였는데 
좌, 우 도 동일하게 구현해서 그룹으로 관리하고 상황에 맞게 어떤 상태를 표시할지 기능을 제공합니다.<br/>
위 플레이어 Sprite 이미지를 이용해서 다음과 같이 **<span style="color: rgb(107, 173, 222);">SpriteAnimation</span>** 객체를 생성 합니다.<br/><br/>

```dart
var playerImage = Flame.images.fromCache("Player.png");

List<Sprite> spritesGo = [
      Sprite(playerImage, srcPosition: Vector2(0, 0), srcSize: Vector2(24, 24)),
      Sprite(playerImage,
          srcPosition: Vector2(25, 0), srcSize: Vector2(24, 24)),
      Sprite(playerImage,
          srcPosition: Vector2(50, 0), srcSize: Vector2(24, 24)),
      Sprite(playerImage,
          srcPosition: Vector2(75, 0), srcSize: Vector2(24, 24)),
];

List<Sprite> spritesLeft = [
      Sprite(playerImage,
          srcPosition: Vector2(2, 50), srcSize: Vector2(24, 24)),
      Sprite(playerImage,
          srcPosition: Vector2(27, 50), srcSize: Vector2(24, 24)),
      Sprite(playerImage,
          srcPosition: Vector2(52, 50), srcSize: Vector2(24, 24)),
      Sprite(playerImage,
          srcPosition: Vector2(77, 50), srcSize: Vector2(24, 24)),
];

List<Sprite> spritesRight = [
      Sprite(playerImage,
          srcPosition: Vector2(1, 25), srcSize: Vector2(24, 24)),
      Sprite(playerImage,
          srcPosition: Vector2(26, 25), srcSize: Vector2(24, 24)),
      Sprite(playerImage,
          srcPosition: Vector2(51, 25), srcSize: Vector2(24, 24)),
      Sprite(playerImage,
          srcPosition: Vector2(76, 25), srcSize: Vector2(24, 24)),
];

var animatedPlayer_go =
      SpriteAnimation.spriteList(spritesGo, stepTime: 0.15);
var animatedPlayer_left =
      SpriteAnimation.spriteList(spritesLeft, stepTime: 0.15);
var animatedPlayer_right =
      SpriteAnimation.spriteList(spritesRight, stepTime: 0.15);
```

> **참고**<br/>
> Flame.images.fromCache 부분은 앱 처음 시작시<br/>
> main에서 Flame.images.loadAll() 메서드로 이미지 로드 처리 후 Cache로 불러왔습니다.

그리고 다음과 같이 go, left, right에 해당되는 **<span style="color: rgb(107, 173, 222);">SpriteAnimation</span>** 객체를 **<span style="color: rgb(107, 173, 222);">SpriteAnimationGroup</span>** 컴포넌트로 처리합니다.
<br/><br/>

```dart
enum PlayerDirection { go, left, right }

final player = SpriteAnimationGroupComponent<PlayerDirection>(
  animations: {
    PlayerDirection.go: animatedPlayer_go,
    PlayerDirection.left: animatedPlayer_left,
    PlayerDirection.right: animatedPlayer_right,
  },
  current: PlayerDirection.go,
);
```

**[game/my_game.dart]**<br/>
```dart
late MyWorld world;

@override
onLoad() async {
  await super.onLoad();
  
  world = MyWorld();
  // 하단 중앙에 위치
  player = Player(position: Vector2((size[0] / 2) - 40, size[1] - 70));
  
  add(world);
  add(player);
}

이렇게 여러개의 **<span style="color: rgb(107, 173, 222);">SpriteAnimation</span>** 객체를 관리해서 current속성으로 어떤 애니매이션을 표현할지 지정할 수 있습니다.


HasDraggableComponents
-

**<span style="color: rgb(107, 173, 222);">HasDraggableComponents</span>** 컴포넌트는 드래그 액션이 필요할때 관련 이벤트를 제공해 주는 컴포넌트 입니다.<br/>
또한 반드시 **<span style="color: rgb(107, 173, 222);">FlameGame</span>** 컴포넌트에서 mixin으로 사용할 수 있습니다.<br/>
**<span style="color: rgb(107, 173, 222);">HasDraggableComponents</span>** 컴포넌트가 사용되면 드래그 액션에 대한 이벤트를 통보 받을 수 있는데 드래그 이벤트관련 처리가 필요한 
컴포넌트에 **<span style="color: rgb(107, 173, 222);">DragCallbacks</span>** mixin 해서 관련 이벤트 처리가 가능합니다.<br/><br/>

여기서는 위에서 구현한 플레이어 컴포넌트가 좌/우 로 드래그 되었을때 플레이어 컴포넌트의 애니매이션 상태를 go, left, right로 처리해보겠습니다.<br/>
제일 먼저 **<span style="color: rgb(107, 173, 222);">FlameGame</span>** 를 사용하는 게임 위젯에 **<span style="color: rgb(107, 173, 222);">HasDraggableComponents</span>** 를 mixin 시킵니다.<br/>
단순히 이 처리만으로 이제 드래그 액션 사용이 가능하게 됩니다.<br/><br/>

플레이어 컴포넌트가 드래그 대상이고 관련 이벤트 처리가 되어야 하기 때문에 플레이어 컴포넌트에 **<span style="color: rgb(107, 173, 222);">DragCallbacks</span>** 를 mixin 시킵니다.<br/>
그러면 onDragStart(DragStartEvent event), onDragUpdate(DragUpdateEvent event), onDragEnd(DragEndEvent event) 메서드들을 오버라이드해서 관련 이벤트 처리를 할 수 있습니다.<br/>
플레이어 컴포넌트가 드래그중일때 onDragUpdate(DragUpdateEvent event) 메서드에서 **<span style="color: rgb(107, 173, 222);">DragUpdateEvent</span>** 클래스로 드래그 위치 정보를 가져올 수 있기 때문에 
여기서 드래그 방향에 따른 플레이어 애니메이션 표시를 처리할 수 있습니다.<br/><br/>

```dart
@override
  void onDragUpdate(DragUpdateEvent event) {
    // 좌측 범위 초과 금지
    if (position.x <= 150) {
      position.x += 10;
      return;
    }

    // 우측 범위 초과 금지
    if (position.x >= gameRef.size[0] - 150) {
      position.x -= 10;
      return;
    }

    // 드래그 하지 않는 경우
    if (event.delta.x == 0) {
      _playerComponent.current = PlayerDirection.go;
    }
    // 우측으로 드래그중인 경우
    else if (event.delta.x > 0) {
      _playerComponent.current = PlayerDirection.right;
    }
    // 좌측으로 드래그중인 경우
    else if (event.delta.x < 0) {
      _playerComponent.current = PlayerDirection.left;
    }
    // 드래그 방향에 따라 이동
    position.x += event.delta.x;
}
```

**[결과 화면]**
![player_aniGroup](https://user-images.githubusercontent.com/13028129/214765890-2ac0c7b2-fe9f-460c-9dbb-c7d42dc8db9f.gif)<br/>

이렇게 해서 플레이어 컴포넌트의 상태별 애니메이션 처리와 드래그 액션 처리의 전체 코드 부분은 다음과 같이 구현 됩니다.<br/><br/>

**[components/player.dart]**<br/>
```dart
import 'package:flame/components.dart';
import 'package:flame/experimental.dart';
import 'package:flame/flame.dart';
import 'package:flame_game/game/my_game.dart';

enum PlayerDirection { go, left, right }

class Player extends PositionComponent with DragCallbacks, HasGameRef<MyGame> {
  late final PlayerComponent _playerComponent;
  bool _isDragging = false;

  Player({super.position})
      : super(
          size: Vector2(35, 35),
          anchor: Anchor.center,
        ) {
    var playerImage = Flame.images.fromCache("Player.png");

    List<Sprite> spritesGo = [
      Sprite(playerImage, srcPosition: Vector2(0, 0), srcSize: Vector2(24, 24)),
      Sprite(playerImage,
          srcPosition: Vector2(25, 0), srcSize: Vector2(24, 24)),
      Sprite(playerImage,
          srcPosition: Vector2(50, 0), srcSize: Vector2(24, 24)),
      Sprite(playerImage,
          srcPosition: Vector2(75, 0), srcSize: Vector2(24, 24)),
    ];

    List<Sprite> spritesLeft = [
      Sprite(playerImage,
          srcPosition: Vector2(2, 50), srcSize: Vector2(24, 24)),
      Sprite(playerImage,
          srcPosition: Vector2(27, 50), srcSize: Vector2(24, 24)),
      Sprite(playerImage,
          srcPosition: Vector2(52, 50), srcSize: Vector2(24, 24)),
      Sprite(playerImage,
          srcPosition: Vector2(77, 50), srcSize: Vector2(24, 24)),
    ];

    List<Sprite> spritesRight = [
      Sprite(playerImage,
          srcPosition: Vector2(1, 25), srcSize: Vector2(24, 24)),
      Sprite(playerImage,
          srcPosition: Vector2(26, 25), srcSize: Vector2(24, 24)),
      Sprite(playerImage,
          srcPosition: Vector2(51, 25), srcSize: Vector2(24, 24)),
      Sprite(playerImage,
          srcPosition: Vector2(76, 25), srcSize: Vector2(24, 24)),
    ];

    var animatedPlayer_go =
        SpriteAnimation.spriteList(spritesGo, stepTime: 0.15);
    var animatedPlayer_left =
        SpriteAnimation.spriteList(spritesLeft, stepTime: 0.15);
    var animatedPlayer_right =
        SpriteAnimation.spriteList(spritesRight, stepTime: 0.15);

    _playerComponent = PlayerComponent<PlayerDirection>({
      PlayerDirection.go: animatedPlayer_go,
      PlayerDirection.left: animatedPlayer_left,
      PlayerDirection.right: animatedPlayer_right
    });
    _playerComponent.current = PlayerDirection.go;
    add(_playerComponent);
  }

  @override
  void update(double dt) {
    super.update(dt);
  }

  @override
  void onDragStart(DragStartEvent event) {
    _isDragging = true;
    priority = 100;
  }

  @override
  void onDragUpdate(DragUpdateEvent event) {
    if (!_isDragging) {
      return;
    }

    // 좌측 범위 초과 금지
    if (position.x <= 150) {
      position.x += 10;
      return;
    }

    // 우측 범위 초과 금지
    if (position.x >= gameRef.size[0] - 150) {
      position.x -= 10;
      return;
    }

    // 드래그 하지 않는 경우
    if (event.delta.x == 0) {
      _playerComponent.current = PlayerDirection.go;
    }
    // 우측으로 드래그중인 경우
    else if (event.delta.x > 0) {
      _playerComponent.current = PlayerDirection.right;
    }
    // 좌측으로 드래그중인 경우
    else if (event.delta.x < 0) {
      _playerComponent.current = PlayerDirection.left;
    }
    // 드래그 방향에 따라 이동
    position.x += event.delta.x;
  }

  @override
  void onDragEnd(DragEndEvent event) {
    if (!_isDragging) {
      return;
    }

    _isDragging = false;
    _playerComponent.current = PlayerDirection.go;
  }

  void playerUpdate(PlayerDirection playerDirection) {
    _playerComponent.playerUpdate(playerDirection);
  }

  void setPosition(Vector2 position) {
    _playerComponent.position = position;
  }
}

class PlayerComponent<T> extends SpriteAnimationGroupComponent<T> {
  PlayerComponent(Map<T, SpriteAnimation> playerAnimationMap)
      : super(size: Vector2(40, 40), animations: playerAnimationMap);

  @override
  void update(double dt) {
    super.update(dt);
  }

  void playerUpdate(PlayerDirection playerDirection) {
    current = playerDirection as T?;
  }
}
```

<br/><br/>

***

지금까지 컴포넌트 애니메이션 처리와 드래그 이벤트 처리 방법에 대해 알아보았습니다.<br/>
다음에는 장애물 표현 처리와 미사일 표현 처리 등을 살펴보겠습니다.<br/><br/>

***

위 코드는 다음 Repository에서 확인할 수 있습니다.<br/>
> [Flutter_flame_game](https://github.com/tyeom/Flutter_flame_game)



{% include content_adsense.html %}
