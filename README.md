# Jarkom-Modul-3-E09-2022

## No 8 - 12(selesai)
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
```


### testing 8-13
Untuk testing bbandwidth limit menggunakan speedtest, akese deny pada http harus dimatikan lebih dahulu agar speedtest bisa diakses karena speedtest menggungakan akses http, lalu jalankan command pada client yang telah terinstall python sebelumnya
```bash
export PYTHONHTTPSVERIFY=0
```
lalu jalankan commadn `speedtest`
 
#### Senin jam 10:00+
- akses http
![image](https://user-images.githubusercontent.com/70748569/201580057-1715774b-2dfb-4b53-9164-88c5e67451e8.png)

- akses https

![image](https://user-images.githubusercontent.com/70748569/201581509-121b6714-aa5d-47bd-a9c8-5192998116db.png)

- akses loid-work.com dan franky-work (sudah berhasil, namun karena domain tidak mengarah ke webservice apapun maka yang tampil adalah error "service unavaliable - connection failed/refused, karena jika terhalang proxy error yang tampil adalah forbiden - acces denied).

![image](https://user-images.githubusercontent.com/70748569/201580526-b595f5a5-9f67-4ad6-b99c-08bfd01045ce.png)
![image](https://user-images.githubusercontent.com/70748569/201580590-829c7af8-c8aa-433e-b20c-331b817ee4f8.png)

- speedtest

![image](https://user-images.githubusercontent.com/70748569/201582951-8d4afb5d-c19b-4657-8fc6-d835e4d6cbbc.png)

#### Senin jam 20:00+
- akses http

![image](https://user-images.githubusercontent.com/70748569/201580812-eefe1bf6-3de6-4211-aab7-5289da5f598b.png)

- akses https

![image](https://user-images.githubusercontent.com/70748569/201581277-69bc02d8-8908-4f9b-9cb0-c05016d5758d.png)

- akses loid dan franky-work.com (disini bisa dilihat jika tidak bisa diakses dan errornya forbiden -access denied bukan service unavaliable seperti yang berhasil diatas)

![image](https://user-images.githubusercontent.com/70748569/201581357-fbf06c7a-f2ec-481e-a6d2-5c9302959976.png)
![image](https://user-images.githubusercontent.com/70748569/201581994-fdb49667-73e6-4a1b-afb6-ccf72f31a7f6.png)


- speedtest

![image](https://user-images.githubusercontent.com/70748569/201583091-be628c95-2ead-4a3a-ab7d-cc0f652270ec.png)

#### Sabtu jam 10:00
- akses http

![image](https://user-images.githubusercontent.com/70748569/201581607-0f6f54df-d52c-4693-a123-872343ebed59.png)

- akses https

![image](https://user-images.githubusercontent.com/70748569/201582200-2c8c40cc-53c0-40bb-b3be-03464c45b627.png)

- akses loid-work dan franky

![image](https://user-images.githubusercontent.com/70748569/201581921-35c004b2-8f51-4dfb-99b2-9a79929a9b47.png)
![image](https://user-images.githubusercontent.com/70748569/201581982-478b0185-ecef-488f-8330-cd9f9c5afe72.png)

- speedtest

![image](https://user-images.githubusercontent.com/70748569/201583196-d6687c7d-7d90-4da6-bd47-9e5c20d5676d.png)
