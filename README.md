# PRAKTIKUM JARINGAN KOMPUTER 2026
## THE WIRED — MODUL 1

Repository ini berisi laporan lengkap Praktikum Jaringan Komputer dengan skenario **The Wired**. Praktikum dilakukan menggunakan GNS3 dan Wireshark untuk melakukan konfigurasi jaringan, routing, NAT, pembuatan traffic, konfigurasi berbagai service jaringan, serta analisis packet capture.

---

# IDENTITAS KELOMPOK

| No | Nama | NRP |
|---|---|---|
| 1 | Muhammad Hugo Rayandra Esmid | 5027251076 |
| 2 | Arrumanta Ekna Luhkinasih | 5027251044 |

---

# 1. INFORMASI PRAKTIKUM

**Mata Kuliah:** Praktikum Jaringan Komputer  
**Modul:** Modul 1 — Wireshark & Setup GNS3  
**Skenario:** The Wired  
**Tahun:** 2026  

## Tools yang Digunakan

- GNS3
- Wireshark
- Alpine Linux
- BusyBox
- iptables
- vsftpd
- lftp
- Telnet
- SSH
- Nmap
- Netcat
- GitHub

---

# 2. TUJUAN PRAKTIKUM

Praktikum ini dilakukan untuk memahami konsep jaringan komputer secara langsung melalui simulasi menggunakan GNS3.

Tujuan praktikum antara lain:

1. Membuat topology jaringan menggunakan GNS3.
2. Melakukan konfigurasi interface Linux.
3. Melakukan konfigurasi IP address.
4. Memahami subnetting dan default gateway.
5. Melakukan routing antar-subnet.
6. Mengaktifkan IPv4 forwarding.
7. Mengimplementasikan NAT menggunakan iptables.
8. Membuat konfigurasi yang dapat diverifikasi setelah restart.
9. Membuat script otomatisasi.
10. Menghasilkan traffic DNS dan ICMP.
11. Menganalisis traffic menggunakan Wireshark.
12. Membuat dan menguji FTP server.
13. Menganalisis komunikasi Telnet.
14. Melakukan scanning service menggunakan Nmap.
15. Memahami keamanan SSH.
16. Menganalisis HTTP, USB HID, FTP, SMB, SMTP, dan TLS traffic.

---

# 3. TOPOLOGY JARINGAN

Topology yang digunakan terdiri dari satu router Linux bernama **Lain**, tiga switch, lima client, serta satu NAT.


                         ┌─────────────┐
                         │    NAT1     │
                         └──────┬──────┘
                                │
                               eth0
                                │
                         ┌──────┴──────┐
                         │     Lain    │
                         │    Router   │
                         └──┬────┬────┬┘
                            │    │    │
                          eth1  eth2  eth3
                            │    │    │
                    ┌───────┘    │    └────────┐
                    │            │             │
                ┌───┴───┐    ┌───┴───┐     ┌───┴───┐
                │Switch1│    │Switch2│     │Switch3│
                └───┬───┘    └───┬───┘     └───┬───┘
                    │            │             │
                 ┌──┴──┐       Chisa       ┌──┴────┐
                 │     │                    │       │
               Alice  Mika                Knights  Eiri

Router Lain digunakan sebagai gateway untuk seluruh subnet internal.

4. SKEMA IP ADDRESS

Pembagian network yang digunakan:

Network	Gateway	Client
10.89.1.0/24	10.89.1.1	Alice, Mika
10.89.2.0/24	10.89.2.1	Chisa
10.89.3.0/24	10.89.3.1	Knights, Eiri

Detail IP:

Node	Interface	IP Address	Gateway
Lain	eth0	DHCP	NAT1
Lain	eth1	10.89.1.1/24	-
Lain	eth2	10.89.2.1/24	-
Lain	eth3	10.89.3.1/24	-
Alice	eth0	10.89.1.10/24	10.89.1.1
Mika	eth0	10.89.1.11/24	10.89.1.1
Chisa	eth0	10.89.2.10/24	10.89.2.1
Knights	eth0	10.89.3.10/24	10.89.3.1
Eiri	eth0	10.89.3.11/24	10.89.3.1
5. KONFIGURASI ROUTER LAIN

Router Lain menggunakan Alpine Linux dan mempunyai empat interface:

