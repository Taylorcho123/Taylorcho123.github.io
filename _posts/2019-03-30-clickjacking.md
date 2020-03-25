---
layout: post
title: 클릭재킹 개념과 실제 사례(ylands.com)
categories: PoC
tags: [Bugbounty, Clickjacking, Translation, Hackerone]
---
# 개념
[클릭재킹](https://ko.wikipedia.org/wiki/%ED%81%B4%EB%A6%AD%EC%9E%AC%ED%82%B9){:target="_blank"}이란 웹 사용자가 자신이 클릭하고 있다고 인지하는 것과 다른 것 즉, 공격자가 원하는 것을 클릭하도록 속이는 해킹 기법을 말합니다. 사용자 입장에서는 마치 정상적인 페이지에 접근해서 원하는 API이벤트를 발생시키고 있다고 착각하게 되지만, 실은 사용자가 보고있는 페이지의 url은 `<iframe>`태그 안에 있는 것이고 공격자는 그 아래에 악의적인 javascript코드를 삽입하여 사용자를 속일 수 있습니다.

예를 들어, 공격자는 사용자를 속이기 위해서 보이지 않는 레이어를 만든 뒤, 보이지 않는 버튼을 만듭니다. 그 후에 공격자의 페이지에서 사용자의 실제 커서를 보이지 않게 만들어 공격자가 만들어둔 버튼 위에 위치시키도록 한 뒤, 실제 사용자가 조작하는 ‘보이는’ 커서는 가짜로 생성합니다. 그래서 사용자가 클릭이라는 이벤트를 발생시키면 ‘보이지 않는’ 실제 커서가 공격자의 버튼을 클릭하게 되는 것 입니다.

이 예시에서의 공격자의 버튼을 클릭하게되면 바이러스가 있는 악성 스팸 사이트로 접근이 되거나 피싱 사이트로 접근이 가능합니다. 또한 예시로 든 클릭재킹 외에도 페이스북의 ‘좋아요’버튼을 누르게되는 라이크재킹 등의 공격이 가능합니다.

# X-Frame-Options
이러한 클릭재킹 공격기법을 방어하기 위해서 브라우저에서는 Http헤더에 X-Frame-Options 속성 중에서 DENY나 SAMEORIGIN 옵션을 선택해야 합니다.
DENY는 연결 요청되는 사이트에 관계 없이 프레임에 표시할 수 없는 옵션이고, SAMEORIGIN은 페이지 자체와 동일한 출처의 프레임만 표시될 수 있는 것을 말합니다

그 외의 클릭재킹 완화 방안으로는 `<iframe>`태그를 깨트려버리는 Framekiller JS가 있고, 보안 위협에 대비할 수 있는 정책을 Content Security Policy headers 에 설정하여 완화시키는 방법이 있습니다.
- - -
# 버그바운티 리포트 번역
다음은 클릭재킹에 관련하여 hackerone의 hacktivity에 올라온 버그바운티 리포트입니다. 보통 버그바운티에 올라오는 의뢰 사이트들의 Out of Scope를 보면 대체로 클릭재킹 기법을 막아놓는 경우가 많은데 ylands의 경우는 그렇지 않았나 봅니다.

출처 : [hackerone hacktivity, 작성자 : KryptoMon (kryptomon)] https://hackerone.com/reports/40534

_[ 본 이슈는 해결 된 상태이며, 취약점은 2019년 3월 22일 오전 12시 28분에 공개되었습니다(한국시각 UTC +9). BOHEMIA INTERACTIVE a.s.에 리포팅 되었습니다. 취약점의 종류는 UI Redressing(Clickjacking)이며, 포상금은 $80달러로 한화로 약 9만원 입니다. 해당 취약점은 해당 도메인에서 4~6.9/10 정도의 심각성을 내포하고 있습니다. ]_
- - -
# 설명
안녕하세요,

ylands의 웹사이트 보안 테스팅을 수행하던 도중 저는 ‘클릭재킹’이라는 취약점을 발견했습니다. (테스팅)범위 안에있는 대부분의 URL들이 클릭재킹에 취약합니다.

클릭재킹이란 무엇일까요?

클릭재킹(User Interface redress attack, UI redress attack, UI redressing)이란 사용자를 속이는 악의적인 기법으로써, 사용자를 자신이 클릭했다고 인식하는 것과는 다른 것을 클릭하게 됨으로써 잠재적으로 기밀 정보를 흘리거나 그들의 컴퓨터를 장악하게 됨을 의미합니다. 겉보기엔 무해해 보일지라도 말입니다.

해당 서버는 X-Frame-Options 헤더를 리턴하지 않는데, 이것은 해당 웹사이트가 클릭재킹 공격으로부터 위험하다는 것을 의미합니다. X-Frame-Options HTTP 응답 헤더는 브라우저가 <frame>이나 <iframe>안의 페이지를 렌더링하기를 허용할 것인지 혹은 그렇지 않은지에 대한 지표로써 사용됩니다. 해당 사이트들은 이 헤더를 이용함으로써 클릭재킹 공격에 방어를 할 수 있는데요, 그들의 컨텐츠가 다른 사이트에 삽입되지 않았다는 것을 보장함으로써 가능합니다. 또한 이 취약점은 웹 서버에 영향을 미칩니다.
- - -
# 공격이 발생되는 과정 / 개념 증명
## 취약한 URL : 
* https://ylands.com/
* https://workshop.ylands.com/
* https://dayz.com/
* http://armamobileops.com/
* https://minidayz.com/

# 위의 모든 url을 하나씩 iframe 코드 안에 넣어보세요. 코드는 아래의 것을 사용하세요 :
{% highlight html %}
<!DOCTYPE HTML>
<html lang="en-US">
	<head>
		<meta charset="UTF-8">
		<title>I Frame</title>
	</head>
	<body>
		<h3>clickjacking vulnerability</h3>
		<iframe src="https://vigorgame.com/" height="550px" width="700px"></iframe>
	</body>
</html>
{% endhighlight %}

By Tahir Javed
tahirjavedbhutta@gmail.com

사이트가 iframe에 보여진다는 것에 주의하세요.

개념 증명(POC)는 첨부 파일에 있습니다. 답변을 기다리겠습니다. 감사합니다.


![clickjacking1]({{site.baseurl}}/images/clickjacking-1.png)
<center>POC 중 하나. 나머지 4개의 첨부 이미지도 같은 형식임. [사진 출처 : https://hackerone.com/reports/405342]</center>

- - -
# 영향
비슷한 기법을 사용함으로써, 키보드 타이핑을 하이재킹할 수 있습니다. 잘 조작된 스타일시트와 iframe 그리고 텍스트박스를 이용한다면 사용자는 패스워드란, 그들의 이메일이나 은행 계좌를 타이핑한다고 믿겠지만 사실은 그게 아니라 공격자에 의해서 통제되는 보이지않는 프레임에 타이핑하게 되는 것입니다.