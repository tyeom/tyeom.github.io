---
title: (Flutter) 필수 위젯 - 반응형 화면 처리 LayoutBuilder
categories: Flutter
key: 20230215_01
comments: true
tags: Flutter LayoutBuilder responsive 반응형 반응형처리 멀티기기 멀티디바이스
---

Flutter에서는 다양한 디바이스에 따라 각 화면 사이즈에 맞게 레이아웃을 유동적으로 처리 할 수 있도록 **<span style="color: rgb(107, 173, 222);">LayoutBuilder</span>** 위젯을 제공하고 있습니다.<br/>
이번 내용은 **<span style="color: rgb(107, 173, 222);">LayoutBuilder</span>** 위젯을 사용하여 화면 사이즈에 따라 반응형으로 레이아웃이 바뀌도록 처리 할 수 있는 간단한 샘플 App을 만들어 보겠습니다.

<!--more-->

> 이 글에서 다루는 코드는 다음 Repository에서 확인할 수 있습니다.<br/>
> [code_check - Responsive_layout](https://github.com/tyeom/code_check/tree/main/TestSample/Flutter/Responsive_layout)

LayoutBuilder Class ?
-

**<span style="color: rgb(107, 173, 222);">LayoutBuilder</span>** 클래스는 디바이스의 사이즈 정보를 제공해서 사이즈에 맞게 동적으로 반응형 레이아웃을 쉽게 처리 할 수 있도록 하는 위젯입니다.<br/>
**<span style="color: rgb(107, 173, 222);">ConstrainedLayoutBuilder&lt;BoxConstraints&gt;</span>** Class를 상속 받고 있어 **<span style="color: rgb(107, 173, 222);">BoxConstraints</span>** 클래스를 통해
현재 **<span style="color: rgb(107, 173, 222);">LayoutBuilder</span>** 위젯이 배치되어 있는 화면의 사이즈(maxWidth, maxHeight) 정보를 받아 올 수 있습니다.<br/><br/>

```dart
@override
Widget build(BuildContext context) {
    return LayoutBuilder(builder: (context, constraints) {
        constraints.maxWidth  // Width size
        constraints.maxHeight  // Height size
    });
}
```

현재 화면의 사이즈가 변경 될때마다 Builder를 통해 사이즈를 알 수 있기 때문에 사이즈에 맞게 바로 반응형 화면 구성 처리가 가능 합니다.


예제
-

앞서 설명 하는 예제는 Mobile 사이즈와 Desktop 사이즈에 따라 동적으로 레이아웃이 변경 되도록 처리한 간단한 예제 입니다.<br/>
화면 레이아웃은 Git Hub 웹 사이트 페이지의 일부 레이아웃을 따라서 간단하게 표현해 보았습니다.<br/>
우선 **<span style="color: rgb(107, 173, 222);">LayoutBuilder</span>** 위젯을 배치하여 현재 디바이스 화면의 사이즈를 가져온 후 특정 사이즈 이상인 경우 Desktop View 위젯으로 표시하고
그렇지 않은 경우 Mobile View 위젯으로 표시 하도록 다음과 같이 분기 처리 합니다.<br/><br/>

**[srs/views/app_view.dart]**<br/>
```dart
import 'package:flutter/material.dart';

class AppView extends StatelessWidget {
  final Widget mobileView;
  final Widget desktopView;
  static const int _maxWidth = 900;

  const AppView({Key? key, required this.mobileView, required this.desktopView})
      : super(key: key);

  @override
  Widget build(BuildContext context) {
    return LayoutBuilder(builder: (context, constraints) {
      if (constraints.maxWidth < _maxWidth) {
        return mobileView;
      } else {
        return desktopView;
      }
    });
  }
}
```

화면 가로 사이즈가 900 초과 인경우 Desktop View로 그렇지 않다면 Mobile View로 사용 되도록 분기 되어 있습니다.<br/>
각 Desktop View와 Mobile View의 위젯 구성은 다음과 같습니다.<br/><br/>

**[srs/views/desktop_view.dart]**<br/>
```dart
import 'package:flutter/material.dart';

class DesktopView extends StatefulWidget {
  const DesktopView({super.key});

  @override
  State<DesktopView> createState() => _DesktopViewState();
}

class _DesktopViewState extends State<DesktopView> {
  // 경계선
  Widget _line() {
    return Container(
      height: 1,
      margin: const EdgeInsets.symmetric(horizontal: 15),
      color: Colors.grey.withOpacity(0.3),
    );
  }

  Widget _bodyWidget() {
    return Row(
      children: [
        // left
        SizedBox(
          width: 350,
          child: SingleChildScrollView(
            scrollDirection: Axis.vertical,
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.center,
              children: [
                Container(
                    alignment: Alignment.topRight,
                    child: Column(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      children: [
                        CircleAvatar(
                          radius: 100,
                          backgroundImage:
                              Image.asset("assets/images/13028129.jpg").image,
                        ),
                        const SizedBox(
                          height: 20,
                        ),
                        const Text(
                          "tyeom",
                          style: TextStyle(
                              fontSize: 17, fontWeight: FontWeight.bold),
                        ),
                        const SizedBox(
                          height: 20,
                        ),
                        ElevatedButton(
                            onPressed: () {},
                            style: ElevatedButton.styleFrom(
                                minimumSize: const Size(300, 50)),
                            child: const Text(
                              'Edit profile',
                              style: TextStyle(
                                  fontSize: 12, fontWeight: FontWeight.bold),
                            )),
                      ],
                    )),
                const SizedBox(
                  height: 20,
                ),
                Row(
                  mainAxisAlignment: MainAxisAlignment.center,
                  children: const [
                    Icon(Icons.people),
                    SizedBox(width: 5),
                    Text(
                      '30',
                      style: TextStyle(fontWeight: FontWeight.bold),
                    ),
                    SizedBox(width: 5),
                    Text('followers'),
                    SizedBox(width: 5),
                    Text('·'),
                    SizedBox(width: 5),
                    Text(
                      '9',
                      style: TextStyle(fontWeight: FontWeight.bold),
                    ),
                    SizedBox(width: 5),
                    Text('following'),
                  ],
                ),
                const SizedBox(
                  height: 20,
                ),
                _line(),
                const SizedBox(
                  height: 20,
                ),
                Padding(
                    padding: const EdgeInsets.all(8.0),
                    child: Container(
                      color: Colors.deepPurple[300],
                      height: 120,
                    )),
                const SizedBox(
                  height: 20,
                ),
                _line(),
                const SizedBox(
                  height: 20,
                ),
                Padding(
                    padding: const EdgeInsets.all(8.0),
                    child: Container(
                      color: Colors.deepPurple[300],
                      height: 120,
                    )),
              ],
            ),
          ),
        ),

        // right
        Expanded(
          child: Column(
            children: [
              Padding(
                padding: const EdgeInsets.all(16.0),
                child: SizedBox(
                  height: 70,
                  child: Row(
                    children: [
                      TextButton(
                          onPressed: () {},
                          child: const Text(
                            'Overview',
                            style: TextStyle(fontSize: 14),
                          )),
                      TextButton(
                          onPressed: () {},
                          child: const Text(
                            'Repositories',
                            style: TextStyle(fontSize: 14),
                          )),
                      TextButton(
                          onPressed: () {},
                          child: const Text(
                            'Projects',
                            style: TextStyle(fontSize: 14),
                          )),
                      TextButton(
                          onPressed: () {},
                          child: const Text(
                            'Packages',
                            style: TextStyle(fontSize: 14),
                          )),
                      TextButton(
                          onPressed: () {},
                          child: const Text(
                            'Stars',
                            style: TextStyle(fontSize: 14),
                          )),
                    ],
                  ),
                ),
              ),
              Expanded(
                child: ListView.separated(
                  itemCount: 10,
                  padding: const EdgeInsets.symmetric(horizontal: 10),
                  physics: const ClampingScrollPhysics(), // bounce 효과 제거
                  itemBuilder: (context, index) {
                    return Padding(
                      padding: const EdgeInsets.all(8.0),
                      child: Container(
                        color: Colors.deepPurple[300],
                        height: 120,
                      ),
                    );
                  },
                  separatorBuilder: (_, __) {
                    return _line();
                  },
                ),
              ),
            ],
          ),
        ),
      ],
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Row(
          children: const [
            Icon(Icons.hub),
            Padding(
              padding: EdgeInsets.only(left: 10),
              child: Text(
                "Git Hub",
                style: TextStyle(fontSize: 17, fontWeight: FontWeight.bold),
              ),
            ),
          ],
        ),
        actions: [
          IconButton(onPressed: () {}, icon: const Icon(Icons.notifications)),
          IconButton(onPressed: () {}, icon: const Icon(Icons.add)),
          IconButton(onPressed: () {}, icon: const Icon(Icons.person)),
        ],
      ),
      body: _bodyWidget(),
    );
  }
}
```

**[Desktop View]**<br/>
![image](https://user-images.githubusercontent.com/13028129/218936253-e5595c86-2ce8-4090-aa0c-ae2c63897d52.png)<br/><br/>

**[srs/views/mobile_view.dart]**<br/>
```dart
import 'package:flutter/material.dart';
import 'package:flutter_speed_dial/flutter_speed_dial.dart';

class MobileView extends StatefulWidget {
  const MobileView({super.key});

  @override
  State<MobileView> createState() => _MobileViewState();
}

class _MobileViewState extends State<MobileView> {
  // 경계선
  Widget _line() {
    return Container(
      height: 1,
      margin: const EdgeInsets.symmetric(horizontal: 15),
      color: Colors.grey.withOpacity(0.3),
    );
  }

  Widget _bodyWidget() {
    return Column(
      children: [
        // left
        SingleChildScrollView(
          scrollDirection: Axis.vertical,
          child: Padding(
            padding: const EdgeInsets.only(left: 20),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                const SizedBox(
                  height: 20,
                ),
                Row(
                  children: [
                    CircleAvatar(
                      radius: 30,
                      backgroundImage:
                          Image.asset("assets/images/13028129.jpg").image,
                    ),
                    const Padding(
                      padding: EdgeInsets.only(left: 20),
                      child: Text(
                        "tyeom",
                        style: TextStyle(
                            fontSize: 17, fontWeight: FontWeight.bold),
                      ),
                    ),
                  ],
                ),
                const SizedBox(
                  height: 25,
                ),
                const Text(
                  ".Net developer (C# / WPF,ASP.NET Core) Node.js developer",
                  style: TextStyle(fontSize: 13),
                ),
                const SizedBox(
                  height: 20,
                ),
                ElevatedButton(
                    onPressed: () {},
                    style: ElevatedButton.styleFrom(
                        minimumSize: const Size.fromHeight(40)),
                    child: const Text(
                      'Edit profile',
                      style:
                          TextStyle(fontSize: 12, fontWeight: FontWeight.bold),
                    )),
                const SizedBox(
                  height: 20,
                ),
                _line(),
                const SizedBox(
                  height: 20,
                ),
                Wrap(
                  spacing: 10,
                  runSpacing: 10,
                  children: [
                    'YOLO',
                    'PullShark',
                    'QuickDraw',
                    'Pair',
                    'Starstruck',
                    'ArcticCode'
                  ]
                      .map(
                        (item) => CircleAvatar(
                          radius: 30,
                          backgroundColor: Colors.lightGreen,
                          child: Text(
                            item,
                            style: const TextStyle(fontSize: 12),
                            textAlign: TextAlign.center,
                          ),
                        ),
                      )
                      .toList(),
                ),
                const SizedBox(
                  height: 20,
                ),
                _line(),
                const SizedBox(
                  height: 20,
                ),
              ],
            ),
          ),
        ),

        Expanded(
          child: ListView.separated(
            itemCount: 10,
            padding: const EdgeInsets.symmetric(horizontal: 10),
            physics: const ClampingScrollPhysics(), // bounce 효과 제거
            itemBuilder: (context, index) {
              return Padding(
                padding: const EdgeInsets.all(8.0),
                child: Container(
                  color: Colors.deepPurple[300],
                  height: 120,
                ),
              );
            },
            separatorBuilder: (_, __) {
              return _line();
            },
          ),
        ),
      ],
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Row(
          mainAxisAlignment: MainAxisAlignment.start,
          children: [
            SpeedDial(
              switchLabelPosition: true,
              icon: Icons.menu,
              activeIcon: Icons.close,
              spacing: 3,
              direction: SpeedDialDirection.down,
              children: [
                SpeedDialChild(
                    onTap: () {
                      print('Dashboard');
                    },
                    label: 'Dashboard'),
                SpeedDialChild(
                    onTap: () {
                      print('Pull requests');
                    },
                    label: 'Pull requests'),
                SpeedDialChild(
                    onTap: () {
                      print('Issues');
                    },
                    label: 'Issues'),
                SpeedDialChild(
                    onTap: () {
                      print('Codespace');
                    },
                    label: 'Codespace'),
                SpeedDialChild(
                    onTap: () {
                      print('Marketplace');
                    },
                    label: 'Marketplace'),
                SpeedDialChild(
                    onTap: () {
                      print('Explore');
                    },
                    label: 'Explore'),
                SpeedDialChild(
                    onTap: () {
                      print('Sponsors');
                    },
                    label: 'Sponsors'),
                SpeedDialChild(
                    onTap: () {
                      print('Settings');
                    },
                    label: 'Settings'),
              ],
            ),
            Expanded(
              child: Container(
                  alignment: Alignment.center, child: const Icon(Icons.hub)),
            ),
          ],
        ),
        actions: [
          IconButton(onPressed: () {}, icon: const Icon(Icons.notifications)),
        ],
      ),
      body: _bodyWidget(),
      //floatingActionButtonLocation: FloatingActionButtonLocation.startTop,
    );
  }
}
```

**[Mobile View]**<br/>
![image](https://user-images.githubusercontent.com/13028129/218936311-e038562e-c458-4b77-9853-8059b20415c1.png)<br/><br/>

***

실행해서 화면 사이즈가 변경 될때 반응형으로 레이아웃이 바뀌는 것을 확인해 볼 수 있습니다.<br/><br/>
**[결과 화면]**<br/>
![flutter_responsive](https://user-images.githubusercontent.com/13028129/218936723-3c9b74f0-7d05-45cb-9fd0-d20206e4d94e.gif)<br/><br/>


이렇게 **<span style="color: rgb(107, 173, 222);">LayoutBuilder</span>** 위젯 사용으로 반응형 레이아웃 처리 구현을 간단하게 처리해 볼 수 있습니다.<br/>
여러 디바이스 환경 또는 App 및 웹 대상으로 UI/UX 처리 구현을 고려 했을때 **<span style="color: rgb(107, 173, 222);">MediaQuery</span>** 클래스를 사용하여 디바이스 사이즈를 계산해 사용도 가능하지만,
화면 사이즈가 변경 될때 마다 반응형으로 간단하게 처리하기 위해서는 필수로 사용 되는 위젯 입니다.

***


위 코드는 다음 Repository에서 확인할 수 있습니다.<br/>
> [code_check - Responsive_layout](https://github.com/tyeom/code_check/tree/main/TestSample/Flutter/Responsive_layout)



{% include content_adsense.html %}
