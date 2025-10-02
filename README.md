

# Jarkom-Modul-1-2025-K-14

| Nama                         | Nrp        |
| ---------------------------- | ---------- |
| Oscaryavat Viryavan          | 5027241053 |
| Mohamad Arkan Zahir Asyafiq  | 5027241118 |


# Laporan Resmi Praktikum Jaringan Komputer
---

## Soal 1: Membuat Topologi Jaringan Dasar

### Tujuan
Membangun topologi jaringan dasar di GNS3 yang terdiri dari satu Router (Eru), dua Switch, dan empat Client (Melkor, Manwe, Varda, Ulmo) sesuai dengan skenario yang diberikan.

### Langkah-langkah Pengerjaan
1.  Buka GNS3 dan buat proyek baru.
2.  Tambahkan satu node Docker container dan ubah namanya menjadi **Eru**.
3.  Tambahkan dua node *Ethernet switch* dan beri nama **Switch1** dan **Switch2**.
4.  Tambahkan empat node Docker container lagi dan beri nama **Melkor**, **Manwe**, **Varda**, dan **Ulmo**.
5.  Hubungkan perangkat-perangkat tersebut dengan kabel virtual sesuai skema yang dijelaskan di soal.
6.  Nyalakan semua node.

### Skrip yang Digunakan
Pengerjaan soal nomor 1 dilakukan sepenuhnya secara manual melalui antarmuka grafis GNS3, sehingga **tidak ada file skrip `.sh` yang digunakan**.

### Bukti Pengerjaan (Screenshot)
* **[➡️ Ambil Screenshot di sini]** Tampilan akhir topologi jaringan di GNS3.
    * *Saran penamaan file: `images/soal1_topologi.png`*

---

## Soal 2, 3, & 4: Konfigurasi Jaringan Manual dan Konektivitas Internet

### Tujuan
Tujuan dari gabungan soal ini adalah:
1.  **(Soal 2)** Mengonfigurasi router Eru agar dapat terhubung ke internet.
2.  **(Soal 3)** Memastikan semua client (Ainur) dapat saling berkomunikasi melalui router Eru.
3.  **(Soal 4)** Mengonfigurasi Eru agar semua client dapat mengakses internet.

### Langkah-langkah Pengerjaan
Konfigurasi untuk ketiga soal ini dilakukan secara manual dengan mengetikkan perintah-perintah berikut di terminal masing-masing node.

**1. Konfigurasi di Node Eru (Router):**
* Aktifkan IP Forwarding:
    ```bash
    sysctl -w net.ipv4.ip_forward=1
    ```
* Atur IP Address untuk interface ke Switch1 (`eth1`):
    ```bash
    ip addr add 192.18.1.1/24 dev eth1
    ip link set eth1 up
    ```
* Atur IP Address untuk interface ke Switch2 (`eth2`):
    ```bash
    ip addr add 192.18.2.1/24 dev eth2
    ip link set eth2 up
    ```
* Atur NAT agar client bisa terhubung ke internet (asumsi `eth0` terhubung ke internet):
    ```bash
    iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
    ```

**2. Konfigurasi di Node Melkor:**
* Atur IP Address dan default gateway:
    ```bash
    ip addr add 192.18.1.2/24 dev eth0
    ip link set eth0 up
    ip route add default via 192.18.1.1
    ```

**3. Konfigurasi di Node Manwe:**
* Atur IP Address dan default gateway:
    ```bash
    ip addr add 192.18.1.3/24 dev eth0
    ip link set eth0 up
    ip route add default via 192.18.1.1
    ```

**4. Konfigurasi di Node Varda:**
* Atur IP Address dan default gateway:
    ```bash
    ip addr add 192.18.2.2/24 dev eth0
    ip link set eth0 up
    ip route add default via 192.18.2.1
    ```

**5. Konfigurasi di Node Ulmo:**
* Atur IP Address dan default gateway:
    ```bash
    ip addr add 192.18.2.3/24 dev eth0
    ip link set eth0 up
    ip route add default via 192.18.2.1
    ```

**6. Verifikasi:**
* **(Verifikasi Soal 3)** Dari **Melkor**, lakukan ping ke **Varda**: `ping 192.18.2.2`.
* **(Verifikasi Soal 4)** Dari **Melkor**, lakukan ping ke internet: `ping 8.8.8.8`.

