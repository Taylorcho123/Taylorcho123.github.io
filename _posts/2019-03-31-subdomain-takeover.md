---
layout: post
title: Subdomain takeover on usclsapipma.cv.ford.com
categories: PoC
tags: [Bugbounty, SubdomainTakeover, Translation, Hackerone]
---
해당 리포트는 포상금은 없을지라도 권한 상승 취약점을 다루기 때문에 정말 가치있는 리포트라고 생각합니다. 그것도 포드 자동차 웹페이지에서요.

출처 : [hackerone hacktivity, 작성자 : March ~~ Michel Gaschet (march)] https://hackerone.com/reports/484420

_[ 본 이슈는 해결 된 상태이며, 취약점은 한국시각으로 2019년 3월 25일 오전 8시 26분에 공개되었습니다. Ford에 리포팅 되었으며, 해당 페이지는 Ford의 자산입니다. 취약점의 종류는 Privilege Escalation이며, 포상금은 없습니다. 해당 취약점은 해당 도메인에서 7~8.9/10 정도의 심각성을 내포하고 있습니다. ]_
- - -
안녕하세요 Ford H1팀 여러분,

저는 이 리포트에서 서브도메인 탈취 취약점(Subdomain takeover vulnerability)에 대해 알리고 싶습니다. 어떠한 맥락에서는 상당히 심각한 이슈라고 할 수 있습니다.

# 개요
ford.com 서브도메인 중 하나는 Azure로 포인팅되고 있는데, CNAME 레코드에 요청되지 않은 상태(unclaimed)입니다. 이 순간 누구라도 ford.com의 서브도메인을 탈취할 수 있습니다.

해당 취약점을 서브도메인 탈취(subdomain takeover)라고 합니다. 더 궁금하시다면 아래의 링크를 열람해주세요 :
* https://blog.sweepatic.com/subdomain-takeover-principles/
* https://labs.detectify.com/tag/hostile-subdomain-takeover/
* https://hackerone.com/reports/325336
  
- - -
# 상세
usclsapipma.cv.ford.com은 feuscspma3fcvapi.eastus.cloudapp.azure.com의 CNAME을 가지고 있는 usclsapipma.trafficmanager.net의 CNAME을 가지고 있습니다. (역자 : CNAME이란 Canonical Name의 줄임말로 “하나의 도메인에 다른 이름”을 부여하는 방식을 의미합니다. 도메인 이름의 또 다른 이름으로 생각하시면 됩니다). 그러나 feuscspma3fcvapi.eastus.cloudapp.azure.com은 더 이상 애저(Azure) 클라우드앱 가상 머신에 등록되어있지 않습니다. 그러므로 누구라도 easus VM에 FQDN으로서 등록할 수 있습니다. 애저 포털의 클라우드 앱 가상머신에 등록한 사람은 트래픽 너머로 usclsapipma.cv.ford.com의 모든 통제권을 가질 수 있습니다(그래서 우리가 가상머신과 그의 운영체제의 모든 통제권을 가질 수 있기 때문에 HTTP/HTTPS 뿐만 아니라 메일 트래픽 등의 네트워크 통신을 할 수 있습니다).
- - -
# 완화 방안
ford.com DNS 영역에서 CNAME 레코드를 완전히 지우거나 애저 포털에서 재신청 하세요.
- - -
# 파일
Azure-check-availability.png -> “eastus” 클라우드앱 가상 머신을 위해 애저 웹사이트 API “check availability”를 스크린샷 하세요. 그 링크에서 FQDN의 한 부분인 DomainNameLabel “feuscspma3fcvapi”와 FQDN의 한 부분인 “eastus”, 그리고 FQDN을 위한 “available : true” 응답 위치를 확인할 수 있습니다.

![Azure-check-availability]({{site.baseurl}}/images/Azure-check-availability.png)
<center>Azure-check-availability.png</center>

dns-proof.png -> 포드 서브도메인의 CNAME 엔트리의 “NXDOMAIN” 응답을 보여주는 현 도메인들의 “dig” 명령의 결과입니다.

![dns-proof.png]({{site.baseurl}}/images/dns-proof.png)
<center>dns-proof.png</center>
- - -
# 영향
서브도메인 탈취는 다음과 같은 행위들을 위해 악용될 수 있습니다 :

* 멀웨어 확산
* 피싱 / 스피어 피싱
* XSS
* 인증 우회
* 포드 서브도메인을 대신하여 합법적(Legitimate) 메일 보내기