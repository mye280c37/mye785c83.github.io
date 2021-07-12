---
layout: post
title : EC2 보안강화하기
category: Web
tags: [AWS, EC2]
complete: false
---

##### 참고사이트
* [fail2ban 으로 SSH 서버 강화하기](https://www.lesstif.com/security/fail2ban-ssh-43843899.html)


## EC2 인스턴스 설정

#### SSH port 변경
<span class="text-highlight warning"> 일단 보류 </span>

기본 SSH 포트는 22인데 이를 다른 포트로 변경해주는 것이 좋다.


#### password authentication for SSH

<span class="text-highlight warning"> 일단 보류: 굳이 밖에서 서버 접속할 일 없음 </span>

```
sudo passwd `username`
```

EC2의 경우 `username=ubuntu`


##### sshd_conf file 업데이트

<span class="text-highlight warning"> 일단 보류 </span>

`/etc/ssh/sshd_config` 파일을 열어 `PasswordAuthentication` 값을 yes로 변경한다.

```
# /etc/ssh/sshd_config

PasswordAuthentication yes
```

#### SSH 재시작
<span class="text-highlight warning"> 일단 보류 </span>

```
sudo service ssh restart
```


## fail2ban 설정


#### fail2ban의 역할