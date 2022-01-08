---
title: (기타) disqus provider를 사용해서 Comment기능 추가하기
categories: Etc
key: 20210108_01
comments: true
tags: github_블로그 Comment disqus
---

Jekyll테마를 사용해서 github 블로그 만들때 Comment기능을 추가 하는 방법 입니다.

참고로 다음 설명에서는 TeXt Theme 사용 기준으로 설명 합니다.

<!--more-->

준비
-

1. disqus 회원 가입<br/>
[disqus 홈페이지](https://disqus.com)에서 회원가입을 합니다.

2. 유형을 선택합니다.<br/>
![image](https://user-images.githubusercontent.com/13028129/148637708-1dc99104-d9f7-426d-885c-e8ed81baf184.png)

3. 사이트 정보를 입력합니다.<br/>
- Website Name은 github Jekyll테마에서 수정할때 사용되므로 기억해 둡니다.<br/>
- Plan은 무료 버전인 Basic으로 선택해도 됩니다.
- Platform은 Jekyll로 선택 합니다.

github Jekyll테마에 적용
-

Universal Embed Code링크를 눌러 해당 페이지로 가서 하단에 보면<br/>
Comment삽입관련 HTML태그 가이드를 볼 수 있습니다.
![image](https://user-images.githubusercontent.com/13028129/148637917-37d63e23-e1c4-4767-a2e6-663a7d714d93.png)


1. github 해당 테마의 레파지토리에서 _config.yml 설정 파일을 수정 합니다.<br/>
해당 부분은 어떤 테마를 쓰는지에 따라 다를 수 있습니다.<br/>
![image](https://user-images.githubusercontent.com/13028129/148639178-b0af0c9c-4306-4b30-a24a-7dab9f902e97.png)

위에 shortname은 disqus에서 Website Name에서 설정했던 이름을 입력하면 됩니다.

2. _include 폴더내에 disqus.html파일을 생성하고 Comment HTML가이드에서 나온 태그를 그대로 복사해서 붙여넣기 해줍니다.

위 설정 까지 했다면 다음과 같이 포스트 하단에 disqus에서 제공하는 Comment기능이 추가 된것을 볼 수 있습니다.
![image](https://user-images.githubusercontent.com/13028129/148647444-691063dd-76ee-45e2-ad79-e3f0f3c0aa49.png)


3. _layout폴더내에 article.html 수정<br/>
_layout폴더내에 post.html 또는 article.html 파일에서<br/>
_includes폴더에 생성 했던 disqus.html 내용을 include 해주어야 합니다.
html내에서 include하는 방법은

![image](https://user-images.githubusercontent.com/13028129/148648727-6317384c-d3cf-43d4-9998-9b923518f5da.png)

형식으로 추가 할 수 있으며, if표현식으로 다음과 같이 처리 할 수 있습니다.<br/>
shortname이 지정되어 있는 경우 특정 html include처리

![image](https://user-images.githubusercontent.com/13028129/148648740-01e3a83e-11b6-451b-afc8-b5df7d370e18.png)

그리고 포스트 작성시 comments: true 옵션을 추가해야 표시 될 수 있으니 이 부분도 체크해봐야 합니다.<br/>
TeXt Theme에서는 _config.yml 설정 옵션에 따라 기본적으로 포스트 작성시 Comment provider가 사용 되도록 설정 되어 있습니다.
