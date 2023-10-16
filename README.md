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

### Soal 7
Seperti yang kita tahu karena banyak sekali informasi yang harus diterima, buatlah subdomain khusus untuk perang yaitu baratayuda.abimanyu.yyy.com dengan alias www.baratayuda.abimanyu.yyy.com yang didelegasikan dari Yudhistira ke Werkudara dengan IP menuju ke Abimanyu dalam folder Baratayuda.

### Soal 8
Untuk informasi yang lebih spesifik mengenai Ranjapan Baratayuda, buatlah subdomain melalui Werkudara dengan akses rjp.baratayuda.abimanyu.yyy.com dengan alias www.rjp.baratayuda.abimanyu.yyy.com yang mengarah ke Abimanyu.

### Soal 9
Arjuna merupakan suatu Load Balancer Nginx dengan tiga worker (yang juga menggunakan nginx sebagai webserver) yaitu Prabakusuma, Abimanyu, dan Wisanggeni. Lakukan deployment pada masing-masing worker.

### Soal 10
Kemudian gunakan algoritma Round Robin untuk Load Balancer pada Arjuna. Gunakan server_name pada soal nomor 1. Untuk melakukan pengecekan akses alamat web tersebut kemudian pastikan worker yang digunakan untuk menangani permintaan akan berganti ganti secara acak. Untuk webserver di masing-masing worker wajib berjalan di port 8001-8003. Contoh
    - Prabakusuma:8001
    - Abimanyu:8002
    - Wisanggeni:8003

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

### Soal 16
Buatlah suatu konfigurasi virtual host agar file asset www.parikesit.abimanyu.yyy.com/public/js menjadi 
www.parikesit.abimanyu.yyy.com/js 

### Soal 17
Agar aman, buatlah konfigurasi agar www.rjp.baratayuda.abimanyu.yyy.com hanya dapat diakses melalui port 14000 dan 14400.

### Soal 18
Untuk mengaksesnya buatlah autentikasi username berupa “Wayang” dan password “baratayudayyy” dengan yyy merupakan kode kelompok. Letakkan DocumentRoot pada /var/www/rjp.baratayuda.abimanyu.yyy.

### Soal 19
Buatlah agar setiap kali mengakses IP dari Abimanyu akan secara otomatis dialihkan ke www.abimanyu.yyy.com (alias)

### Soal 20
Karena website www.parikesit.abimanyu.yyy.com semakin banyak pengunjung dan banyak gambar gambar random, maka ubahlah request gambar yang memiliki substring “abimanyu” akan diarahkan menuju abimanyu.png.