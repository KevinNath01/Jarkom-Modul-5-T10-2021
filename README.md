# Jarkom-Modul-5-T10-2021

PRAKTIKUM MODUL 5 JARINGAN KOMPUTER 2021

Anggota Kelompok T10:<br>

1. Tri Rizki Yuliawan (05311940000024) <br>
2. Kevin Nathaniel (05311940000032) <br>
3. I Gde Ardha Semaranatha Gunasatwika (05311940000034) <br>

# Soal <a name="Soal"></a>

### A) Topologi

<br>
<img src="https://github.com/KevinNath01/Jarkom-Modul-5-T10-2021/blob/main/Dokumentasi/topologi5.jpeg">
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
<br>
**Foosha**

```
apt-get install isc-dhcp-relay -y
echo '
SERVERS="192.216.7.131"
INTERFACES="eth2 eth1"
OPTIONS=""
' > /etc/default/isc-dhcp-relay
service isc-dhcp-relay restart
```

 <br>
 **Water7**
 
```
apt install isc-dhcp-relay -y
echo '
SERVERS="192.216.7.131"
INTERFACES="eth2 eth3 eth0 eth1"
OPTIONS=""
' > /etc/default/isc-dhcp-relay
service isc-dhcp-relay restart
```

<br>
 **Guanhao**
 
```
apt install isc-dhcp-relay -y
echo '
SERVERS="192.216.7.131"
INTERFACES="eth2 eth3 eth1 eth0"
OPTIONS=""
' > /etc/default/isc-dhcp-relay
service isc-dhcp-relay restart
```


### No. 1

### Agar topologi yang kalian buat dapat mengakses keluar, kalian diminta untuk mengkonfigurasi Foosha menggunakan iptables, tetapi Luffy tidak ingin menggunakan MASQUERADE.

pada persoalan ini, kami menggunakan SNAT sebagai metode pengganti masquerade. Karena Foosha kami menggunakan IP DHCP, maka IP tersebut perlu diambil dan disimpan di dalam variabel `IPETH0` yang nantinya akan digunakan pada command iptables SNAT

```
IPETH0="$(ip -br a | grep eth0 | awk '{print $NF}' | cut -d'/' -f1)"
iptables -t nat -A POSTROUTING -o eth0 -j SNAT --to-source "$IPETH0" -s 192.216.0.0/21
```

berikut adalah dokumentasi saat Foosha ping ke google.com
<br>
<img src="https://github.com/KevinNath01/Jarkom-Modul-5-T10-2021/blob/main/Dokumentasi/1.png">

### No. 2

### Kalian diminta untuk mendrop semua akses HTTP dari luar Topologi kalian pada server yang merupakan DHCP Server dan DNS Server demi menjaga keamanan.

untuk mendrop akses HTTP, kami memberikan command iptables untuk mendrop akses port 80 eth0 pada IP Doriki dan IP JIpanggu

```
iptables -A FORWARD -d 192.216.7.131 -i eth0 -p tcp --dport 80 -j DROP
iptables -A FORWARD -d 192.216.7.130 -i eth0 -p tcp --dport 80 -j DROP
```

berikut adalah dokumentasi saat CIPHER melakukan nmap port 80 pada Jipanggu
<br>
<img src="https://github.com/KevinNath01/Jarkom-Modul-5-T10-2021/blob/main/Dokumentasi/2.1.png">

berikut adalah dokumentasi saat CIPHER melakukan nmap port 80 pada Doriki
<br>
<img src="https://github.com/KevinNath01/Jarkom-Modul-5-T10-2021/blob/main/Dokumentasi/2.2.png">

### No. 3

### Karena kelompok kalian maksimal terdiri dari 3 orang. Luffy meminta kalian untuk membatasi DHCP dan DNS Server hanya boleh menerima maksimal 3 koneksi ICMP secara bersamaan menggunakan iptables, selebihnya didrop.

untuk menjawab persoalan ini, kami memberikan command connection limit dengan menggunakan iptables, yang dimana akan mendrop koneksi ICMP apabila melebihi 3 koneksi

```
iptables -A INPUT -p icmp -m connlimit --connlimit-above 3 --connlimit-mask 0 -j DROP
```

berikut adalah dokumentasi saat CIPHER melakukan ping pada IP Jipanggu
<br>
<img src="https://github.com/KevinNath01/Jarkom-Modul-5-T10-2021/blob/main/Dokumentasi/3.1.png">

berikut adalah dokumentasi saat BLUENO melakukan ping pada IP Jipanggu
<br>
<img src="https://github.com/KevinNath01/Jarkom-Modul-5-T10-2021/blob/main/Dokumentasi/3.2.png">

berikut adalah dokumentasi saat GUANHAO melakukan ping pada IP Jipanggu
<br>
<img src="https://github.com/KevinNath01/Jarkom-Modul-5-T10-2021/blob/main/Dokumentasi/3.3.png">

berikut adalah dokumentasi saat WATER7 melakukan ping pada IP Jipanggu
<br>
<img src="https://github.com/KevinNath01/Jarkom-Modul-5-T10-2021/blob/main/Dokumentasi/3.4.png">

### No. 4 & 5

#### Kemudian kalian diminta untuk membatasi akses ke Doriki yang berasal dari subnet Blueno, Cipher, Elena dan Fukuro dengan beraturan sebagai berikut :

