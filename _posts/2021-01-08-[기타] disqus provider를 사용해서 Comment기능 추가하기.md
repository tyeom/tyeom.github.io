---
title: (기타) disqus provider를 사용해서 Comment기능 추가하기
tags: github_블로그 Comment disqus
---

Jekyll테마를 사용해서 github 블로그 만들때 Comment기능을 추가 하는 방법 입니다.

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
- Platform은 Jekyll로 선택 합니다. (꼭 Jekyll선택이 아닌 다른걸로 해도 되는 것 같지만 확인은 해보지 않았습니다.)

github Jekyll테마에 적용
-

Universal Embed Code링크를 눌러 해당 페이지로 가서 하단에 보면<br/>
Comment삽입관련 HTML코드 가이드를 볼 수 있습니다.
![image](https://user-images.githubusercontent.com/13028129/148637917-37d63e23-e1c4-4767-a2e6-663a7d714d93.png)


1. github 해당 테마의 레파지토리에서 _config.yml 설정 파일을 수정 합니다.<br/>
해당 부분은 어떤 테마를 쓰는지에 따라 다를 수 있습니다.<br/>
![image](https://user-images.githubusercontent.com/13028129/148639178-b0af0c9c-4306-4b30-a24a-7dab9f902e97.png)
