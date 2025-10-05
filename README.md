
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
<img width="1712" height="802" alt="Screenshot 2025-09-29 194845" src="https://github.com/user-attachments/assets/fce14020-1456-4b68-9f43-b546eb8cc4be" />

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
<img width="1281" height="551" alt="Screenshot 2025-10-04 230808" src="https://github.com/user-attachments/assets/645b22b9-6f3a-4151-9d16-86329f519c69" />

<img width="893" height="384" alt="Screenshot 2025-10-04 230846" src="https://github.com/user-attachments/assets/44f9b2a2-e83d-421c-bd71-559e235992a4" />

---

## Soal 5: Membuat Konfigurasi Permanen

### Tujuan
Menyimpan semua konfigurasi jaringan agar tidak hilang saat node di-restart.

### Langkah-langkah Pengerjaan
1.  **Konfigurasi IP Permanen (Client):** Edit file `/etc/network/interfaces` di setiap node client (Melkor, Manwe, Varda, Ulmo) untuk IP statis.
2.  **Konfigurasi IP Forwarding & NAT Permanen di Eru:** Jalankan skrip `soal_5_permanen.sh` di node **Eru**.

### Skrip yang Digunakan

**`soal_5_permanen.sh` (di Node Eru)**\

         !/bin/bash
         apt update
            apt install iptables -y
            echo 1 > /proc/sys/net/ipv4/ip_forward
            iptables -t nat -F
            iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.218.0.0/16
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
2.  Dari **Manwe**, hasilkan trafik dengan menjalankan file traffic.sh.
3.  Di Wireshark, gunakan filter `ip.addr == 192.18.1.3`.

### Skrip yang Digunakan
Tidak ada skrip utama, pengerjaan berfokus pada GNS3 dan Wireshark.

### Bukti Pengerjaan (Screenshot)
<img width="1920" height="1080" alt="Screenshot (172)" src="https://github.com/user-attachments/assets/b94d8b8c-2d28-4d54-83f2-721176adfbc7" />

---

   ## Soal 7: Konfigurasi FTP Server

### Tujuan
Memasang FTP server di Eru dengan hak akses berbeda untuk `ainur` dan `melkor`.

### Langkah-langkah Pengerjaan
1.  Jalankan skrip `soal_7.sh` di **Eru**.
2.  Dari node lain (misal **Manwe**), tes login FTP ke Eru sebagai `ainur` (harus berhasil) dan `melkor` (harus gagal).

### Skrip yang Digunakan

