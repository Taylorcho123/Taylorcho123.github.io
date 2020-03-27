---
layout: post
title: 안드로이드 모의해킹 실습 01 – 인시큐어뱅크(InsecureBankv2) 활용
categories: MobileHacking
tags: [Android, InsecureBank]
---
# 목차
&nbsp;&nbsp;<a href="#{{ 1| downcase }}">1. Disclaimer</a><br>
&nbsp;&nbsp;<a href="#{{ 2| downcase }}">2. 인시큐어뱅크(InsecureBankv2)란?</a><br>
&nbsp;&nbsp;<a href="#{{ 3| downcase }}">3. 브로드캐스트 리시버 결함</a><br>
&nbsp;&nbsp;<a href="#{{ 4| downcase }}">4. 취약한 인증 메커니즘</a><br>
&nbsp;&nbsp;<a href="#{{ 5| downcase }}">5. 취약점 대응 방안</a><br>
&nbsp;&nbsp;<a href="#{{ 6| downcase }}">6. References</a>
- - -
  
<h1 id="{{ 1| downcase }}">1. Disclaimer</h1>
<font color='red'>
All the content shown in this post is only for educational purposes. Any misuse of this content is completely at your own risk.<br>
Do not try to attempt to break laws or do any illegal stuff by using this.<br><br>

