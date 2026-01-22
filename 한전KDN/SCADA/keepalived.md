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

## 3. kernel parameter 수정(/etc/sysctl.conf)
```
net.ipv4.ip_nonlocal_bind = 1
net.ipv4.ip_forward = 1

sysctl -p
```

## 3. keepalived 설정 (/etc/keepalived/keepalived.conf
```
! Configuration File for keepalived

global_defs {
   router_id HAPROXY_01
   vrrp_skip_check_adv_addr
   vrrp_garp_interval 0
   vrrp_gna_interval 0
}

vrrp_script check_haproxy {
    script "pidof haproxy"
    interval 2
    weight -20
}

vrrp_instance VI_1 {
    state MASTER
    interface ens192
    virtual_router_id 51
    priority 101
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        10.60.1.60
    }
}
```
