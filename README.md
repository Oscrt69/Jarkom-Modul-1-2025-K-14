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
**`soal_5_permanen.sh` (di Node Eru)**
```bash
#!/bin/bash
echo "=== Membuat Konfigurasi Permanen di Eru (Soal 5) ==="
sed -i 's/^#net.ipv4.ip_forward=1/net.ipv4.ip_forward=1/' /etc/sysctl.conf
sysctl -p
echo iptables-persistent iptables-persistent/autosave_v4 boolean true | debconf-set-selections
echo iptables-persistent iptables-persistent/autosave_v6 boolean true | debconf-set-selections
apt-get install -y iptables-persistent
echo "--> Konfigurasi permanen selesai."