### Skrip yang Digunakan
Pengerjaan untuk soal 2, 3, dan 4 dilakukan secara manual dengan mengetikkan perintah-perintah di atas langsung di terminal setiap node, sehingga **tidak ada file skrip `.sh` yang digunakan** untuk bagian ini.

### Bukti Pengerjaan (Screenshot)
* **[➡️ Ambil Screenshot di sini]** Tampilan terminal Melkor yang berhasil `ping` ke Varda (bukti Soal 3).
* **[➡️ Ambil Screenshot di sini]** Tampilan terminal Melkor yang berhasil `ping` ke `8.8.8.8` (bukti Soal 4).
    * *Saran penamaan file: `images/soal3_interkoneksi.png`, `images/soal4_koneksi_internet.png`*

---

## Soal 5: Membuat Konfigurasi Permanen

### Tujuan
Menyimpan semua konfigurasi jaringan agar tidak hilang saat node di-restart.

### Langkah-langkah Pengerjaan
1.  **Konfigurasi IP Permanen (Client):** Edit file `/etc/network/interfaces` di setiap node client (Melkor, Manwe, Varda, Ulmo) untuk IP statis.
2.  **Konfigurasi IP Forwarding & NAT Permanen di Eru:** Jalankan skrip `soal_5_permanen.sh` di node **Eru**.

### Skrip yang Digunakan

**`soal_5_permanen.sh` (di Node Eru)**\

    ```bash
    #!/bin/bash
     echo "=== Membuat Konfigurasi Permanen di Eru (Soal 5) ==="
     sed -i 's/^#net.ipv4.ip_forward=1/net.ipv4.ip_forward=1/' /etc/sysctl.conf
    sysctl -p
    echo iptables-persistent iptables-persistent/autosave_v4 boolean true | debconf-set-selections
    echo iptables-persistent iptables-persistent/autosave_v6 boolean true | debconf-set-selections
    apt-get install -y iptables-persistent
    echo "--> Konfigurasi permanen selesai."
   

## Soal 6: Packet Sniffing dengan Wireshark

### Tujuan
Latihan menangkap dan memfilter trafik jaringan antara Manwe dan Eru.

### Langkah-langkah Pengerjaan
1.  Di GNS3, mulai *capture* pada koneksi antara Eru dan Switch1.
2.  Dari **Manwe**, hasilkan trafik dengan `ping -c 5 192.18.1.1`.
3.  Di Wireshark, gunakan filter `ip.addr == 192.18.1.3`.

### Skrip yang Digunakan
Tidak ada skrip utama, pengerjaan berfokus pada GNS3 dan Wireshark.

### Bukti Pengerjaan (Screenshot)
* **[➡️ Ambil Screenshot di sini]** Tampilan jendela Wireshark yang sudah difilter.
    * *Saran penamaan file: `images/soal6_wireshark_filter.png`*

---

   ## Soal 7: Konfigurasi FTP Server

### Tujuan
Memasang FTP server di Eru dengan hak akses berbeda untuk `ainur` dan `melkor`.

### Langkah-langkah Pengerjaan
1.  Jalankan skrip `soal_7.sh` di **Eru**.
2.  Dari node lain (misal **Manwe**), tes login FTP ke Eru sebagai `ainur` (harus berhasil) dan `melkor` (harus gagal).

### Skrip yang Digunakan

**`soal_7.sh` (di Node Eru)**

   ```bash
   #!/bin/bash
   apt-get update > /dev/null 2>&1 && apt-get install -y vsftpd
   useradd -m -s /bin/bash ainur && echo "ainur:ftpaman" | chpasswd
   useradd -m -s /bin/bash melkor && echo "melkor:ftpgagal" | chpasswd
   cp /etc/vsftpd.conf /etc/vsftpd.conf.bak
   cat > /etc/vsftpd.conf << EOF
   listen=YES
   anonymous_enable=NO
   local_enable=YES
   write_enable=YES
   chroot_local_user=YES
   userlist_enable=YES
   userlist_file=/etc/vsftpd.userlist
   userlist_deny=NO
   EOF
   echo "ainur" > /etc/vsftpd.userlist
   service vsftpd restart
   echo "FTP Server Siap."
