# Jarkom_Modul5_Lapres_D06

> A. Tugas pertama kalian yaitu membuat topologi jaringan sesuai dengan rancangan yang diberikan
Bibah seperti dibawah ini :

![Topologi](https://link)

> SURABAYA diberikan IP TUNTAP<br>
MALANG merupakan DNS Server diberikan IP DMZ<br>
MOJOKERTO merupakan DHCP Server diberikan IP DMZ<br>
MADIUN dan PROBOLINGGO merupakan WEB Server<br>
Setiap Server diberikan memory sebesar 128M<br>
Client dan Router diberikan memori sebesar 96M<br>
Jumlah host pada subnet SIDOARJO 200 Host<br>
Jumlah host pada subnet GRESIK 210 Host<br>

```
JAWAB HERE
```

<br>

> B. karena kalian telah mempelajari Subnetting dan Routing, Bibah meminta kalian untuk membuat
topologi tersebut menggunakan teknik CIDR atau VLSM.

```
JAWAB HERE
```

<br>

> C. Setelah melakukan subnetting, kalian
juga diharuskan melakukan routing agar setiap perangkat pada jaringan tersebut dapat terhubung.

```
HERE ANSWER
```

<br>

> D. Tugas berikutnya adalah memberikan ip pada subnet SIDOARJO dan GRESIK secara dinamis
menggunakan bantuan DHCP SERVER (Selain subnet tersebut menggunakan ip static). Kemudian
kalian mengingat bahwa kalian harus setting DHCP RELAY pada router yang menghubungkannya,
seperti yang kalian telah pelajari di masa lalu.

```
ANSWER
```

<br>

> 1. Agar topologi yang kalian buat dapat mengakses keluar, kalian diminta untuk mengkonfigurasi
SURABAYA menggunakan iptables, namun Bibah tidak ingin kalian menggunakan
MASQUERADE.

UML **SURABAYA**

```
iptables -t nat -A POSTROUTING -o eth0 -j SNAT -s 192.168.0.0/16 --to-source 10.151.78.30
```

<br>

> 2. Kalian diminta untuk mendrop semua akses SSH dari luar Topologi (UML) Kalian pada server
yang memiliki ip DMZ (DHCP dan DNS SERVER) pada SURABAYA demi menjaga keamanan.

UML **SURABAYA**

```
iptables -N VAR
iptables -A VAR -j LOG --log-prefix '2--DROPPED PACKET FROM SSH =>' --log-level 6
iptables -A VAR -j DROP

iptables -A FORWARD -i eth0 -p tcp -d 10.151.79.56/29 --dport 22 -j VAR
```

<br>

> 3. Karena tim kalian maksimal terdiri dari 3 orang, Bibah meminta kalian untuk membatasi DHCP
dan DNS server hanya boleh menerima maksimal 3 koneksi ICMP secara bersamaan yang berasal dari
mana saja menggunakan iptables pada masing masing server, selebihnya akan di DROP.

UML **MALANG**

```
iptables -N VAR3
iptables -A VAR3 -j LOG --log-prefix '3--DROPPED INCOMING ICMP =>' --log-level 6

iptables -A VAR3 -j DROP

iptables -A INPUT -p icmp -m connlimit --connlimit-above 3 --connlimit-mask 0 -j VAR3
```

UML **MOJOKERTO**

```
iptables -N VAR3
iptables -A VAR3 -j LOG --log-prefix '3--DROPPED INCOMING ICMP =>' --log-level 6

iptables -A VAR3 -j DROP

iptables -A INPUT -p icmp -m connlimit --connlimit-above 3 --connlimit-mask 0 -j VAR3
```

<br>

> kemudian kalian diminta untuk membatasi akses ke MALANG yang berasal dari SUBNET
SIDOARJO dan SUBNET GRESIK dengan peraturan sebagai berikut:

> 4. Akses dari subnet SIDOARJO hanya diperbolehkan pada pukul 07.00 - 17.00 pada hari Senin sampai Jumat. Selain itu paket akan di REJECT.

UML **MALANG**

```
iptables -N VAR4
iptables -A VAR4 -j LOG --log-prefix '4--DROPPED PACKET =>' --log-level 6

iptables -A VAR4 -j REJECT

iptables -A INPUT -s 192.168.0.0/24 -m time --timestart 07:00 --timestop 17:00 --weekdays Mon,Tue,Wed,Thu,Fri -j ACCEPT

iptables -A INPUT -s 192.168.0.0/24 -j VAR4

```

> 5. Akses dari subnet GRESIK hanya diperbolehkan pada pukul 17.00 hingga pukul 07.00 setiap
harinya. Selain itu paket akan di REJECT.

UML **MALANG**

```
iptables -N VAR5
iptables -A VAR5 -j LOG --log-prefix '5--DROPPED PACKET =>' --log-level 6
iptables -A VAR5 -j REJECT

iptables -A INPUT -s 192.168.2.0/24 -m time --timestart 07:00 --timestop 17:00 -j VAR5

```
<br>

> 6. Bibah ingin SURABAYA disetting sehingga setiap
request dari client yang mengakses DNS Server akan didistribusikan secara bergantian pada
PROBOLINGGO port 80 dan MADIUN port 80.

UML **SURABAYA**

```
iptables -t nat -A PREROUTING -p tcp -d 10.151.79.58 -m statistic --mode nth --every 2 --packet 0 -j DNAT --to-destination 192.168.3.2:80
iptables -t nat -A PREROUTING -p tcp -d 10.151.79.58 -j DNAT --to-destination 192.168.3.3:80
```

<br>

> 7. Bibah ingin agar semua paket didrop oleh firewall (dalam topologi) tercatat dalam log pada setiap UML yang memiliki aturan drop.

Logging digabung dalam jawaban setiap soal

```
iptables -N SOMEVAR
iptables -A SOMEVAR -j LOG --log-prefix 'MESSEGE DROPPED PACKET =>' --log-level 6
iptables -A SOMEVAR -j DROP
```