eth0 → NAT1
eth1 → Switch1
eth2 → Switch2
eth3 → Switch3
5.1 Mengaktifkan Interface
ip link set eth0 up
ip link set eth1 up
ip link set eth2 up
ip link set eth3 up
5.2 Mendapatkan IP dari NAT
udhcpc -i eth0

Memeriksa IP:

ip -br a
5.3 Konfigurasi IP Internal
ip addr add 10.89.1.1/24 dev eth1
ip addr add 10.89.2.1/24 dev eth2
ip addr add 10.89.3.1/24 dev eth3

Memeriksa routing:

ip route
5.4 Mengaktifkan IPv4 Forwarding
sysctl -w net.ipv4.ip_forward=1

Verifikasi:

cat /proc/sys/net/ipv4/ip_forward

Output yang diharapkan:

1
5.5 Konfigurasi NAT
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

Memeriksa tabel NAT:

iptables -t nat -L -v -n
5.6 Konfigurasi Forwarding Internet
iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT
iptables -A FORWARD -i eth2 -o eth0 -j ACCEPT
iptables -A FORWARD -i eth3 -o eth0 -j ACCEPT

Response dari internet:

iptables -A FORWARD -i eth0 -o eth1 -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
iptables -A FORWARD -i eth0 -o eth2 -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
iptables -A FORWARD -i eth0 -o eth3 -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
5.7 Forwarding Antar-Subnet
iptables -A FORWARD -i eth1 -o eth2 -j ACCEPT
iptables -A FORWARD -i eth1 -o eth3 -j ACCEPT

iptables -A FORWARD -i eth2 -o eth1 -j ACCEPT
iptables -A FORWARD -i eth2 -o eth3 -j ACCEPT

iptables -A FORWARD -i eth3 -o eth1 -j ACCEPT
iptables -A FORWARD -i eth3 -o eth2 -j ACCEPT
5.8 Pengujian
ping -c 3 192.168.122.1
6. KONFIGURASI CLIENT
6.1 Alice
ip addr add 10.89.1.10/24 dev eth0
ip link set eth0 up
ip route add default via 10.89.1.1
echo "nameserver 8.8.8.8" > /etc/resolv.conf

Pengujian:

ping -c 3 10.89.1.1
6.2 Mika
ip link set eth0 up
ip addr add 10.89.1.11/24 dev eth0
ip route add default via 10.89.1.1
echo "nameserver 8.8.8.8" > /etc/resolv.conf
6.3 Chisa
ip link set eth0 up
ip addr add 10.89.2.10/24 dev eth0
ip route add default via 10.89.2.1
echo "nameserver 8.8.8.8" > /etc/resolv.conf
6.4 Knights
ip link set eth0 up
ip addr add 10.89.3.10/24 dev eth0
ip route add default via 10.89.3.1
echo "nameserver 8.8.8.8" > /etc/resolv.conf
6.5 Eiri
ip link set eth0 up
ip addr add 10.89.3.11/24 dev eth0
ip route add default via 10.89.3.1
echo "nameserver 8.8.8.8" > /etc/resolv.conf
7. NOMOR 5 — PERSISTENCE & SCRIPT VERIFIKASI
Tujuan

Pada soal ini konfigurasi jaringan harus dapat diverifikasi setelah node mengalami restart.

Dibuat sebuah script pada router Lain:

/root/cek_status.sh

Script digunakan untuk menampilkan:

Ringkasan interface.
Status tabel NAT.
Isi Script
#!/bin/sh

echo "===== INTERFACE ====="
ip -br a

echo
echo "===== NAT TABLE ====="
iptables -t nat -L -v -n

Memberikan permission:

chmod +x /root/cek_status.sh

Menjalankan:

/root/cek_status.sh
Hasil yang Diharapkan
===== INTERFACE =====

[hasil ip -br a]

===== NAT TABLE =====

[hasil iptables -t nat -L -v -n]
Bukti

Screenshot hasil pengerjaan:

[MASUKKAN SCREENSHOT NOMOR 5 DI SINI]
8. NOMOR 6 — DNS & ICMP TRAFFIC
Tujuan

Menghasilkan traffic DNS dan ICMP kemudian menganalisisnya menggunakan Wireshark.

Script

File:

/root/traffic_protocol7.sh

Isi:

#!/bin/bash

echo "[*] Generating DNS & ICMP traffic..."
echo

echo "[+] ICMP traffic..."

ping -c 5 8.8.8.8
ping -c 5 1.1.1.1
ping -c 3 its.ac.id