```

### Soal 8: Analisis Upload FTP
## Tujuan
Menganalisis proses upload file ke FTP dan mengidentifikasi perintah STOR di Wireshark.

## Langkah-langkah Pengerjaan
Jalankan skrip soal_8.sh di Ulmo.

Mulai capture di GNS3 pada koneksi Eru-Switch2.

Dari Ulmo, jalankan ftp 192.18.2.1, login sebagai ainur, lalu put ramalan.txt.

Hentikan capture dan cari paket STOR di Wireshark.

## Skrip yang Digunakan

soal_8.sh (di Node Ulmo)

Bash

   #!/bin/bash
   echo "=== Persiapan Ulmo (Soal 8) ==="
   apt-get update > /dev/null 2>&1 && apt-get install -y ftp
   echo "data cuaca untuk Eru" > /root/ramalan.txt
   echo "--> Siap untuk upload. Jalankan 'ftp 192.18.2.1'"


Bukti Pengerjaan (Screenshot)
[➡️ Ambil Screenshot di sini] Screenshot Wireshark yang menyorot paket Request: STOR ramalan.txt.

Saran penamaan file: images/soal8_wireshark_stor.png

Soal 9: Analisis Download FTP & Read-Only
Tujuan
Mengubah hak akses menjadi read-only dan menganalisis proses download (RETR).

Langkah-langkah Pengerjaan
Jalankan skrip soal_9.sh di Eru.

Mulai capture pada koneksi Manwe-Eru.

Dari Manwe, jalankan ftp 192.18.1.1, login sebagai ainur, lalu get kitab.txt. Coba juga put file.txt untuk membuktikan kegagalannya.

Hentikan capture dan cari paket RETR di Wireshark.

Skrip yang Digunakan
soal_9.sh (di Node Eru)

Bash

#!/bin/bash
echo "=== Konfigurasi Read-Only FTP (Soal 9) ==="
sed -i 's/write_enable=YES/write_enable=NO/' /etc/vsftpd.conf
service vsftpd restart
echo "isi kitab" > /home/ainur/kitab.txt
echo "--> Konfigurasi read-only aktif."
Bukti Pengerjaan (Screenshot)
[➡️ Ambil Screenshot di sini] Screenshot terminal Manwe yang menunjukkan get berhasil dan put gagal.

[➡️ Ambil Screenshot di sini] Screenshot Wireshark yang menyorot paket Request: RETR kitab.txt.

Saran penamaan file: images/soal9_test_akses.png, images/soal9_wireshark_retr.png

Soal 10: Simulasi Serangan Ping Flood
Tujuan
Mengirim 100 paket ping dari Melkor ke Eru dan menganalisis hasilnya.

Langkah-langkah Pengerjaan
Jalankan skrip soal_10.sh di node Melkor.

Tunggu hingga selesai dan amati blok statistik di akhir output.

Skrip yang Digunakan
soal_10.sh (di Node Melkor)

Bash

#!/bin/bash
echo "=== Memulai Ping Flood (Soal 10) ==="
ping -c 100 192.18.1.1
echo "=== Ping Flood Selesai ==="
Bukti Pengerjaan (Screenshot)
[➡️ Ambil Screenshot di sini] Screenshot blok statistik di akhir output ping.

Saran penamaan file: images/soal10_ping_statistik.png

Soal 11: Membuktikan Kelemahan Telnet
Tujuan
Mendemonstrasikan bahwa Telnet tidak aman dengan menangkap kredensial login.

Langkah-langkah Pengerjaan
Jalankan skrip soal_11.sh di Melkor.

Mulai capture pada koneksi Eru-Melkor.

Dari Eru, jalankan telnet 192.18.1.2 dan login.

Analisis di Wireshark menggunakan "Follow TCP Stream".

Skrip yang Digunakan
soal_11.sh (di Node Melkor)

Bash

#!/bin/bash
echo "=== Persiapan Server Telnet (Soal 11) ==="
apt-get update > /dev/null 2>&1 && apt-get install -y inetutils-inetd telnetd
sed -i 's/^#<off># telnet.*/telnet\tstream\ttcp\tnowait\troot\t/usr/sbin/tcpd\t/usr/sbin/telnetd/' /etc/inetd.conf
USERNAME="eru_login"
if id "$USERNAME" &>/dev/null; then userdel -r "$USERNAME"; fi
useradd -m -d /home/$USERNAME -s /bin/bash "$USERNAME"
echo "eru_login:password_terlihat_jelas" | chpasswd
killall inetutils-inetd > /dev/null 2>&1
# Menggunakan nama biner yang benar yang kita temukan saat troubleshooting
/usr/sbin/inetutils-inetd
echo "--> Server Telnet Siap."
Bukti Pengerjaan (Screenshot)
[➡️ Ambil Screenshot di sini] Screenshot jendela "Follow TCP Stream" di Wireshark yang menunjukkan username dan password.

Saran penamaan file: images/soal11_telnet_plaintext.png

Soal 12: Pemindaian Port dengan Netcat
Tujuan
Latihan memindai port dengan nc untuk memeriksa status port 21, 80 (terbuka) dan 666 (tertutup).

Langkah-langkah Pengerjaan
Jalankan skrip soal_12.sh di Melkor.

Dari Eru, jalankan tiga perintah nc -zv 192.18.1.2 <port> untuk setiap port (21, 80, 666).

Skrip yang Digunakan
soal_12.sh (di Node Melkor)

Bash

#!/bin/bash
echo "=== Menjalankan Listener Netcat (Soal 12) ==="
killall nc > /dev/null 2>&1
apt-get update > /dev/null 2>&1 && apt-get install -y netcat-traditional
nc -l -p 21 &
nc -l -p 80 &
echo "--> Listener di port 21 dan 80 aktif."
Bukti Pengerjaan (Screenshot)
[➡️ Ambil Screenshot di sini] Screenshot terminal Eru yang menunjukkan hasil dari ketiga perintah scan nc.

Saran penamaan file: images/soal12_hasil_scan.png

Soal 13: Mendemonstrasikan Keamanan SSH
Tujuan
Menunjukkan bahwa SSH aman dengan menganalisis sesi login yang terenkripsi.

Langkah-langkah Pengerjaan
Jalankan skrip soal_13_server.sh di Eru, lalu atur password root dengan passwd.

Jalankan skrip soal_13_client.sh di Varda.

Mulai capture pada koneksi Varda-Eru.

Dari Varda, jalankan ssh root@192.18.2.1 dan login.

Analisis paket SSH di Wireshark untuk menunjukkan data terenkripsi.

Skrip yang Digunakan
soal_13_server.sh (di Node Eru)

Bash

#!/bin/bash
echo "=== Konfigurasi Server SSH di Eru (Soal 13) ==="
apt-get update > /dev/null 2>&1 && apt-get install -y openssh-server
sed -i 's/^#*PermitRootLogin.*/PermitRootLogin yes/' /etc/ssh/sshd_config
service ssh restart
echo "--> Server SSH Siap. Jalankan 'passwd' untuk set password root."
soal_13_client.sh (di Node Varda)

Bash

#!/bin/bash
echo "=== Konfigurasi Client SSH di Varda (Soal 13) ==="
apt-get update > /dev/null 2>&1 && apt-get install -y openssh-client
# IP Address sudah diatur oleh langkah-langkah manual di soal 3
echo "--> Client SSH Siap."
Bukti Pengerjaan (Screenshot)
[➡️ Ambil Screenshot di sini] Jendela Wireshark yang menampilkan detail paket SSH dan menunjukkan bagian "Encrypted Data".

Saran penamaan file: images/soal13_ssh_encrypted.png


### Soal 14

Command di bash untuk menyelesaikan soal.

'''
nc 10.15.43.32 3401
'''

Setelah itu akan muncul Sub-soal yang jawabannya dapat ditemukan di wireshark.

### Sub-Soal 1

How many packets are recorded in the pcapng file?

### Jawaban

500358

### Sub-Soal 2

What are the user that successfully logged in? Format: user:pass

### Jawaban

n1enna:y4v4nn4_k3m3nt4r1

### Sub-Soal 3

In which stream were the credentials found? Format: int

### Jawaban

41824

Jawaban dari ketiga Sub-soal bisa dilihat dengan mengikuti prosedur berikut:

Buka file > right click > follow

TARO SS DI SINI

### Flag

### Soal 15

### Sub-soal

### Flag

### Soal 16

### Sub-soal

### Flag

### Soal 19

### Sub-soal

### Flag




