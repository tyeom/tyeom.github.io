---
title: (Flutter) 플러터로 게임 만들어보기 03 - 몹 만들기 (with flame)
categories: Flutter
key: 20230130_01
comments: true
tags: Flutter flame game 게임 2D게임 2D 게임엔진 GameEngine overlay CollisionCallbacks hitbox
---

이번에는 게임 컨텐츠에 적을 출현하게 하고 플레이어가 적에게 닿았을때 Hit 처리 방법에 대해 알아보겠습니다.<br/>
그 전에 우선 게임 메뉴 관련 화면이 없기에 메뉴 화면을 먼저 만들어 보겠습니다.

<!--more-->

> 이 글에서 다루는 코드는 다음 Repository에서 확인할 수 있습니다.<br/>
> [Flutter_flame_game](https://github.com/tyeom/Flutter_flame_game)

Overlay
-

Flame 엔진의 **<span style="color: rgb(107, 173, 222);">GameWidget</span>** 은 기본 위젯을 따르기 때문에 Flutter의 다른 위젯과 함께 사용할 수 있는 장점이 있습니다.<br/>
게임상의 메뉴 화면도 다른 위젯으로 메뉴 화면을 꾸미고 **<span style="color: rgb(107, 173, 222);">GameWidget</span>** 의 overlayBuilderMap 속성 으로 게임 화면 위에 메뉴 위젯을 표시할 수 있습니다.<br/>
먼저 다음과 같이 메인 메뉴 화면을 추가 합니다.<br/><br/>

**[main_menu_overlay.dart]**<br/>
```dart
import 'package:flame_game/game/my_game.dart';
import 'package:flame_game/managers/game_manager.dart';
import 'package:flutter/material.dart';
import 'package:flame/game.dart';

class MainMenuOverlay extends StatefulWidget {
  final Game game;

  const MainMenuOverlay(this.game, {super.key});

  @override
  State<MainMenuOverlay> createState() => _MainMenuOverlayState();
}

class _MainMenuOverlayState extends State<MainMenuOverlay> {
  @override
  Widget build(BuildContext context) {
    MyGame game = widget.game as MyGame;

    return LayoutBuilder(builder: (_, constraints) {
      late String buttonStr;
      late double backOpacity;
      if (game.gameManager.currentState == GameState.intro) {
        backOpacity = 1;
        buttonStr = '시작하기';
      } else if (game.gameManager.currentState == GameState.pause) {
        backOpacity = 0.8;
        buttonStr = '계속하기';
      }

      return Material(
          color: Colors.transparent,
          child: Opacity(
            opacity: backOpacity,
            child: Padding(
              padding: const EdgeInsets.all(0.0),
              child: Container(
                color: Theme.of(context).colorScheme.background,
                child: Center(
                  child: SingleChildScrollView(
                    child: Column(
                      children: [
                        const Text(
                          '2D 슈팅게임',
                          style: TextStyle(
                              fontSize: 20, fontWeight: FontWeight.bold),
                          textAlign: TextAlign.center,
                        ),
                        const SizedBox(
                          height: 100,
                        ),
                        ElevatedButton(
                            onPressed: () {
                              if (game.gameManager.currentState ==
                                  GameState.intro) {
                                game.startGame();
                              } else if (game.gameManager.currentState ==
                                  GameState.pause) {
                                game.pauseAndresumeGame();
                              }
                            },
                            child: Text(
                              buttonStr,
                              style: const TextStyle(fontSize: 15),
                            )),
                      ],
                    ),
                  ),
                ),
              ),
            ),
          ));
    });
  }
}
```

이렇게 메인 메뉴에 해당하는 위젯을 만들고 **<span style="color: rgb(107, 173, 222);">GameWidget</span>** 의 overlayBuilderMap로 등록해서 사용할 수 있습니다.<br/><br/>

**[main.dart]**<br/>
```dart
class GameWrapper extends StatelessWidget {
  final MyGame myGame;
  const GameWrapper(this.myGame, {super.key});

  @override
  Widget build(BuildContext context) {
    return GameWidget(
      game: myGame,
      // overlay 위젯 등록
      overlayBuilderMap: <String, Widget Function(BuildContext, Game)>{
        'gameOverlay': (context, game) => GameOverlay(game),
        'mainMenuOverlay': (context, game) => MainMenuOverlay(game),
      },
    );
  }
}
```

등록된 overlay는 **<span style="color: rgb(107, 173, 222);">OverlayManager</span>** 클래스의 add() / remove() 메서드를 사용해 화면에 추가하고 제거할 수 있습니다.<br/>
**<span style="color: rgb(107, 173, 222);">OverlayManager</span>** 클래스는 **<span style="color: rgb(107, 173, 222);">GameWidget</span>** 클래스가 상속받고 있는 **<span style="color: rgb(107, 173, 222);">Game</span>** 클래스에서 
인스턴스를 가지고 있기 때문에 바로 다음과 같이 사용 가능합니다.<br/><br/>

```dart
// 메인 메뉴 표시
overlays.add('mainMenuOverlay');
// 메인 메뉴 제거
overlays.remove('mainMenuOverlay');
```

**[결과 화면]**<br/>
![image](https://user-images.githubusercontent.com/13028129/215414091-9ee07724-5bbe-46f5-88b3-5c67a9612c0f.png)<br/>

위와 같은 방식으로 게임진행 상태(현재 점수, 게임 정보 표시 등)도 게임 플레이 화면에 표시할 수 있습니다.<br/><br/>

**[game_overlay.dart]**<br/>
```dart
import 'package:flame/game.dart';
import 'package:flame_game/game/my_game.dart';
import 'package:flame_game/game/score_display.dart';
import 'package:flutter/material.dart';

class GameOverlay extends StatefulWidget {
  final Game game;

  const GameOverlay(this.game, {super.key});

  @override
  State<GameOverlay> createState() => _GameOverlayState();
}

class _GameOverlayState extends State<GameOverlay> {
  @override
  Widget build(BuildContext context) {
    return Material(
        color: Colors.transparent,
        child: Container(
          margin: const EdgeInsets.fromLTRB(10, 10, 10, 0),
          child: Row(
            mainAxisAlignment: MainAxisAlignment.spaceBetween,
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              ScoreDisplay(game: widget.game),
              ElevatedButton(
                onPressed: () => (widget.game as MyGame).pauseAndresumeGame(),
                child: const Icon(
                  Icons.pause,
                  size: 30,
                ),
              ),
            ],
          ),
        ));
  }
}
```

플에이어의 점수를 표시하는 ScoreDisplay 위젯을 다음과 같이 생성해서 사용했습니다.<br/><br/>

**[score_display.dart]**<br/>
```dart
import 'package:flame/game.dart';
import 'package:flame_game/game/my_game.dart';
import 'package:flutter/material.dart';

class ScoreDisplay extends StatelessWidget {
  final Game game;

  const ScoreDisplay({super.key, required this.game});

  @override
  Widget build(BuildContext context) {
    // GameManager의 ValueNotifier 값 변경시 처리
    return ValueListenableBuilder(
      valueListenable: (game as MyGame).gameManager.score,
      builder: (context, value, child) {
        return Text('Score: $value',
            style: Theme.of(context).textTheme.displaySmall!);
      },
    );
  }
}
```

그리고 현재 게임의 상태[첫 메인화면, 플레이중, 일시정지, 게임종료]와 플레이어 점수 정보를 관리할 수 있는 GameManager 클래스를 별도로 추가해서 관리하도록 했습니다.<br/><br/>

**[managers/game_manager.dart]**<br/>
```dart
import 'package:flame/components.dart';
import 'package:flame_game/game/my_game.dart';
import 'package:flutter/material.dart';

enum GameState { intro, playing, pause, gameOver }

class GameManager extends Component with HasGameRef<MyGame> {
  GameManager();

  GameState _state = GameState.intro;
  // 참고
  // ValueNotifier<T> 클래스는 값 변경시 통보되고
  // ValueListenableBuilder로 구독해서 사용할 수 있다.
  // ScoreDisplay 위젯에서 사용된다.
  ValueNotifier<int> score = ValueNotifier(0);

  GameState get currentState => _state;

  void changeState(GameState state) {
    _state = state;
  }

  void increaseScore(int point) {
    score.value += point;
  }

  void reset() {
    score.value = 0;
  }
}

```

**[결과 화면]**<br/>
![image](https://user-images.githubusercontent.com/13028129/215415337-76073b50-b6a9-4d93-bb62-c2e8f495bca2.png)


플레이어 bullet 처리
-

플레이어 컴포넌트에서 총알이 발사 되는 효과를 나타내기 위해 다음과 같은 Sprites 이미지를 사용해서 bullet 컴포넌트를 만들어 보겠습니다.<br/>
![Bullets](https://user-images.githubusercontent.com/13028129/215416129-b8c2cc03-c118-4912-bd8d-9e446945dedc.png)<br/>
지난번 배경화면 이미지 만들기에서 사용했던 **<span style="color: rgb(107, 173, 222);">SpriteComponent</span>** 클래스를 이용해서 3개의 영역을 합쳐서 총알 이미지로 다음과 같이 표시하도록 합니다.<br/><br/>

**[components/bullet.dart]**<br/>
```dart
final double _speed = 800;

Bullet() {
  var bulletsSprites = Flame.images.fromCache("Bullets.png");
  var bullet01 = SpriteComponent.fromImage(bulletsSprites,
    srcPosition: Vector2(3, 0), srcSize: Vector2(4, 7));
  var bullet02 = SpriteComponent.fromImage(bulletsSprites,
    srcPosition: Vector2(12, 0), srcSize: Vector2(11, 11));
  var bullet03 = SpriteComponent.fromImage(bulletsSprites,
    srcPosition: Vector2(3, 0), srcSize: Vector2(4, 7));

  bullet01.position = Vector2(0, 0);
  bullet02.position = Vector2(6, 0);
  bullet03.position = Vector2(17, 0);
  
  add(bullet01);
  add(bullet02);
  add(bullet03);
}
```

이렇게 하면 하나의 총알 이미지로 표현되는 컴포넌트 완성 입니다. ^0^ <br/>
여기에 플레이어 컴포넌트에서 총알이 발사 되는 효과를 추가하면 됩니다. 총알이 발사 되는 효과도 지난번 움직이는 배경처리와 같이 Position의 Y축을 이동시켜 이동되는 효과를 표현할 수 있습니다.<br/><br/>

```dart
@override
void update(double dt) {
    super.update(dt);
    
    position.y -= dt * _speed;

    // 스크린 범위 밖으로 나가면 객체 제거
    if (position.y < 0) {
      removeFromParent();
    }
}
```

간단합니다. 추가로 확인할 것은 해당 객체가(Bullet) 현재 보여지는 스크린 범위 밖으로 나가면  **<span style="color: rgb(107, 173, 222);">Component</span>** 클래스의 removeFromParent() 메서드로 해당 객체를 반드시 제거합니다.<br/>
이렇게 만들어진 Bullet컴포넌트를 지난번 만들었던 Player에 붙이면 총알 발사 효과를 나타낼 수 있습니다.<br/>
Player컴포넌트의 update() 메서드 부분에서 Bullet컴포넌트를 생성하고 메인 게임 위젯 화면에 add하면 됩니다.<br/><br/>

**[player.dart]**<br/>
```dart
double bulletTime = 0;

@override
void update(double dt) {
  super.update(dt);
  
  bulletTime += dt;
  // 0.3초 마다 한번씩 발사
  if (bulletTime < 0.3) return;
  bulletTime = 0;
  
  Bullet bullet = Bullet()
    // Bullet의 사이즈 설정
    ..size = Vector2(19, 25)
    // Bullet의 위치 설정
    ..position = position.clone()
    // Bullet의 기준점 설정
    ..anchor = Anchor.center;
  gameRef.add(bullet);
  }
```

**[결과 화면]**<br/>
![bullet](https://user-images.githubusercontent.com/13028129/215420484-b5349e12-d10a-4e18-8f25-a64c67204098.gif)<br/><br/>

추가로 Bullet이 몹에 맞았을때 몹 hp가 얼마나 깎일지에 대한 Power정보를 GameManager 클래스에서 관리할 수 있도록 Bullet power정보를 추가하였습니다.<br/><br/>

**[managers/game_manager.dart]**<br/>
```dart
import 'package:flame/components.dart';
import 'package:flame_game/game/my_game.dart';
import 'package:flutter/material.dart';

enum GameState { intro, playing, pause, gameOver }

class GameManager extends Component with HasGameRef<MyGame> {
  GameManager();

  GameState _state = GameState.intro;
  // 참고
  // ValueNotifier<T> 클래스는 값 변경시 통보되고
  // ValueListenableBuilder로 구독해서 사용할 수 있다.
  // ScoreDisplay 위젯에서 사용된다.
  ValueNotifier<int> score = ValueNotifier(0);
  // bullet power 값
  int _bulletPowerPoint = 10;
  int get bulletPowerPoint => _bulletPowerPoint;

  GameState get currentState => _state;

  void changeState(GameState state) {
    _state = state;
  }

  void increaseScore(int point) {
    score.value += point;
  }

  // bullet power 값 변경
  void upgradePowrerPoint(int point) {
    _bulletPowerPoint = point;
  }

  void reset() {
    score.value = 0;
    // bullet power reset 추가
    _bulletPowerPoint = 10;
  }
}
```

이렇게 추가된 bulletPowerPoint는 추후에 플레이어가 특정 아이템을 먹었을때 power를 업그레이드 시킬때 사용될 수 있습니다.


몹(적) 만들기 및 CollisionCallbacks 처리
-

몹은 몹의 특성에 따라 능력치가 각각 다를 수 있고 플레이어가 그 몹을 잡았을때 획득할 수 있는 점수 또한 다르게 처리 해야하는 상황이 있을 수 있습니다.<br/>
이런 상황을 고려해서 추상화 하여 공통으로 처리할 수 있는 부분과 특정한 몹에 해당되는 부분을 분리하여 설계하는 것이 좋습니다.<br/>
우선 몹의 능력치 정보에 대한 모델 클래스를 다음과 같이 만들었습니다.<br/><br/>

**models/enemy_model.dart**<br/>
```dart
class EnemyModel {
  final double speed;
  final bool isHMove;
  final int killPoint;
  final int level;
  final bool isBoss;

  const EnemyModel({
    required this.speed,
    required this.isHMove,
    required this.killPoint,
    required this.level,
    required this.isBoss,
  });
}
```

해당 몹의 속도, 좌우 이동 가능 여부, 플레이어가 잡았을때 획득 점수, 레벨 등 정보를 가지고 있는 모델 입니다.<br/>
이렇게 만든 EnemyModel을 이용하는 Enemy의 추상 클래스를 작성해 봅니다. Enemy 추상 클래스는 각 몹들의 공통 특성과 기능을 담당하게 됩니다.<br/><br/>

**[components/enemy.dart]**<br/>
```dart
abstract class Enemy extends SpriteGroupComponent
    with CollisionCallbacks, HasGameRef<MyGame> {
  final _random = Random();
  double _speed = 250;
  Vector2 _moveDirection = Vector2(0, 1);
  EnemyModel enemyData;
  int _hitPoints = 10;

  Enemy(this.enemyData) {
    angle = pi;
    _speed = enemyData.speed;
    _hitPoints = (enemyData.level * 10);

    if (enemyData.isHMove) {
      _moveDirection = getRandomDirection();
    }
  }

  @override
  void onMount() {
    super.onMount();
    
    // enemy 객체 사이즈의 반지름 0.8배 작은 원형 히트박스 추가
    final shape = CircleHitbox.relative(
      0.8,
      parentSize: size,
      position: size / 2,
      anchor: Anchor.center,
    );
    add(shape);
  }

  @override
  void update(double dt) {
    super.update(dt);

    // hp가 0이면 제거
    if (_hitPoints <= 0) {
      destroy();
    }

    // 기본 Sprite 으로 변경
    if (current == 2) current = 1;

    position += _moveDirection * _speed * dt;

    // 적이 boss 가 아니라면 스크린 세로 사이즈 범위 밖으로 사라지면 객체 제거
    if (enemyData.isBoss == false && position.y > gameRef.size.y) {
      removeFromParent();
    }
    // 적이 boss 라면 세로 사이즈 범위 밖으로 사라졌을때 다시 y = 0 으로 변경
    else if (enemyData.isBoss && position.y > gameRef.size.y) {
      position = Vector2(_random.nextDouble() * gameRef.size.x, 0);
    }
    // 스크린 가로 사이즈 범위 밖으로 사라지지 않도록 x 방향 반전
    else if (enemyData.isBoss &&
        ((position.x < size.x / 2) ||
            (position.x > (gameRef.size.x - size.x / 2)))) {
      _moveDirection.x *= -1;
    }
    // 스크린 가로 사이즈 범위 밖으로 사라지지 않도록 x 방향 반전
    else if ((position.x < size.x / 2) ||
        (position.x > (gameRef.size.x - size.x / 2))) {
      _moveDirection.x *= -1;
    }
  }

  @override
  void onCollision(Set<Vector2> intersectionPoints, PositionComponent other) {
    super.onCollision(intersectionPoints, other);

    // Bullet과 충돌시
    if (other is Bullet) {
      // hp 감소처리
      _hitPoints -= gameRef.gameManager.bulletPowerPoint;
      current = 2;
    }
    // 플레이어와 충돌시 게임오버
    else if (other is Player) {
      destroy(isGameOver: true);
      gameRef.gameOver();
    }
  }

  Vector2 getRandomDirection() {
    return (Vector2.random(_random) - Vector2(0.5, -1)).normalized();
  }

  void destroy({bool isGameOver = false}) {
    // 객체 제거
    removeFromParent();

    if (isGameOver) {
      return;
    }

    gameRef.gameManager.increaseScore(enemyData.killPoint);

    // boss 처리 완료시 게임오버
    if (enemyData.isBoss) {
      gameRef.gameOver();
    }
  }
}
```

**<span style="color: rgb(107, 173, 222);">SpriteGroupComponent</span>** 클래스와 **<span style="color: rgb(107, 173, 222);">CollisionCallbacks</span>** 클래스가 mixin 되어 있는데 
몹의 기본 상태 이미지와 몹이 총알에 맞았을때 상태 이미지 두개의 Sprite로 처리하기 위해서 **<span style="color: rgb(107, 173, 222);">SpriteGroupComponent</span>** 클래스를 사용합니다.<br/>
**<span style="color: rgb(107, 173, 222);">CollisionCallbacks</span>** mixin 클래스는 해당 컴포넌트가 다른 컴포넌트의 **<span style="color: rgb(107, 173, 222);">Hitbox&lt;T&gt;</span>** 에 닿았을때 즉 서로 부딛혔을때 
Callback를 제공해주는 클래스 입니다. 이 처리는 잠시 후 자세하게 살펴 보겠습니다.<br/><br/>

먼저 몹의 특성 정보를 알 수 있는 위에서 만들었던 EnemyModel 클래스를 생성자로 받고 기본적인 속도, hp(level * 10) 를 설정합니다.<br/>
그리고 몹의 움직임 방향 정보를 이용해서 update(double dt) 메서드에서 해당 몹이 가지고 있는 속도만큼 이동되도록 처리합니다.<br/><br/>

```dart
final _random = Random();
double _speed = 250;
Vector2 _moveDirection = Vector2(0, 1);
EnemyModel enemyData;
int _hitPoints = 10;

Enemy(this.enemyData) {
  angle = pi;
  _speed = enemyData.speed;
  _hitPoints = (enemyData.level * 10);
  
  // 좌.우 움직임 가능한 경우 X 시작지점 랜덤으로 초기화
  if (enemyData.isHMove) {
    _moveDirection = getRandomDirection();
  }
}
  
@override
void update(double dt) {
  super.update(dt);
  ...[중간 생략]...
  
  position += _moveDirection * _speed * dt;
  
  ...[중간 생략]...
}

Vector2 getRandomDirection() {
  return (Vector2.random(_random) - Vector2(0.5, -1)).normalized();
}
```

위 코드에서 처럼 현재 위치에서 _moveDirection 값 기준으로 컴포넌트를 이동시키는데 만약 몹의 특성중 좌.우로 움직일 수 있다면 랜덤으로 X 시작지점을 정해서 _moveDirection를 초기화 합니다.<br/><br/>

또한 몹이 좌.우 스크린 범위로 사라지지 않도록 좌.우 끝 위치까지 이동 됬다면 다시 반전 시켜 범위를 벗어나지 않도록 처리하고, Y축 아래쪽 끝 스크린 범위 밖으로 사라졌다면 
사라진 객체는 **<span style="color: rgb(107, 173, 222);">Component</span>** 클래스의 removeFromParent() 메서드로 제거하도록 합니다.<br/>
다만 몹이 boss 인 경우 다시 시작 지점인 Y축을 0으로 설정해서 다시 나타나도록 분기 처리 합니다. 그 밖에도 hp(**_hitPoints**)가 0이 되었을때도 제거 하도록 처리 합니다.<br/><br/>

```dart
@override
void update(double dt) {
  super.update(dt);
  
  // hp가 0이면 제거
  if (_hitPoints <= 0) {
    destroy();
  }
  
  ...[중간 생략]...
  
  // 적이 boss 가 아니라면 스크린 세로 사이즈 범위 밖으로 사라지면 객체 제거
  if (enemyData.isBoss == false && position.y > gameRef.size.y) {
    removeFromParent();
  }
  // 적이 boss 라면 세로 사이즈 범위 밖으로 사라졌을때 다시 y = 0 으로 변경
  else if (enemyData.isBoss && position.y > gameRef.size.y) {
    position = Vector2(_random.nextDouble() * gameRef.size.x, 0);
  }
  // 스크린 가로 사이즈 범위 밖으로 사라지지 않도록 x 방향 반전
  else if (enemyData.isBoss &&
      ((position.x < size.x / 2) || (position.x > (gameRef.size.x - size.x / 2)))) {
    _moveDirection.x *= -1;
  }
  // 스크린 가로 사이즈 범위 밖으로 사라지지 않도록 x 방향 반전
  else if ((position.x < size.x / 2) || (position.x > (gameRef.size.x - size.x / 2))) {
    _moveDirection.x *= -1;
  }
}
```

이렇게 Enemy 추상 클래스가 구현 되었으면 실제 몹에 해당 되는 각각의 몹 클래스를 구현합니다.<br/>
먼저 Enemy Sprites 이미지는 다음과 같습니다.<br/>
![Enemies](https://user-images.githubusercontent.com/13028129/215644662-46aaf4ff-5f4a-4eef-9ca9-eb99cf35e2eb.png)<br/>
모두 4종류고 위 이미지를 이용해서 각각의 Enemy를 구현합니다.<br/><br/>

**[components/enemy.dart]**<br/>
```dart
class NormalEnemy01 extends Enemy {
  NormalEnemy01()
      // enemy 정보
      : super(const EnemyModel(
            speed: 200,
            isHMove: false,
            killPoint: 1,
            level: 1,
            isBoss: false)) {
    size = Vector2(20, 17);
  }

  @override
  Future<void>? onLoad() async {
    // 기본 이미지
    var enemySprite = await gameRef.loadSprite("Enemies.png",
        srcPosition: Vector2(125, 67), srcSize: Vector2(12, 11));
    // hit 되었을때 이미지
    var enemyHitSprite = await gameRef.loadSprite("Enemies.png",
        srcPosition: Vector2(125, 95), srcSize: Vector2(12, 11));

    sprites = <int, Sprite>{
      // 기본 이미지
      1: enemySprite,
      // hit 이미지
      2: enemyHitSprite,
    };

    current = 1;

    return super.onLoad();
  }
}
```

Enemy 추상 클래스를 확장시키고 해당 고유의 Enemy 특성이 담긴 EnemyModel 클래스를 초기화 합니다.<br/>
> 속도는 200, 좌.우 움직이지 못함, 해당 몹을 죽였을때 획득하는 점수 1point, 해당 몹의 hp(level * 10)

그리고 기본 상태일때 이미지와 총알에 맞았을때 이미지 두개의 **<span style="color: rgb(107, 173, 222);">Sprite</span>** 를 **<span style="color: rgb(107, 173, 222);">SpriteGroupComponent</span>** 의 sprites로 지정합니다.<br/>
마찬가지로 위와 같이 여러 형태로 각각의 Enemy를 구현해볼 수 있습니다.<br/><br/>

이렇게 만들어진 Enemy 컴포넌트를 게임상에 랜덤한 위치로 부터 출현 되도록 처리 하면 됩니다.<br/>
이를 구현하기 위해 Enemy 컴포넌트를 관리하기 위한 EnemyManager 컴포넌트를 구현해 보겠습니다.<br/>
일단 전체 코드는 다음과 같습니다.<br/><br/>

**[managers/enemy_manager.dart]**<br/>
```dart
import 'dart:math';
import 'package:flame/components.dart';
import 'package:flame_game/components/enemy.dart';
import 'package:flame_game/game/my_game.dart';

final Random _rand = Random();

class EnemyManager extends Component with HasGameRef<MyGame> {
  late Timer _enemyTimer;
  final Random _random = Random();
  bool isBossDisplay = false;

  EnemyManager() : super() {
    _enemyTimer = Timer(1, onTick: _enemyTick, repeat: true);
  }

  @override
  void onMount() {
    super.onMount();
    _enemyTimer.start();
  }

  @override
  void onRemove() {
    super.onRemove();
    _enemyTimer.stop();
  }

  @override
  void update(double dt) {
    super.update(dt);
    _enemyTimer.update(dt);
  }

  void _enemyTick() {
    if (gameRef.buildContext == null) return;

    // 0 ~ 1 사이 난수 생성 * 스크린 너비
    // 스크린 너비 사이즈 만큼 랜덤 X 위치
    Vector2 position = Vector2(_random.nextDouble() * gameRef.size.x, 0);

    int currentScore = gameRef.gameManager.score.value;
    int level = 1;
    if (isBossDisplay) {
      // 레벨5 boss는 1개만 나오도록 처리
      // boss는 출현시 나머지 적 레벨별로 랜덤 출현
      level = _random.nextInt(3) + 1;
    } else {
      level = getLevel(currentScore);
    }

    // 해당 level에 맞는 랜덤 enumy 추출
    late Enemy enemy;
    switch (level) {
      case 1:
        // level1에 해당되는 Enemy가 2개 라서 랜덤으로 생성
        enemy = (_random.nextBool()) ? NormalEnemy01() : NormalEnemy02();
        break;
      case 2:
        enemy = NormalEnemy02();
        break;
      case 3:
        enemy = NormalEnemy03();
        break;
      case 4:
        enemy = NormalEnemy04();
        break;
      case 5:
        isBossDisplay = true;
        enemy = BossEnemy();
        break;
    }

    // Enemy 컴포넌트가 화면안에 유지 되도록 고정
    position.clamp(
      Vector2.zero() + enemy.size / 2,
      gameRef.size - enemy.size / 2,
    );

    enemy.position = position;
    enemy.anchor = Anchor.center;
    gameRef.add(enemy);
  }

  void reset() {
    _enemyTimer.start();
  }

  int getLevel(int score) {
    int level = 1;

    if (score > 40) {
      level = 5;
    } else if (score > 30) {
      level = 4;
    } else if (score > 20) {
      level = 3;
    } else if (score > 10) {
      level = 2;
    }

    return level;
  }

  void destroy() {
    _enemyTimer.stop();
    removeFromParent();
  }
}
```

해당 코드를 살펴보면 **<span style="color: rgb(107, 173, 222);">Timer</span>** 를 사용해서 1초 마다 Enemy 컴포넌트를 생성하고, 스크린 너비의 범위 내에 위치를 설정합니다.<br/>
그리고 score(점수) 별로 어떤 Enemy 컴포넌트를 생성할지 분기 되어 있습니다. 이렇게 만든 EnemyManager 컴포넌트를 메인 **<span style="color: rgb(107, 173, 222);">FlameGame</span>** 컴포넌트에 추가 하면 됩니다.<br/><br/>

**[game/my_game.dart]**<br/>
```dart
@override
onLoad() async {
  await super.onLoad();
  
  world = MyWorld();
  // 하단 중앙에 위치
  player = Player(position: Vector2((size[0] / 2) - 40, size[1] - 70));
  EnemyManager enemyManager = EnemyManager();
  
  add(world);
  add(player);
  add(enemyManager);
}
```


### HasCollisionDetection

flame은 컴포넌트가 서로 닿았을때 처리를 **<span style="color: rgb(107, 173, 222);">Hitbox&lt;T&gt;</span>** 를 사용해서 서로 닿았을때를 Callback 형태로 전달해서 처리할 수 있도록 지원합니다.<br/>
여기서는 Player 컴포넌트가 Enemy 컴포넌트에 닿았을때, Enemy 컴포넌트가 Bullet 컴포넌트에 닿았을때 처리를 구현해 보겠습니다.<br/>
그러기 위해서 먼저 메인 FlameGame 컴포넌트에 **<span style="color: rgb(107, 173, 222);">HasCollisionDetection</span>** mixin 클래스를 추가 합니다.<br/><br/>

**[game/my_game.dart]**<br/>
```dart
// HasCollisionDetection mixin 클래스 추가
class MyGame extends FlameGame
    with HasTappableComponents, HasDraggableComponents, HasCollisionDetection
```

그리고 충돌 처리가 필요한 컴포넌트에 **<span style="color: rgb(107, 173, 222);">CollisionCallbacks</span>** mixin 클래스를 사용 합니다.<br/>
여기서는 Enemy 컴포넌트 / Player 컴포넌트 / Bullet 컴포넌트 에서 사용 합니다.<br/><br/>

충돌 감지는 **<span style="color: rgb(107, 173, 222);">Hitbox&lt;T&gt;</span>** 를 사용하는데 OnLoaded() 또는 onMount()에 **<span style="color: rgb(107, 173, 222);">Hitbox&lt;T&gt;</span>** 를 생성하고 추가합니다.<br/>
**<span style="color: rgb(107, 173, 222);">Hitbox&lt;T&gt;</span>** 의 종류는 **<span style="color: rgb(107, 173, 222);">CircleHitbox</span>**, **<span style="color: rgb(107, 173, 222);">RectangleHitbox</span>**, 그리고 다양한 모양으로 만들 수 있는 **<span style="color: rgb(107, 173, 222);">PolygonHitbox</span>** 가 있습니다.<br/>
특별한 설정 없이 바로 **<span style="color: rgb(107, 173, 222);">Hitbox&lt;T&gt;</span>** 를 추가하면 해당 컴포넌트의 사이즈만큼 추가 되며 여기서 enemy 컴포넌트의 hitbox는 반지름 0.8배의 작은 원형인 hitbox를 사용하겠습니다.<br/>
이유는 hitbox를 크게 하면 Bullet이 조금만 닿아도 쉽게 맞을 수 있기 때문입니다.<br/><br/>

**[components/enemy.dart]**<br/>
```dart
@override
void onMount() {
  super.onMount();
  
  // enemy 객체 사이즈의 반지름 0.8배 작은 원형 히트박스 추가
  final shape = CircleHitbox.relative(
    0.8,
    parentSize: size,
    position: size / 2,
    anchor: Anchor.center,
  );
  add(shape);
}
```

이렇게 **<span style="color: rgb(107, 173, 222);">CollisionCallbacks</span>** mixin 클래스를 추가 하면 onCollision(Setx&lt;Vector2&gt; intersectionPoints, PositionComponent other) 메서드를 override해서 
충돌 감지에 대한 Callback을 처리할 수 있습니다.<br/><br/>

```dart
@override
void onCollision(Set<Vector2> intersectionPoints, PositionComponent other) {
  super.onCollision(intersectionPoints, other);

  // Bullet과 충돌시
  if (other is Bullet) {
    // hp 감소처리
    _hitPoints -= gameRef.gameManager.bulletPowerPoint;
    // 총알에 맞았을때 Sprite 변경
    current = 2;
  }
  // 플레이어와 충돌시 게임오버
  else if (other is Player) {
    destroy(isGameOver: true);
    gameRef.gameOver();
  }
}
```

위와 같이 onCollision(Setx&lt;Vector2&gt; intersectionPoints, PositionComponent other) 메서드 other 파라메터로 충돌 감지된 컴포넌트 객체를 알 수 있습니다.<br/><br/>

추가로 **Bullet 컴포넌트**와 **Player 컴포넌트**에서도 Enemy 컴포넌트와 충돌되었을때를 다음과 같이 구현합니다.<br/><br/>
**[components/player.dart]**<br/>
```dart
@override
void onCollision(Set<Vector2> intersectionPoints, PositionComponent other) {
  super.onCollision(intersectionPoints, other);
  
  // Enemy 충돌시
  if (other is Enemy) {
    _playerComponent.current = PlayerDirection.boom;
  }
}
```

**[components/bullet.dart]**<br/>
```dart
@override
void onCollision(Set<Vector2> intersectionPoints, PositionComponent other) {
  super.onCollision(intersectionPoints, other);
  
  // Enemy 충돌시
  if (other is Enemy) {
    destroy();
  }
}

void destroy() {
  removeFromParent();
}
```

Player 컴포넌트에서는 Enemy 컴포넌트와 충돌시 효과를 다른 상태의 **<span style="color: rgb(107, 173, 222);">SpriteAnimationComponent</span>** 가 보여지게 하기 위해 
앞서 구현했던 PlayerDirection enum에 boom 상수를 추가하고 그 상태에 따른 **<span style="color: rgb(107, 173, 222);">SpriteAnimation</span>** 을 추가 하였습니다.<br/>
해당 Sprites 이미지는 다음과 같습니다.<br/>
![Boom](https://user-images.githubusercontent.com/13028129/215652697-96d22dd9-46e3-4879-a295-9adae675f8c2.png)<br/><br/>

**[components/player.dart]**<br/>
```dart
enum PlayerDirection { go, left, right, boom }

class Player extends PositionComponent
    with DragCallbacks, CollisionCallbacks, HasGameRef<MyGame> {
  late final PlayerComponent _playerComponent;
  bool _isDragging = false;

  Player({super.position})
      : super(
          size: Vector2(35, 35),
          anchor: Anchor.center,
        ) {
    var playerImage = Flame.images.fromCache("Player.png");
    var boomImage = Flame.images.fromCache("Boom.png");
    
    ...[중간 생략]...
    
    List<Sprite> spriteBoom = [
      Sprite(boomImage, srcPosition: Vector2(0, 0), srcSize: Vector2(72, 72)),
      Sprite(boomImage, srcPosition: Vector2(74, 0), srcSize: Vector2(72, 72)),
      Sprite(boomImage, srcPosition: Vector2(146, 0), srcSize: Vector2(72, 72)),
    ];
    
    ...[중간 생략]...
    
    var animatedBoom = SpriteAnimation.spriteList(spriteBoom, stepTime: 0.15, loop: false);
    // 플레이어가 죽고 Boom 애니메이션 종료 후 객체 제거
    animatedBoom.onComplete = () => destroy();

    _playerComponent = PlayerComponent<PlayerDirection>({
      PlayerDirection.go: animatedPlayer_go,
      PlayerDirection.left: animatedPlayer_left,
      PlayerDirection.right: animatedPlayer_right,
      PlayerDirection.boom: animatedBoom,
    });
    _playerComponent.current = PlayerDirection.go;
    add(_playerComponent);
  }
  
  void destroy() {
    removeFromParent();
  }
}
```

Player의 **<span style="color: rgb(107, 173, 222);">SpriteAnimation</span>** 이 boom 으로 표시 되면 onComplete 으로 애니매이션이 완료 된 후 destroy() 메서드 호출로 객체가 제거 되어 집니다.


Rendering - Particles
-

Flame은 기본적인 Particles 처리를 제공하는데 여러 입자에 속도와 가속도를 주어 튀기는듯(?) 한 효과를 나타낼 수 있습니다.<br/>
몹의 hp가 0이 되고 사라질때 이러한 효과를 주어 처리해 보겠습니다.<br/>
Enemy 추상 클래스에서 몹이 제거될때 destroy() 메서드안에서 객체가 제거 되는데 이때 **<span style="color: rgb(107, 173, 222);">ParticleSystemComponent</span>** 를 화면에 추가해서 
효과를 나타낼 수 있습니다.<br/><br/>

```dart
Vector2 getRandomVector() {
  return (Vector2.random(_random) - Vector2.random(_random)) * 500;
}
  
void destroy({bool isGameOver = false}) {
  ...[중간 생략]...

  // 적이 사라질때 효과

  // Particle 효과 추가
  // 0.1초 동안 유효한 20개의 흰색 원을 생성
  // Particle 속도를 랜덤으로 설정
  final particleComponent = ParticleSystemComponent(
    particle: Particle.generate(
      count: 20,
      lifespan: 0.1,
      generator: (i) => AcceleratedParticle(
      acceleration: getRandomVector(),
          speed: getRandomVector(),
          position: position.clone(),
          child: CircleParticle(
            radius: 2,
            paint: Paint()..color = Colors.white,
          ),
        ),
      ),
    );

    gameRef.add(particleComponent);

  ...[중간 생략]...
}
```

이렇게 객체가 사라진 후 **<span style="color: rgb(107, 173, 222);">ParticleSystemComponent</span>** 를 사용해서 20개의 **<span style="color: rgb(107, 173, 222);">AcceleratedParticle</span>** 가속도 속성이 있는 애니매이션 효과 Particles을 만들어 내고 랜덤한 
가속력과 속도로 0.1초 이후에 사라지는 효과인 컴포넌트를 생성하고 추가 합니다.<br/>


아이템
-

게임상에 Player가 아이템을 먹고 특정 능력치 부여 등의 처리도 지금까지의 처리 과정과 동일하게 **<span style="color: rgb(107, 173, 222);">CollisionCallbacks</span>** mixin 클래스 사용으로 Player 컴포넌트가 특정 아이템을 먹었을때 능력치 향상 처리 등을 구현할 수 있을 것입니다.<br/>
이 부분은 위 과정과 동일하기 때문에 전체 코드만 작성하고 추가 설명은 없어도 될것으로 보입니다.<br/>
아래 코드에서 사용된  Sprites 이미지는 다음과 같습니다.<br/>
![Items](https://user-images.githubusercontent.com/13028129/215659670-feb8af01-3ef9-4ad7-b016-75e254c740ed.png)<br/><br/>

**[components/item.dart]**<br/>
```dart
import 'package:flame/collisions.dart';
import 'package:flame/components.dart';
import 'package:flame_game/components/player.dart';
import 'package:flame_game/game/my_game.dart';

abstract class Item extends SpriteComponent
    with CollisionCallbacks, HasGameRef<MyGame> {
  int bulletPowerPoint = 10;
  final double _speed = 150;
  final Vector2 _moveDirection = Vector2(0, 1);

  @override
  void onMount() {
    super.onMount();

    // enemy 객체 사이즈의 반지름 0.8배 작은 원형 히트박스 추가
    final shape = CircleHitbox.relative(
      0.8,
      parentSize: size,
      position: size / 2,
      anchor: Anchor.center,
    );
    add(shape);
  }

  @override
  void update(double dt) {
    super.update(dt);

    position += _moveDirection * _speed * dt;

    // 스크린 세로 사이즈 범위 밖으로 사라지면 객체 제거
    if (position.y > gameRef.size.y) {
      destroy();
    }
  }

  @override
  void onCollision(Set<Vector2> intersectionPoints, PositionComponent other) {
    super.onCollision(intersectionPoints, other);

    // 플레이어와 충돌시 게임오버
    if (other is Player) {
      destroy();

      // 파워 업 아이템
      if (this is PowerUpgradeItem) {
        gameRef.gameManager.upgradePowrerPoint(bulletPowerPoint);
      }
    }
  }

  void destroy() {
    // 객체 제거
    removeFromParent();
  }
}

class PowerUpgradeItem extends Item {
  PowerUpgradeItem() {
    size = Vector2(20, 20);
    // 파워 +5
    bulletPowerPoint += 5;
  }

  @override
  Future<void>? onLoad() async {
    // 기본 이미지
    var powerUpgradeItemSprite = await gameRef.loadSprite("Items.png",
        srcPosition: Vector2(68, 2), srcSize: Vector2(16, 12));
    sprite = powerUpgradeItemSprite;
  }
}
```

이제 게임상 랜덤하게 특정 조건하에 Item이 노출 되도록 처리하면 됩니다. 이를 위해서 ItemManager 컴포넌트를 만들어 관리 되도록 하겠습니다.<br/><br/>

**[managers/item_manager.dart]**<br/>
```dart
import 'dart:math';
import 'package:flame/components.dart';
import 'package:flame_game/components/item.dart';
import 'package:flame_game/game/my_game.dart';

final Random _rand = Random();

class ItemManager extends Component with HasGameRef<MyGame> {
  late Timer _itemTimer;
  final Random _random = Random();

  ItemManager() : super() {
    _itemTimer = Timer(15, onTick: _enemyTick, repeat: true);
  }

  @override
  void onMount() {
    super.onMount();
    _itemTimer.start();
  }

  @override
  void onRemove() {
    super.onRemove();
    _itemTimer.stop();
  }

  @override
  void update(double dt) {
    super.update(dt);
    _itemTimer.update(dt);
  }

  void _enemyTick() {
    if (gameRef.buildContext == null) return;

    // powerup 아이템은 최대 power 20이 되면 표시 안되도록
    if (gameRef.gameManager.bulletPowerPoint < 20) {
      // 0 ~ 1 사이 난수 생성 * 스크린 너비
      // 스크린 너비 사이즈 만큼 랜덤 X 위치
      Vector2 position = Vector2(_random.nextDouble() * gameRef.size.x, 0);

      Item powerUpgradeItem = PowerUpgradeItem();

      // 컴포넌트가 화면안에 유지 되도록 고정
      position.clamp(
        Vector2.zero() + powerUpgradeItem.size / 2,
        gameRef.size - powerUpgradeItem.size / 2,
      );

      powerUpgradeItem.anchor = Anchor.center;
      powerUpgradeItem.position = position;
      gameRef.add(powerUpgradeItem);
    }
  }

  void reset() {
    _itemTimer.start();
  }

  void destroy() {
    _itemTimer.stop();
    removeFromParent();
  }
}
```

15초 마다 랜덤한 위치에 아이템이 표시 되도록 구현하고 이렇게 만든 ItemManager 컴포넌트는 메인 **<span style="color: rgb(107, 173, 222);">FlameGame</span>** 컴포넌트에 추가 합니다.<br/><br/>

**[game/my_game.dart]**<br/>
```dart
@override
onLoad() async {
  await super.onLoad();
  
  world = MyWorld();
  // 하단 중앙에 위치
  player = Player(position: Vector2((size[0] / 2) - 40, size[1] - 70));
  EnemyManager enemyManager = EnemyManager();
  ItemManager itemManager = ItemManager();
  
  add(world);
  add(player);
  add(enemyManager);
  add(itemManager);
}
```

이렇게 구현된 지금까지의 최종 화면은 다음과 같습니다.<br/><br/>


***

이번엔 조금 많은 작업이 되었는데 지금까지 게임 메뉴 화면 처리 부분, 그리고 다양한 몹에 대한 공통 처리 부분을 추상화 하고 랜덤으로 배치하여 출현되도록 처리하는 부분과 
Hitbox를 사용해서 컴포넌트 충돌 감지 처리, 간단한 파티클 효과 까지 알아보았습니다.<br/>
다음에는 마지막 몹의 bullet 처리에 대해 알아보겠습니다. <br/><br/>

![777](https://user-images.githubusercontent.com/13028129/215655584-3612897c-6cf6-48cb-9547-f8468ffe57ee.gif)<br/><br/>

***


위 코드는 다음 Repository에서 확인할 수 있습니다.<br/>
> [Flutter_flame_game](https://github.com/tyeom/Flutter_flame_game)



{% include content_adsense.html %}
