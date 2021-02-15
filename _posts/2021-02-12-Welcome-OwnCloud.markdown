---
layout: post
title:  "OwnCloud란 무엇인가"
date:   2021-02-12 19:00:00 +0900
categories: jekyll update
---
#1. OwnCloud 란 무엇인지와 용도
===
OwnCloud 란 무엇일까요? 말씀드리기 앞서 클라우드에 대해서 설명드리겠습니다.
- 클라우드란 인터넷 통해 서비스를 제공받는 서비스 입니다. 
- 그래서 OwnCloud란 인터넷을 통한NAS 형태의 오픈소스 파일 호스팅 서비스를 만들고 사용하기 위한 클라이언트-서버 소프트웨어 입니다.
- 파일을 저장할때는 네트워크 기반 파일 저장(클라우드 스토리지)을 이용하며 OwnCloud의 사용자에게 장치에서의 모든 파일에 액세스를 제공, 보안, 제어 및 통합을 제공합니다.  

#2. OwnCloud 의 설치 과정
===
* 2-1. DB, http 패키지 설치  
`yum -y install mysql-community-server wget mariadb-server mariadb httpd`
* 2-2. 필요한 php 패키지 설치  
`yum -y install php56w php56w-opcache php56w-mysql php56w-gd php56w-mbstring php56w-xml php56w-intl` 
* 2-3. 방화벽 데몬 disable 및 영구 disable 설정  
`systemctl disable --now firewalld` 
* 2-4. 압축 파일 다운로드  
`wget https://download.owncloud.org/community/owncloud-complete-20201216.zip`
* 2-5. 서비스 할 서버에서 압축파일 해제  
`unzip 20201216.zip`
* 2-6. mysql config 추가, mysql 기본 인코딩 UTF-8 설정  
`vi /etc/my.cnf`

```
[client]
default-character-set = utf8

[mysqld]
init_connect="SET NAMES utf8"
character-set-server = utf8
collation-server = utf8_general_ci
character-set-client-handshake=FALSE
init_connect="SET collation_connection = utf8_general_ci"

[mysqldump]
default-character-set = utf8

[mysql]
default-character-set = utf8
```

* 2-7. mysql 서비스 enable 설정  
`systemctl enable --now mysqld`

```
 + mysql -u root 
  - (mysql 서비스 root 접속)

 + mysql> update user set password=password('새비밀번호') WHERE user='root';
  - (mysql 서비스 root PW 설정) [option]

 + mysql> create database <DB이름> default character set utf8;
  - (owncloud에서 사용할 DB 생성)

 + mysql> create user '<계정명>'@'%' identified by '<password>' 
  - (외부접속 가능한 계정 생성)

 + mysql> create user '<계정명>'@'localhost' identified by '<password>' 
  - (로컬접속만 가능한 계정 생성) [option]

 + mysql> grant all privileges on <DB이름>.* to '아이디'@'%'; 
  - (owncloud DB에 새로 생성한 계정 권한 부여)

 + mysql> grant all privileges on <DB이름>.* to '아이디'@'localhost'; 
  - (owncloud DB에 새로 생성한 계정 권한 부여)

 + mysql> flush privileges; 
  - (mysql 설정 저장)
```

* 2-8. owncloud config 종속 옵션 및  설정  
`vi /etc/httpd/conf.d/owncloud.conf`

```
Alias /owncloud "/var/www/html/owncloud/"

<Directory "/var/www/html/owncloud">

    Options +FollowSymLinks

    AllowOverride All



    <IfModule mod_dav.c>

      Dav off

    </IfModule>



    SetEnv HOME /var/www/html/owncloud

    SetEnv HTTP_HOME /var/www/html/owncloud

</Directory>



<Directory "/var/www/html/owncloud/data/">

  # just in case if .htaccess gets disabled

  Require all denied

</Directory>
```

* 2-9. apache 계정 owncloud 디렉토리 소유권 설정  
`chown -R apache:apache owncloud`
* 2-10. apache 계정 owncloud 디렉토리 권한 설정  
`chmod -R 770 owncloud`
* 2-11. http 서비스 재시작  
`systemctl restart httpd`


#3. OwnCloud 의 공통기능
===
기능이 여러가지 있기에 제일 많이 사용하는 기능들에 대해서만 소개하도록 하겠습니다.

 - 파일 업로드 : 가장 기본적인 NAS와 같은 형태로 Drag and Drop 또는 Upload 이용한 파일 업로드가 가능합니다.
네트워크 기반의 클라우드 스토리지의 가장 기본적이고도 핵심 기술이라고 할 수 있겠는데요.
OwnCloud를 이용하는 사용자끼리 공유를 하거나 파일을 저장하기 위하여 사용합니다.
또한 WebDAV 기능을 통해 저장소내 파일을 관리할 수 있습니다.
그리고 업로드를 통해 파일을 올렸을 경우에 언제 올렸는지에 대한 히스토리를 확인하실 수 있습니다.

 - 댓글 기능
업로드한 파일에 대해 댓글을 작성하고 간단한 내용들을 적어서 히스토리를 남길 수 있습니다.

 - 공유 기능
업로드한 파일에 대해 OwnCloud를 사용하는 특정 User들에게 파일을 공유하여 권한에 따라 다운로드 혹은
파일 삭제, 수정 이 가능합니다.
또한 "공유 링크" 라는 추가적인 기능을 통해서 URL을 만들어 공유하거나 암호를 설정하여 보안에 한층을 더할수 있습니다.

 - 사용자관리
권한 또는 보안에서 중요하다고 볼 수 있는 사용자를 관리하는 기능입니다.
admin이라는 그룹에 속해있는 사용자는 OwnCloud에서 사용자들을 관리할 수 있는 SuperUser(root)가 됩니다.
일반그룹을 생성하여 여러 사용자들을 나누어 관리할 수 있습니다.
그리고 그 일반그룹에 대한 admin 권한을 부여하여 그룹에 대한 관리를 할 수 있도록 설정할 수 있습니다.