**`soal_7.sh` (di Node Eru)**

    #!/bin/bash
    # SCRIPT NOMOR 7 - FTP SERVER
    
    echo "=========================================="
    echo "   NOMOR 7: FTP SERVER SETUP"
    echo "=========================================="
    
    # Cleanup
    echo "[0/8] Cleanup..."
    pkill vsftpd 2>/dev/null || true
    umount /home/ainur/ftp/shared 2>/dev/null || true
    rm -rf /home/ainur/ftp /home/melkor/ftp 2>/dev/null || true
    
    # Persistence script
    cat > /root/ftp_startup.sh << 'STARTUP_SCRIPT'
    #!/bin/bash
    mkdir -p /home/ainur/ftp/shared
    mount --bind /home/shared /home/ainur/ftp/shared 2>/dev/null
    if ! pgrep vsftpd > /dev/null; then
        /usr/sbin/vsftpd /etc/vsftpd.conf &
    fi
    STARTUP_SCRIPT
    
    chmod +x /root/ftp_startup.sh
    
    # Add to rc.local
    if [ -f /etc/rc.local ]; then
        sed -i '/ftp_startup.sh/d' /etc/rc.local
        sed -i '/^exit 0/i /root/ftp_startup.sh' /etc/rc.local
    else
        echo -e '#!/bin/bash\n/root/ftp_startup.sh\nexit 0' > /etc/rc.local
        chmod +x /etc/rc.local
    fi
    
    echo "  ✓ Persistence configured"
    
    # Install vsftpd
    echo "[1/8] Installing vsftpd..."
    apt-get update > /dev/null 2>&1
    apt-get install -y vsftpd > /dev/null 2>&1
    echo "  ✓ vsftpd installed"
    
    # Create users
    echo "[2/8] Creating users..."
    if ! id ainur &>/dev/null; then
        useradd -m ainur -s /bin/bash
    fi
    echo "ainur:password123" | chpasswd
    
    if ! id melkor &>/dev/null; then
        useradd -m melkor -s /bin/bash
    fi
    echo "melkor:password123" | chpasswd
    echo "  ✓ Users created"
    
    # Create shared folder and files
    echo "[3/8] Creating shared folder..."
    mkdir -p /home/shared
    chmod 755 /home/shared
    
    echo "File test untuk membuktikan akses user FTP - $(date)" > /home/shared/bukti_nomor7.txt
    
    cat > /home/shared/kitab_penciptaan.txt << 'KITAB_FILE'
    === KITAB PENCIPTAAN ARDA ===
    Pada mulanya adalah Eru, Yang Satu, yang di Arda disebut Ilúvatar.
    Dia menciptakan Ainur, Yang Suci, yang menyanyi musik penciptaan.
    Melkor memasukkan disonansi, menciptakan kejahatan.
    Manwë, Varda, dan Ulmo membantu pembentukan Arda.
    === Akhir Kitab ===
    KITAB_FILE
    
    echo "Test file di shared folder - $(date)" > /home/shared/test.txt
    
    chmod 644 /home/shared/*
    chown root:root /home/shared/*
    echo "  ✓ Shared folder created"
    
    # Setup ainur FTP structure
    echo "[4/8] Setting up ainur FTP..."
    mkdir -p /home/ainur/ftp/files
    mkdir -p /home/ainur/ftp/shared
    
    chown nobody:nogroup /home/ainur/ftp
    chmod 555 /home/ainur/ftp
    
    chown ainur:ainur /home/ainur/ftp/files
    chmod 755 /home/ainur/ftp/files
    
    echo "Welcome to Ainur's FTP! Upload your files here. - $(date)" > /home/ainur/ftp/files/welcome.txt
    echo "Sample file in files folder - $(date)" > /home/ainur/ftp/files/sample.txt
    echo "You can upload and download files in this folder." > /home/ainur/ftp/files/readme.txt
    
    chown ainur:ainur /home/ainur/ftp/files/*
    chmod 644 /home/ainur/ftp/files/*
    
    mount --bind /home/shared /home/ainur/ftp/shared
    echo "  ✓ Ainur FTP structure created"
    
    # Setup melkor FTP structure
    echo "[5/8] Setting up melkor FTP..."
    mkdir -p /home/melkor/ftp
    
    chown nobody:nogroup /home/melkor/ftp
    chmod 555 /home/melkor/ftp
    
    echo "Access restricted by Eru Iluvatar - No shared files available" > /home/melkor/ftp/.message
    chown nobody:nogroup /home/melkor/ftp/.message
    chmod 444 /home/melkor/ftp/.message
    echo "  ✓ Melkor FTP structure (RESTRICTED)"
    
    # Configure vsftpd
    echo "[6/8] Configuring vsftpd..."
    [ -f /etc/vsftpd.conf ] && cp /etc/vsftpd.conf /etc/vsftpd.conf.backup
    
    cat > /etc/vsftpd.conf << 'VSFTPD_CONFIG'
    listen=NO
    listen_ipv6=YES
    anonymous_enable=NO
    local_enable=YES
    write_enable=YES
    local_umask=022
    dirmessage_enable=YES
    use_localtime=YES
    xferlog_enable=YES
    xferlog_std_format=YES
    connect_from_port_20=YES
    chroot_local_user=YES
    allow_writeable_chroot=YES
    secure_chroot_dir=/var/run/vsftpd/empty
    pam_service_name=vsftpd
    user_sub_token=$USER
    local_root=/home/$USER/ftp
    pasv_enable=YES
    pasv_min_port=40000
    pasv_max_port=50000
    message_file=.message
    VSFTPD_CONFIG
    
    mkdir -p /var/run/vsftpd/empty
    chown root:root /var/run/vsftpd/empty
    echo "  ✓ vsftpd configured"
    
    # Start FTP service
    echo "[7/8] Starting FTP service..."
    /usr/sbin/vsftpd /etc/vsftpd.conf &
    sleep 2
    
    if pgrep vsftpd > /dev/null; then
        echo "  ✓ FTP service running"
    else
        /usr/sbin/vsftpd /etc/vsftpd.conf &
        sleep 1
    fi
    
    # Verification
    echo "[8/8] Verification..."
    echo ""
    echo "FTP Users:"
    id ainur | grep uid
    id melkor | grep uid
    
    echo ""
    echo "Shared files:"
    ls -lh /home/shared/
    
    echo ""
    echo "Ainur files folder:"
    ls -lh /home/ainur/ftp/files/
    
    echo ""
    echo "FTP Service:"
    if pgrep vsftpd > /dev/null; then
        echo "  ✓ vsftpd running (PID: $(pgrep vsftpd))"
    else
        echo "  ✗ vsftpd NOT running"
    fi
    
    # Summary
    echo ""
    echo "=========================================="
    echo "   NOMOR 7 SETUP COMPLETED!"
    echo "=========================================="
    echo ""
    echo "FTP Server: 192.218.2.1"
    echo ""
    echo "User 1: ainur / password123"
    echo "  - /files (read/write)"
    echo "  - /shared (read/write)"
    echo ""
    echo "User 2: melkor / password123"
    echo "  - RESTRICTED (no shared access)"
    echo ""
    echo "Ready for testing!"
    echo "=========================================="

lalu jalanin ftp nya di melkor dengan ketentuan yang ada di soal

<img width="847" height="626" alt="Screenshot 2025-10-04 231457" src="https://github.com/user-attachments/assets/d6c5d1ed-963f-4156-8760-55e40106f445" />


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

       #!/bin/bash
       echo "=== Persiapan Ulmo (Soal 8) ==="
       apt-get update > /dev/null 2>&1 && apt-get install -y ftp
       echo "data cuaca untuk Eru" > /root/ramalan.txt
       echo "--> Siap untuk upload. Jalankan 'ftp 192.18.2.1'"


<img width="1920" height="1080" alt="Screenshot (173)" src="https://github.com/user-attachments/assets/7c3cd4b4-ed8c-4064-9584-de175a94094f" />


## Soal 9: Analisis Download FTP & Read-Only
Tujuan
Mengubah hak akses menjadi read-only dan menganalisis proses download (RETR).

Langkah-langkah Pengerjaan
Jalankan skrip soal_9.sh di Eru.

Mulai capture pada koneksi Manwe-Eru.

Dari Manwe, jalankan ftp 192.18.1.1, login sebagai ainur, lalu get kitab.txt. Coba juga put file.txt untuk membuktikan kegagalannya.

Hentikan capture dan cari paket RETR di Wireshark.

Skrip yang Digunakan

soal_9.sh (di Node Eru)

    #!/bin/bash
    # Script Nomor 9 - Setup FTP Read-Only di Eru
    
    echo "=== NOMOR 9: FTP READ-ONLY SETUP ==="
    
    # Copy file kitab ke FTP directory
    echo "Copying kitab_penciptaan.txt to FTP directory..."
    cp kitab_penciptaan.txt /home/ainur/ftp/files/
    chown ainur:ainur /home/ainur/ftp/files/kitab_penciptaan.txt
    chmod 644 /home/ainur/ftp/files/kitab_penciptaan.txt
    
    # Backup original config
    cp /etc/vsftpd.conf /etc/vsftpd.conf.backup
    
    # Modify config untuk READ-ONLY
    echo "Setting FTP server to READ-ONLY mode..."
    sed -i 's/write_enable=YES/write_enable=NO/' /etc/vsftpd.conf
    
    # Verify config change
    echo "Current write_enable setting:"
    grep "write_enable" /etc/vsftpd.conf
    
    # Restart FTP server
    echo "Restarting FTP server..."
    pkill vsftpd
    sleep 2
    /usr/sbin/vsftpd /etc/vsftpd.conf &
    
    # Verify FTP server running
    echo "FTP Server status:"
    pgrep vsftpd && echo "FTP Server is running" || echo "FTP Server failed to start"
    
    # List files in FTP directory
    echo "Files available for download:"
    ls -la /home/ainur/ftp/files/
    
    echo "=== ERU SETUP COMPLETE - FTP SERVER IS READ-ONLY ==="
    
Bukti Pengerjaan (Screenshot)
<img width="1438" height="359" alt="Screenshot 2025-10-04 232439" src="https://github.com/user-attachments/assets/47a4eb6f-3a71-4f20-8d6d-5f8bb65c8fed" />


<img width="1920" height="1080" alt="Screenshot (174)" src="https://github.com/user-attachments/assets/b0a18921-07c7-4f0c-a341-764f7c2ff3ac" />


Soal 10: Simulasi Serangan Ping Flood
Tujuan
Mengirim 100 paket ping dari Melkor ke Eru dan menganalisis hasilnya.

Langkah-langkah Pengerjaan
Jalankan skrip soal_10.sh di node Melkor.

Tunggu hingga selesai dan amati blok statistik di akhir output.

Skrip yang Digunakan
soal_10.sh (di Node Melkor)

    #!/bin/bash
    echo "=== Memulai Ping Flood (Soal 10) ==="
    ping -c 100 192.18.1.1
    echo "=== Ping Flood Selesai ==="

Tidak ada paket yang loss untuk nomor 10 ini dan tidak mempengaruhi kinerja eru

## Soal 11: Membuktikan Kelemahan Telnet
Tujuan
Mendemonstrasikan bahwa Telnet tidak aman dengan menangkap kredensial login.

Langkah-langkah Pengerjaan
Jalankan skrip soal_11.sh di Melkor.

Mulai capture pada koneksi Eru-Melkor.

Dari Eru, jalankan telnet 192.18.1.2 dan login.

Analisis di Wireshark menggunakan "Follow TCP Stream".

Skrip yang Digunakan
soal_11.sh (di Node Melkor)

    #!/bin/bash
    echo "=== Memulai Setup Telnet dengan Metode Alternatif (xinetd) - VERSI PERBAIKAN ==="
    apt-get update > /dev/null 2>&1
    echo "[1/4] Menginstal xinetd dan telnetd..."
    apt-get purge -y inetutils-inetd > /dev/null 2>&1
    apt-get install -y xinetd telnetd
    echo "[2/4] Membuat file konfigurasi /etc/xinetd.d/telnet..."
    cat > /etc/xinetd.d/telnet << EOF
    service telnet
    {
        disable         = no
        flags           = REUSE
        socket_type     = stream
        wait            = no
        user            = root
        server          = /usr/sbin/telnetd
        log_on_failure  += USERID
    }
    EOF
    echo "[3/4] Membuat user 'eru_login' dengan shell yang benar..."
    USERNAME="eru_login"
    PASSWORD="password_terlihat_jelas"
    if id "$USERNAME" &>/dev/null; then userdel -r "$USERNAME"; fi
    # --- PERBAIKAN DI SINI: dari /bash menjadi /bin/bash ---
    useradd -m -d /home/$USERNAME -s /bin/bash "$USERNAME"
    echo "$USERNAME:$PASSWORD" | chpasswd
    echo "[4/4] Merestart layanan xinetd..."
    service xinetd restart
    echo ""
    echo "=== Setup Selesai. Server Telnet Siap ==="
    

Setelah menjalankannya lalu lakukan langkah2 sebagai berikut :
1. Koneksi Telnet dari Eru ke Melkor
bashtelnet 192.218.1.2
Login:

Username: eruadmin
Password: telnetpass123

Jalankan beberapa command:
bashwhoami
pwd
ls
exit

2. Stop Wireshark dan Analisis
Apply display filter:
telnet
Follow TCP Stream:

Klik kanan pada packet telnet
Follow → TCP Stream
Username dan password terlihat jelas sebagai plain text

<img width="1920" height="1080" alt="Screenshot (176)" src="https://github.com/user-attachments/assets/2f6ffe20-3c4a-4964-99ec-1b8493a514a8" />

## Soal 12: Pemindaian Port dengan Netcat
Tujuan
Latihan memindai port dengan nc untuk memeriksa status port 21, 80 (terbuka) dan 666 (tertutup).

Langkah-langkah Pengerjaan
Jalankan skrip soal_12.sh di Melkor.

Dari Eru, jalankan tiga perintah nc -zv 192.18.1.2 <port> untuk setiap port (21, 80, 666).

Skrip yang Digunakan
soal_12.sh (di Node Melkor)

    #!/bin/bash
    echo "=== Memulai Skrip Persiapan Port Scan ==="
    echo "[1/3] Membersihkan proses 'nc' yang mungkin sudah berjalan..."
    killall nc > /dev/null 2>&1
    echo "--> Selesai."
    echo "[2/3] Menjalankan listener di port 21 (background)..."
    nc -l -p 21 &
    echo "[3/3] Menjalankan listener di port 80 (background)..."
    nc -l -p 80 &
    sleep 1
    echo ""
    echo "=== Verifikasi Proses yang Berjalan ==="
    ps aux | grep "[n]c -l"
    echo ""
    echo "Skrip selesai. Node Melkor sekarang siap untuk dipindai dari Eru."
    echo "Pastikan Anda melihat dua proses 'nc' di atas."


<img width="696" height="219" alt="Screenshot 2025-10-04 235107" src="https://github.com/user-attachments/assets/0f48c522-241b-4a97-a7f8-bd7d30afb33e" />


## Soal 13: SSH (Secure Shell) Connection Analysis
# Tujuan
Membuktikan bahwa SSH mengenkripsi data (username, password, command) sehingga tidak dapat dibaca di Wireshark, berbeda dengan Telnet yang mengirim plain text.

Langkah-langkah Pengerjaan
1. Setup SSH Server di Node Eru
Jalankan skrip setup_ssh_eru.sh di Eru.
2. Koneksi SSH dengan Wireshark

Buka Wireshark di interface Varda-Eru → Start Capturing
Dari Varda, jalankan: ssh root@192.218.2.1
Login dengan password: password123
Jalankan beberapa command (whoami, hostname, exit)
Stop Wireshark → Apply filter: ssh.encrypted_packet
Save capture: ssh_varda_eru.pcapng

3. Screenshot Bukti
Ambil screenshot Wireshark yang menampilkan Encrypted Packet dengan detail SSH Protocol terlihat.

Skrip yang Digunakan
setup_ssh_eru.sh (di Node Eru)

    bash#!/bin/bash
    echo "=== Setup SSH Server di Node Eru ==="
    echo "[1/5] Update dan install OpenSSH Server..."
    apt-get update > /dev/null 2>&1
    apt-get install openssh-server -y > /dev/null 2>&1
    echo "--> Selesai."
    echo "[2/5] Generate SSH host keys..."
    ssh-keygen -A > /dev/null 2>&1
    mkdir -p /var/run/sshd
    echo "--> Selesai."
    echo "[3/5] Konfigurasi SSH..."
    cat > /etc/ssh/sshd_config << EOF
    Port 22
    ListenAddress 0.0.0.0
    PermitRootLogin yes
    PasswordAuthentication yes
    EOF
    echo "--> Selesai."
    echo "[4/5] Set password root..."
    echo "root:password123" | chpasswd
    echo "--> Selesai."
    echo "[5/5] Start SSH service..."
    /usr/sbin/sshd
    sleep 1
    echo "--> Selesai."
    echo ""
    echo "=== Verifikasi SSH Server ==="
    ps aux | grep "[s]shd"
    netstat -tlnp | grep :22
    echo ""
    echo "SSH Server siap di 192.218.2.1:22"
    echo "Username: root | Password: password123"


<img width="1920" height="1080" alt="Screenshot (177)" src="https://github.com/user-attachments/assets/4be4dd6f-0824-45bf-8517-cdb5d3a9346f" />


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

### Soal 17

```
nc 10.15.43.32 3404
```

### Sub-soal 1

What is the name of the first suspicious file? Format: file.exe

1. What is the name of the first suspicious file? Format: file.exe

Jawaban

```
Invoice&MSO-Request.doc
```

### Sub-soal 2

What is the name of the second suspicious file? Format: file.exe

Jawaban

```
knr.exe
```

### Sub-soal 3

What is the hash of the second suspicious file (knr.exe)? Format: sha256

1. File > Export Objects > HTTP > Save
2. ``` sha256 knr.exe ```

Jawaban

```
749e161661290e8a2d190b1a66469744127bc25bf46e5d0c6f2e835f4b92db18
```

Flag

```
KOMJAR25{M4ster_4n4lyzer_1xcg3WUUjKpux80gm8tEdFgL0}

```

## Soal 18: Deteksi File Malware via SMB Protocol
Tujuan
Mengidentifikasi file berbahaya yang ditransfer melalui protokol SMB dengan metode yang berbeda dari soal sebelumnya (HTTP).

Langkah-langkah Pengerjaan
1. Download File Capture

bashnc 10.15.43.32 3405 > soal18.pcap
3. Buka dengan Wireshark

wireshark soal18.pcap

4. Export SMB Objects
PENTING: Soal ini menggunakan protokol SMB, bukan HTTP!

Menu: File → Export Objects → SMB/SMB2

Window "SMB object list" akan terbuka menampilkan file yang ditransfer
5. lalu filter dengan .exe

6. Identifikasi File Malware
   
Dari SMB object list, terlihat 2 file .exe:
File 1 (Malware Pertama):

Size: 712 kB
Filename: d0p2nc6ka3f_fixhohlyc j4ovqfcy_smchzo_ub83urjpphrwahjwhv_o5c0fvf6.exe

File 2 (Malware Kedua):

Size: 115 kB
Filename: oiku9bu68cxqenfmcsos2aek6t07_guuisgxhllixv8dx2eemqddnhyhH6l8n_di.exe

8. Jawab Challenge
bashnc 10.15.43.32 3405
Pertanyaan 1: How many files are suspected of containing malware?
> 2
Pertanyaan 2: What is the name of the first malicious file?

> d0p2nc6ka3f_fixhohlyc j4ovqfcy_smchzo_ub83urjpphrwahjwhv_o5c0fvf6.exe
Pertanyaan 3: What is the hash of the first malicious file?

> 59896ae5f3edcb999243c7bfdc0b17eb7fe28f3a66259d797386ea470c010040
Pertanyaan 4: What is the hash of the second malicious file?

> cf99990bee6c378cbf56239b3cc88276eec348d82740f84e9d5c343751f82560

Flag: KOMJAR25{Y0u_4re_g0d1lke_YXy69mpWg42RMOralWIzMD9fZS}

<img width="1920" height="1080" alt="Screenshot (181)" src="https://github.com/user-attachments/assets/0cd842ab-a608-4678-ae07-942a8b3cd8b3" />

<img width="1223" height="444" alt="Screenshot 2025-10-05 021911" src="https://github.com/user-attachments/assets/4776d15c-2548-47f5-a5b6-74d40535f553" />

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

### Sub-soal 3

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

### Soal 20

```
nc 10.15.43.32 3407
```

### Sub-soal 1

What encryption method is used? Format: string

1. gunakan filter ```tls.handshake.type == 2```

Jawaban

```
TLS
```

### Sub-soal 2

What is the name of the malicious file placed by the attacker? Format: file.exe

1. gunakan filter

```
http.response.code == 200
```

Answer

```
invest_20.dll
```

### Sub-soal 3

What is the hash of the file containing the malware? Format: sha256

```
sha256sum invest_20.ddl
```

Answer

```
31cf42b2a7c5c558f44cfc67684cc344c17d4946d3a1e0b2cecb8eb58173cb2f
```

Flag

```
KOMJAR25{B3ware_0f_M4lw4re_Q1lBjLY0oJ3rCllqfcUS8GO4a}
```

## Soal 20: Deteksi Malware Terenkripsi dengan TLS Decryption
Tujuan
Mengidentifikasi file berbahaya yang disembunyikan dengan enkripsi TLS menggunakan bantuan keylog file dari Manwë untuk mendekripsi traffic.

Langkah-langkah Pengerjaan
1. Download File Capture
bashnc 10.15.43.32 3407 > soal20.pcap
2. Download Keylog File (Bantuan Manwë)
Manwë memberikan bantuan berupa TLS keylog file untuk mendekripsi traffic. File ini bernama:

keylogfile.txt 

Download atau extract keylog file dari challenge.
3. Load Keylog File ke Wireshark
Cara 1: Via Preferences

Buka Wireshark
Edit → Preferences
Expand Protocols → TLS (atau SSL di Wireshark versi lama)
Di field (Pre)-Master-Secret log filename, browse ke file keylog
Klik OK

4. Buka File Capture
bashwireshark soal20.pcap
Setelah keylog di-load, traffic TLS akan otomatis ter-decrypt dan muncul sebagai HTTP biasa.
5. Export HTTP Objects

Menu: File → Export Objects → HTTP
Sekarang file yang sebelumnya terenkripsi akan terlihat

File yang ditemukan:

Filename: invest_20.dll
Content Type: application/octet-stream

6. Identifikasi Metode Enkripsi
Dari packet TLS sebelum didekripsi:

Klik packet TLS Handshake
Expand Transport Layer Security → Handshake Protocol
Lihat cipher suite yang digunakan

Metode enkripsi: TLS
7. Hash File Malware
Export file invest_20.dll lalu hitung hash:
bash# Linux
sha256sum invest_20.dll

Windows PowerShell
Get-FileHash invest_20.dll -Algorithm SHA256
Hash: 31cf42b2a7c5c558f44cfc67684cc344c17d49d3a1e0b2cecb8eb58173cb2f
8. Jawab Challenge
bashnc 10.15.43.32 3407
> TLS
> invest_20.dll
> 31cf42b2a7c5c558f44cfc67684cc344c17d49d3a1e0b2cecb8eb58173cb2f
Flag: KOMJAR25{B3ware_0f_M4lw4re_VDVGw0SO41hokPcl4mIqJGUef}

<img width="1059" height="432" alt="Screenshot 2025-10-05 083258" src="https://github.com/user-attachments/assets/75a425e2-6f67-4d05-824c-4fd087cf529a" />
<img width="1920" height="1080" alt="Screenshot (183)" src="https://github.com/user-attachments/assets/cca40e30-1770-4638-a81f-7f64d8801892" />



