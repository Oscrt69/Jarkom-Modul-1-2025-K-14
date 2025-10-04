
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

    #!/bin/bash
# Script Internet Eru
apt update
apt install iptables -y

# Enable IP forwarding
echo 1 > /proc/sys/net/ipv4/ip_forward

# Setup NAT (kasih internet ke semua client)
iptables -t nat -F
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.218.0.0/16
# Allow forwarding
iptables -A FORWARD -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT
iptables -A FORWARD -i eth2 -o eth0 -j ACCEPT

echo nameserver 192.168.122.1 > /etc/resolv.conf
echo "Eru internet configured!"
   

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

`nc 10.15.43.32 3401`

### Sub-Soal 1

How many packets are recorded in the pcapng file?

### Jawaban

```
500358
```

### Sub-Soal 2

What are the user that successfully logged in? Format: user:pass

<img width="800" height="1200" alt="ws142" src="https://github.com/user-attachments/assets/01cf855e-3864-49bc-9a49-a78522f3e50d" />

### Jawaban

```
n1enna:y4v4nn4_k3m3nt4r1
```

### Sub-Soal 3

In which stream were the credentials found? Format: int

### Jawaban

```
41824
```

### Sub-Soal 4

What tools are used for brute force? Format: Hydra v1.8.0-dev

### Jawaban

``` Fuzz Faster U Fool v2.1.0-dev ```

Jawaban dari keempat Sub-soal bisa dilihat dengan mengikuti prosedur berikut:

Buka file > right click > follow

### Flag

```
KOMJAR25{Brut3_F0rc3_kEo19FVGnWOwcUk1mvLDKAmCa}  
```

<img width="800" height="1200" alt="ws146" src="https://github.com/user-attachments/assets/bdbcb609-3806-47e1-bb88-63a9f7eb8ce3" />

### Soal 15

``` 
nc 10.15.43.32 3402
```

### Sub-soal 1

What device does Melkor use?

### Jawaban

```
Keyboard
```

<img width="800" height="1200" alt="ws152" src="https://github.com/user-attachments/assets/4d9ecdde-cdac-4d27-92a9-71db3737072b" />

### Sub-soal 2

What did Melkor write?

Gunakan filter ```(Frame.len==35)``` > Hilangkan semua kolom kecuali HID ID > File > Export Packet Dissections > as csv

buat script .py (decode_hid.py) untuk encode, dan jadikan file .csv (dataws.csv) satu folder dengan script .py.

Ini adalah isi decode_hid.py:

```
hid_map = {
    0x04: "a", 0x05: "b", 0x06: "c", 0x07: "d",
    0x08: "e", 0x09: "f", 0x0A: "g", 0x0B: "h",
    0x0C: "i", 0x0D: "j", 0x0E: "k", 0x0F: "l",
    0x10: "m", 0x11: "n", 0x12: "o", 0x13: "p",
    0x14: "q", 0x15: "r", 0x16: "s", 0x17: "t",
    0x18: "u", 0x19: "v", 0x1A: "w", 0x1B: "x",
    0x1C: "y", 0x1D: "z",
    0x1E: "1", 0x1F: "2", 0x20: "3", 0x21: "4",
    0x22: "5", 0x23: "6", 0x24: "7", 0x25: "8",
    0x26: "9", 0x27: "0",
    0x2C: " ",   
    0x28: "\n", 
    0x2E: "="
}

shift_map = {
    0x1E: "!", 0x1F: "@", 0x20: "#", 0x21: "$",
    0x22: "%", 0x23: "^", 0x24: "&", 0x25: "*",
    0x26: "(", 0x27: ")",
    0x04: "A", 0x05: "B", 0x06: "C", 0x07: "D",
    0x08: "E", 0x09: "F", 0x0A: "G", 0x0B: "H",
    0x0C: "I", 0x0D: "J", 0x0E: "K", 0x0F: "L",
    0x10: "M", 0x11: "N", 0x12: "O", 0x13: "P",
    0x14: "Q", 0x15: "R", 0x16: "S", 0x17: "T",
    0x18: "U", 0x19: "V", 0x1A: "W", 0x1B: "X",
    0x1C: "Y", 0x1D: "Z"
}

def decode_key(modifier, keycode):
    is_shift = (modifier & 0x22) != 0  
    if is_shift and keycode in shift_map:
        return shift_map[keycode]
    elif keycode in hid_map:
        return hid_map[keycode]
    else:
        return ""

def decode_line(hexline):
    hexline = hexline.strip().strip('"')
    bytestr = [hexline[i:i+2] for i in range(0, len(hexline), 2)]
    if len(bytestr) > 2:
        modifier = int(bytestr[0], 16)   
        keycode  = int(bytestr[2], 16)
        return decode_key(modifier, keycode)
    return ""


def main():
    infile = "/mnt/c/tugas/JARKOM/dataws.csv"  
    output = ""
    with open(infile, "r") as f:
        for line in f:
            line = line.strip().strip('"')
            # Skip header atau baris kosong
            if not line or not all(c in "0123456789abcdefABCDEF" for c in line.replace('"','')):
                continue
            output += decode_line(line)
    print("Decoded output:")
    print(output)

if __name__ == "__main__":
    main()

```
gunakan command untuk decode:

``` 
python3 decode_hid.py
```

<img width="946" height="274" alt="ws156" src="https://github.com/user-attachments/assets/9746c077-1128-442f-af09-f4f1e889e2c1" />

### Jawaban 
```
UGx6X3ByMHYxZGVfeTB1cl91czNybjRtZV80bmRfcDRzc3cwcmQ= 
```