- Akses dari subnet Blueno dan Cipher hanya diperbolehkan pada pukul 07.00 - 15.00 pada hari Senin sampai Kamis
- Akses dari subnet Elena dan Fukuro hanya diperbolehkan pada pukul 15.01 hingga pukul 06.59 setiap harinya.

Pada file `run.sh` di dalam fungsi Doriki, kami berikan command sebagai berikut :

```
#Blueno
iptables -A INPUT -s 192.216.7.0/25 -m time --weekdays Fri,Sat,Sun -j REJECT
iptables -A INPUT -s 192.216.7.0/25 -m time --timestart 00:00 --timestop 06:59 --weekdays Mon,Tue,Wed,Thu -j REJECT
iptables -A INPUT -s 192.216.7.0/25 -m time --timestart 15:01 --timestop 23:59 --weekdays Mon,Tue,Wed,Thu -j REJECT
#Cipher
iptables -A INPUT -s 192.216.0.0/22 -m time --weekdays Fri,Sat,Sun -j REJECT
iptables -A INPUT -s 192.216.0.0/22 -m time --timestart 00:00 --timestop 06:59 --weekdays Mon,Tue,Wed,Thu -j REJECT
iptables -A INPUT -s 192.216.0.0/22 -m time --timestart 15:01 --timestop 23:59 --weekdays Mon,Tue,Wed,Thu -j REJECT
```

yang artinya pada subnet `Blueno` dan `Cipher` tidak akan bisa mengakses Doriki pada hari `jumat, sabtu dan minggu`, sekaligus pada jam `00.00 - 06.59` dan jam `15:01 - 23:59` pada hari `senin sampai kamis`

Berikut adalah dokumentasi saat bisa mengakses Doriki
<br>
<img src="https://github.com/KevinNath01/Jarkom-Modul-5-T10-2021/blob/main/Dokumentasi/4.1.png">

Berikut saat tidak bisa mengakses Doriki
<br>
<img src="https://github.com/KevinNath01/Jarkom-Modul-5-T10-2021/blob/main/Dokumentasi/4.2.png">

Untuk no 5 caranya juga sama, kami berikan command sebagai berikut pada file `run.sh` di dalam fungsi Doriki:

```
iptables -A INPUT -s 192.216.4.0/23 -m time --timestart 07:00 --timestop 15:00 -j REJECT #Elena
iptables -A INPUT -s 192.216.6.0/24 -m time --timestart 07:00 --timestop 15:00 -j REJECT #Fukuro
```

yang artinya adalah untuk memblok subnet `Elena` dan `Fukuro` mulai jam `07:00 - 15:00`

Berikut adalah dokumentasi saat bisa mengakses Doriki
<Br>
<img src="https://github.com/KevinNath01/Jarkom-Modul-5-T10-2021/blob/main/Dokumentasi/5.1.png">

Berikut saat tidak bisa mengakses Doriki
<br>
<img src="https://github.com/KevinNath01/Jarkom-Modul-5-T10-2021/blob/main/Dokumentasi/5.2.png">

### No. 6 Karena kita memiliki 2 Web Server, Luffy ingin Guanhao disetting sehingga setiap request dari client yang mengakses DNS Server akan didistribusikan secara bergantian pada Jorge dan Maingate

Kami tambahkan command pada fungsi Guanhao, karena request yang dimodifikasi berasal dari client yang terhubung pada router Guanhao

```
iptables -A PREROUTING -t nat -p tcp -d 192.216.7.130 -m statistic --mode nth --every 2 --packet 0 -j DNAT --to-destination 192.216.7.138:80
iptables -A PREROUTING -t nat -p tcp -d 192.216.7.130 -j DNAT --to-destination 192.216.7.139:80
```

maksud dari command diatas adalah, kita melakukan `PREROUTING` pada setiap paket yang menuju DNS Server yang mempunyai IP `192.216.7.130`. `-m statistic --mode nth --every 2 --packet 0` adalah algoritma round robin yang akan mendistribusikan `p paket` kepada `n host` dan algoritma ini membutuhkan 2 parameter, pertama `every (n)` dan `packet (p)`. Lalu perintah `DNAT` adalah untuk mengubah destinasi yang awalnya adalah `192.216.7.130` menjadi `192.216.7.138:80` (IP Jorge) dan `192.216.7.139:80` (IP Maingate)

Berikut adalah pesan yang dikirim oleh client Elena :
<br>
<img src="https://github.com/KevinNath01/Jarkom-Modul-5-T10-2021/blob/main/Dokumentasi/6.1.png">

Berikut adalah pesan yang dikirim oleh client Fukorou :
<br>
<img src="https://github.com/KevinNath01/Jarkom-Modul-5-T10-2021/blob/main/Dokumentasi/6.2.png">

Berikut adalah pesan yang diterima oleh Web Server pertama yaitu Jorge :
<br>
<img src="https://github.com/KevinNath01/Jarkom-Modul-5-T10-2021/blob/main/Dokumentasi/6.3.png">

Berikut adalah pesan yang diterima oleh Web Server kedua yaitu Maingate :
<br>
<img src="https://github.com/KevinNath01/Jarkom-Modul-5-T10-2021/blob/main/Dokumentasi/6.4.png">