이 포스트에서 보여지는 모든 컨텐츠는 교육적인 목적으로만 제작되었습니다. 해당 컨텐츠를 오용한 결과는 전적으로 오용한 본인 책임입니다.<br>
이 포스트에서 다루는 내용을 이용해 불법적인 일을 저지르려고 하지 마십시오.
</font>
- - -
<h1 id="{{ 2| downcase }}">2. 인시큐어뱅크(InsecureBankv2)란?</h1>
인시큐어뱅크는 모바일 뱅킹 취약점 진단을 위한 테스트 용도로 제작된 애플리케이션입니다. 안드로이드 모바일 개발자 및 보안 관리자를 위해 만들어졌으며, 백엔드 서버는 파이썬으로 제작되었습니다. 현재까지 총 23개의 취약점을 테스트할 수 있으며, 소스파일은 인시큐어 뱅크 깃허브 페이지에서 [다운로드](https://github.com/dineshshetty/Android-InsecureBankv2){:target="_blank"} 하실 수 있습니다.  
(해당 포스트에서는 진단 환경 구성에 관한 내용을 생략했습니다.)
- - -
<h1 id="{{ 3| downcase }}">3. 브로드캐스트 리시버 결함</h1>
## – 소개
브로드캐스트 리시버는 안드로이드의 4대 컴포넌트 중에 하나입니다. 브로드캐스트 리시버의 역할은 단말기 안에서 이루어지는 수 많은 일들을 대신해서 알려주는 것 입니다. 예를 들어, 배터리 부족, SMS 문자 메시지, 전화 알림 등의 방송(이벤트)를 캐치 후 리시버로 처리할 수 있도록 줍니다. “방송하기 -> 수신하기”가 하나의 사이클로 동작됩니다.  

브로드캐스트 리시버를 구현하기 위해서는 두 가지 방법이 존재합니다.  
코드상에서 BroadCastReceiver를 등록하는 방법인 동적인 방법과 AndroidManifest.xml의 `<receiver></receiver>`의 형태로 등록하는 정적인 방법이 있습니다.
- - -
## – 취약점 진단 과정
해당 취약점을 살펴보기에 앞서, .apk파일의 코드를 살펴보는 방법을 적어보겠습니다.


.apk를 .zip파일로 확장자 변경을 하고 압축 해제하면 왼쪽과 같습니다. 이 중 classes.dex 파일을 .jar파일로 변환해서 애플리케이션의 코드를 살펴볼 수 있습니다.



[dex2jar](https://github.com/pxb1988/dex2jar){:target="_blank"}는 APK파일이나 APK파일에 포함된 classes.dex 파일을 자바 클래스 파일로 변환해주는 도구입니다. .jar파일을 [jd-gui](http://java-decompiler.github.io/#jd-gui-download){:target="_blank"}로 열어보면 아래와 같이 자바 패키지 파일들과 메서드들을 확인해볼 수 있습니다.
- - -
.apk를 .zip으로 변환하여 압축 해제한 파일들 중에서 AndroidManifest.xml 파일을 살펴보겠습니다. 리시버 선언 부분에 해당하는 xml 코드는 다음과 같습니다.


여기에 선언된 브로드캐스트의 이름은 theBroadcast이며, 브로드캐스트 신호를 받으면 MyBroadCastReceiver에 설정된 작업을 수행합니다. 그리고 exported값이 true로 되어 있기 때문에 외부 애플리케이션으로부터 intent를 받을 수 있는 상태 입니다.
- - -
ADB 쉘 명령을 이용하여 임의의 브로드캐스트를 생성, 리시버를 속이는 작업을 해보겠습니다. 여기서 ADB(Android Debug Bridge)란 안드로이드 에뮬레이터나 PC에 실제 연결된 장치를 제어하기 위한 안드로이드 디버깅 도구 중 하나 입니다. ADB 쉘 명령어 중 am 명령어는 액티비티 매니저로써, 이것을 사용하면 안드로이드 시스템에 포함된 다양한 액션을 명령으로 수행할 수 있습니다.

ADB를 이용하여 확인하는 명령어는 다음과 같습니다 :
{% highlight text %}
adb shell am start [앱이 설치된 주소]/[호출하고 싶은 패키지 주소]
{% endhighlight %}
phonenumber와 newpass 값을 포함하여 브로드캐스트를 생성합니다.
–es 옵션을 이용하여 변수와 함께 값을 추가합니다.

{% highlight text %}
> adb shell broadcast -a theBroadcast -n com.android.insecurebankv2/.MyBroadCastReceiver --es phonenumber 5555 --es newpass test
{% endhighlight %}

logcat을 이용해 로그를 확인해봅니다.
{% highlight text %}
> adb logcat > log01.txt
{% endhighlight %}

logcat 정보를 확인해보시면 기존에 사용했던 비밀번호가 평문으로 보여짐을 확인할 수 있습니다(계정의 비밀번호는 변경되지 않습니다).
- - -
<h1 id="{{ 4| downcase }}">4. 취약한 인증 매커니즘</h1>
## – 소개
취약한 인증 매커니즘은 정상적인 인증 절차를 우회하여 잘못된(비정상적인) 인증으로 접근 권한을 취득하는 취약점을 말합니다.
- - -
## – 취약점 진단 과정
AndroidManifest.xml 코드의 안드로이드 액티비티 속성이 android:exported=”true”로 설정되어 있다는 것을 알 수 있습니다.


인시큐어 뱅크에 ADB 쉘을 이용해 액티비티를 호출하는 명령어 입니다.
{% highlight text %}
> adb shell am start com.android.insecurebankv2/com.android.insecurebankv2.PostLogin
> adb shell am start com.android.insecurebankv2/com.android.insecurebankv2.DoTransfer
{% endhighlight %}
위 명령어를 입력했을 때 실제 애플리케이션에서 별도의 인증 절차 없이 해당 액티비티가 직접 호출됨을 확인할 수 있었습니다.

<blockquote class="imgur-embed-pub" lang="en" data-id="ZKj6PGI"><a href="//imgur.com/ZKj6PGI"></a></blockquote><script async src="//s.imgur.com/min/embed.js" charset="utf-8"></script>
- - -
<h1 id="{{ 5| downcase }}">5. 취약점 대응 방안</h1>
AndroidManifest.xml 리시버와 액티비티 항목에 위치하는 android:exported=”true”항목을 “false”로 바꿔주어야 합니다.

android:exported=”true”로 설정해야 하는 경우, 각 리시버에 별도의 권한을 주어야 하며 별도의 인텐트 필터로 검증해야 합니다.
- - -
<h1 id="{{ 6| downcase }}">6. References</h1>
&#91;안드로이드 모바일 앱 모의해킹&#93; / 조정원, 김명근, 조승현, 류진영, 김광수 지음 / 에이콘 출판사
