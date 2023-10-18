# Jarkom-Modul-2-A10-2023

## Anggota Kelompok

| NRP        | Name                        |
| ---------- | --------------------------- |
| 5025211081 | Gabriella Natasya Br Ginting|
| 5025211102 | Adhira Riyanti Amanda       |

### Daftar Isi
- [Soal 1](#soal-1)
- [Soal 2](#soal-2)
- [Soal 3](#soal-3)
- [Soal 4](#soal-4)
- [Soal 5](#soal-5)
- [Soal 6](#soal-6)
- [Soal 7](#soal-7)
- [Soal 8](#soal-8)
- [Soal 9](#soal-9)
- [Soal 10](#soal-10)
- [Soal 11](#soal-11)
- [Soal 12](#soal-12)
- [Soal 13](#soal-13)
- [Soal 14](#soal-14)
- [Soal 15](#soal-15)
- [Soal 16](#soal-16)
- [Soal 17](#soal-17)
- [Soal 18](#soal-18)
- [Soal 19](#soal-19)
- [Soal 20](#soal-20)

### Topologi
![Topologi](/images/Topologi/05.png)

### Soal 1
Yudhistira akan digunakan sebagai DNS Master, Werkudara sebagai DNS Slave, Arjuna merupakan Load Balancer yang terdiri dari beberapa Web Server yaitu Prabakusuma, Abimanyu, dan Wisanggeni. Buatlah topologinya sesuai dengan gambar topologi diatas

![Soal 1](/images/Soal%201/topologi.png)

#### Konfigurasi:
- **Pandudewanata**
  ```
    auto eth0
    iface eth0 inet dhcp

    auto eth1
    iface eth1 inet static
        address 10.4.1.1
        netmask 255.255.255.0

    auto eth2
    iface eth2 inet static
        address 10.4.2.1
        netmask 255.255.255.0

    auto eth3
    iface eth3 inet static
        address 10.4.3.1
        netmask 255.255.255.0
  ```
- **Nakula**
  ```
    auto eth0
    iface eth0 inet static
        address 10.4.1.2
        netmask 255.255.255.0
        gateway 10.4.1.1
  ```
- **Sadewa**
  ```
    auto eth0
    iface eth0 inet static
        address 10.4.1.3
        netmask 255.255.255.0
        gateway 10.4.1.1
  ```
- **Yudhistira**
  ```
    auto eth0
    iface eth0 inet static
        address 10.4.2.2
        netmask 255.255.255.0
        gateway 10.4.2.1
  ```
- **Werkudara**
  ```
    auto eth0
    iface eth0 inet static
        address 10.4.2.3
        netmask 255.255.255.0
        gateway 10.4.2.1
  ```
- **Prabukusuma**
  ```
    auto eth0
    iface eth0 inet static
        address 10.4.3.2
        netmask 255.255.255.0
        gateway 10.4.3.1
  ```
- **Abimanyu**
  ```
    auto eth0
    iface eth0 inet static
        address 10.4.3.3
        netmask 255.255.255.0
        gateway 10.4.3.1
  ```
- **Wisanggeni**
  ```
    auto eth0
    iface eth0 inet static
        address 10.4.3.4
        netmask 255.255.255.0
        gateway 10.4.3.1
  ```
- **Arjuna**
  ```
    auto eth0
    iface eth0 inet static
        address 10.4.3.5
        netmask 255.255.255.0
        gateway 10.4.3.1
  ```
- **Servers Summary**
  ```
  Pandudewanata	: 10.4.1.1 (Switch 1)
  Nakula	        : 10.4.1.2
  Sadewa	        : 10.4.1.3
  Pandudewanata	: 10.4.2.1 (Switch 2)
  Yudhistira	: 10.4.2.2
  Werkudara	: 10.4.2.3
  Pandudewanata	: 10.4.3.1 (Switch 3)
  Prabukusuma	: 10.4.3.2
  Abimanyu	: 10.4.3.3
  Wisanggeni	: 10.4.3.4
  Arjuna	        : 10.4.3.5
  ```

#### Inisiasi ``.bashrc``

- **Pandudewanata**
  ```
  iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 10.4.0.0/16
  cat /etc/resolv.conf
  echo 'nameserver 192.168.122.1' > /etc/resolv.conf
  ```
- **Master & Slave**
  ```
  echo 'nameserver 192.168.122.1' > /etc/resolv.conf
  apt-get update
  apt-get install bind9 -y      
  ```
- **Client**
  ```
  echo -e '
  nameserver 10.4.2.2 # IP Yudhistira
  nameserver 10.4.2.3 # IP Werkudara
  nameserver 192.168.122.1
  ' > /etc/resolv.conf
  apt-get update
  apt-get install dnsutils -y
  ```

### Soal 2
Buatlah website utama pada node arjuna dengan akses ke arjuna.yyy.com dengan alias www.arjuna.yyy.com dengan yyy merupakan kode kelompok.

#### Script

Setup file `/etc/bind9/named.conf.local` dan `/etc/bind/jarkom/arjuna.A10.com` 

**DNS Master (Yudhistira)**
```
echo 'zone "arjuna.A10.com" {
        type master;
        file "/etc/bind/jarkom/arjuna.A10.com";
        allow-transfer { 10.4.3.5; }; // IP Arjuna
};' > /etc/bind9/named.conf.local

mkdir /etc/bind/jarkom

cp /etc/bind/db.local /etc/bind/jarkom/arjuna.A10.com

echo '
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     arjuna.A10.com. root.arjuna.A10.com. (
                        2023101201      ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      arjuna.A10.com.
@       IN      A       10.4.2.2     ; IP Yudhistira
www     IN      CNAME   arjuna.A10.com.' > /etc/bind/jarkom/arjuna.A10.com

service bind9 restart
```

#### Test Ping

![Soal 2](/images/Soal%202/No2.png)

### Soal 3
Dengan cara yang sama seperti soal nomor 2, buatlah website utama dengan akses ke abimanyu.yyy.com dan alias www.abimanyu.yyy.com.

#### Script

Menambahkan konfigurasi abimanyu.A10.com pada file `/etc/bind9/named.conf.local` dan melakukan setup pada file `/etc/bind/jarkom/abimanyu.A10.com` 

**DNS Master (Yudhistira)**
```
echo 'zone "arjuna.A10.com" {
        type master;
        file "/etc/bind/jarkom/arjuna.A10.com";
        allow-transfer { 10.4.3.5; }; // IP Arjuna
};

zone "abimanyu.A10.com" {
        type master;
        file "/etc/bind/jarkom/abimanyu.A10.com";
        allow-transfer { 10.4.3.3; }; // IP Abimanyu
};' > /etc/bind/named.conf.local

cp /etc/bind/db.local /etc/bind/jarkom/abimanyu.A10.com

echo '
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     abimanyu.A10.com. root.abimanyu.A10.com. (
                        2023101201      ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      abimanyu.A10.com.
@       IN      A       10.4.2.2     ; IP Yudhistira
www     IN      CNAME   abimanyu.A10.com.' > /etc/bind/jarkom/abimanyu.A10.com

service bind9 restart
```
#### Test Ping

![Soal 3](/images/Soal%203/No3.png)

### Soal 4
Kemudian, karena terdapat beberapa web yang harus di-deploy, buatlah subdomain parikesit.abimanyu.yyy.com yang diatur DNS-nya di Yudhistira dan mengarah ke Abimanyu.

#### Script

Menambahkan line `parikesit IN    A       10.4.3.3     ; IP Abimanyu ` pada `/etc/bind/jarkom/abimanyu.A10.com` untuk membuat subdomain parikesit.abimanyu.A10.com

**DNS Master (Yudhistira)**
```
echo '
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     abimanyu.A10.com. root.abimanyu.A10.com. (
                        2023101201      ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      abimanyu.A10.com.
@       IN      A       10.4.2.2     ; IP Yudhistira
www     IN      CNAME   abimanyu.A10.com.
parikesit IN    A       10.4.3.3     ; IP Abimanyu' > /etc/bind/jarkom/abimanyu.A10.com

service bind9 restart
```

#### Test Ping

![Soal 4](/images/Soal%204/No4.png)

### Soal 5
Buat juga reverse domain untuk domain utama. (Abimanyu saja yang direverse)

#### Script

Sebelum melakukan reverse domain dari Abimanyu, kita perlu untuk membalik IPnya. IP Abimanyu adalah 10.4.3.3 sehingga setelah dibalik menjadi 3.3.4.10.

Kemudian, kita menambahkan konfigurasi 3.4.10.in-addr.arpa pada file `/etc/bind9/named.conf.local` dan melakukan setup pada file `/etc/bind/jarkom/3.4.10.in-addr.arpa` 

**DNS Master (Yudhistira)**
```
echo 'zone "arjuna.A10.com" {
        type master;
        file "/etc/bind/jarkom/arjuna.A10.com";
        allow-transfer { 10.4.3.5; }; // IP Arjuna
};

zone "abimanyu.A10.com" {
        type master;
        file "/etc/bind/jarkom/abimanyu.A10.com";
        allow-transfer { 10.4.3.3; }; // IP Abimanyu
};

zone "3.4.10.in-addr.arpa" {
        type master;
        file "/etc/bind/jarkom/3.4.10.in-addr.arpa";
};' > /etc/bind/named.conf.local

cp /etc/bind/db.local /etc/bind/jarkom/3.4.10.in-addr.arpa

echo ';
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     abimanyu.A10.com. root.abimanyu.A10.com. (
                        2003101201      ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
3.4.10.in-addr.arpa. IN      NS      abimanyu.A10.com.
3                       IN      PTR     abimanyu.A10.com.
5                       IN      PTR     arjuna.A10.com.' > /etc/bind/jarkom/3.4.10.in-addr.arpa

service bind9 restart
```

#### Testing

![Soal 5](/images/Soal%205/No5.png)

### Soal 6
Agar dapat tetap dihubungi ketika DNS Server Yudhistira bermasalah, buat juga Werkudara sebagai DNS Slave untuk domain utama.

#### Script

Pada DNS Master diperlukan setup `also-notify` dan `allow-transfer` agar memberikan izin kepada IP yang dituju.

**DNS Master (Yudhistira)**
```
echo 'zone "arjuna.A10.com" {
        type master;
        file "/etc/bind/jarkom/arjuna.A10.com";
        allow-transfer { 10.4.2.3; }; // IP Werkudara
        also-notify { 10.4.2.3; }; // IP Werkudara
};

zone "abimanyu.A10.com" {
        type master;
        notify yes;
        also-notify { 10.4.2.3; }; // IP Werkudara
        allow-transfer { 10.4.2.3; }; // IP Werkudara
        file "/etc/bind/jarkom/abimanyu.A10.com";
};

zone "3.4.10.in-addr.arpa" {
        type master;
        file "/etc/bind/jarkom/3.4.10.in-addr.arpa";
};' > /etc/bind/named.conf.local

service bind9 restart

service bind9 stop
```

**DNS Slave (Werkudara)**

```
echo 'zone "abimanyu.A10.com" {
    type slave;
    masters { 10.4.2.2; }; // Masukan IP Yudhistira
    file "/var/lib/bind/abimanyu.A10.com";
};' >> /etc/bind/named.conf.local

service bind9 restart
```

#### Test Ping
![Soal 6](/images/Soal%206/No6.png)
![Soal 6](/images/Soal%206/No6(1).png)

### Soal 7
Seperti yang kita tahu karena banyak sekali informasi yang harus diterima, buatlah subdomain khusus untuk perang yaitu baratayuda.abimanyu.yyy.com dengan alias www.baratayuda.abimanyu.yyy.com yang didelegasikan dari Yudhistira ke Werkudara dengan IP menuju ke Abimanyu dalam folder Baratayuda.

#### Script

Menambahkan line `ns1     IN      A       10.4.2.3     ; IP Werkudara` dan
`baratayuda IN   NS      ns1` pada `/etc/bind/jarkom/abimanyu.A10.com`. Kemudian comment out line `dnssec-validation auto;`dan menambahkan `allow-query{any;};` pada `/etc/bind/named.conf.options`

**DNS Master (Yudhistira)**
```
echo ';
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     abimanyu.A10.com. root.abimanyu.A10.com. (
                        2023101201      ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      abimanyu.A10.com.
@       IN      A       10.4.2.2     ; IP Yudhistira
www     IN      CNAME   abimanyu.A10.com.
parikesit IN    A       10.4.3.3     ; IP Abimanyu
ns1     IN      A       10.4.2.3     ; IP Werkudara
baratayuda IN   NS      ns1' > /etc/bind/jarkom/abimanyu.A10.com

echo "options {
    directory \"/var/cache/bind\";

    // If there is a firewall between you and nameservers you want
    // to talk to, you may need to fix the firewall to allow multiple
    // ports to talk.  See http://www.kb.cert.org/vuls/id/800113

    // If your ISP provided one or more IP addresses for stable
    // nameservers, you probably want to use them as forwarders.
    // Uncomment the following block, and insert the addresses replacing
    // the all-0's placeholder.

    // forwarders {
    //      0.0.0.0;
    // };

    //========================================================================
    // If BIND logs error messages about the root key being expired,
    // you will need to update your keys.  See https://www.isc.org/bind-keys
    //========================================================================
    //dnssec-validation auto;

    allow-query { any; };
    auth-nxdomain no;
    listen-on-v6 { any; };
};" > /etc/bind/named.conf.options

service bind9 restart
```
Konfigurasi baratayuda pada file `/etc/bind/named.conf.local` dan comment out line `dnssec-validation auto;`dan menambahkan `allow-query{any;};` pada `/etc/bind/named.conf.options`

**DNS Slave (Werkudara)**
```
echo 'zone "baratayuda.abimanyu.A10.com" {
        type master;
        file "/etc/bind/baratayuda/baratayuda.abimanyu.A10.com";
};' >> /etc/bind/named.conf.local

mkdir /etc/bind/baratayuda

cp /etc/bind/db.local /etc/bind/baratayuda/baratayuda.abimanyu.A10.com

echo '
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     baratayuda.abimanyu.A10.com. root.baratayuda.abimanyu.A10.com. (
                        2023101201      ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      baratayuda.abimanyu.A10.com.
@       IN      A       10.4.3.3     ; IP Abimanyu
www     IN      CNAME   baratayuda.abimanyu.A10.com.' > /etc/bind/baratayuda/baratayuda.abimanyu.A10.com

echo "options {
    directory \"/var/cache/bind\";

    // If there is a firewall between you and nameservers you want
    // to talk to, you may need to fix the firewall to allow multiple
    // ports to talk.  See http://www.kb.cert.org/vuls/id/800113

    // If your ISP provided one or more IP addresses for stable
    // nameservers, you probably want to use them as forwarders.
    // Uncomment the following block, and insert the addresses replacing
    // the all-0's placeholder.

    // forwarders {
    //      0.0.0.0;
    // };

    //========================================================================
    // If BIND logs error messages about the root key being expired,
    // you will need to update your keys.  See https://www.isc.org/bind-keys
    //========================================================================
    //dnssec-validation auto;

    allow-query { any; };
    auth-nxdomain no;
    listen-on-v6 { any; };
};" > /etc/bind/named.conf.options

service bind9 restart
```

#### Test Ping
![Soal 7](/images/Soal%207/No7.png)

### Soal 8
Untuk informasi yang lebih spesifik mengenai Ranjapan Baratayuda, buatlah subdomain melalui Werkudara dengan akses rjp.baratayuda.abimanyu.yyy.com dengan alias www.rjp.baratayuda.abimanyu.yyy.com yang mengarah ke Abimanyu.

#### Script

Menambahkan line `rjp             IN      A       10.4.3.3     ; IP Abimanyu` dan `www.rjp         IN      CNAME   rjp.baratayuda.abimanyu.A10.com.` untuk membuat subdomain rjp

**DNS Slave (Werkudara)**
```
echo ';
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     baratayuda.abimanyu.A10.com. root.baratayuda.abimanyu.A10.com. (
                        2023101201      ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@               IN      NS      baratayuda.abimanyu.A10.com.
@               IN      A       10.4.3.3     ; IP Abimanyu
www             IN      CNAME   baratayuda.abimanyu.A10.com.
rjp             IN      A       10.4.3.3     ; IP Abimanyu
www.rjp         IN      CNAME   rjp.baratayuda.abimanyu.A10.com.' > /etc/bind/baratayuda/baratayuda.abimanyu.A10.com

service bind9 restart
```

#### Test Ping
![Soal 8](/images/Soal%208/No8.png)

### Soal 9
Arjuna merupakan suatu Load Balancer Nginx dengan tiga worker (yang juga menggunakan nginx sebagai webserver) yaitu Prabakusuma, Abimanyu, dan Wisanggeni. Lakukan deployment pada masing-masing worker.

#### Script
Untuk ketiga nginx worker ditambahkan konfigurasi pada file `/etc/nginx/sites-available/jarkom`. Dan untuk load balancer ditambahkan juga konfigurasi pada file `/etc/nginx/sites-available/lb-jarkom`.

**Nginx Worker ( Prabakusuma, Abimanyu, dan Wisanggeni )**

```
 server {

 	listen 80;

 	root /var/www/jarkom;

 	index index.php index.html index.htm;
 	server_name _;

 	location / {
 			try_files $uri $uri/ /index.php?$query_string;
 	}

 	# pass PHP scripts to FastCGI server
 	location ~ \.php$ {
 	include snippets/fastcgi-php.conf;
 	fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
 	}

 location ~ /\.ht {
 			deny all;
 	}

 	error_log /var/log/nginx/jarkom_error.log;
 	access_log /var/log/nginx/jarkom_access.log;
 }
```

Membuat `symnlink` dan restart `nginx`
```
ln -s /etc/nginx/sites-available/jarkom /etc/nginx/sites-enabled

service nginx restart
```

**Load Balancer Nginx ( Arjuna )**
```
 # Default menggunakan Round Robin
upstream backend {
  server 10.4.3.2; # IP PrabaKusuma
  server 10.4.3.3; # IP Abimanyu
  server 10.4.3.4; # IP Wisanggeni
}

server {
  listen 80;
  server_name arjuna.A10.com www.arjuna.A10.com;

  location / {
    proxy_pass http://backend;
  }
}
```

Membuat `symnlink`
```
ln -s /etc/nginx/sites-available/lb-jarkom /etc/nginx/sites-enabled
```

#### Test Lynx

Lakukan testing pada client `Nakula` atau `Sadewa`
```
lynx http://10.4.3.2
lynx http://10.4.3.3
lynx http://10.4.3.4
```

- Pada ip 10.4.3.2

  ![Soal 9](/images/Soal%209/2.jpg)

- Pada ip 10.4.3.3

  ![Soal 9](/images/Soal%209/1.jpg)

- Pada ip 10.4.3.4

  ![Soal 9](/images/Soal%209/3.jpg)

### Soal 10
Kemudian gunakan algoritma Round Robin untuk Load Balancer pada Arjuna. Gunakan server_name pada soal nomor 1. Untuk melakukan pengecekan akses alamat web tersebut kemudian pastikan worker yang digunakan untuk menangani permintaan akan berganti ganti secara acak. Untuk webserver di masing-masing worker wajib berjalan di port 8001-8003. Contoh
    - Prabakusuma:8001
    - Abimanyu:8002
    - Wisanggeni:8003

#### Script
Lakukan update konfigurasi untuk ketiga nginx worker pada file `/etc/nginx/sites-available/jarkom`, dan untuk load balancer pada file `/etc/nginx/sites-available/lb-jarkom`.

**Nginx Worker ( Prabakusuma, Abimanyu, dan Wisanggeni )**
```
 server {

 	listen [port masing masing worker];

 	root /var/www/jarkom;

 	index index.php index.html index.htm;
 	server_name _;

 	location / {
 			try_files $uri $uri/ /index.php?$query_string;
 	}

 	# pass PHP scripts to FastCGI server
 	location ~ \.php$ {
 	include snippets/fastcgi-php.conf;
 	fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
 	}

 location ~ /\.ht {
 			deny all;
 	}

 	error_log /var/log/nginx/jarkom_error.log;
 	access_log /var/log/nginx/jarkom_access.log;
 }
```

**Load Balancer Nginx ( Arjuna )**
```
upstream backend {
  server 10.4.3.2:8001; # IP PrabaKusuma
  server 10.4.3.3:8002; # IP Abimanyu
  server 10.4.3.4:8003; # IP Wisanggeni
}
```

#### Test Lynx

Lakukan testing pada client `Nakula` atau `Sadewa`
```
lynx http://10.4.3.2:8001
lynx http://10.4.3.3:8002
lynx http://10.4.3.4:8003
```

- Pada port 8001

  ![Soal 10](/images/Soal%2010/1.jpg)

- Pada port 8002

  ![Soal 10](/images/Soal%2010/2.jpg)

- Pada port 8003

  ![Soal 10](/images/Soal%2010/3.jpg)

### Soal 11
Selain menggunakan Nginx, lakukan konfigurasi Apache Web Server pada worker Abimanyu dengan web server www.abimanyu.yyy.com. Pertama dibutuhkan web server dengan DocumentRoot pada /var/www/abimanyu.yyy



### Soal 12
Setelah itu ubahlah agar url www.abimanyu.yyy.com/index.php/home menjadi www.abimanyu.yyy.com/home.

### Soal 13
Selain itu, pada subdomain www.parikesit.abimanyu.yyy.com, DocumentRoot disimpan pada /var/www/parikesit.abimanyu.yyy

### Soal 14
Pada subdomain tersebut folder /public hanya dapat melakukan directory listing sedangkan pada folder /secret tidak dapat diakses (403 Forbidden).

### Soal 15
Buatlah kustomisasi halaman error pada folder /error untuk mengganti error kode pada Apache. Error kode yang perlu diganti adalah 404 Not Found dan 403 Forbidden.

#### Script
Lakukan update konfigurasi pada file `/etc/apache2/sites-available/parikesit.abimanyu.A10.com.conf`

**Abimanyu**
```
echo '<VirtualHost *:80>
  ServerAdmin webmaster@localhost
  DocumentRoot /var/www/parikesit.abimanyu.A10
  ServerName parikesit.abimanyu.A10.com
  ServerAlias www.parikesit.abimanyu.A10.com

  <Directory /var/www/parikesit.abimanyu.A10/public>
          Options +Indexes
  </Directory>

  <Directory /var/www/parikesit.abimanyu.A10/secret>
          Options -Indexes
  </Directory>

  Alias "/public" "/var/www/parikesit.abimanyu.A10/public"
  Alias "/secret" "/var/www/parikesit.abimanyu.A10/secret"

  ErrorDocument 404 /error/404.html
  ErrorDocument 403 /error/403.html

  ErrorLog ${APACHE_LOG_DIR}/error.log
  CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>' > /etc/apache2/sites-available/parikesit.abimanyu.A10.com.conf

service apache2 restart
```

#### Test Lynx

Lakukan testing pada client `Nakula` atau `Sadewa`
```
lynx parikesit.abimanyu.A10.com/testerror
lynx parikesit.abimanyu.A10.com/secret
```

- 404 Not Found

  ![Soal 15](/images/Soal%2015/1.jpg)

  ![Soal 15](/images/Soal%2015/2.jpg)

- 403 Forbidden

  ![Soal 15](/images/Soal%2015/3.jpg)

  ![Soal 15](/images/Soal%2015/4.jpg)

### Soal 16
Buatlah suatu konfigurasi virtual host agar file asset www.parikesit.abimanyu.yyy.com/public/js menjadi 
www.parikesit.abimanyu.yyy.com/js 

#### Script
Lakukan update konfigurasi pada file `/etc/apache2/sites-available/parikesit.abimanyu.A10.com.conf` agar file asset `/var/www/parikesit.abimanyu.A10/public/js` 

**Abimanyu**
```
echo '<VirtualHost *:80>
  ServerAdmin webmaster@localhost
  DocumentRoot /var/www/parikesit.abimanyu.A10
  ServerName parikesit.abimanyu.A10.com
  ServerAlias www.parikesit.abimanyu.A10.com

  <Directory /var/www/parikesit.abimanyu.A10/public>
          Options +Indexes
  </Directory>

  <Directory /var/www/parikesit.abimanyu.A10/secret>
          Options -Indexes
  </Directory>

  Alias "/public" "/var/www/parikesit.abimanyu.A10/public"
  Alias "/secret" "/var/www/parikesit.abimanyu.A10/secret"
  Alias "/js" "/var/www/parikesit.abimanyu.A10/public/js"

  ErrorDocument 404 /error/404.html
  ErrorDocument 403 /error/403.html

  ErrorLog ${APACHE_LOG_DIR}/error.log
  CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>' > /etc/apache2/sites-available/parikesit.abimanyu.A10.com.conf
```

#### Test Lynx

Lakukan testing pada client `Nakula` atau `Sadewa`
```
lynx parikesit.abimanyu.A10.com/js
```

![Soal 16](/images/Soal%2016/1.jpg)

### Soal 17
Agar aman, buatlah konfigurasi agar www.rjp.baratayuda.abimanyu.yyy.com hanya dapat diakses melalui port 14000 dan 14400.

#### Script
Lakukan update konfigurasi pada file `/etc/apache2/sites-available/rjp.baratayuda.abimanyu.A10.com.conf`

**Abimanyu**
```
echo -e '<VirtualHost *:14000 *:14400>
  ServerAdmin webmaster@localhost
  DocumentRoot /var/www/rjp.baratayuda.abimanyu.A10
  ServerName rjp.baratayuda.abimanyu.A10.com
  ServerAlias www.rjp.baratayuda.abimanyu.A10.com

  ErrorDocument 404 /error/404.html
  ErrorDocument 403 /error/403.html

  ErrorLog ${APACHE_LOG_DIR}/error.log
  CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>' > /etc/apache2/sites-available/rjp.baratayuda.abimanyu.A10.com.conf

a2ensite rjp.baratayuda.abimanyu.A10.com.conf

service apache2 restart
```

#### Test Lynx

Lakukan testing pada client `Nakula` atau `Sadewa`
```
lynx rjp.baratayuda.abimanyu.A10.com:14000
lynx rjp.baratayuda.abimanyu.A10.com:14400
```

- Port 14000

  ![Soal 17](/images/Soal%2017/1.jpg)

- Port 14400

  ![Soal 17](/images/Soal%2017/2.jpg)

- Port 14444 ( Port selain 14000 & 14400 )

  ![Soal 17](/images/Soal%2017/3.jpg)

### Soal 18
Untuk mengaksesnya buatlah autentikasi username berupa “Wayang” dan password “baratayudayyy” dengan yyy merupakan kode kelompok. Letakkan DocumentRoot pada /var/www/rjp.baratayuda.abimanyu.yyy.

#### Script
Lakukan update konfigurasi pada file `/etc/apache2/sites-available/rjp.baratayuda.abimanyu.A10.com.conf`. Untuk melakukan autentikasi pada server tersebut, kita memerlukan `AuthType` dan Require `Valid-User`.

**Abimanyu**
```
echo -e '<VirtualHost *:14000 *:14400>
  ServerAdmin webmaster@localhost
  DocumentRoot /var/www/rjp.baratayuda.abimanyu.A10
  ServerName rjp.baratayuda.abimanyu.A10.com
  ServerAlias www.rjp.baratayuda.abimanyu.A10.com

  <Directory /var/www/rjp.baratayuda.abimanyu.A10>
          AuthType Basic
          AuthName "Restricted Content"
          AuthUserFile /etc/apache2/.htpasswd
          Require valid-user
  </Directory>

  ErrorDocument 404 /error/404.html
  ErrorDocument 403 /error/403.html

  ErrorLog ${APACHE_LOG_DIR}/error.log
  CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>' > /etc/apache2/sites-available/rjp.baratayuda.abimanyu.A10.com.conf

a2ensite rjp.baratayuda.abimanyu.A10.com.conf

service apache2 restart
```

- Menjalankan command berikut untuk menginisiasi username dan password.
```
htpasswd -c -b /etc/apache2/.htpasswd Wayang baratayudaA10
```

#### Test Lynx

Lakukan testing pada client `Nakula` atau `Sadewa`
```
lynx rjp.baratayuda.abimanyu.A10.com:14000
lynx rjp.baratayuda.abimanyu.A10.com:14400
```

![Soal 18](/images/Soal%2018/1.jpg)

![Soal 18](/images/Soal%2018/2.jpg)


### Soal 19
Buatlah agar setiap kali mengakses IP dari Abimanyu akan secara otomatis dialihkan ke www.abimanyu.yyy.com (alias)

#### Script
Lakukan update konfigurasi pada file `/etc/apache2/sites-available/000-default.conf`

**Abimanyu**
```
echo -e '<VirtualHost *:14000 *:14400>
  ServerAdmin webmaster@localhost
  DocumentRoot /var/www/rjp.baratayuda.abimanyu.A10
  ServerName rjp.baratayuda.abimanyu.A10.com
  ServerAlias www.rjp.baratayuda.abimanyu.A10.com

  ErrorDocument 404 /error/404.html
  ErrorDocument 403 /error/403.html

  ErrorLog ${APACHE_LOG_DIR}/error.log
  CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>' > /etc/apache2/sites-available/rjp.baratayuda.abimanyu.A10.com.conf

apache2ctl configtest

service apache2 restart
```

#### Test Lynx

Lakukan testing pada client `Nakula` atau `Sadewa`
```
lynx 10.4.3.3
```

![Soal 19](/images/Soal%2019/1.jpg)

### Soal 20
Karena website www.parikesit.abimanyu.yyy.com semakin banyak pengunjung dan banyak gambar gambar random, maka ubahlah request gambar yang memiliki substring “abimanyu” akan diarahkan menuju abimanyu.png.