### Sub-soal 3

What did Melkor write?

Gunakan command ``` echo "UGx6X3ByMHYxZGVfeTB1cl91czNybjRtZV80bmRfcDRzc3cwcmQ=" | base64 --decode ```

```Plz_pr0v1de_y0ur_us3rn4me_4nd_p4ssw0rd```

### Flag

```KOMJAR25{K3yb0ard_W4rr10r_BRxsRQ8etjElDYMOJBbksIR0d}```

<img width="1505" height="770" alt="w151" src="https://github.com/user-attachments/assets/de78ed5c-6f86-4997-a06d-1c18959624a6" />

### Soal 16

```
nc 10.15.43.32 3403
```
### Sub-soal 1

What credential did the attacker use to log in? Format: user:pass

1. Gunakan filter ```tcp.stream eq 1```

<img width="600" height="400" alt="ws161" src="https://github.com/user-attachments/assets/67baf020-582c-42cd-bfc3-5816abaeba2c" />

2. Right Click > Follow > TCP Stream

<img width="1200" height="800" alt="ws162" src="https://github.com/user-attachments/assets/f5969a10-e264-45b8-ad3f-6cf89f893253" />

Jawaban 

```
ind@psg420.com:{6r_6e#TfT1p
```
### Sub-soal 2

How many files are suspected of containing malware? Format: int

1. Ditemukan bahwa terdapat 5 jenis file `.exe` yang berbeda diantaranya:
> q.exe <br>
> w.exe <br>
> e.exe <br>
> r.exe <br>
> t.exe <BR>

Jawaban

```
5 
```
### Sub-soal 3

What is the hash of the first file (q.exe)? Format: sha256

<img width="1200" height="676" alt="ws165" src="https://github.com/user-attachments/assets/43f64eea-c3dc-41e9-b4c4-e6540dd688ef" />

Klik salah satu file q.exe > Follow > TCP Stream > Save as raw > beri nama file (encrypt1.exe)

```
sha256sum encrypt1 
```

<img width="1218" height="132" alt="ws166" src="https://github.com/user-attachments/assets/6c021e09-8b8b-47cc-8ee4-d0c03afdda76" />

Jawaban

``` 
ca34b0926cdc3242bbfad1c4a0b42cc2750d90db9a272d92cfb6cb7034d2a3bd
```

### Sub-soal 4

What is the hash of the second file (w.exe)? Format: sha256

Lakukan langkah yang sama dengan soal sebelumya, Klik salah satu file q.exe > Follow > TCP Stream > Save as raw > beri nama file (encrypt2.exe)

Jawaban 

```
08eb941447078ef2c6ad8d91bb2f52256c09657ecd3d5344023edccf7291e9fc
 ```

### Sub-soal 5

What is the hash of the third file (e.exe)? Format: sha256

Lakukan langkah yang sama dengan soal sebelumya, Klik salah satu file q.exe > Follow > TCP Stream > Save as raw > beri nama file (encrypt3.exe)

Jawaban

```
32e1b3732cd779af1bf7730d0ec8a7a87a084319f6a0870dc7362a15ddbd3199
```

### Sub-soal 6

What is the hash of the fourth file (r.exe)? Format: sha256

Lakukan langkah yang sama dengan soal sebelumya, Klik salah satu file q.exe > Follow > TCP Stream > Save as raw > beri nama file (encrypt4.exe)

Jawaban

```
4ebd58007ee933a0a8348aee2922904a7110b7fb6a316b1c7fb2c6677e613884
```

### Sub-soal 7
 
What is the hash of the fifth file (t.exe)? Format: sha256

Lakukan langkah yang sama dengan soal sebelumya, Klik salah satu file q.exe > Follow > TCP Stream > Save as raw > beri nama file (encrypt5.exe)

Jawaban

```
10ce4b79180a2ddd924fdc95951d968191af2ee3b7dfc96dd6a5714dbeae613a 
```

### Flag

<img width="2880" height="1716" alt="ws164" src="https://github.com/user-attachments/assets/7f02d5e5-e941-453c-a4a0-1290b29f1e8f" />

```
KOMJAR25{Y0u_4r3_4_g00d_4nalyz3r_iDYON1mAHIfDzG2S49awKOBRm}
```
### Soal 19

```
nc 10.15.43.32 3406
```

### Sub-soal 1

Who sent the threatening message? Format: string (name)

1. Follow > TCP Stream

<img width="1388" height="375" alt="ws191 - Copy" src="https://github.com/user-attachments/assets/743466f6-686c-413a-be54-0dc0ae6046c4" />

Jawaban

``` 
Your Life
```

## Sub-soal 2

How much ransom did the attacker demand ($)? Format: int

1. Follow > TCP Stream

<img width="1824" height="227" alt="ws191" src="https://github.com/user-attachments/assets/86ef70c9-e98e-42ed-b704-45d14683bd6e" />

Jawaban

``` 
1600
```

### Sub-soal3

What is the hash of the second suspicious file (knr.exe)? Format: sha256

<img width="1832" height="587" alt="ws192" src="https://github.com/user-attachments/assets/9884281a-af13-462a-b2ec-b11a28c780b7" />

Jawaban

```
1CWHmuF8dHt7HBGx5RKKLgg9QA2GmE3UyL
```

### Flag

```
KOMJAR25{Y0u_4re_J4rk0m_G0d_uYgq9qyc7cdrJ2F5gyBt0otqK}
```
x<img width="1486" height="884" alt="ws193" src="https://github.com/user-attachments/assets/c1ecd241-f075-41bd-a180-83c88975d9b5" />


