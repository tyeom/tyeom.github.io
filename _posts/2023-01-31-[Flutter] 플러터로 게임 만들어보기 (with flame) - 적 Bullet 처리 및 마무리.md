---
title: (Flutter) 플러터로 게임 만들어보기 04 - 적 Bullet 처리 및 마무리 (with flame)
categories: Flutter
key: 20230131_01
comments: true
tags: Flutter flame game 게임 2D게임 2D 게임엔진 GameEngine SpriteAnimation SpriteComponent CollisionCallbacks hitbox
---

플러터로 게임 만들어 보기 마지막 포스팅 입니다.<br/>

이번 글은 지난 번 처리했던 몹에 Bullet를 추가해서 각 몹에 맞는 Bullet가 표시 되도록 처리해 보겠습니다.

<!--more-->

> 이 글에서 다루는 코드는 다음 Repository에서 확인할 수 있습니다.<br/>
> [Flutter_flame_game](https://github.com/tyeom/Flutter_flame_game)

적의 Bullet 처리
-

이번 처리는 지난번 Bullet 컴포넌트를 만들고 Player 컴포넌트에 추가 했던 것과 동일 합니다.<br/>
EnemyBullet 컴포넌트를 만들고 각각의 Enemy 컴포넌트에 추가하면 끝 입니다. Bullet의 종류별로 각 Enemy 컴포넌트에서 사용 될 수 있도록 Enemy 처리와 동일하게 
각각의 EnemyBullet을 구현해서 처리해보겠습니다.<br/>
사용된 Sprites 이미지는 다음과 같습니다.<br/>
![Bullets](https://user-images.githubusercontent.com/13028129/215682153-532f1ed9-3833-4ca4-b768-cfe9ba115007.png)<br/>
**[components/enemies_bullet.dart]**<br/>
```dart
import 'package:flame/collisions.dart';
import 'package:flame/components.dart';
import 'package:flame_game/components/player.dart';
import 'package:flame_game/game/my_game.dart';

abstract class EnemiesBullet extends PositionComponent
    with CollisionCallbacks, HasGameRef<MyGame> {
  final double _speed = 300;

  @override
  void onMount() {
    super.onMount();

    // player 객체 사이즈의 반지름 0.8배 작은 원형 히트박스 추가
    final shape = CircleHitbox.relative(
      0.8,
      parentSize: size,
      position: size / 2,
      anchor: Anchor.center,
    );
    add(shape);
  }

  @override
  void onCollision(Set<Vector2> intersectionPoints, PositionComponent other) {
    super.onCollision(intersectionPoints, other);

    // Player 충돌시
    if (other is Player) {
      destroy();
    }
  }

  @override
  void update(double dt) {
    super.update(dt);

    position.y += dt * _speed;

    // 스크린 세로 사이즈 범위 밖으로 사라지면 객체 제거
    if (position.y > gameRef.size.y) {
      destroy();
    }
  }

  void destroy() {
    removeFromParent();
  }
}

class EnemyBullet01 extends EnemiesBullet {
  EnemyBullet01() {
    size = Vector2(5, 13);
  }

  @override
  Future<void>? onLoad() async {
    var bullerImage = await gameRef.images.load("Bullets.png");
    var bullet = SpriteComponent.fromImage(bullerImage,
        srcPosition: Vector2(58, 0), srcSize: Vector2(4, 12));

    add(bullet);
  }
}

class EnemyBullet02 extends EnemiesBullet {
  EnemyBullet02() {
    size = Vector2(7, 10);
  }

  @override
  Future<void>? onLoad() async {
    var bullerImage = await gameRef.images.load("Bullets.png");
    var bullet = SpriteComponent.fromImage(bullerImage,
        srcPosition: Vector2(35, 0), srcSize: Vector2(6, 9));

    add(bullet);
  }
}

class EnemyBullet03 extends EnemiesBullet {
  EnemyBullet03() {
    size = Vector2(11, 11);
  }

  @override
  Future<void>? onLoad() async {
    var bullerImage = await gameRef.images.load("Bullets.png");
    var bullet = SpriteComponent.fromImage(bullerImage,
        srcPosition: Vector2(66, 0), srcSize: Vector2(10, 10));

    add(bullet);
  }
}
```

Bullet의 속도가 300 고정인 추상 클래스인 EnemiesBullet 클래스를 확장해서 각각 EnemyBullet01, EnemyBullet02, EnemyBullet03 컴포넌트를 구현하였고, 이제 각 Enemy 컴포넌트에 추가 하기만 하면 됩니다.<br/><br/>

**[components/enemy.dart]**<br/>
```dart
class NormalEnemy01 extends Enemy {
  double bulletTime = 0;

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

  // update()에 Bullet 추가
  @override
  void update(double dt) {
    super.update(dt);

    bulletTime += dt;
    // 1.3초 마다 한번씩 발사
    if (bulletTime < 1.3) return;
    bulletTime = 0;

    EnemiesBullet bullet = EnemyBullet01()
      // Bullet의 위치 설정
      ..position = position.clone()
      // Bullet의 기준점 설정
      ..anchor = Anchor.center;
    gameRef.add(bullet);
  }
}
```

Enemy컴포넌트가 각각 개별로 설계 되어 있기 때문에 각 Enemy Level에 맞게 동시에 여러개의 Bullet이 표시 되도록 커스텀하게 처리도 가능할 것 입니다.<br/><br/>

그리고 Player 컴포넌트에서도 EnemiesBullet 컴포넌트와 충돌시 GameOver 되도록 처리 해야 합니다.<br/><br/>

**[components/player.dart]**<br/>
```dart
@override
void onCollision(Set<Vector2> intersectionPoints, PositionComponent other) {
  super.onCollision(intersectionPoints, other);
  
  // Enemy or EnemiesBullet 충돌시
  if (other is Enemy || other is EnemiesBullet) {  // other is EnemiesBullet 조건 추가
    _playerComponent.current = PlayerDirection.boom;
  }
}
```
<br/><br/>

**[최종 게임 영상]**<br/>

https://user-images.githubusercontent.com/13028129/215684893-a80a1991-7fb0-413f-aebd-c7ae0006b949.mp4

***

지금 까지 Flutter의 flame 미니멀리스트의 게임 엔진을 이용해서 간단한 2D Vertical Scrolling Game을 만들어 보았습니다.<br/>
flame 자체적으로 제공 되는 기능은 이 밖에도 이펙트 효과, 카메라 처리 등 다양 하기에 잘 활용하면 실제 릴리즈 해도 될만큼 퀄리티 높은 게임 제작이 가능할 것 입니다.<br/>
추가로 모바일 뿐 아니라 데스크탑과 웹 플랫폼 모두 지원하기 때문에 웹 게임도 간단하게 제작 가능합니다.<br/><br/>

**[Web 구동 화면]**<br/>
![web](https://user-images.githubusercontent.com/13028129/215686992-5d30925e-f903-4765-843c-62aba1f6775a.gif)<br/><br/>


***


위 코드는 다음 Repository에서 확인할 수 있습니다.<br/>
> [Flutter_flame_game](https://github.com/tyeom/Flutter_flame_game)



{% include content_adsense.html %}
