---
title: (NAS) Synology NAS 도메인 연결 및 SSL 인증서 설정
categories: NAS
key: 20220727_01
comments: true
tags: Synology NAS 인증서설정 도메인연결
---

개인 도메인을 하나 구입해서 Synology NAS에 적용하고 해당 도메인이 SSL 인증서 발급 및 적용하는 과정을 기록합니다.<br/>
우선 적용할 도메인이 없다면 가비아, cafe24등 도메인 관리 업체에서 구입할 수 있습니다.

<!--more-->

도메인을 구입했다면 해당 도메인에 사용되는 호스트를 연결해야 하는데 보통 일반 가정 환경에서는 고정 IP가 아닌 ISP업체에서 제공받는 유동 IP를 사용합니다. 그리고 공유기를 통해 N개의 PC를 사용합니다.<br/>
고정 IP가 아니기 때문에 DDNS를 사용해야 하는데 iptimne 공유기에서 자체적으로 DDNS를 제공하거나 [dnszi.com](https://dnszi.com)(무료) 같은 곳이 있습니다.<br/>
저는 [dnszi.com](https://dnszi.com)에서 DDNS서비스를 받아 연결해 보겠습니다.

DDNS 설정 및 네임서버 설정
-

[dnszi.com](https://dnszi.com) 회원가입 후 도메인 관리 페이지에서 다음과 같이 설정합니다.<br/>
![image](https://user-images.githubusercontent.com/13028129/181153631-47c1f25a-c0e9-458f-90c2-9e52348dd104.png)
<br/>
호스트 IP 관리에서 A레코드를 추가 합니다. 2차 도메인 형식이 아닌 대표 도메인으로 등록하기 위해 A레코드 입력란엔 빈항목으로,<br/>
IP 주소란에는 외부 IP주소를 입력합니다. (ISP업체에서 발급되는 IP주소) 그리고 DDNS설정은 '○'로 설정 합니다.<br/><br/>

A레코드가 추가 되었다면 도메인을 구입한 관리 사이트에서 네임서버를 [dnszi.com](https://dnszi.com)의 네임서버로 변경합니다.<br/>
![image](https://user-images.githubusercontent.com/13028129/181154258-ad634096-be14-4105-984c-2e28e5d7f80f.png)<br/>
설정 후 반영되기 까지는 대략 10분 ~ 길게는 1시간 정도 걸릴 수 있습니다.<br/><br/>

그리고 Synology NAS의 DSM 접속 도메인으로 사용할 2차 도메인을 CName으로 설정합니다.<br/>
![image](https://user-images.githubusercontent.com/13028129/181160875-43775d8c-2a24-4864-a00b-93091a35329a.png)<br/>
저의 경우는 nas.arong.info를 DSM 접속 도메인으로 사용하기 위해 CName으로 추가해 주었습니다.


nslookup {도메인주소} 명령으로 실제 Address가 A레코드 호스트 IP주소로 나온다면 정상적으로 처리 된 것 입니다.

NAS 역방향 프록시 설정
-

여기 까지 해당 도메인으로 공유기 까지 접속되고 공유기를 통해 Synology NAS의 DSM 사이트까지 접속이 가능한 상태 입니다.<br/>
이제 해당 도메인 기준으로 SSL 인증서를 발급 받고 적용해 보겠습니다.<br/>

먼저 공유기 DMZ설정을 해주어야 합니다.<br/>
(사실 이부분은 필요 없는 부분인데 저 같은 경우 인증서 발급시 호스트 IP 80번 443번 포트에 도달하지 못하는 오류가 발생해서 설정하였습니다.)<br/>
![image](https://user-images.githubusercontent.com/13028129/181160372-d4a34e5a-b995-43eb-bbbc-9c5d33771b35.png)<br/><br/>


그리고 nas.arong.info 도메인을 역방향 프록시를 통해서 DSM 페이지로 설정합니다.<br/>
[제어판] > [로그인 포털] > [고급 탭] > 역방향 프록시<br/>
![image](https://user-images.githubusercontent.com/13028129/181161192-23c2ea7a-9eae-4853-89ac-86505534431f.png)<br/>
여기서 추가적으로 [사용자 지정 머리글] 탭에서 생성 > WebSocket를 추가해주어야 NAS GUI 도커에서 터미널을 생성할 수 있습니다.<br/>
이 부분이 설정되어 있지 않다면 DSM 에서 도커 > 터미널이 열리지 않게 됩니다. (실제 SSH 에서는 정상 동작)<br/>
![image](https://user-images.githubusercontent.com/13028129/181161585-f3883b59-b95a-4c6f-ae61-fe9ccb8bf8a9.png)<br/>
여기까지 되었다면 해당 도메인 기준으로 새로운 인증서 발급을 통해 적용할 수 있습니다.

NAS 인증서 발급 및 적용
-

[제어판] > [보안] > [인증서 탭] > [추가] 를 통해 새 인증서를 추가 합니다.<br/>
꼭 기본 인증서로 체크 하고 인증서 발급은 Let's Encrypt를 통해서 무료로 발급받을 수 있습니다.<br/>
![image](https://user-images.githubusercontent.com/13028129/181162218-52d2e253-e226-417d-9adf-0fee8910e4f5.png)<br/>
위 처럼 도메인을 입력하고 여기서 중요한 부분은 주제 대체 이름에 해당 도메인에 소속되어 있는 2차 도메인을 ';'(세미콜론) 기준으로 나열해야 합니다.<br/>
완료를 누르고 잠시 기다리면 새로운 인증서가 발급되었음을 확인할 수 있습니다.<br/>
![image](https://user-images.githubusercontent.com/13028129/181162452-527310be-dad5-4c87-acee-bf83cf4b01be.png)<br/><br/>

<details>
<summary><b>인증서 발급 오류 발생시?</b></summary>
<div markdown="1">  
> 만약 인증서 발급시 잘못된 호스트 IP, 또는 역방향 프록시 관련 오류가 나온다면 DDNS A레코드 호스트 IP가 잘못 설정 되었는지
> NAS 방화벽에서 (제어판 > 보안 > 방화벽 > 규칙 편집) HTTP, Reverse Proxy HTTPS (80번), Reverse Proxy(443번) 허용되어 있는지 체크가 필요합니다.

|![image](https://user-images.githubusercontent.com/13028129/181163751-57453d40-4d29-4a27-814f-658eb312f6b6.png)|![image](https://user-images.githubusercontent.com/13028129/181163821-12b6df71-30de-471c-97af-3d1ee5715693.png)|
|---|---|
</div>
</details>



추가로 상단 [설정] 메뉴에서 [구성 탭]에 보여지는 서비스란에 조금전 역방향 프록시 설정으로 지정된 도메인 서비스와 시스템 기본 설정 서비스를<br/>
새로 발급 받은 인증서로 선택해야 합니다.<br/>
![image](https://user-images.githubusercontent.com/13028129/181162777-cf12fa20-4a92-4ad2-bdfb-eaa46a4bb4c9.png)<br/>
위 설정이 되어 있지 않으면 기존 서비스 되는 DSM 인증서와 역방향 프록시로 사용되는 도메인 인증서 불일치로 안전하지 않은 인증서로 브라우저에서 접속을 일시 차단 하게 됩니다.<br/>
브라우저에서 해당 도메인에 https로 접속이 잘 되는걸 확인해 볼 수 있습니다.

DDNS 자동 갱신 처리
-

만약 ISP에서 발급된 IP가 변경되어 DNS호스트 IP가 변경 되었다면 DDNS를 다시 갱신해주어야 하는데 dnszi에서 get http 요청을 통해 자동으로 업데이트 해주는 url을 제공해 줍니다.<br/>
이를 이용해서 스크립트를 만들고 NAS 작업스케줄로 등록해서 자동으로 업데이트 되도록 처리할 수 있습니다.<br/><br/>

dnszi의 도메인관리 페이지 고급관리에서 [인증키생성]을 통해 인증키를 생성합니다.<br/>
생성된 인증키를 이용해서 스크립트를 작성하고 NAS에 업로드 합니다.<br/>
**[ddns_dnsever.sh]**
```
#!/bin/sh


# 저장되어지는 위치 (directory)
# DIR=/volume1/시놀로지폴더/하위폴더   <<log확인 시 주석 풀고 사용
  
# 저장되어지는 파일 명
# dailylog.20091112 와 같은 형식
#FILE00=/ddns_dnsever[`date +"%Y%m%d%I%M%S"`]00.txt  << 저장 폴더 사용 시 여기도 주석 해제


  
# wget 의 실행 로그가 쌓입니다.
#LOGFILE=/some/where/logdir/wget.log
  
# wget이 읽어오는 URL
#URL=
   
# 실제 파일을 읽어옵니다.
#wget $URL -O $DIR$FILE -o $LOGFILE
/usr/bin/wget  -O - 'http://ddns.dnszi.com/set.html?user={아이디}&auth={인증번호}&domain={도메인주소(arong.info)}&record='
```

위 파일을 NAS에 업로드 하고 [제어판] > [작업 스케줄러] > [생성]을 통해 매시간 마다 해당 스크립트를 실행하는 작업을 생성 합니다.<br/>
|![image](https://user-images.githubusercontent.com/13028129/181166284-80f2e74b-9a65-47b7-bd7f-a3d061021d6e.png)|![image](https://user-images.githubusercontent.com/13028129/181166334-a8f97918-6d5c-420b-a80a-20e7f19df713.png)|![image](https://user-images.githubusercontent.com/13028129/181166455-2af78c80-218b-467d-a57a-69680557508e.png)
|---|---|---|

{% include content_adsense.html %}
