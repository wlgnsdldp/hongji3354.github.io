---
layout: post
title: "CentOS7에 MariaDB 설치하기"
tags: [설치]
comments: true
---

CentOS7에 MariaDB 10.4 설치하기

# 1. yum Repository 추가

아래 홈페이지에 접속하시면 OS와 MairaDB Version에 따라 Repository 추가 방법을 제공해 줍니다.

[Setting up MariaDB Repositories](https://downloads.mariadb.org/mariadb/repositories/#mirror=hosting90)

저는 CentOS7과 MariaDB 10.4 버전이 필요하므로 아래와 같이 선택하였습니다.

![](./images/centos7-mariadb-install/1.PNG)



``/etc/yum.repos.d/MriaDB.repo``에 MariaDB 설치를 위한 Repository 정보를 추가해 줍니다.

```
# MariaDB 10.4 CentOS repository list - created 2019-12-26 14:54 UTC
# http://downloads.mariadb.org/mariadb/repositories/
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.4/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
```



# 2. MariaDB 설치

MariaDB 를 설치합니다.

```
sudo yum install MariaDB-server MariaDB-clienty
```
설치가 정상적으로 되었는지 확인합니다. 
```
rpm -qa | grep MariaDB
```
설치가 정상적으로 진행 되었다면 아래의 그림과 같이 나타납니다.
![](./images/centos7-mariadb-install/2.PNG)

MariaDB 를 시작 합니다.
```
systemctl start mariadb
```

MariaDB가 정상적으로 실행되었는지 확인합니다.
```
systemctl status mariadb
```
정상적으로 실행 되었다면 아래의 그림 같이 **Active가 active (running)**로 되어 있습니다.
![](./images/centos7-mariadb-install/3.PNG)



# 3. MariaDB Setting

## 3-1. 보안설정

```
sudo mysql_secure_installation
```
보안 설정의 질문은 아래와 같습니다.

- Switch to unix_socket authentication [Y/n]
  - unix_socket란 시스템 root 사용자인 root@localhost의 경우 **비밀번호 없이 로그인 할 수 있는 기능** 으로 저는 비밀번호를 사용하기 위해 n를 선택했습니다.
- Change the root password? [Y/n]
  - **root 사용자의 비밀번호 변경** 여부 입니다. 설치시 root 계정에 대한 비밀번호를 설정한 적이 없으므로 Y를 선택 했습니다.
- Remove anonymous users? [Y/n]
  - **익명사용자 제거여부**로, 보안측면에서는 제거하는 것 이 좋기 때문에 Y를 선택했습니다.
- Disallow root login remotely? [Y/n]
  - **원격지에서 root 로그인 비허용 여부**로, 저는 root 계정을 외부에서 사용하기 위해 Y를 선택하였습니다.  보안을 생각한다면 N를 선택해야 합니다.
- Remove test databases and access to it? [Y/n]
  - **test database의 삭제 여부**로 test database가 불필요 하므로 Y를 선택했습니다.
- Reload privilege tables now? [Y/N]
  - **설정 정보 저장 여부**로 잘못 설정하신 부분이 있다면 N를 선택 후 설정을 다시 진행하시면 됩니다.

![](./images/centos7-mariadb-install/4.PNG)

## 3-2. encoding 설정

개발을 하다보면 character set 문제가 생각보다 많이 발생합니다. DB의 경우 기본 character set으로 개발을 진행하다가 다른 character set으로 변경시 데이터를 다 삭제 후 INSERT 해야 할 수 있기 때문에 **설치시 꼭 character set을 설정해 줘야 합니다!**

저는 character set을 모두 **UTF8로 설정**하였습니다.

1. /etc/my.cnf.d/mysql-client.cnf

```
[mysql]
default-character-set=utf8

[mysqldump]
default-character-set=utf8
```

2. /etc/my.cnf.d/server.cnf
```
[mysqld]
collation-server = utf8_unicode_ci
init-connect='SET NAMES utf8'
character-set-server = utf8
```

변경 사항을 적용하기 위해 mariaDB를 재시작 합니다.
```
systemctl restart mariadb
```

character set이 정상적으로 변경되었는지 확인하기 위해 mariaDB Console에 접속합니다.
```
mysql -u root -p
```

character set이 정상적으로 적용되었다면 아래 이미지 처럼 Server,Db, Client, Conn의 Characterset이 변경되어 있습니다.
```
status
```
![](./images/centos7-mariadb-install/4.PNG)



# 4. MariaDB 외부 접속 설정

외부 접속을 하기 위해서는 설정 파일 변경과 계정에 대한 작업이 필요합니다.

## 4-1. MairaDB에서 외부접속 허용

/etc/my.cnf.d/server.cnf에 들어가셔서 **bind-address**를 주석처리 하거나 또는 0.0.0.0으로 세팅해 줍니다. 

```
#bind-address=0.0.0.0
```

그 다음으로 변경사항을 적용하기 위해 MariaDB를 재시작 합니다.

```
systemctl restart mariadb
```

## 4-2. 계정 생성 및 권한 부여

계정 생성 및 권한 부여는 필요에 따라서 설정해주시면 됩니다.

### 4-2-1. 계정 생성

```
/* local에서만 접속 가능한 계정*/
create user '계정명'@'localhost' identified by '계정 비밀번호'

/* 모든 외부 IP에서 접속 가능한 계정*/
create user '계정명'@'%' identified by '계정 비밀번호'

/* 특정 외부 IP에서 접속 가능한 계정*/
create user '계정명'@'192.168.0.%' identified by '계정 비밀번호'
```



### 4-2-2. 계정 권한 부여

```
/* local에서만 접속 가능한 계정*/
grant all privileges on 데이터베이스이름.* to '계정명'@'localhost'

/* 모든 외부 IP에서 접속 가능한 계정*/
grant all privileges on 데이터베이스이름.* to '계정명'@'%'

/* 특정 외부 IP에서 접속 가능한 계정*/
grant all privileges on 데이터베이스이름.* to '192.168.0.%'
```

>만약 위의 과정을 모두 진행후에도 외부에서 Connection이 안되신다면 방화벽에 걸렸을 확률이 높으므로 MariaDB 기본 포트인 3306을 풀어 주시면 됩니다.