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
