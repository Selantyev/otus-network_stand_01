# network_stand_01

# Теоритическая часть

Построить следующую архитектуру

Сеть office1

    192.168.2.0/26 - адрес подсети dev
    192.168.2.1 - 192.168.2.62/26 - адрес подсети dev
    192.168.2.63/26 - broadcast адрес dev

    192.168.2.64/26 - адрес подсети test servers
    192.168.2.65 - 192.168.2.126/26 - адреса хостов test servers
    192.168.2.127/26 - broadcast адрес test servers

    192.168.2.128/26 - адрес подсети managers
    192.168.2.129 - 192.168.2.190/26 - адреса хостов managers
    192.168.2.191/26 - broadcast адрес managers
    
    192.168.2.192/26 - адрес подсети office hardware
    192.168.2.193 - 192.168.2.254/26 - адреса хостов office hardware
    192.168.2.255/26 - broadcast адрес office hardware

Сеть office2

    192.168.1.0/25 - адрес подсети dev
    192.168.1.1 - 192.168.126/25 - адреса хостов dev
    192.168.1.127/25 - broadcast адрес dev

    192.168.1.128/26 - адрес подсети test servers
    192.168.1.129 - 192.168.1.190/26 - адреса хостов test servers
    192.168.1.191 - broadcast адрес test servers

    192.168.1.192/26 - адрес подсети office hardware
    192.168.1.193 - 192.168.2.254/26 - адреса хостов office hardware
    192.168.1.255 - broadcast адрес office hardware

Сеть central

    192.168.0.0/28 - адрес подсети directors
    192.168.0.1 - 192.168.0.14/28 - адреса хостов directors
    192.168.0.15 - broadcast адрес directors

    192.168.0.32/28 - адрес подсети office hardware
    192.168.0.33 - 192.168.0.46/28 - адреса хостов office hardware
    192.168.0.47/28 - broadcast адрес office hardware
    
    192.168.0.64/26 - адрес подсети wifi
    192.168.0.65 - 192.168.0.126/26 - адреса хостов wifi
    192.168.0.127/26 - broadcast адрес wifi

Для роутеров выбрана подсеть 192.168.255.0/29

# Практическая часть

Вагрантом развёрнуты 7 виртуалок: inetRouter, centralRouter, office1Router, office2Router, centralServer, office1Server, office2Server

На всех хостах кроме inetRouter отключены NAT интерфейсы:

`ifdown eth0`

```
vi /etc/sysconfig/network-scripts/ifcfg-eth0
ONBOOT=no
```

На inetRouter, centralRouter, office1Router и office2Router включен ip forwarding:

`echo 'net.ipv4.conf.all.forwarding=1' >> /etc/sysctl.conf`

На inetRouter добавлено правило POSTROUTING:

`iptables -t nat -A POSTROUTING ! -d 192.168.0.0/16 -o eth0 -j MASQUERADE`

```
yum install iptables-services -y
systemctl enable iptables
service iptables save
```

На inetRouter добавлены маршруты:

`vi /etc/sysconfig/network-scripts/route-eth1`

```
192.168.0.0/28 via 192.168.255.2 dev eth1
192.168.0.32/28 via 192.168.255.2 dev eth1
192.168.0.64/26 via 192.168.255.2 dev eth1
192.168.1.0/26 via 192.168.255.4 dev eth1
192.168.1.128/26 via 192.168.255.4 dev eth1
192.168.1.192/26 via 192.168.255.4 dev eth1
192.168.2.0/26 via 192.168.255.3 dev eth1
192.168.2.64/26 via 192.168.255.3 dev eth1
192.168.2.128/26 via 192.168.255.3 dev eth1
192.168.2.192/26 via 192.168.255.3 dev eth1
```

Сервера centralServer, office1Server и office2Server ходят в инет через inetRouter (см. screenshots)
Сервера centralServer, office1Server и office2Server видят друг-друга (см. screenshots)
