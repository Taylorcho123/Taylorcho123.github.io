---
layout: post
title: 라즈베리파이2 웹서버 구축
categories: Coding
tags: [Linux, Server, RaspberryPi]
---
# 0. 준비물
1. <a href="#{{ 1| downcase }}">1. 라즈베리파이</a>
1. 키보드/마우스/모니터(+HDMI케이블)/휴대폰 충전기
1. 랜선(혹은 usb무선 랜카드)
1. sd카드★

주의사항1 : 라즈베리파이3의 경우는 와이파이 설정이 되지만, 버전2는 그렇지 않습니다. 와이파이를 사용하시려면 별도의 usb랜카드를 구입해주셔야 합니다.  

주의사항2 : sd카드는 반드시 FAT32로 해주셔야 합니다. FAT도 안됨.
- - -
# 1. 라즈비안 설치
<h1 id="{{ 1| downcase }}">1. 라즈비안 설치</h1>
일단 라즈베리파이에 라즈비안을 설치하는 것은 어렵지 않습니다. 라즈베리파이 [NOOBS 다운로드 페이지](https://www.raspberrypi.org/downloads/noobs/){:target="_blank"}에 들어가셔서 NOOBS Lite를 다운받아줍니다.


NOOBS의 압축을 풀고, 해당 파일들을 FAT32로 포맷된 sd카드로 복사합니다. 그 후, 라즈베리파이에 충전기를 꼿아 부팅되게 만듭니다. 그러면 NOOBS가 켜지고, 라즈비안을 설치할 수 있습니다.


기타 내용은 [제 이전 블로그의 포스팅](https://blog.naver.com/pentester357/221213608851){:target="_blank"}을 참고해 주세요 ㅎㅎ
- - -
# 2. APM 설치
APM은 Apache, PHP, MySQL의 두문자어 입니다.
APM 설치 명령어와 그 과정을 적어보겠습니다.
{% highlight text %}
sudo apt-get update && sudo apt-get upgrade
{% endhighlight %}
업데이트를 해줍니다.
- - -
## 2.1. Apache2 설치
{% highlight text %}
sudo apt install apache2
{% endhighlight %}
브라우저에 localhost를 입력해서 정상적으로 설치가 되었는지 확인합니다.

방화벽 기본 설정을 변경합니다.
{% highlight text %}
sudo ufw enable
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw allow 80
sudo ufw allow 443
sudo ufw status numbered
{% endhighlight %}
방화벽 프로그램인 ufw를 활성화 한 후, 방화벽 기본설정을 합니다.  
incoming 정책을 deny로 설정하고, outgoing 정책을 allow합니다.  
ssh접속도 허용합니다.  

웹서버에서 사용할 http 80번 포트와 https 443번 포트를 선택합니다.
- - -
## 2.1. MySQL 서버 설치
{% highlight text %}
sudo apt install mysql-server
sudo mysql_secure_installation
{% endhighlight %}
MySQL 서버 패키지를 설치합니다.

설치 진행 과정을 따라 [y/n] 로 물어볼 때는 y를 입력합니다.  
설치 중 패스워드 묻는 부분은 root 패스워드를 설정하는 것입니다.  
{% highlight text %}
sudo mysql -u root -p
{% endhighlight %}

sudo mysql -u root -p 혹은 sudo mysql 입력한다. 
{% highlight text %}
create database db DEFAULT CHARACTER SET utf8;
create user 사용자이름 identified by '패스워드';
GRANT ALL PRIVILEGES ON db.* TO '사용자이름'@'localhost' identified by '패스워드';
{% endhighlight %}
db라는 이름의 데이터베이스를 생성한 후, 유저를 생성합니다. 그리고 그 유저에게 해당 데이터베이스에 관한 모든 권한을 부여해줍니다.

{% highlight text %}
mysql -u 사용자이름 -p
{% endhighlight %}
방금 생성한 사용자이름으로 MySQL에 로그인 합니다.
{% highlight text %}
show databases;
use db;
create table tb1(
-> id int unsigned not null auto_increment, 
-> name varchar(255) not null, 
-> contents varchar(255) not null,
-> primary key (id),
) DEFAULT CHARACTER SET utf8;
show tables;
insert into tb1(name, contents) values('Kim', 'Hello World!');
exit
{% endhighlight %}
테이블을 생성해봅니다.
- - -
## 3. PHP 설치
{% highlight text %}
sudo apt install php php-mysql
{% endhighlight %}

{% highlight text %}
sudo vi /var/www/html/info.php
{% endhighlight %}
/var/www/html/info.php를 열어서  
<?php phpinfo(); ?>를 입력합니다.  

위와같이 php정보가 뜬다면 잘 설치가 된 것입니다.
- - -
## 4. phpMyAdmin 설치 (옵션)
{% highlight text %}
sudo apt install phpmyadmin
{% endhighlight %}

phpMyAdmin이 실행되도록 자동으로 설정할 웹 서버를 선택합니다. apache2를 선택합니다.  




위와같이 사례대로 설정한 후, phpMyAdmin에서 사용할 MySQL 암호를 설정합니다.  
브라우저에서 localhost/phpmyadmin/으로 접속하고 MySQL 계정정보를 입력하고 실행(로그인)합니다.  


 
사용할 수 있도록 권한을 부여받은 데이터베이스 db에 대한 작업 외에는 다른 일을 할 수 없도록 되어있습니다.  

## ※ phpmyadmin 페이지 안보일 때 재설치 참고 :
{% highlight text %}
apt-get update
apt-get install --reinstall phpmyadmin
ln -s /etc/phpmyadmin/apache.conf /etc/apache2/sites-available/phpmyadmin.conf
a2ensite phpmyadmin
service apache2 restart
{% endhighlight %}
- - -
# 3. PHP 코딩
제가 만든 php코드를 참고해주세요. 여기서 다운로드 받으실 수 있습니다.

중요한 코드는 아래와 같습니다 :  
{% highlight php %}
<!-- index.php의 78~90번째줄 코드 -->

<?php
$conn = mysqli_connect('localhost', 'jaycho', '******', 'db');
$sql = "SELECT * FROM Posts ORDER BY id DESC";
$result = mysqli_query($conn, $sql);
while($row = mysqli_fetch_row($result)) {
	echo '<div class="in">';
	echo '<p><b>'.$row[2].'</b></p>';
	echo 'Written By <u>'.$row[1].'</u>';
	echo '</div>';
	echo '<br><br>';
}
?>
{% endhighlight %}
MySQL에 계정정보를 입력해서 연결하고, SQL 쿼리문을 변수에 저장하고, 요청해서 한줄씩 받아옵니다. db에 튜플이 더 이상 없을 때 까지 내림차순(DESC)으로 반복문을 이용해 받아옵니다.
{% highlight html %}
<!-- index.html 69~75줄 -->

<div class="in">
	<form action="process_create.php" method="POST">
		<p><input type="text" name="name" placeholder="Who?" /></p>
		<p><textarea name="post" rows="5" cols="47" placeholder="What's Happening?"></textarea></p>
		<input class="button" name="submit" type="submit" value="Upload!" />
	</form>
</div>
{% endhighlight %}
POST 방식으로 요청하는 php코드입니다. 여기서 name, post 애트리뷰트로 전달할 텍스트를 입력받습니다. submit을 누르면 process_create.php 페이지로 이동합니다.
{% highlight php %}
<!-- process_create.php 50~74줄 -->

<?php

$conn = mysqli_connect('localhost', 'jaycho', '*****','db');
if($conn === false) {
	die("ERROR: Could not connect. " . mysqli_connect_error());
}

if(isset($_POST['submit'])) {
	$name = $_POST['name'];
	$post = $_POST['post'];

	$query = "INSERT INTO Posts(name, post) VALUES ('$name','$post')";
	$result = mysqli_query($conn, $query);

	if($result) {
		echo "<h2>INSERTED SUCCESSFULLY</h2>";
	}
	else {
		echo "<h2>FAILED TO INSERT</h2>";
	}
}
mysqli_close($conn);
?>
{% endhighlight %}
MySQL 계정으로 연결하고, 에러처리도 해주고, index.html 에서 POST방식으로 넘겨준 텍스트들을 name과 post 변수에 저장한 후, INSERT 구분을 query라는 변수에 저장합니다. 그 후 MySQL에 방금의 쿼리를 날려서 리턴 값이 참이면 글이 잘 등록됬다는 메시지를 화면에 출력합니다. 마지막으로 연결을 닫습니다.
- - -
# 4. 포트포워딩
_(안타깝게도 제 홈네트워크에서 라우터 장비가 두개가 있는데 상단에 있는 라우터의 관리자페이지에 접근이 안되서(..) 포트포워딩을 하지 못했지만 어쨌든 포트포워딩 방법에 관해 간략하게 설명하고 싶습니다. 참고로 다른 곳에서는 네트워크 구성이 단순해서 포트포워딩에 성공했었는데 이번에는 안타깝게 외부접속을 못하게 되었네요.)_

중요한 코드는 아래와 같습니다 :  
{% highlight text %}
route
ip route
{% endhighlight %}
route 혹은 ip route를 입력해서 게이트웨이 주소를 확인합니다.  


공유기 관리자 페이지에 접속합니다. 제 경우는 iptime이고 로그인해줍니다. 이름과 암호는 네트워크 이름과 비밀번호 동일하게 입력합니다.  


관리도구를 클릭합니다.  


고급설정 -> NAT/라우터 관리 -> 포트포워드 설정에 들어갑니다.  
규칙 이름은 아무거나 적으셔도 괜찮습니다.  
내부 IP주소는 라즈베리파이의 ip주소,  
프로토콜은 TCP 그대로,  
외부포트는 3000번대로 해주시구요,  
내부포트는 80으로 설정해두었습니다.  


고급설정 -> 특수기능 -> DDNS 설정으로 들어갑니다.  
라즈베리파이의 IP주소를 등록 IP주소로 적습니다.  
DDNS를 설정해주어야 하는 이유는, 제 공유기를 포함해서 대부분의 홈네트워크에선 유동 IP를 쓰기 때문에 IP가 자주 변해서 DDNS서버에 바뀐 IP를 지속적으로 업데이트 시켜줘야 동일한 도메인을 접속자가 정상적으로 이용할 수 있기 때문입니다.  
- - -
그러나 안타깝게도 제가 네트워크 설정을 제대로 못한 탓에 외부접속이 불가능해서 결국엔 내부망에서만 사용 가능한 서버를 구축하게 되었습니다.. 아무튼 긴글 읽어주셔서 감사합니다.
- - -
## 참고자료
[Ubuntu 18.04에 LAMP ( Apache2, MySQL , PHP 7) 설치하는 방법](https://webnautes.tistory.com/1185){:target="_blank"}