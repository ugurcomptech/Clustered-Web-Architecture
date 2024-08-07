# HAProxy ve Keepalived Kurulumu ve Yapılandırması

Bu belge, HAProxy ve Keepalived'in nasıl kurulacağını ve yapılandırılacağını adım adım açıklar. HAProxy, yük dengelemesi sağlar, Keepalived ise yüksek erişilebilirlik sağlar.

## 1. HAProxy Kurulumu

### 1.1. HAProxy'yi Kurun

Ubuntu sisteminizde HAProxy'yi kurmak için aşağıdaki komutu çalıştırın:

```bash
sudo apt update
sudo apt install haproxy
```

### 1.2. HAProxy Yapılandırması

HAProxy'nin yapılandırma dosyası genellikle /etc/haproxy/haproxy.cfg yolundadır. Aşağıdaki örnek, temel bir HAProxy yapılandırmasını göstermektedir:

```
global
    log /dev/log local0
    log /dev/log local1 notice
    chroot /var/lib/haproxy
    stats socket /run/haproxy/admin.sock mode 660
    stats timeout 30s
    user haproxy
    group haproxy
    daemon

defaults
    log     global
    option  httplog
    option  dontlognull
    timeout connect 5000ms
    timeout client  50000ms
    timeout server  50000ms

frontend http_front
    mode 80
    bind *:80
    default_backend http_back

backend http_back,
    mode 80
    balance roundrobin
    server webserver1 192.168.1.10:80 check
    server webserver2 192.168.1.20:80 check

```


Bu yapılandırma, gelen HTTP trafiğini webserver1 ve webserver2 sunucularına dengeler.

### 1.3. HAProxy'yi Başlatma ve Etkinleştirme

```
sudo systemctl start haproxy
sudo systemctl enable haproxy
```

### 2. Keepalived Kurulumu

Ubuntu sisteminizde Keepalived'i kurmak için aşağıdaki komutu çalıştırın:

```
sudo apt update
sudo apt install keepalived
```

2.2. Keepalived Yapılandırması

Keepalived'in yapılandırma dosyası genellikle /etc/keepalived/keepalived.conf yolundadır. Aşağıdaki örnek, HAProxy ile birlikte çalışacak bir Keepalived yapılandırmasını göstermektedir:

Primary HAProxy Sunucusu İçin:

```
vrrp_instance VI_1 {
    state MASTER
    interface ens5
    virtual_router_id 51
    priority 101
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1234
    }
    virtual_ipaddress {
        172.31.47.100
    }
}
```

Secondary HAProxy Sunucusu İçin:

```
vrrp_instance VI_1 {
    state BACKUP
    interface ens5
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1234
    }
    virtual_ipaddress {
        172.31.47.100
    }
}
```

2.3. Keepalived'i Başlatma ve Etkinleştirme

Keepalived'i başlatın ve sistem başlangıcında otomatik olarak başlatılması için etkinleştirin:

```
sudo systemctl start keepalived
sudo systemctl enable keepalived
```


## 3. Son Kontroller

HAProxy ve Keepalived yapılandırmalarınızı kontrol edin

```
sudo systemctl status haproxy
sudo systemctl status keepalived
```


## 4. Ekstra Notlar

1. DNS ayarlarınızı, HAProxy'nin  IP adresine yönlendirdiğinizden emin olun.
2. Keepalived yapılandırmalarında priority değerleri, master ve yedek sunucular arasındaki önceliği belirler.
3. Keepalived ve HAProxy'yi doğru yapılandırdığınızdan ve sistemlerin doğru çalıştığından emin olmak için testler yapın.


----------------------------------------------------------------
HAProxy ve Keepalived kurulum ve yapılandırma adımlarını kapsamlı bir şekilde açıklar. Her adımın doğru uygulandığından emin olun ve yapılandırmalarınızı test edin.

Okuduğunuz için teşekkürler.






