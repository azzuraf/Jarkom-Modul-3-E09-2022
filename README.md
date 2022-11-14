# Jarkom-Modul-3-E09-2022
### Anggota:
1. Lia Kharisma Putri (5025201034)
2. Azzura Ferliani Ramadhani (5025201190)
3. Ingwer Ludwig Nommensen (5025201259) <br/>
<br/> ![image](https://user-images.githubusercontent.com/52819640/201662428-beca3b2f-f5c5-4a69-8b0b-72540404ed3c.png)<br/>
## Nomor 1
**Loid bersama Franky berencana membuat peta tersebut dengan kriteria WISE sebagai DNS Server, Westalis sebagai DHCP Server, Berlint sebagai Proxy Server** <br/> <br/>
Network configuration pada Ostania:
```
auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet static
	address 10.26.1.1
	netmask 255.255.255.0

auto eth2
iface eth2 inet static
	address 10.26.2.1
	netmask 255.255.255.0

auto eth3
iface eth3 inet static
	address 10.26.3.1
	netmask 255.255.255.0
```
Network configuration pada WISE:
```
auto eth0
iface eth0 inet static
	address 10.26.2.2
	netmask 255.255.255.0
	gateway 10.26.2.1
```
Network Configuration pada Berlint:
```
auto eth0
iface eth0 inet static
	address 10.26.2.3
	netmask 255.255.255.0
	gateway 10.26.2.1
```
Network Configuration pada Westalis:
```
auto eth0
iface eth0 inet static
	address 10.26.2.4
	netmask 255.255.255.0
	gateway 10.26.2.1
```
Setelah itu lakukan ```iptables``` pada Ostania <br/>
``` iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 10.26.0.0/16 ```
Instalasi update dan service pada WISE sebagai DNS Server:
```
echo nameserver 192.168.122.1 > /etc/resolv.conf

apt-get update
apt-get install bind9 -y
service bind9 start
```
Instalasi update dan service pada Berlint sebagai Proxy Server:
```
echo nameserver 192.168.122.1 > /etc/resolv.conf

apt-get update
apt-get install apt-utils
apt-get install squid
service squid restart
```
Instalasi update dan service pada Westalis sebagai DHCP Server:
```
echo nameserver 192.168.122.1 > /etc/resolv.conf

apt-get update -y
apt-get install isc-dhcp-server -y
dhcpd --version

echo 'INTERFACES="eth0"' > /etc/default/isc-dhcp-server
```
## Nomor 2
**dan Ostania sebagai DHCP Relay** <br/> <br/>
Instalasi update dan service pada Ostania:
```
apt-get update
apt-get install isc-dhcp-relay -y
dhcrelay --version

echo 'SERVERS="10.26.2.4"
INTERFACES="eth1 eth2 eth3"
OPTIONS=""' > /etc/default/isc-dhcp-relay
```
Pada file ``` /etc/default/isc-dhcp-relay ```, SERVERS diisikan dengan IP Westalis sebagai DHCP Server.

## Nomor 3 dan 6
**Loid dan Franky menyusun peta tersebut dengan hati-hati dan teliti.** 
**Ada beberapa kriteria yang ingin dibuat oleh Loid dan Franky, yaitu:** <br/>
**1. Semua client yang ada HARUS menggunakan konfigurasi IP dari DHCP Server.** <br/>
**2. Client yang melalui Switch1 mendapatkan range IP dari [prefix IP].1.50 - [prefix IP].1.88 dan [prefix IP].1.120 - [prefix IP].1.155** <br/>
**Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch1 selama 5 menit. Dengan waktu maksimal yang dialokasikan untuk peminjaman alamat IP selama 115 menit.** <br/><br/>
Network Configuration dari SSS, Garden, Eden, NewstonCastle, dan KemonoPark:
```
auto eth0
iface eth0 inet dhcp
```
Pada Westalis sebagai DHCP server, konfigurasi pada file ``` /etc/dhcp/dhcpd.conf ``` untuk client yang ada pada Switch1:
```
subnet 10.26.1.0 netmask 255.255.255.0 {
    range 10.26.1.50 10.26.1.88; 
    range 10.26.1.120 10.26.1.155;
    option routers 10.26.1.1;
    option broadcast-address 10.26.1.255;
    option domain-name-servers 10.26.2.2;
    default-lease-time 300;
    max-lease-time 6900;
```

## Nomor 4 dan 6
**Client yang melalui Switch3 mendapatkan range IP dari [prefix IP].3.10 - [prefix IP].3.30 dan [prefix IP].3.60 - [prefix IP].3.85** <br/>
**Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch3 selama 10 menit. Dengan waktu maksimal yang dialokasikan untuk peminjaman alamat IP selama 115 menit.** <br/><br/>
Pada Westalis sebagai DHCP server, konfigurasi pada file ``` /etc/dhcp/dhcpd.conf ``` untuk client yang ada pada Switch3:
```
subnet 10.26.3.0 netmask 255.255.255.0 {
    range 10.26.3.10 10.26.3.30; 
    range 10.26.3.60 10.26.3.85;
    option routers 10.26.3.1;
    option broadcast-address 10.26.3.255;
    option domain-name-servers 10.26.2.2;
    default-lease-time 600;
    max-lease-time 6900;
```

## No 8 - 12(selesai)
- Buat konfigurasi proxy server pada `berlint` dengan menggunakan `squid` yang telah terinstall.
```bash
echo "
http_port 8080
visible_hostname Berlint
acl available_hour1 time MTWHF 08:00-17:00
acl available_hour2 time AS 00:00-23:59
acl loid-work dstdomain loid-work.com
acl franky-work dstdomain franky-work.com
http_access allow loid-work available_hour1
http_access allow franky-work available_hour1
http_access deny all
" > /etc/squid/squid.conf
service squid restart

service squid restart

echo "bash berlint.sh : done"
```
- Jalankan proxy pada client server dalam hal ini adalah `SSS`  dengan command berkut:
```bash
# Menjalankan proxy pada http dan https (192.177.2.3 adalah ip berlint sebagai proxy server, 8000 adalah port yang digunakan)
export http_proxy="http://192.177.2.3:8080"
export https_proxy="https://192.177.2.3:8080"
```
- Periksa apakah proxy sudah berjalan di client server dengan command berikut  
`` env | grep -i proxy``  
jika sudah berjalan akan ditampilkan proxy yang sedang berjalan sebagai berikut
``
http_proxy=http://192.177.2.3:8080
https_proxy=http://192.177.2.3:8080
``

####  1. Client hanya dapat mengakses internet diluar (selain) hari & jam kerja (senin-jumat 08.00 - 17.00) dan hari libur (dapat mengakses 24 jam penuh)
- Tambahkan konfigurasi berikut pada proxy server yaitu `berlint` pada file `/etc/squid/squid.conf`
```bash 
http_port 8080
visible_hostname berlint

#membuat acl untuk sebagai veriable untuk jam kerja senin-jumat (MTWHF) jam 08:00-17:00
acl AVAILABLE_WORKING time MTWHF 08:00-17:00

# melarang akses internet pada jam kerja
http_a ccess deny AVAILABLE_WORKING
```
#### 2. Adapun pada hari dan jam kerja sesuai nomor (1), client hanya dapat mengakses domain loid-work.com dan franky-work.com (IP tujuan domain dibebaskan) 
- Buat domain loid-work.xom dan francky-work.com terlebih dahulu pada `wise` sebagai DNS server, dengan konfigurasi dalam script file sebagai berikut (pastikan bind9 telah terinstall):
``` bash
echo '
zone "loid-work.com" {
        type master;
        file "/etc/bind/jarkom/loid-work.com";
}; 

zone "franky-work.com" {
        type master;
        file "/etc/bind/jarkom/franky-work.com";
};

' > /etc/bind/named.conf.local

mkdir -p /etc/bind/jarkom

echo '
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     franky-work.com. root.franky-work.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      franky-work.com.
@       IN      A       192.177.2.3     ;IP Berlint 192.177.2.3

' > /etc/bind/jarkom/franky-work.com

echo '
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     loid-work.com. root.loid-work.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      loid-work.com.
@       IN      A       192.177.2.3     ;IP Berlint 192.177.2.3

' > /etc/bind/jarkom/loid-work.com

service bind9 restart
```
- setelah domain terbuat, tambahkan domain loid dan franky-work sebagai acl dnsdomain yanga akan kita allow pada jam kerja dan deny diluar jam kerja. dengan konfigurasi pada `/etc/squid/squid.conf` di `berlint` sehingga konfigurasi menjadi sebagai berkut:
```bash
http_port 8080
visible_hostname berlint

acl AVAILABLE_WORKING time MTWHF 08:00-17:00
acl WORKSITES dstdomain loid-work.com franky-work.com

# membolehkan worksite pada jam kerja
http_access allow WORKSITES AVAILABLE_WORKING
# melarang worksite diluar jam kerja
http_access deny WORKSITES !AVAILABLE_WORKING
http_access deny AVAILABLE_WORKING

# restart squid
# service squid restart
# cek status squid
# service squid status
```
#### 3. Saat akses internet dibuka, client dilarang untuk mengakses web tanpa HTTPS. (Contoh web HTTP: http://example.com)
- Gunakan url_regex untuk melarang semua url yang memiliki pola berupa `diawali dengan "http:" dan "https:"` dengan konfigurasi tambahan pada `squid.conf` pada berlint sebagai berkut:
```bash
http_port 8080
visible_hostname berlint

acl AVAILABLE_WORKING time MTWHF 08:00-17:00
acl WORKSITES dstdomain loid-work.com franky-work.com
#url regex sebagai acl
acl HTTP_REGEX url_regex ^http:.*$
acl HTTPS_REGEX url_regex ^https:.*$

http_access allow WORKSITES AVAILABLE_WORKING
http_access deny WORKSITES !AVAILABLE_WORKING
http_access deny AVAILABLE_WORKING
# menolak akses https pada jam kerja
http_access deny AVAILABLE_WORKING HTTPS_REGEX
# menolak semua akses http pada jam kerja apapun (tidak termasuk worksite yang telah di allow dibagian atas)
http_access deny HTTP_REGEX

# service squid restart
# service squid status
```
#### 4. Agar menghemat penggunaan, akses internet dibatasi dengan kecepatan maksimum 128 Kbps pada setiap host (Kbps = kilobit per second; lakukan pengecekan pada tiap host, ketika 2 host akses internet pada saat bersamaan, keduanya mendapatkan speed maksimal yaitu 128 Kbps)
- Tambahkan bandwidth limit pada konfigurasi `squid.conf` di berlint
```bash
http_port 8080
visible_hostname berlint

acl AVAILABLE_WORKING time MTWHF 08:00-17:00
acl WORKSITES dstdomain loid-work.com franky-work.com
acl HTTP_REGEX url_regex ^http:.*$
acl HTTPS_REGEX url_regex ^https:.*$

http_access allow WORKSITES AVAILABLE_WORKING
http_access deny WORKSITES !AVAILABLE_WORKING
http_access deny AVAILABLE_WORKING
http_access deny AVAILABLE_WORKING HTTPS_REGEX
http_access deny HTTP_REGEX

# konfigurasi limit bandwidth
delay_pools 1
delay_class 1 1
delay_parameters 1 16000/64000

# service squid restart
# service squid status
```

#### 5. Setelah diterapkan, ternyata peraturan nomor (4) mengganggu produktifitas saat hari kerja, dengan demikian pembatasan kecepatan hanya diberlakukan untuk pengaksesan internet pada hari libur
- Tambahkan konfigurasi untuk menentukan bahwa bandwidth limit hanya akan dijalankan pada hari libur yait sabtu dan minggu
```bash
http_port 8080
visible_hostname berlint

acl AVAILABLE_WORKING time MTWHF 08:00-17:00
acl WORKSITES dstdomain loid-work.com franky-work.com
acl HTTP_REGEX url_regex ^http:.*$
acl HTTPS_REGEX url_regex ^https:.*$

http_access allow WORKSITES AVAILABLE_WORKING
http_access deny WORKSITES !AVAILABLE_WORKING
http_access deny AVAILABLE_WORKING
http_access deny AVAILABLE_WORKING HTTPS_REGEX
http_access deny HTTP_REGEX

# acl weekdays sebagi waktu untuk hari minggu (S) dan sabtu (A)
acl WEEKDAYS time SA
delay_pools 1
delay_class 1 1
# melalkukan limit bandwidth hanya pada hari weekdays seperti yang telah didefinisikan pada acl di atas
delay_access 1 allow WEEKDAYS
delay_parameters 1 16000/64000

# service squid restart
# service squid status
```## No 8 - 12(selesai)
- Buat konfigurasi proxy server pada `berlint` dengan menggunakan `squid` yang telah terinstall.
```bash
# backup file dari /etc/squid/squid.conf ke /etc/squid/squid.conf.bak 
mv /etc/squid/squid.conf /etc/squid/squid.conf.bak 

# tambahkan konfigurasi berikut ke /etc/squid/squid.conf
echo '
http_port 8080
visible_hostname Berlint
' > /etc/squid/squid.conf

#restart dan check status dari squid apakan sudah dapat berjalan
service squid restart
service squid status
```
- Jalankan proxy pada client server dalam hal ini adalah `SSS`  dengan command berkut:
```bash
echo "nameserver 10.26.3.13" >> /etc/resolv.conf.d

#now sss could access loid-work.com & franky-work.com
#and also the internet

apt-get update
apt-get upgrade
apt-get install lynx -y

#activate the proxy, makesure to run berlint.sh first
export http_proxy="http://10.26.2.69:8080"
export https_proxy="http://10.26.2.69:8080"
```
- Periksa apakah proxy sudah berjalan di client server dengan command berikut  
`` env | grep -i proxy``  
jika sudah berjalan akan ditampilkan proxy yang sedang berjalan sebagai berikut
``
http_proxy=http://192.177.2.3:8080
https_proxy=http://192.177.2.3:8080
``

####  1. Client hanya dapat mengakses internet diluar (selain) hari & jam kerja (senin-jumat 08.00 - 17.00) dan hari libur (dapat mengakses 24 jam penuh)
- Tambahkan konfigurasi berikut pada proxy server yaitu `berlint` pada file `/etc/squid/squid.conf`
```bash 
http_port 8080
visible_hostname berlint

#membuat acl untuk sebagai veriable untuk jam kerja senin-jumat (MTWHF) jam 08:00-17:00
acl AVAILABLE_WORKING time MTWHF 08:00-17:00

# melarang akses internet pada jam kerja
http_a ccess deny AVAILABLE_WORKING
```
#### 2. Adapun pada hari dan jam kerja sesuai nomor (1), client hanya dapat mengakses domain loid-work.com dan franky-work.com (IP tujuan domain dibebaskan) 
- Buat domain loid-work.xom dan francky-work.com terlebih dahulu pada `wise` sebagai DNS server, dengan konfigurasi dalam script file sebagai berikut (pastikan bind9 telah terinstall):
``` bash
echo '
zone "loid-work.com" {
        type master;
        file "/etc/bind/jarkom/loid-work.com";
}; 

zone "franky-work.com" {
        type master;
        file "/etc/bind/jarkom/franky-work.com";
};

' > /etc/bind/named.conf.local

mkdir -p /etc/bind/jarkom

echo '
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     franky-work.com. root.franky-work.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      franky-work.com.
@       IN      A       192.177.2.3     ;IP Berlint 192.177.2.3

' > /etc/bind/jarkom/franky-work.com

echo '
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     loid-work.com. root.loid-work.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      loid-work.com.
@       IN      A       192.177.2.3     ;IP Berlint 192.177.2.3

' > /etc/bind/jarkom/loid-work.com

service bind9 restart
```
- setelah domain terbuat, tambahkan domain loid dan franky-work sebagai acl dnsdomain yanga akan kita allow pada jam kerja dan deny diluar jam kerja. dengan konfigurasi pada `/etc/squid/squid.conf` di `berlint` sehingga konfigurasi menjadi sebagai berkut:
```bash
http_port 8080
visible_hostname berlint

acl AVAILABLE_WORKING time MTWHF 08:00-17:00
acl WORKSITES dstdomain loid-work.com franky-work.com

# membolehkan worksite pada jam kerja
http_access allow WORKSITES AVAILABLE_WORKING
# melarang worksite diluar jam kerja
http_access deny WORKSITES !AVAILABLE_WORKING
http_access deny AVAILABLE_WORKING

# restart squid
# service squid restart
# cek status squid
# service squid status
```
#### 3. Saat akses internet dibuka, client dilarang untuk mengakses web tanpa HTTPS. (Contoh web HTTP: http://example.com)
- Gunakan url_regex untuk melarang semua url yang memiliki pola berupa `diawali dengan "http:" dan "https:"` dengan konfigurasi tambahan pada `squid.conf` pada berlint sebagai berkut:
```bash
http_port 8080
visible_hostname berlint

acl AVAILABLE_WORKING time MTWHF 08:00-17:00
acl WORKSITES dstdomain loid-work.com franky-work.com
#url regex sebagai acl
acl HTTP_REGEX url_regex ^http:.*$
acl HTTPS_REGEX url_regex ^https:.*$

http_access allow WORKSITES AVAILABLE_WORKING
http_access deny WORKSITES !AVAILABLE_WORKING
http_access deny AVAILABLE_WORKING
# menolak akses https pada jam kerja
http_access deny AVAILABLE_WORKING HTTPS_REGEX
# menolak semua akses http pada jam kerja apapun (tidak termasuk worksite yang telah di allow dibagian atas)
http_access deny HTTP_REGEX

# service squid restart
# service squid status
```
#### 4. Agar menghemat penggunaan, akses internet dibatasi dengan kecepatan maksimum 128 Kbps pada setiap host (Kbps = kilobit per second; lakukan pengecekan pada tiap host, ketika 2 host akses internet pada saat bersamaan, keduanya mendapatkan speed maksimal yaitu 128 Kbps)
- Tambahkan bandwidth limit pada konfigurasi `squid.conf` di berlint
```bash
http_port 8080
visible_hostname berlint

acl AVAILABLE_WORKING time MTWHF 08:00-17:00
acl WORKSITES dstdomain loid-work.com franky-work.com
acl HTTP_REGEX url_regex ^http:.*$
acl HTTPS_REGEX url_regex ^https:.*$

http_access allow WORKSITES AVAILABLE_WORKING
http_access deny WORKSITES !AVAILABLE_WORKING
http_access deny AVAILABLE_WORKING
http_access deny AVAILABLE_WORKING HTTPS_REGEX
http_access deny HTTP_REGEX

# konfigurasi limit bandwidth
delay_pools 1
delay_class 1 1
delay_parameters 1 16000/64000

# service squid restart
# service squid status
```

#### 5. Setelah diterapkan, ternyata peraturan nomor (4) mengganggu produktifitas saat hari kerja, dengan demikian pembatasan kecepatan hanya diberlakukan untuk pengaksesan internet pada hari libur
- Tambahkan konfigurasi untuk menentukan bahwa bandwidth limit hanya akan dijalankan pada hari libur yait sabtu dan minggu
```bash
http_port 8080
visible_hostname berlint

acl AVAILABLE_WORKING time MTWHF 08:00-17:00
acl WORKSITES dstdomain loid-work.com franky-work.com
acl HTTP_REGEX url_regex ^http:.*$
acl HTTPS_REGEX url_regex ^https:.*$

http_access allow WORKSITES AVAILABLE_WORKING
http_access deny WORKSITES !AVAILABLE_WORKING
http_access deny AVAILABLE_WORKING
http_access deny AVAILABLE_WORKING HTTPS_REGEX
http_access deny HTTP_REGEX

# acl weekdays sebagi waktu untuk hari minggu (S) dan sabtu (A)
acl WEEKDAYS time SA
delay_pools 1
delay_class 1 1
# melalkukan limit bandwidth hanya pada hari weekdays seperti yang telah didefinisikan pada acl di atas
delay_access 1 allow WEEKDAYS
delay_parameters 1 16000/64000

# service squid restart
# service squid status
```
