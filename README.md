# Clustered Web Architecture


![Untitled-removebg-preview](https://github.com/user-attachments/assets/daf9b836-dd02-41a3-9dd7-eccd5b2db768)

Bu sistemde, iki tane HAProxy sunucusunu kullanarak yüksek erişilebilirlik sağlayan bir yapı kuruyoruz. İstemci istekleri önce bu HAProxy sunucularına yönlendirilir. Bu sunucular, istekleri arka plandaki web sunucularına yönlendirmekle sorumlu olur. Eğer birinci HAProxy sunucusu çalışmazsa, diğeri devreye girer. HAProxy sunucuları, sağlık kontrolü yaparak arka plandaki sunucuların erişilebilir olduğunu doğrular. Web sunucularının SSL sertifikaları HAProxy üzerinden yönetilir; bu da, SSL sertifikalarının doğru şekilde yapılandırıldığından ve geçerli olduğundan emin olunmasını sağlar. Ayrıca, HAProxy sunucuları için Keepalived kullanarak sanal bir IP (VIP) yapılandırılır ve bu VIP, dışarıdan erişim için kullanılır. Eğer bir HAProxy sunucusu kapalı olursa, Keepalived başka bir sunucuyu etkinleştirir ve bu sayede kesintisiz hizmet sağlanır. Bu yapı, yüksek erişilebilirlik ve kesintisiz hizmet için esnek ve güvenilir bir çözüm sunar.

https://amigdala.net.tr Web sitesine giriş sağlayarak kontrol edebilirsiniz.

## 1. HAProxy Kurulumu

### 1.1. HAProxy'yi Kurun

Ubuntu sisteminizde HAProxy'yi kurmak için aşağıdaki komutu çalıştırın:

```bash
sudo apt update
sudo apt install haproxy
```

### 1.2. HAProxy Yapılandırması

HAProxy'nin yapılandırma dosyası genellikle `/etc/haproxy/haproxy.cfg` yolundadır. Aşağıdaki örnek, temel bir HAProxy yapılandırmasını göstermektedir:

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

Keepalived'in yapılandırma dosyası genellikle `/etc/keepalived/keepalived.conf` yolundadır. Aşağıdaki örnek, HAProxy ile birlikte çalışacak bir Keepalived yapılandırmasını göstermektedir:

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

## 2.3. Keepalived'i Başlatma ve Etkinleştirme

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


## 5. HaProxy Üzerinde SSL Kurulumu

SSL (Secure Sockets Layer), internet üzerindeki veri iletimini şifreleyen bir güvenlik protokolüdür. Bu, kullanıcıların web siteleri aracılığıyla bilgi gönderirken, üçüncü şahısların bu bilgilere erişmesini zorlaştırarak güvenli bir iletişim sağlar. Şimdi serverımıza SSL kuracağız. Aşağıdaki komutu yazarak gerekli indirmeleri yapalım.

```
sudo apt install certbot python3-certbot-apache
```

SSL sertfika kurulumunu gerçekleştirmek için HaProxy servisini durdurmanız gerekmektedir. 

```
systemctl stop haproxy.service
```

Terminale aşağıdaki gibi yazıyoruz

```
sudo certbot certonly --standalone -d domain.com--non-interactive --agree-tos --email user@domain.com
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Requesting a certificate for domain.com

Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/domain.com/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/domain.com/privkey.pem
This certificate expires on 2024-11-07.
These files will be updated when the certificate renews.
Certbot has set up a scheduled task to automatically renew this certificate in the background.

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
If you like Certbot, please consider supporting our work by:
 * Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
 * Donating to EFF:                    https://eff.org/donate-le
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```

HAProxy, SSL sertifikalarını ve özel anahtarları aynı dosyada ister. Bu yüzden fullchain.pem ve privkey.pem dosyalarını birleştirmelisin.

```
cat /etc/letsencrypt/live/domain.com/fullchain.pem /etc/letsencrypt/live/domain.com/privkey.pem > /etc/letsencrypt/live/domain.com/haproxy.pem
```

HAProxy'yi SSL trafiği için yapılandırmak üzere /etc/haproxy/haproxy.cfg dosyasını güncelle. Aşağıdaki örnek konfigürasyonu kullanabilirsin:

```
frontend https_front
    bind *:443 ssl crt /etc/letsencrypt/live/domain.com/haproxy.pem
    mode http
    option httplog
    default_backend http_back
```

Bu konfigürasyon, gelen HTTPS isteklerini 443 portu üzerinden alır ve onları belirtilen backend sunucularına yönlendirir.


Yaptığın değişikliklerin etkili olması için HAProxy'yi yeniden başlat:

```
sudo systemctl restart haproxy
```

Web sitenize giderek test edebilirsiniz.

----------------------------------------------------------------
HAProxy ve Keepalived kurulum ve yapılandırma adımlarını kapsamlı bir şekilde açıklar. Her adımın doğru uygulandığından emin olun ve yapılandırmalarınızı test edin.

Okuduğunuz için teşekkürler.






