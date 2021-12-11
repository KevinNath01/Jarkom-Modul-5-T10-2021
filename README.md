# Jarkom-Modul-5-T10-2021

PRAKTIKUM MODUL 5 JARINGAN KOMPUTER 2021

Anggota Kelompok T10:<br>

1. Tri Rizki Yuliawan (05311940000024) <br>
2. Kevin Nathaniel (05311940000032) <br>
3. I Gde Ardha Semaranatha Gunasatwika (05311940000034) <br>

# Soal <a name="Soal"></a>

### A) Topologi
<br>
<img src="https://github.com/KevinNath01/Jarkom-Modul-5-T10-2021/blob/main/Dokumentasi/Topologi5.jpeg">
<br>

### B) Subnetting
Subnetting dilakukan dengan menggunakan metode VLSM dan menggunakan pembagian subneting sebagai berikut
<br>
<img src="https://github.com/KevinNath01/Jarkom-Modul-5-T10-2021/blob/main/Dokumentasi/subnet5.png">
<br>
Dari pembagian subnetting terseut dibuatlah tree untuk menentukan ip subnetnya masing masing
<br>
<img src="https://github.com/KevinNath01/Jarkom-Modul-5-T10-2021/blob/main/Dokumentasi/Tree5.png">
<br>

### C) Routing IP
Routing akan dilakukan di Foosha yang akan merutekan ke seluruh subnet agar setiap subnet dan pc bisa saling terhubung
**Foosha**
```
route add -net 192.216.7.0 netmask 255.255.255.128 gw 192.216.7.146 #BLUENO
route add -net 192.216.0.0 netmask 255.255.252.0 gw 192.216.7.146 #CIPHER
route add -net 192.216.7.128 netmask 255.255.255.248 gw 192.216.7.146 #DORIKI & JIPANGU

route add -net 192.216.4.0 netmask 255.255.254.0 gw 192.216.7.150 #ELENA
route add -net 192.216.6.0 netmask 255.255.255.0 gw 192.216.7.150 #FUKORO
route add -net 192.216.7.136 netmask 255.255.255.248 gw 192.216.7.150 #Maingate & Jourge
```

### D) IP Address (DHCP Server)
IP Address disetting untuk Blueno,Cipher,Fukurou dan Elena , pengaturannya sendiri dilakukan di node Jipangu sebagai DHCP Server
di node Jipangu akan di unduh untuk DHCP servernya yang kemudian diatur interfacenya `eth0` , kemudian tiap subnet yang membutuhkan ip dinamis akan disetting , dari range yang bisa dihitung dari perhitungan subnet, router yang terhubung dengan subnetnya , broadcast address, dns server yang memiliki IP 192.216.7.130 dan waktu peminjaman ip maksimal 2 jam yang sehabis itu akan direset lagi dengan ip baru 
**Jipangu**
```
apt install isc-dhcp-server -y
echo '
INTERFACES="eth0"
' > /etc/default/isc-dhcp-server

echo '
ddns-update-style none;
option domain-name "example.org";
option domain-name-servers ns1.example.org, ns2.example.org;
default-lease-time 600;
max-lease-time 7200;
log-facility local7;
subnet 192.216.0.0 netmask 255.255.252.0 {
    range 192.216.0.2 192.216.3.254;
    option routers 192.216.0.1;
    option broadcast-address 192.216.3.255;
    option domain-name-servers 192.216.7.130;
    default-lease-time 360;
    max-lease-time 7200;
}
subnet 192.216.7.0 netmask 255.255.255.128 {
    range 192.216.7.2 192.216.7.126;
    option routers 192.216.7.1;
    option broadcast-address 192.216.7.127;
    option domain-name-servers 192.216.7.130;
    default-lease-time 720;
    max-lease-time 7200;
}
subnet 192.216.4.0 netmask 255.255.254.0 {
    range 192.216.4.2 192.216.5.254;
    option routers 192.216.4.1;
    option broadcast-address 192.216.5.255;
    option domain-name-servers 192.216.7.130;
    default-lease-time 720;
    max-lease-time 7200;
}
subnet 192.216.6.0 netmask 255.255.255.0 {
    range 192.216.6.2 192.216.6.254;
    option routers 192.216.6.1;
    option broadcast-address 192.216.6.255;
    option domain-name-servers 192.216.7.130;
    default-lease-time 720;
    max-lease-time 7200;
}
subnet 192.216.7.128 netmask 255.255.255.248 {}
subnet 192.216.7.144 netmask 255.255.255.252 {}
subnet 192.216.7.148 netmask 255.255.255.252 {}
subnet 192.216.7.136 netmask 255.255.255.248 {}
' > /etc/dhcp/dhcpd.conf
service isc-dhcp-server restart
 ```
 
 ### D.1) DHCP Relay
 DHCP relay utamanya akan diinstal di semua router dan akan dikonfigurasikan port eth yang digunakannya
 **Foosha**
 ```
 apt-get install isc-dhcp-relay -y

echo '
SERVERS="192.216.7.131"
INTERFACES="eth2 eth1"
OPTIONS=""
' > /etc/default/isc-dhcp-relay
service isc-dhcp-relay restart
 `
 <br>
 **Water7**
 `apt install isc-dhcp-relay -y
echo '
SERVERS="192.216.7.131"
INTERFACES="eth2 eth3 eth0 eth1"
OPTIONS=""
' > /etc/default/isc-dhcp-relay
service isc-dhcp-relay restart
`
<br>
**Guanhao**
`
apt install isc-dhcp-relay -y
echo '
SERVERS="192.216.7.131"
INTERFACES="eth2 eth3 eth1 eth0"
OPTIONS=""
' > /etc/default/isc-dhcp-relay
service isc-dhcp-relay restart
```
