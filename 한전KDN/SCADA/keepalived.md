# KeepAlived를 이용한 haproxy서비스 이중화
> 준비사항
> 1. keepalived 패키지 (AppStream Repository)
> 2. haproxy가 설치된 2대의 서버 (haproxy01, haproxy02)
> 3. Virtual IP Address (10.60.1.60)

## 1. 패키지를 설치한다. 
```
dnf install -y keepalived
```

## 2. keepalived 서버(haproxy01, haproxy02)간 방화벽을 오픈한다.
```
firewall-cmd --add-rich-rule='rule protocol value="vrrp" accept' --permanent
firewall-cmd --reload
```

## 3. keepalived 설정 (/etc/keepalived/keepalived.conf
```
! C
