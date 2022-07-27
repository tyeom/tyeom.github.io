---
title: (ETC) github 블로그 개인 도메인 연결
categories: ETC
key: 20220727_02
comments: true
tags: github_blog githubblog github 도메인연결
---

이번 아티클은 이전 [Synology NAS 도메인 연결 및 SSL 인증서 설정](https://blog.arong.info/nas/2022/07/27/NAS-Synology-NAS-%EB%8F%84%EB%A9%94%EC%9D%B8-%EC%97%B0%EA%B2%B0-%EB%B0%8F-SSL-%EC%9D%B8%EC%A6%9D%EC%84%9C-%EC%84%A4%EC%A0%95.html) 아티클에 이어서 깃 블로그에 도메인 연결하는 방법 입니다.<br/>

<!--more-->

도메인 CName 설정
-

[dnszi.com](dnszi.com) 에서 기존 github blog 주소를 CName으로 추가 설정 합니다.<br/>
![image](https://user-images.githubusercontent.com/13028129/181168160-3e24a790-f846-4dd0-ac44-08e9069a3587.png)<br/>
＊ 2차 도메인이 아닌 대표 도메인으로 하는 경우<br/>
CName에 '@'(골뱅이)로 추가 하면 됩니다. (이 부분은 직접 설정해 보진 않아서 정확하지 않음)

git reposltory 설정
-

이후 해당 git reposltory의 [Settings]의 [Pages] 메뉴에 Custom domain에 CName으로 등록한 도메인을 설정해 줍니다. 

{% include content_adsense.html %}
