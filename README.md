# Jarkom-Modul-2-F13-2022


## Penyelesaian Soal Praktikum Modul 1 Jaringan Komputer

* Muhammad Andi Akbar Ramadhan (5025201264)
* Natya Madya Marciola	(5025201238)
* Neisa Hibatillah Alif	(5025201170

## Nomor 1
WISE akan dijadikan sebagai DNS Master, Berlint akan dijadikan DNS Slave, dan Eden akan digunakan sebagai Web Server. Terdapat 2 Client yaitu SSS, dan Garden. Semua node terhubung pada router Ostania, sehingga dapat mengakses internet

### Penyelesaian
![image](https://user-images.githubusercontent.com/80830860/198825987-39875322-033a-4325-b188-4194191e9e43.png)

Diatas merupakan gambar dari topografi yang telah terbuat.

Konfigurasinya : <br>
**Router Ostania**
```
# DHCP config for eth0
auto eth0
iface eth0 inet dhcp

# Static config for eth1
auto eth1
iface eth1 inet static
	address 10.35.1.1
	netmask 255.255.255.0

# Static config for eth2
auto eth2
iface eth2 inet static
	address 10.35.2.1
	netmask 255.255.255.0

# Static config for eth3
auto eth3
iface eth3 inet static
	address 10.35.3.1
	netmask 255.255.255.0

```
**WISE**
```
# Static config for eth0
auto eth0
iface eth0 inet static
	address 10.35.3.2
	netmask 255.255.255.0
	gateway 10.35.3.1

```
**SSS**
```
# Static config for eth0
auto eth0
iface eth0 inet static
	address 10.35.1.2
	netmask 255.255.255.0
	gateway 10.35.1.1

```
**Garden**
```
# Static config for eth0
auto eth0
iface eth0 inet static
	address 10.35.1.3
	netmask 255.255.255.0
	gateway 10.35.1.1

```
**Berlint**
```
# Static config for eth0
auto eth0
iface eth0 inet static
	address 10.35.2.2
	netmask 255.255.255.0
	gateway 10.35.2.1

```
**Eden**
```
# Static config for eth0
auto eth0
iface eth0 inet static
	address 10.35.2.3
	netmask 255.255.255.0
	gateway 10.35.2.1

```
Kemudian untuk menghubungkannya ke internet gunakan ```iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 10.35.0.0/16``` di Ostania,
dan ```echo nameserver 192.168.122.1 > /etc/resolv.conf``` di SSS, Garden, Berlindt, Eden.

## Nomor 2
Untuk mempermudah mendapatkan informasi mengenai misi dari Handler, bantulah Loid membuat website utama dengan akses wise.yyy.com dengan alias www.wise.yyy.com pada folder wise

### Penyelesaian
- Lakukan perintah pada WISE
```
nano /etc/bind/named.conf.local
```
- Kemudian isi konfigurasi domain wise.f13.com sebagai berikut
```zone "wise.f13.com" {
    type master;
    file "/etc/bind/wise/wise.f13.com";
};
```
- Buat folder baru bernama wise di dalam /etc/bind
```
mkdir /etc/bind/wise
```
- Copykan file db.local pada path /etc/bind ke dalam folder wise yang baru saja dibuat dan ubah namanya menjadi wise.f13.com
```
cp /etc/bind/db.local /etc/bind/wise/wise.f13.com
```
- Kemudian buka file wise.f13.com tadi dan edit seperti berikut
```
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     wise.f13.com. root.wise.f13.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      wise.f13.com.
@       IN      A       10.35.3.2       ; IP WISE
www     IN      CNAME   wise.f13.com. 
```
- Restart bind dengan perintah
```
service bind9 restart
```
- Pada client SSS dan Garden arahkan nameserver menuju IP WISE dengan mengedit file resolv.conf dengan mengetikkan perintah
``` 
nano /etc/resolv.conf
```
- Kemudian isi dengan IP dari WISE 
```nameserver 10.35.3.2```
- Untuk mencoba koneksi DNS, lakukan perintah berikut pada client SSS dan Garden
```
ping wise.f13.com -c 5
ping www.wise.f13.com -c 5
host -t CNAME www.wise.f13.com
```

### NOMOR 3
Setelah itu ia juga ingin membuat subdomain eden.wise.yyy.com dengan alias www.eden.wise.yyy.com yang diatur DNS-nya di WISE dan mengarah ke Eden

### Penyelesaian
- Pada WISE, edit file /etc/bind/wise/wise.f13.com dengan menambahkan konfigurasi berikut
```
eden    IN      A       10.35.2.3       ; IP Eden
www.eden        IN      CNAME   eden.wise.f13.com.
```
![image](https://user-images.githubusercontent.com/91374949/197987328-c8d47110-498b-41de-ac79-379ed0539bfa.png)
- Restart bind dengan perintah
```
service bind9 restart
```
- Kemudian coba ping ke subdomain dengan perintah berikut pada client SSS atau Garden
```
ping eden.wise.f13.com -c 5
ping www.eden.wise.f13.com -c 5
host -t CNAME www.eden.wise.f13.com
```

### NOMOR 4
Buat juga reverse domain untuk domain utama

### Penyelesaian
- Edit file /etc/bind/named.conf.local 
```
nano /etc/bind/named.conf.local
```
- Tambahkan konfiguasi berikut
```
zone "3.35.10.in-addr.arpa" {
    type master;
    file "/etc/bind/wise/3.35.10.in-addr.arpa";
};
```
![image](https://user-images.githubusercontent.com/91374949/197989577-5379def2-0a47-4fe8-b303-3d96f5e28c99.png)
- Copykan file db.local pada path /etc/bind ke dalam folder wise yang baru dibuat dan ubah namanya menjadi 3.35.10.in-addr.arpa
```
cp /etc/bind/db.local /etc/bind/wise/3.35.10.in-addr.arpa
```
- Kemudian edit file tersebut seperti gambar di bawah ini 
![image](https://user-images.githubusercontent.com/91374949/197990169-1a402f05-2431-41e5-b6a5-7362b93c7234.png)
- Restart bind 
```
service bind9 restart
```
- Untuk mengecek konfigurasi sudah benar atau belum, lakukan perintah berikut pada client SSS atau Garden
```
host -t PTR 10.35.3.2
```
![image](https://user-images.githubusercontent.com/91374949/198828930-4135e686-3ab3-4e12-a2c2-5aeabfa72490.png)

### NOMOR 5

Agar dapat tetap dihubungi jika server WISE bermasalah, buatlah juga Berlint sebagai DNS Slave untuk domain utama

Pada Berlint, lakukan ```apt-get update``` dan ```apt-get install bind9 -y```.
Kemudian, ```nano /etc/bind/named.conf.local``` dan masukkan konfigurasi zone berikut :
```
zone "wise.f13.com" {
	type slave;
	masters { "IP dari WISE"; };
	file "/var/lib/bind/wise/wise.f13.com";
};
```
Restart bind9 service pada Berlint setelah konfigurasi selesai.

Pada WISE, lakukan konfigurasi dengan ```nano /etc/bind/named.conf.local```.
Kemudian, edit zone wise.f13.com yang sudah ada dengan :
```
zone "wise.f13.com" {		
	type master;
	notify yes;
	also-notify { "IP Berlint"; }; 
	allow-transfer { "IP Berlint"; }; 
	file "/etc/bind/wise/wise.f13.com";
};
```
Lakukan restart pada service bind9 di Wise juga.

Untuk mengetes, stop service bind9 pada Wise, masukkan IP Berlint pada etc/resolv.conf di SSS atau Garden, dan lakukan ping.

### NOMOR 6
Karena banyak informasi dari Handler, buatlah subdomain yang khusus untuk operation yaitu operation.wise.yyy.com dengan alias www.operation.wise.yyy.com yang didelegasikan dari WISE ke Berlint dengan IP menuju ke Eden dalam folder operation

### Penyelesaian
- Pada WISE, tambahkan konfigurasi berikut pada file ```/etc/bind/wise/wise.f13.com```
```
ns1     IN      A       10.35.2.2       ; IP Berlint
operation       IN      NS      ns1
```
![image](https://user-images.githubusercontent.com/91374949/198836622-d63118ad-1ba1-431e-b194-0ef95f3d8597.png)
- Kemudian, pada file ```/etc/bind/named.conf.options```, comment line ```dnssec-validation auto;``` lalu tambahkan ```allow-query{any;};```
- Restart bind 
```
service bind9 restart
```
- Kemudian pada Berlint hal yang sama pada file ```/etc/bind/named.conf.options```, comment line ```dnssec-validation auto;``` lalu tambahkan ```allow-query{any;};```
- Kemudian, tambahkan konfigurasi berikut pada ```/etc/bind/named/conf.local```
```
zone "operation.wise.f13.com" {
    type master;
    file "/etc/bind/operation/operation.wise.f13.com";
};
```
![image](https://user-images.githubusercontent.com/91374949/198837472-91f8493e-eaf7-43de-8cce-5770888b098f.png)

- Buat folder baru bernama ```operation``` dengan 
```
mkdir /etc/bind/operation
```
- Tambahkan konfigurasi berikut pada file ```/etc/bind/operation/operation.wise.f13.com```
```
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     operation.wise.f13.com. root.operation.wise.f13.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      operation.wise.f13.com.
@       IN      A       10.35.2.3
www     IN      CNAME   operation.wise.f13.com.
```
- Restart bind 
```
service bind9 restart
```
- Untuk mencoba testing, lakukan perintah berikut pada client SSS atau Garden
```
ping operation.wise.f13.com
ping www.operation.wise.f13.com
host -t CNAME www.operation.wise.f13.com
```
### NOMOR 7
Untuk informasi yang lebih spesifik mengenai Operation Strix, buatlah subdomain melalui Berlint dengan akses strix.operation.wise.yyy.com dengan alias www.strix.operation.wise.yyy.com yang mengarah ke Eden

### Penyelesaian
- Pada Berlint, tambahkan konfigurasi berikut pada file ```/etc/bind/operation/operation.wise.f13.com```
```
strix   IN      A       10.35.2.3
www.strix       IN      CNAME   strix.operation.wise.f13.com.
```
![image](https://user-images.githubusercontent.com/91374949/198837273-b391b61e-7c5a-4fcc-b5c4-c0a2cbd083e2.png)
- Restart bind 
```
service bind9 restart
```
- Kemudian, lakukan testing pada client SSS atau Garden dengan melakukan perintah berikut
```
ping strix.operation.wise.f13.com
ping www.strix.operation.wise.f13.com
```

### NOMOR 8
Setelah melakukan konfigurasi server, maka dilakukan konfigurasi Webserver. Pertama dengan webserver www.wise.yyy.com. Pertama, Loid membutuhkan webserver dengan DocumentRoot pada /var/www/wise.yyy.com

### Penyelesaian
- Pada Eden, install aplikasi apache, PHP, dan libapache2-mod-php7.0
```
apt-get update
apt-get install apache2 -y
apt-get install php -y
apt-get install libapache2-mod-php7.0 -y
```

- Copy file 000-default.conf ke wise.f13.com.conf 
```
cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/wise.f13.com.conf
```

- Tambahkan bagian berikut pada file wise.f13.com.conf
![image](https://user-images.githubusercontent.com/72701806/198831067-722b6f71-041f-4348-8a77-602d41113090.png)


- Aktifkan konfigurasi ```a2ensite wise.f13.com.conf```

- Lakukan instalasi berikut untuk mendownload dan meng-unzip file yang akan digunakan
```
apt-get install wget -y
apt-get install unzip -y
```

- Pindah ke directory /var/www menggunakan perintah ```cd /var/www```

- Download file yang dibutuhkan menggunakan perintah berikut

### NOMOR 9
Setelah itu, Loid juga membutuhkan agar url www.wise.yyy.com/index.php/home dapat menjadi www.wise.yyy.com/home

### Penyelesaian
- Restart apache pada Eden menggunakan ```service apache2 restart```, kemudian jalankan perintah ```a2enmod rewrite```

-