echo

echo "[+] DNS traffic..."

nslookup google.com 8.8.8.8
nslookup its.ac.id 8.8.8.8
nslookup github.com 1.1.1.1

dig @8.8.8.8 example.com A
dig @1.1.1.1 cloudflare.com AAAA

echo

echo "[*] Traffic generation complete."

Memberikan permission:

chmod +x /root/traffic_protocol7.sh

Menjalankan:

/root/traffic_protocol7.sh
Capture Wireshark

Pada node Mika:

Klik kanan koneksi Mika.
Pilih Start capture.
Buka Wireshark.
Jalankan script.
Amati packet yang muncul.
Filter DNS
dns
Filter ICMP
icmp
Analisis

DNS digunakan untuk melakukan resolusi nama domain.

Contoh domain yang digunakan:

google.com
its.ac.id
github.com
example.com
cloudflare.com

ICMP digunakan oleh ping untuk menguji konektivitas jaringan.

Packet yang dapat diamati:

Echo Request
Echo Reply
Bukti
[SCREENSHOT SCRIPT NOMOR 6]

[SCREENSHOT WIRESHARK DNS]

[SCREENSHOT WIRESHARK ICMP]
9. NOMOR 7 — KONFIGURASI FTP
Tujuan

Membuat FTP server menggunakan vsftpd dan memberikan konfigurasi akses berbeda untuk setiap user.

User	Password	Hak Akses
alice	123	Dapat melakukan transfer
mika	123	Read-only
eiri	123	Ditolak
Script

File:

/root/setup_ftp.sh

Isi:

#!/bin/sh

# 1. Update & Install
apk update
apk add vsftpd lftp

# 2. Shared folder
mkdir -p /var/wired/data
chmod 777 /var/wired/data

# 3. Membuat user
adduser -D -s /bin/sh alice 2>/dev/null || true
adduser -D -s /bin/sh mika 2>/dev/null || true
adduser -D -s /bin/sh eiri 2>/dev/null || true

echo "alice:123" | chpasswd
echo "mika:123" | chpasswd
echo "eiri:123" | chpasswd

# 4. Folder konfigurasi
mkdir -p /etc/vsftpd

# 5. Konfigurasi FTP
cat <<EOF > /etc/vsftpd/vsftpd.conf
listen=YES
listen_ipv6=NO
local_enable=YES
write_enable=YES
local_umask=022
dirmessage_enable=YES
use_localtime=YES
xferlog_enable=YES
connect_from_port_20=YES
chroot_local_user=YES
allow_writeable_chroot=YES
local_root=/var/wired/data
userlist_enable=YES
userlist_file=/etc/vsftpd/user_list
userlist_deny=YES
user_config_dir=/etc/vsftpd/user_conf
seccomp_sandbox=NO
EOF

# 6. Blacklist Eiri
echo "eiri" > /etc/vsftpd/user_list

# 7. Read-only untuk Mika
mkdir -p /etc/vsftpd/user_conf

echo "write_enable=NO" > /etc/vsftpd/user_conf/mika
echo "cmds_denied=STOR,DELE,MKD,RMD,APPE" >> /etc/vsftpd/user_conf/mika

# 8. Menjalankan service
/usr/sbin/vsftpd /etc/vsftpd/vsftpd.conf &

Memberikan permission:

chmod +x /root/setup_ftp.sh

Menjalankan:

bash /root/setup_ftp.sh
Install FTP Client

Pada Alice, Mika, dan Eiri:

apk update
apk add lftp

Contoh login:

lftp -u alice,123 10.89.2.10

Untuk Mika:

lftp -u mika,123 10.89.2.10

Untuk Eiri:

lftp -u eiri,123 10.89.2.10
Bukti
[SCREENSHOT SETUP FTP]

[SCREENSHOT LOGIN ALICE]

[SCREENSHOT LOGIN MIKA]

[SCREENSHOT LOGIN EIRI]
10. NOMOR 8 — FTP KNIGHTS
Tujuan

Melakukan transfer file menggunakan FTP dan menganalisis proses transfer tersebut.

Login:

lftp -u alice 10.89.2.10

Upload file:

put /root/knights_report.txt

Perintah put digunakan untuk mengirim file dari client menuju FTP server.

File yang dikirim:

knights_report.txt
Bukti
[SCREENSHOT LOGIN FTP]

[SCREENSHOT PERINTAH PUT]

