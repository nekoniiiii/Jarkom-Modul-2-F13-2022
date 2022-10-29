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
