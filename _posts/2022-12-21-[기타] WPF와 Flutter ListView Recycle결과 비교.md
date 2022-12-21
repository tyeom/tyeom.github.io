---
title: (기타) WPF와 Flutter ListView Recycle결과 비교
categories: Etc
key: 20221221_01
comments: true
tags: Flutter WPF ListView Performance RecycleView UI가상화 Virtualizing UIVirtualizing VirtualizingPanel
---

Flutter에서 RecycleView기반의 ListView.builder로 데이터 10만건 로드 및 스크롤링 처리와<br/>
WPF에서 UI가상화를 사용하고 Recycling모드로 데이터 10만건 로드 및 스크롤링 처리 비교 입니다.<br/>

<!--more-->

개인적으론 아직까진 Flutter는 모바일에 맞춰져 있는 것 같습니다.<br/>
WPF와 Flutter(Windows)가 비교가 대상이 맞진 않는거 같은데 결과적으론 Flutter에서는 너무 많은 데이터가 있는 상태에서 빠르게 스크롤링하면 
스크롤 할 때마다 해당 데이터의 뷰포트 위치 계산과 그 위치에 따르는 데이터 검색 및 접근 연산이 느린 것 같습니다. 반면 WPF는 매끄럽게 잘 동작합니다.<br/><br/>

Flutter는 모바일 디바이스에 맞게 한번에 너무 많은 데이터를 표시해주는 것은 맞지 않고,<br/>
무한 스크롤(Infinite Scroll)같은 것을 이용해서 데이터를 끊어서 페이징 처리하는것이 좋을 것 같습니다.

WPF ListView (UI 가상화 사용, Recycling모드)
-

**[WPF ListView - 100,000 data]**<br/>
![wpf_listview](https://user-images.githubusercontent.com/13028129/208826357-bdac359c-436e-4590-bcd1-3d81e06de02e.gif)

Flutter ListView (RecycleView 사용)
-

**[Flutter ListView - 100,000 data]**<br/>
![flutter_listview](https://user-images.githubusercontent.com/13028129/208826417-1901f3d8-6da8-4323-b00d-2865db6b3bfa.gif)
<br/><br/>



위 코드는 다음 Repository에서 확인할 수 있습니다.<br/>
[Code_check - ListView_Performance_Test](https://github.com/tyeom/code_check/tree/main/TestSample/csharp/ListView_Performance_Test)



{% include content_adsense.html %}