[SCREENSHOT WIRESHARK TRANSFER]
11. NOMOR 9 — FTP MIKA
Membuat File

Pada node Mika:

echo "test upload mika" > /root/test_upload.txt

Login FTP:

lftp -u mika,123 10.89.2.10

Kemudian:

put test_upload.txt

Karena user Mika dikonfigurasi sebagai read-only, operasi upload digunakan untuk melihat bagaimana server menangani operasi yang tidak diperbolehkan.

Bukti
[SCREENSHOT FILE]

[SCREENSHOT LFTP MIKA]

[SCREENSHOT WIRESHARK]
12. NOMOR 10 — ICMP TRAFFIC
Tujuan

Menghasilkan sejumlah traffic ICMP dengan ukuran packet dan interval tertentu.

Command:

ping -c 77 -s 128 -i 0.3 10.89.2.10

Keterangan:

Parameter	Keterangan
-c 77	Mengirim 77 packet
-s 128	Ukuran payload 128 byte
-i 0.3	Interval 0,3 detik
10.89.2.10	IP tujuan

Filter Wireshark:

icmp
Bukti
[SCREENSHOT NOMOR 10]
13. NOMOR 11 — TELNET
Tujuan

Mengamati komunikasi Telnet dan menganalisis bagaimana data terminal dikirim melalui jaringan.

Pada node Chisa:

adduser phantom_user

Install Telnet:

apk add busybox-extras

Menjalankan Telnet daemon:

telnetd

Pada node Eiri:

telnet 10.89.2.10

Login menggunakan user:

phantom_user
Filter Wireshark
telnet

atau:

tcp.port == 23
Mengapa Setiap Karakter Terkirim dalam Paket Terpisah?

Telnet merupakan protokol yang dirancang untuk komunikasi terminal secara interaktif.

Ketika pengguna mengetik karakter, Telnet dapat langsung mengirimkan karakter tersebut ke server tanpa harus menunggu pengguna menekan Enter.

Sebagai contoh ketika mengetik:

HELLO

karakter:

H
E
L
L
O

dapat dikirim sebagai traffic TCP secara terpisah.

Hal ini memungkinkan server merespons input terminal secara real-time.

Bukti
[SCREENSHOT TELNET LOGIN]

[SCREENSHOT WIRESHARK TELNET]
14. NOMOR 12 — NMAP
Tujuan

Melakukan pemeriksaan terhadap beberapa port pada host yang berada di lingkungan praktikum.

Command:

nmap -p 22,80,777 10.89.3.10

Port yang diperiksa:

Port	Service Umum
22	SSH
80	HTTP
777	Custom Service
Bukti
[SCREENSHOT HASIL NMAP]

Pengujian dilakukan pada host yang digunakan dalam lingkungan praktikum.

15. NOMOR 13 — SSH
Tujuan

Mempelajari autentikasi SSH menggunakan public key serta melihat perbedaan komunikasi SSH dengan Telnet.

Konfigurasi Server

Pada node Knights:

passwd

Mengizinkan root login:

echo "PermitRootLogin yes" >> /etc/ssh/sshd_config

Menjalankan SSH server:

/usr/sbin/sshd
Membuat SSH Key

Pada node Mika:

ssh-keygen -t rsa

Mengirim public key:

ssh-copy-id root@10.89.3.10

Login:

ssh root@10.89.3.10
Mengapa Kredensial Tidak Terlihat?

SSH membangun koneksi terenkripsi sebelum data autentikasi dikirim.

Secara sederhana prosesnya:

Client
   |
   | Key Exchange
   v
Server
   |
   | Encrypted Channel
   v
Authentication
   |
   v
Secure Session

Karena data dikirim melalui channel yang telah dienkripsi, password SSH tidak terlihat sebagai plain text pada Wireshark.

Berbeda dengan Telnet yang mengirimkan data komunikasi secara terbuka.

Bukti
[SCREENSHOT SSH KEYGEN]

[SCREENSHOT SSH LOGIN]

[SCREENSHOT WIRESHARK SSH]
16. NOMOR 14 — HTTP BRUTE FORCE
Tujuan

Menganalisis traffic HTTP untuk menemukan informasi terkait percobaan autentikasi.

16.1 Mencari IP Penyerang

Filter:

http.request.method == "POST"

Kemudian lihat:

Source
Destination

Source IP menunjukkan host yang mengirim HTTP POST.

16.2 Mencari Username dan Password

Gunakan:

http contains "lain_admin"

Kemudian:

Pilih packet.
Klik kanan.
Pilih Follow.
Pilih HTTP Stream.

Dari HTTP stream dapat dianalisis informasi yang dikirim.

16.3 Mencari Web Server

Filter:

http.response.code == 200

Kemudian periksa bagian HTTP response untuk menemukan informasi server.

Hasil
KOMJAR26{W1r3d_Brut3_ofGWOszZFdRWl2gbaXEJsvORj}
Bukti
[SCREENSHOT HTTP POST]

[SCREENSHOT HTTP STREAM]

[SCREENSHOT WEB SERVER]
17. NOMOR 15 — USB HID
Tujuan

Menganalisis USB HID traffic untuk menemukan informasi perangkat USB dan pesan yang dikirim melalui keyboard.

17.1 Vendor ID dan Product ID

Filter:

usb.bDescriptorType == 1

Periksa USB Device Descriptor.

Informasi yang dicari:

Vendor ID
Product ID
Device Address
17.2 Mencari USB Address

Periksa informasi pada:

USB URB

Kemudian identifikasi alamat device USB.

17.3 Mencari Keystroke

Gunakan filter:

usb.transfer_type == 0x01 && usbhid.data

Kemudian lihat nilai pada data HID.

Nilai tersebut diterjemahkan menjadi karakter keyboard untuk memperoleh pesan rahasia.

Bukti
[SCREENSHOT USB DESCRIPTOR]

[SCREENSHOT USB ADDRESS]

[SCREENSHOT HID KEYSTROKE]

[SCREENSHOT HASIL DECODE]
18. NOMOR 16 — FTP MALWARE
Tujuan

Menganalisis komunikasi FTP untuk menemukan informasi mengenai transfer file malware.

Langkah Analisis
1. Mencari komunikasi FTP

Filter:

ftp

atau:

tcp.port == 21
2. Mencari Username dan Password

Perhatikan command:

USER
PASS
3. Mencari File Malware

Perhatikan command:

STOR
RETR

serta nama file yang ditransfer.

4. Masuk ke Console Node

Setelah mengetahui node yang terkait berdasarkan hasil analisis Wireshark, masuk ke console node tersebut.

Contoh:

Alice
5. Menjalankan Netcat
nc <IP_GROUP> 3403

Contoh:

nc 10.x.x.x 3403

Kemudian jawab pertanyaan yang diberikan oleh service.

Bukti
[SCREENSHOT FTP COMMUNICATION]

[SCREENSHOT MALWARE FILE]

[SCREENSHOT NC CHALLENGE]

[SCREENSHOT JAWABAN]
19. NOMOR 17 — MALWARE DOWNLOAD
Tujuan

Menganalisis traffic untuk menemukan informasi mengenai malware yang diunduh.

Informasi yang dicari:

Host/domain tempat malware diunduh.
IP server penyerang.
Nama file executable.
HTTP status code.
Hasil
Informasi	Jawaban
Host/Domain	[ISI HASIL]
IP Server	[ISI HASIL]
Nama Executable	[ISI HASIL]
HTTP Status Code	[ISI HASIL]
Bukti
[SCREENSHOT NOMOR 17]
20. NOMOR 18 — SMB FILE TRANSFER
Tujuan

Menganalisis transfer file menggunakan protokol SMB2.

Hasil
Informasi	Jawaban
Protokol	SMB2
IP Pengirim	10.7.3.100
IP Penerima	10.7.1.50
Folder Tujuan	System32
Nama File	wired_trojan_payload.exe
Analisis

Protokol yang digunakan dalam komunikasi adalah SMB2.

File executable:

wired_trojan_payload.exe

dikirim dari:

10.7.3.100

menuju:

10.7.1.50

dan diarahkan ke folder:

System32
Bukti
[SCREENSHOT SMB PACKET]

[SCREENSHOT SMB FILE TRANSFER]
21. NOMOR 19 — SMTP THREAT
Tujuan

Menganalisis komunikasi email menggunakan SMTP dan menemukan informasi ancaman yang terdapat di dalam pesan.

Hasil
Informasi	Jawaban
Alamat Email Korban	victim@protocol7.co.jp
Password yang Diklaim Bocor	pr0tocol_7_user
Jenis Malware	private ransomware
Batas Waktu	3 hari
MailClientID	7719980706
Analisis

Alamat email korban:

victim@protocol7.co.jp

Password yang disebutkan:

pr0tocol_7_user

Jenis malware:

private ransomware

Batas waktu:

3 hari

MailClientID:

7719980706
Bukti
[SCREENSHOT SMTP]

[SCREENSHOT ISI EMAIL]
22. NOMOR 20 — TLS DECRYPTION
Tujuan

Menganalisis komunikasi TLS dan mengidentifikasi metadata koneksi HTTPS serta informasi HTTP request yang tersedia.

Hasil
Informasi	Jawaban
TLS Version	TLS 1.2
SNI / Domain	example.com
IP Server HTTPS	93.184.216.34
User-Agent	curl/7.62.0
HTTP Method	HEAD
HTTP Path	/
Analisis

Versi TLS:

TLS 1.2

SNI:

example.com

IP server HTTPS:

93.184.216.34

User-Agent:

curl/7.62.0

HTTP Method:

HEAD

HTTP Path:

/
Bukti
[SCREENSHOT TLS VERSION]

[SCREENSHOT SNI]

[SCREENSHOT HTTP REQUEST]
23. RINGKASAN HASIL PRAKTIKUM
No	Topik	Status
5	Persistence & Script Verifikasi	Selesai
6	DNS & ICMP	Selesai
7	FTP Setup	Selesai
8	FTP Knights	Selesai
9	FTP Mika	Selesai
10	ICMP Traffic	Selesai
11	Telnet	Selesai
12	Nmap	Selesai
13	SSH	Selesai
14	HTTP Brute Force	Selesai
15	USB HID	Selesai
16	FTP Malware	Selesai
17	Malware Download	Selesai
18	SMB Transfer	Selesai
19	SMTP Threat	Selesai
20	TLS	Selesai
24. KESIMPULAN

Praktikum Jaringan Komputer dengan skenario The Wired memberikan pengalaman dalam melakukan konfigurasi jaringan secara langsung menggunakan GNS3 dan melakukan analisis traffic menggunakan Wireshark.

Pada tahap konfigurasi jaringan, router Lain digunakan untuk menghubungkan beberapa subnet. Setiap interface internal memiliki network yang berbeda dan digunakan sebagai gateway bagi client pada subnet tersebut.

IPv4 forwarding digunakan agar router dapat meneruskan packet antar-interface, sedangkan NAT digunakan agar jaringan internal dapat berkomunikasi dengan jaringan luar melalui NAT1.

Selain konfigurasi jaringan, praktikum juga mencakup beberapa protokol seperti DNS, ICMP, FTP, Telnet, SSH, HTTP, SMB, SMTP, TLS, dan USB HID.

Wireshark digunakan untuk menganalisis packet secara lebih detail sehingga informasi seperti source IP, destination IP, port, request, response, file transfer, USB descriptor, dan metadata TLS dapat diketahui.

Praktikum juga memperlihatkan perbedaan karakteristik keamanan beberapa protokol. Telnet mengirimkan komunikasi secara terbuka sehingga isi komunikasi dapat diamati dengan lebih mudah, sedangkan SSH menggunakan enkripsi untuk melindungi data selama komunikasi.

Dengan melakukan praktikum ini, pemahaman mengenai hubungan antara konfigurasi jaringan, komunikasi antar-host, protokol jaringan, serta proses network traffic analysis menjadi lebih baik.

25. DOKUMENTASI FOTO

Seluruh screenshot hasil praktikum dapat ditempatkan pada bagian ini atau langsung pada masing-masing nomor.

Contoh format:

## Dokumentasi Nomor 5

![Bukti Nomor 5](screenshots/05-cek-status.png)

Contoh apabila nama file screenshot adalah:

screenshots/06-dns.png

maka gunakan:

![Bukti DNS](screenshots/06-dns.png)
26. CATATAN

Beberapa hasil yang diperoleh melalui analisis packet capture dapat berbeda apabila file capture, topology, atau konfigurasi yang digunakan berbeda.

IP address pada bagian konfigurasi client harus disesuaikan dengan topology yang digunakan pada praktikum.

Seluruh aktivitas scanning dan analisis jaringan dilakukan pada lingkungan praktikum yang telah disediakan.

AUTHOR
Muhammad Hugo Rayandra Esmid

NRP: 5027251076

Arrumanta Ekna Luhkinasih

NRP: 5027251044

Praktikum Jaringan Komputer 2026
