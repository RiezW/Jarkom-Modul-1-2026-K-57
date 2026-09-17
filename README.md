# Jarkom-Modul-1-2026-K-57

| | |
|---|---|
| **Nama** | : Riezco Eka Bayu Witantra & Sulthan Daffa Al Hasyimi|
| **NRP** | : 5027251057 & 5027251091|
| **Kelompok** | : K-57 |
| **Mata Kuliah** | : JarKom |

---

## I. Pembahasan per Nomor Soal

### Nomor 1 — Membangun Topologi The Wired & Konfigurasi IP Dasar

**Permintaan Soal**
Lain (Router) membuat 3 switch/gateway: Switch 1 menghubungkan Alice dan Mika, Switch 2 menghubungkan Chisa, Switch 3 menghubungkan Knights dan Eiri. Kelima entitas dikonfigurasi sebagai client di GNS3 dengan prefix IP kelompok (10.92.x.x).

**Konfigurasi yang Digunakan**
```bash
# Router Lain — /root/script.sh
#!/bin/bash
cat << 'NET' > /etc/network/interfaces
auto eth1
iface eth1 inet static
address 10.92.1.1/24
auto eth2
iface eth2 inet static
address 10.92.2.1/24
auto eth3
iface eth3 inet static
address 10.92.3.1/24
NET
ip addr add 10.92.1.1/24 dev eth1 2>/dev/null
ip addr add 10.92.2.1/24 dev eth2 2>/dev/null
ip addr add 10.92.3.1/24 dev eth3 2>/dev/null
ip link set eth1 up
ip link set eth2 up
ip link set eth3 up

# Client (contoh Alice, 10.92.1.2) — /root/script.sh
echo "auto eth0" > /etc/network/interfaces
echo "iface eth0 inet static" >> /etc/network/interfaces
echo " address 10.92.1.2" >> /etc/network/interfaces
echo " netmask 255.255.255.0" >> /etc/network/interfaces
ip addr flush dev eth0
ip addr add 10.92.1.2/24 dev eth0 2>/dev/null
ip link set eth0 up
```

**Langkah Menjalankan**
1. Buka GNS3, tambahkan Docker node menggunakan image `ardhptr21/alpinet:latest` atau `ardhptr21/debinet:latest` untuk Router Lain dan kelima client (Alice, Mika, Chisa, Knights, Eiri).
2. Atur jumlah adapter Router Lain menjadi 4 (eth0 untuk internet, eth1–eth3 untuk tiap switch) lewat Configure > Network.
3. Susun topologi: Router–Switch1–{Alice, Mika}, Router–Switch2–{Chisa}, Router–Switch3–{Knights, Eiri}.
4. Buka console Router Lain, buat file `/root/script.sh` berisi konfigurasi IP static eth1–eth3, lalu jalankan dengan `bash /root/script.sh`.
5. Buka console tiap client, buat `/root/script.sh` dengan IP sesuai subnet gateway-nya (Alice/Mika di 10.92.1.x, Chisa di 10.92.2.x, Knights/Eiri di 10.92.3.x), lalu jalankan.
6. Verifikasi dengan perintah `ip a` di router (memastikan eth1–eth3 muncul) dan di tiap client (memastikan eth0 muncul).

**Bukti / Dokumentasi**
> 📷 ![Screenshot](Assets/Screenshot%202026-09-16%20215059.png) Screenshot topologi lengkap The Wired di canvas GNS3 (Router, 3 switch, 5 client beserta koneksinya).
>
> 📷 ![Screenshot](Assets/Screenshot%202026-09-16%20223935.png) Screenshot hasil `ip a` di Router Lain yang menampilkan IP eth1, eth2, eth3.
>
> 📷 ![Screenshot](Assets/Screenshot%202026-09-16%20225654.png) Screenshot hasil `ip a` di salah satu client (mis. Alice) yang menampilkan IP eth0.

---

### Nomor 2 — Router Terhubung ke Internet Publik (DHCP & NAT)

**Permintaan Soal**
The Wired masih terisolasi dari dunia luar, sehingga Router Lain perlu dikonfigurasi agar tersambung langsung ke jaringan internet publik melalui NAT/DHCP pada interface eth0.

**Konfigurasi yang Digunakan**
```bash
# Router Lain — /root/script.sh
auto eth0
iface eth0 inet dhcp

iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

**Langkah Menjalankan**
1. Tambahkan blok `auto eth0` dan `iface eth0 inet dhcp` ke `/etc/network/interfaces` (lewat script.sh) agar eth0 Router Lain mendapat IP otomatis dari NAT GNS3/host.
2. Tambahkan aturan NAT Masquerade: `iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE` agar semua paket keluar lewat eth0 disamarkan menjadi IP router.
3. Jalankan ulang `bash /root/script.sh`, lalu cek IP eth0 dengan `ip a` — pastikan Router Lain mendapat IP DHCP (mis. 192.168.122.x).
4. Cek aturan NAT sudah aktif dengan `iptables -t nat -L -v -n` dan pastikan chain POSTROUTING berisi rule MASQUERADE.

**Bukti / Dokumentasi**
> 📷 ![Screenshot](Assets/Screenshot%202026-09-16%20223337.png) Screenshot `ip a` pada Router Lain yang menunjukkan eth0 sudah mendapat IP DHCP publik.
>
> 📷 ![Screenshot](Assets/Screenshot%202026-09-17%20002937.png) Screenshot output `iptables -t nat -L -v -n` yang menampilkan rule MASQUERADE pada eth0.
>
> 📷 ![Screenshot](Assets/Screenshot%202026-09-16%20232309.png) Screenshot konfigurasi Router Lain yang memuat DHCP pada eth0 dan rule NAT MASQUERADE.

---

### Nomor 3 — Routing Antar-Client (Inter-Subnet Communication)

**Permintaan Soal**
Setelah Router Lain terhubung ke internet, seluruh entitas (client) di bawah Switch 1, Switch 2, dan Switch 3 harus dapat saling terhubung dan berkomunikasi satu sama lain melalui konfigurasi routing.

**Konfigurasi yang Digunakan**
```bash
# Router Lain — /root/script.sh
sysctl -w net.ipv4.ip_forward=1
```

**Langkah Menjalankan**
1. Aktifkan IP forwarding di kernel Router Lain dengan `sysctl -w net.ipv4.ip_forward=1` agar router meneruskan paket antar interface eth1, eth2, eth3.
2. Pastikan tiap client sudah punya default gateway menuju Router Lain (baris `gateway 10.92.x.1` pada `/etc/network/interfaces`, ditambah `ip route add default via 10.92.x.1`).
3. Uji konektivitas antar subnet, misalnya dari Alice (Switch 1) ping ke Chisa (Switch 2) dan ke Knights (Switch 3).
4. Cek nilai `net.ipv4.ip_forward` sudah = 1 dengan `cat /proc/sys/net/ipv4/ip_forward` di Router Lain.

**Bukti / Dokumentasi**
> 📷 ![Screenshot](Assets/Screenshot%202026-09-16%20223935.png) Screenshot output `cat /proc/sys/net/ipv4/ip_forward` bernilai 1 di Router Lain.
>
> 📷 ![Screenshot](Assets/Screenshot%202026-09-16%20232857.png) Screenshot hasil ping antar client beda subnet (mis. Alice ke Chisa dan Alice ke Knights) yang berhasil reply.

---

### Nomor 4 — Kemandirian Akses Internet Client (NAT + DNS Resolver)

**Permintaan Soal**
Setiap entitas (client) di The Wired harus memiliki kemandirian akses internet: mampu melakukan ping ke 8.8.8.8 dan membuka domain web google.com, melalui konfigurasi firewall/iptables (NAT Masquerade) dan DNS resolver.

**Konfigurasi yang Digunakan**
```bash
# Client (semua node) — /root/script.sh
echo "nameserver 8.8.8.8" > /etc/resolv.conf

# Memanfaatkan konfigurasi Router Lain yang sudah ada:
#  sysctl -w net.ipv4.ip_forward=1        (dari Nomor 3)
#  iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE   (dari Nomor 2)
```

**Langkah Menjalankan**
1. Pastikan konfigurasi ip_forward (Nomor 3) dan NAT Masquerade (Nomor 2) di Router Lain sudah aktif — kedua ini menjadi syarat agar paket client bisa keluar ke internet.
2. Di setiap client, tambahkan baris `echo "nameserver 8.8.8.8" > /etc/resolv.conf` ke dalam script.sh agar client dapat menerjemahkan nama domain.
3. Jalankan ulang script.sh pada tiap client, lalu uji: `ping -c 2 8.8.8.8` (uji NAT) dan `ping -c 2 google.com` (uji DNS + NAT sekaligus).

**Bukti / Dokumentasi**
> 📷 ![Screenshot](Assets/Screenshot%202026-09-16%20233934.png) Screenshot hasil `ping -c 2 8.8.8.8` dari salah satu client yang berhasil reply.
>
> 📷 ![Screenshot](Assets/Screenshot%202026-09-16%20233954.png) Screenshot hasil `ping -c 2 google.com` dari client yang menunjukkan domain berhasil di-resolve dan reply.

---

### Nomor 5 — Persistensi Konfigurasi & Script Verifikasi Status

**Permintaan Soal**
Untuk mengantisipasi restart tiba-tiba (ulah Eiri), seluruh konfigurasi jaringan tidak boleh hilang saat semua node di-restart. Dibuat script verifikasi `/root/cek_status.sh` pada Router Lain yang menampilkan ringkasan interface (`ip -br a`) dan status tabel NAT (`iptables -t nat -L -v -n`).

**Konfigurasi yang Digunakan**
```bash
# Router Lain — /root/cek_status.sh
#!/bin/bash
ip -br a
echo "----------------------------------------"
iptables -t nat -L -v -n

# (opsional) supaya script.sh berjalan otomatis saat boot:
echo "sh /root/script.sh" > /etc/local.d/script.start
chmod +x /etc/local.d/script.start
rc-update add local default
```

**Langkah Menjalankan**
1. Buat file `/root/cek_status.sh` di Router Lain berisi perintah `ip -br a` dan `iptables -t nat -L -v -n`, lalu beri izin eksekusi: `chmod +x /root/cek_status.sh`.
2. Konfigurasi utama (IP interfaces & NAT) tetap disimpan di `/root/script.sh` dan `/etc/network/interfaces` agar bisa dipanggil ulang kapan saja setelah reboot.
3. Simulasikan kondisi restart tanpa reboot node: flush semua IP dan rule iptables (`ip addr flush`, `iptables -F`, `iptables -t nat -F`), lalu jalankan ulang `sh /root/script.sh`.
4. Jalankan `sh /root/cek_status.sh` dan pastikan interface serta rule NAT kembali tampil normal seperti sebelum di-flush.
5. (Opsional) Daftarkan script.sh ke `/etc/local.d/script.start` agar konfigurasi otomatis diterapkan setiap kali node benar-benar di-restart.

**Bukti / Dokumentasi**
> 📷 ![Screenshot](Assets/Screenshot%202026-09-17%20001722.png) Screenshot isi file `/root/cek_status.sh` (`cat /root/cek_status.sh`).
>
> 📷 ![Screenshot](Assets/Screenshot%202026-09-17%20001722.png) Screenshot hasil eksekusi `sh /root/cek_status.sh` setelah simulasi flush/restart, menampilkan ringkasan interface dan tabel NAT.

---

### Nomor 6 — Analisis Traffic Mencurigakan di Node Mika (Wireshark)

**Permintaan Soal**
Mika mencurigai adanya anomali traffic pada segmen jaringannya. Generator traffic dijalankan pada node Mika, kemudian dilakukan packet sniffing dengan Wireshark pada interface node Mika, menerapkan display filter khusus untuk paket berprotokol DNS atau ICMP.

**Konfigurasi yang Digunakan**
```bash
# Node Mika
curl -L "https://docs.google.com/uc?export=download&id=<FILE_ID_GENERATOR>" -o traffic_protocol17.sh
cat traffic_protocol17.sh          # pastikan isinya script, bukan HTML
chmod +x traffic_protocol17.sh
./traffic_protocol17.sh

# Wireshark display filter:
dns || icmp
```

**Langkah Menjalankan**
1. Unduh file `traffic_protocol17.sh` di terminal node Mika menggunakan curl/wget dari link file yang diberikan pada soal.
2. Periksa isi file (`cat traffic_protocol17.sh`) untuk memastikan file benar berupa script, lalu beri izin eksekusi (`chmod +x`) dan jalankan (`./traffic_protocol17.sh`).
3. Di GNS3, klik kanan pada kabel yang terhubung ke node Mika, pilih **Start capture** untuk membuka Wireshark pada interface tersebut.
4. Pada kolom filter Wireshark, masukkan `dns || icmp` lalu tekan Enter untuk menyaring hanya paket DNS dan ICMP.
5. Amati paket yang lolos: query DNS (A/AAAA) hasil nslookup/dig, dan paket ICMP Echo Request/Reply hasil ping dari `traffic_protocol17.sh`.

**Bukti / Dokumentasi**
> 📷 ![Screenshot](Assets/Screenshot%202026-09-17%20003054.png) Screenshot terminal Mika saat traffic_protocol17.sh dijalankan.
>
> 📷 ![Screenshot](Assets/Screenshot%202026-09-17%20003854.png) Screenshot Wireshark dengan filter `dns || icmp` aktif beserta daftar paket DNS dan ICMP yang lolos, lengkap dengan ringkasan (summary) tiap paket.

---

### Nomor 7 — FTP Server di Node Chisa dengan Kebijakan Akses per-User

**Permintaan Soal**
Chisa mendirikan FTP Server dengan shared folder di `/var/wired/data`. Kebijakan akses: user alice (read & write), user mika (read-only), user eiri (tanpa izin akses / blacklist). Dibuktikan dengan pembuatan file `signal_alice.txt` dari user alice, dan penolakan akses saat user eiri login.

**Konfigurasi yang Digunakan**
```bash
# Node Chisa
apk add vsftpd
mkdir -p /var/wired/data
echo -e "123\n123" | adduser -h /var/wired/data alice
echo -e "123\n123" | adduser -h /var/wired/data mika
echo -e "123\n123" | adduser -h /var/wired/data eiri

# Batasi mika hanya read-only pada folder shared
setfacl -m u:mika:r-x /var/wired/data 2>/dev/null || chmod 555 /var/wired/data

# Blacklist eiri (userlist_deny)
echo "eiri" > /etc/vsftpd/user_list

# /etc/vsftpd/vsftpd.conf
anonymous_enable=NO
local_enable=YES
write_enable=YES
chroot_local_user=YES
allow_writeable_chroot=YES
userlist_enable=YES
userlist_file=/etc/vsftpd/user_list
userlist_deny=YES

vsftpd /etc/vsftpd/vsftpd.conf &
```

**Langkah Menjalankan**
1. Install vsftpd dan buat folder shared `/var/wired/data` di node Chisa.
2. Buat tiga user FTP (alice, mika, eiri) dengan home directory `/var/wired/data` menggunakan `adduser -h`.
3. Batasi folder agar mika hanya bisa read-only (setfacl/chmod), sementara alice tetap memiliki akses baca-tulis penuh.
4. Masukkan eiri ke `/etc/vsftpd/user_list` dan aktifkan `userlist_deny=YES` pada vsftpd.conf sehingga eiri diblokir login sama sekali.
5. Jalankan vsftpd dengan `vsftpd /etc/vsftpd/vsftpd.conf &`.
6. Uji dari client lain: login sebagai alice lalu buat file `signal_alice.txt` (`lftp -u alice,123 <IP_Chisa> -e "put -o signal_alice.txt /dev/null; bye"`) untuk membuktikan hak read/write.
7. Uji login sebagai eiri dan buktikan koneksi ditolak oleh server.

**Bukti / Dokumentasi**
> 📷 ![Screenshot](Assets/Screenshot%202026-09-17%20021710.png) Screenshot isi `/etc/vsftpd/vsftpd.conf` dan `/etc/vsftpd/user_list` di node Chisa.
>
> 📷 ![Screenshot](Assets/Screenshot%202026-09-17%20022841.png) Screenshot sesi FTP user alice berhasil membuat/mengunggah `signal_alice.txt`.
>
> 📷 ![Screenshot](Assets/Screenshot%202026-09-17%20023017.png) Screenshot percobaan login FTP user eiri yang ditolak oleh server.

---

### Nomor 8 — Upload Dokumen Rahasia Knights ke FTP Server Chisa

**Permintaan Soal**
Kelompok rahasia Knights mengirimkan dokumen laporan intelijen ke FTP Server Chisa menggunakan akun alice dari node Knights. Sesi dianalisis dengan Wireshark: perintah FTP untuk upload (STOR), kode status sukses server (226), dan port data TCP yang dinegosiasikan pada mode PASV.

**Konfigurasi yang Digunakan**
```bash
# Node Knights
lftp -u alice,123 10.92.2.2
# di dalam sesi lftp:
put laporan
bye
```

**Langkah Menjalankan**
1. Dari node Knights, lakukan koneksi FTP client ke IP Chisa (10.92.2.2) menggunakan akun alice.
2. Upload file laporan yang diberikan pada soal menggunakan perintah put/STOR.
3. Aktifkan capture Wireshark pada interface Knights sebelum/selama proses upload berlangsung.
4. Telusuri sesi FTP di Wireshark (Follow TCP Stream) untuk menemukan perintah `STOR` yang dikirim client, kode balasan `226 Transfer complete` dari server, serta respons `227 Entering Passive Mode (h1,h2,h3,h4,p1,p2)` untuk mode PASV.
5. Hitung port data TCP dari p1 dan p2 dengan rumus `(p1 × 256) + p2`.

**Jawaban**
Pada sesi FTP, client mengirim perintah `STOR laporan` untuk mengunggah file. Server mengembalikan kode `226 Transfer complete` yang menandakan transfer berhasil. Server juga mengirim respons `227 Entering Passive Mode (10,92,2,2,156,121)`, sehingga `p1 = 156` dan `p2 = 121`.

Perhitungan port data TCP:
`(156 × 256) + 121 = 39936 + 121 = 40057`

Jadi, port data TCP pada mode PASV adalah **40057**.

**Bukti / Dokumentasi**
> 📷 ![Screenshot](Assets/Screenshot%202026-09-17%20023815.png) Screenshot proses upload file dari terminal Knights (perintah put/STOR hingga selesai).

---

### Nomor 9 — Akses Read-Only Mika ke FTP Server Chisa

**Permintaan Soal**
Mika mengakses dokumen Protokol Tujuh dari FTP Server Chisa menggunakan akun mika dan mengunduhnya. Selanjutnya dibuktikan pembatasan read-only dengan mencoba mengunggah file baru dari akun mika, dan ditunjukkan pesan error respons server (550 Permission denied).

**Konfigurasi yang Digunakan**
```bash
# Node Mika
lftp -u mika,123 10.92.2.2
# di dalam sesi lftp:
get <nama_dokumen_protokol_tujuh>
put file_baru.txt   # diharapkan gagal (550 Permission denied)
bye
```

**Langkah Menjalankan**
1. Dari node Mika, login FTP ke Chisa (10.92.2.2) menggunakan akun mika.
2. Unduh dokumen Protokol Tujuh dari server menggunakan perintah get, dan pastikan berhasil.
3. Coba unggah file baru menggunakan perintah put dari akun mika untuk membuktikan pembatasan read-only.
4. Amati respons server yang menampilkan error `550 Permission denied` karena mika hanya memiliki hak baca.

**Bukti / Dokumentasi**
> 📷 ![Screenshot](Assets/Screenshot%202026-09-17%20024719.png) Screenshot proses download dokumen Protokol Tujuh berhasil oleh akun mika.
>
> 📷 ![Screenshot](Assets/Screenshot%202026-09-17%20025346.png) Screenshot percobaan upload akun mika yang gagal dengan pesan error 550 Permission denied.

---

### Nomor 10 — Uji Ketahanan Koneksi (Ping Statistik) Knights ke Chisa

**Permintaan Soal**
Knights menguji latensi jaringan The Wired dengan mengirim paket ping ke node Chisa: payload khusus 128 bytes, interval 0.3 detik, sebanyak 77 paket. Dianalisis nilai ICMP Type dan Code untuk Echo Request vs Echo Reply, packet loss, dan RTT (min/avg/max).

**Konfigurasi yang Digunakan**
```bash
# Node Knights
ping -c 77 -s 128 -i 0.3 10.92.2.2
```

**Langkah Menjalankan**
1. Aktifkan capture Wireshark pada interface Knights sebelum menjalankan ping.
2. Jalankan perintah `ping -c 77 -s 128 -i 0.3 10.92.2.2` dari terminal Knights menuju Chisa.
3. Di Wireshark, periksa paket ICMP Echo Request (Type 8, Code 0) dari Knights dan Echo Reply (Type 0, Code 0) dari Chisa.
4. Baca ringkasan statistik ping di terminal: jumlah paket terkirim/diterima, persentase packet loss, serta RTT min/avg/max/mdev.

**Bukti / Dokumentasi**
> 📷 ![Screenshot](Assets/Screenshot%202026-09-17%20025855.png) Screenshot output ringkasan statistik ping di terminal Knights (77 packets transmitted/received, 0% loss, RTT min/avg/max/mdev).
>
> 📷 ![Screenshot](Assets/Screenshot%202026-09-17%20030031.png) Screenshot Wireshark yang menampilkan detail paket ICMP Echo Request dan Echo Reply beserta nilai Type dan Code-nya.

---

### Nomor 11 — Membuktikan Kelemahan Protokol Telnet

**Permintaan Soal**
Dibuktikan kelemahan protokol Telnet dengan membuat akun `phantom_user` (password `wired_ghost`) pada layanan telnetd di node Chisa. Login Telnet dilakukan dari node Eiri ke node Chisa, sesi ditangkap dengan Wireshark, kredensial plain text ditunjukkan melalui fitur Follow TCP Stream, serta dijelaskan mengapa tiap karakter terkirim dalam paket TCP terpisah.

**Konfigurasi yang Digunakan**
```bash
# Node Chisa
adduser phantom_user   # password: wired_ghost
# aktifkan layanan telnetd (mis. apk add busybox-extras / in.telnetd)

# Node Eiri
telnet 10.92.2.2
# login: phantom_user / wired_ghost
```

**Langkah Menjalankan**
1. Buat user `phantom_user` dengan password `wired_ghost` di node Chisa, lalu jalankan layanan telnetd.
2. Aktifkan capture Wireshark pada interface Chisa (atau Eiri) sebelum sesi Telnet dimulai.
3. Dari node Eiri, jalankan `telnet 10.92.2.2` dan login menggunakan kredensial phantom_user / wired_ghost.
4. Di Wireshark, klik kanan salah satu paket TCP sesi tersebut lalu pilih **Follow > TCP Stream** untuk melihat seluruh transkrip sesi.
5. Amati bahwa username dan password terlihat jelas dalam bentuk plain text (tidak terenkripsi).
6. Jelaskan bahwa Telnet beroperasi dalam mode character-at-a-time (NVT), sehingga tiap karakter yang diketik dikirim sebagai satu paket TCP terpisah, termasuk saat mengetik password meski tidak di-echo ke layar.

**Bukti / Dokumentasi**
> 📷 ![Screenshot](Assets/Screenshot%202026-09-17%20031509.png) Screenshot daftar paket Wireshark hasil capture sesi Telnet dari Eiri ke Chisa (tcp.stream sesi login).
>
> 📷 ![Screenshot](Assets/Screenshot%202026-09-17%20032542.png) Screenshot hasil Follow TCP Stream yang menampilkan kredensial phantom_user / wired_ghost dalam bentuk plain text.

---

### Nomor 12 — Pemindaian Port Rahasia di Node Knights

**Permintaan Soal**
Alice mencurigai Knights menjalankan layanan rahasia. Dilakukan pemindaian port dari node Alice ke node Knights menggunakan Netcat (nc) untuk memeriksa port 22 (SSH) dan 80 (HTTP) dalam keadaan terbuka, serta port rahasia 7777 dalam keadaan tertutup. Dianalisis perbedaan TCP Flag antara port terbuka (SYN-ACK) dengan port tertutup (RST-ACK).

**Konfigurasi yang Digunakan**
```bash
# Node Knights (menyalakan layanan agar port 22 & 80 terbuka)
apk add openssh apache2
rc-service sshd start
rc-service apache2 start

# Node Alice (pemindaian port)
nc -zv 10.92.3.2 22
nc -zv 10.92.3.2 80
nc -zv 10.92.3.2 7777
```

**Langkah Menjalankan**
1. Di node Knights, install dan jalankan layanan OpenSSH (port 22) dan Apache (port 80) agar kedua port berstatus terbuka; port 7777 dibiarkan tertutup (tidak ada layanan).
2. Aktifkan capture Wireshark pada interface Alice sebelum melakukan pemindaian.
3. Dari node Alice, jalankan `nc -zv 10.92.3.2 22`, `nc -zv 10.92.3.2 80`, dan `nc -zv 10.92.3.2 7777` secara berurutan.
4. Di Wireshark, bandingkan respons TCP: port 22 dan 80 akan dibalas SYN-ACK, sedangkan port 7777 akan dibalas RST-ACK.
5. Simpulkan bahwa SYN-ACK menandakan ada layanan yang menerima koneksi, sedangkan RST-ACK menandakan tidak ada layanan pada port tersebut.

**Bukti / Dokumentasi**
> 📷 ![Screenshot](Assets/Screenshot%202026-09-17%20033033.png) Screenshot output terminal Alice untuk ketiga percobaan `nc -zv` (port 22, 80, dan 7777).
>
> 📷 ![Screenshot](Assets/Screenshot%202026-09-17%20033122.png) Screenshot Wireshark yang menampilkan perbandingan TCP flag SYN-ACK (port terbuka) dan RST-ACK (port tertutup).

---

### Nomor 13 — SSH Key-Based Authentication (Passwordless) Mika ke Knights

**Permintaan Soal**
Lain memerintahkan administrasi jarak jauh menggunakan SSH secara aman tanpa password. OpenSSH server diinstall pada node Knights, pasangan kunci SSH dibuat pada node Mika untuk user mika_admin, dan dikonfigurasi public key authentication (PasswordAuthentication no). Koneksi SSH dilakukan dari Mika ke Knights, sesi ditangkap Wireshark, diidentifikasi paket Protocol Version Exchange dan Key Exchange, serta dijelaskan mengapa kredensial tidak terlihat dalam bentuk teks terbuka seperti pada Telnet.

**Konfigurasi yang Digunakan**
```bash
# Node Knights (server)
apk add openssh
rc-service sshd start
mkdir -p /home/mika_admin/.ssh && chmod 700 /home/mika_admin/.ssh
# tempel public key Mika ke:
# /home/mika_admin/.ssh/authorized_keys
chmod 600 /home/mika_admin/.ssh/authorized_keys
sed -i 's/^#\?PasswordAuthentication.*/PasswordAuthentication no/' /etc/ssh/sshd_config
pkill sshd; /usr/sbin/sshd

# Node Mika (client)
adduser mika_admin
su - mika_admin
ssh-keygen -t ed25519
cat ~/.ssh/id_ed25519.pub   # disalin ke authorized_keys Knights
ssh mika_admin@10.92.3.2
```

**Langkah Menjalankan**
1. Install dan jalankan OpenSSH server pada node Knights.
2. Buat user `mika_admin` di node Mika, lalu buat pasangan kunci SSH dengan `ssh-keygen` (mis. tipe ed25519).
3. Salin isi public key (id_ed25519.pub) ke file `~/.ssh/authorized_keys` milik mika_admin pada node Knights, dengan permission direktori 700 dan file 600.
4. Nonaktifkan login password di Knights dengan mengatur `PasswordAuthentication no` pada `/etc/ssh/sshd_config`, lalu restart layanan sshd.
5. Aktifkan capture Wireshark pada interface sebelum menjalankan koneksi SSH.
6. Dari node Mika, jalankan `ssh mika_admin@10.92.3.2` dan pastikan berhasil login tanpa diminta password.
7. Di Wireshark, identifikasi paket **Protocol Version Exchange** (pertukaran versi SSH awal) dan **Key Exchange Init**, lalu amati bahwa paket-paket setelahnya tampil sebagai **Encrypted Packet**.
8. Jelaskan bahwa seluruh isi komunikasi setelah key exchange dienkripsi, sehingga kredensial/perintah tidak dapat dibaca sebagai plaintext seperti pada sesi Telnet di Nomor 11.

**Bukti / Dokumentasi**
> 📷 ![Screenshot](Assets/Screenshot%202026-09-17%20034303.png) Screenshot hasil grep PasswordAuthentication di `/etc/ssh/sshd_config` Knights (menunjukkan nilai 'no').
>
> 📷 ![Screenshot](Assets/Screenshot%202026-09-17%20034622.png) Screenshot proses login `ssh mika_admin@10.92.3.2` dari Mika yang berhasil tanpa memasukkan password.
>
> 📷 ![Screenshot](Assets/Screenshot%202026-09-17%20035121.png) Screenshot Wireshark yang menampilkan paket Protocol Version Exchange, Key Exchange, dan paket-paket Encrypted setelahnya.

---


## Soal 14 - Analisis Brute Force pada Form Login (HTTP)

**Berkas:** `soal14_wired_bruteforce.pcapng` | **Port validasi:** 3401

### Yang diminta
IP penyerang, IP + port target, kata sandi user `lain_admin` yang berhasil ditembus, serta software + versi web server.

### Langkah analisis

**1. Menyaring percobaan login.** Submit form login adalah HTTP POST (kredensial berada di body permintaan), sehingga serangan brute force terlihat sebagai banjir permintaan POST. Filter:
```
http.request.method == "POST"
```
Terlihat puluhan POST identik ke `/login.php`. Sumbernya `172.26.7.50` (penyerang), tujuannya `172.26.7.100`. Pada panel detail TCP, **Destination Port = 8080**, jadi target adalah `172.26.7.100:8080`.

![Q14.1 - Filter HTTP POST](Assets/Q14.1%20-%20Filter%20HTTP%20POST%20(flood%20bruteforce).png)

**2. Menemukan login yang berhasil.** Keberhasilan pada HTTP ditandai kode status `200`. Filter:
```
http.response.code == 200
```
Muncul dua respons `200 OK`: satu ke `172.26.7.60` (login sah "Welcome Alice", ini **decoy**), dan satu ke `172.26.7.50` (penyerang, paket terakhir capture). Yang menuju IP penyerang adalah keberhasilan brute force.

![Q14.2 - Filter response 200](Assets/Q14.2%20-%20Filter%20response%20200%20(dua%20hit).png)

**3. Memastikan decoy.** Follow HTTP Stream pada respons ke `172.26.7.60` hanya berisi `GET /` dengan body "Welcome Alice" - bukan serangan. Dipakai sebagai pembanding agar tidak salah pilih.

![Q14.3 - Follow Stream decoy](Assets/Q14.3%20-%20Follow%20Stream%20decoy%20(Welcome%20Alice).png)

**4. Membaca kredensial yang tembus.** Follow HTTP Stream pada sesi penyerang (stream 59) menampilkan body `username=lain_admin&password=wired_pr0tocol_7`, respons `<h1>Success! Login successful.</h1>`, dan header `Server: Apache/2.4.62`. Header `Server` inilah software web server (sedangkan `X-Powered-By: PHP/8.3.14` adalah runtime aplikasi, bukan jawaban). User-Agent `Fuzz Faster U Fool` menandakan tool serangan `ffuf`.

![Q14.4 - Follow Stream 59](Assets/Q14.4%20-%20Follow%20Stream%2059%20(password%20%2B%20Server%20Apache).png)

### Temuan

| Item | Nilai |
|---|---|
| IP penyerang | `172.26.7.50` |
| IP + port target | `172.26.7.100:8080` |
| Password `lain_admin` | `wired_pr0tocol_7` |
| Web server | `Apache/2.4.62` |

### Validasi

![Q14.5 - Validasi nc 3401](Assets/Q14.5%20-%20Validasi%20nc%203401%20(flag).png)

Flag: `KOMJAR26{W1r3d_Brut3_jlUNLCw7Q8uIgOkU3ap80Xz53}`

---

## Soal 15 - Dekode Keystroke Keyboard USB (HID)

**Berkas:** `soal15_wired_usb_hid.pcap` | **Port validasi:** 3402

### Yang diminta
Vendor ID, Product ID, nomor device USB, dan pesan rahasia hasil ketikan.

### Langkah analisis

**1. Membaca Vendor/Product ID.** VID dan PID adalah field pada **DEVICE DESCRIPTOR** yang dikirim saat enumerasi. Filter:
```
usb.idVendor
```
Buka **USB URB -> DEVICE DESCRIPTOR**: `idVendor = 0x046d (Logitech, Inc.)`, `idProduct = 0xc31c (Keyboard K120)`.

![Q15.1 - Device Descriptor](Assets/Q15.1%20-%20USB%20Device%20Descriptor%20(VID%20PID).png)

**2. Menemukan nomor device dan paket keystroke.** Pada paket deskriptor, `Device address` masih `0` (alamat belum diberikan). Nomor asli dibaca dari paket keystroke. Filter:
```
usb.capdata
```
Muncul deretan paket `URB_INTERRUPT in` dari keyboard ke `host`, semuanya `usb.device_address = 7`. Setiap paket berisi laporan HID 8 byte pada field **Leftover Capture Data**.

![Q15.2 - Filter usb.capdata](Assets/Q15.2%20-%20Filter%20usb.capdata%20(device%207%20keystroke).png)

**3. Dekode HID.** Struktur laporan: byte 0 = modifier (Shift = `0x02`), byte 1 = reserved, **byte 2 = keycode**. Laporan bernilai nol seluruhnya adalah pelepasan tombol (dilewati). Keycode dipetakan lewat HID Usage Table (`0x04`-`0x1d` = a-z, `0x1e`-`0x27` = 1..0, `0x2c` = spasi, `0x2d` = `-`/`_` bila Shift). Hasil urutan: `W i r e d _ P r o t o c o l _ 7 _ i s _ a l i v e _ 2 0 2 6` -> **`Wired_Protocol_7_is_alive_2026`** (huruf kapital dan `_` berasal dari modifier Shift).

### Temuan

| Item | Nilai |
|---|---|
| Vendor ID | `0x046d` (Logitech, Inc.) |
| Product ID | `0xc31c` (Keyboard K120) |
| Nomor device | `7` |
| Pesan rahasia | `Wired_Protocol_7_is_alive_2026` |

### Validasi

![Q15.3 - Validasi nc 3402](Assets/Q15.3%20-%20Validasi%20nc%203402%20(flag).png)

Flag: `KOMJAR26{USB_K3ystr0k3_wa37QzOwnVRSO5Bz39SBgsJu8}`

---

## Soal 16 - Pencurian Malware via FTP

**Berkas:** `soal16_wired_ftp_theft.pcapng` | **Port validasi:** 3403

### Yang diminta
IP server FTP penyerang, banner software FTP, kredensial login penyerang, dan ukuran (byte) berkas `knights_payload.exe`.

### Langkah analisis

**1. Membuka kanal kontrol FTP.** FTP kontrol adalah plaintext (port 21). Filter:
```
ftp
```
Terdapat **dua sesi** (ini jebakannya): `10.7.3.30` ("220 wired-drop FTP server", login `guest`, `RETR laporan.pdf`) adalah **decoy**; `198.51.100.7` ("220 Welcome to Wired FTP Server (vsftpd 3.0.5)", login `knights_agent`, mode biner) adalah server malware. Banner `vsftpd 3.0.5` dan kredensial `knights_agent` / `N4v1_s3cur3_2026` dibaca dari respons `220` dan perintah USER/PASS.

![Q16.1 - Filter ftp](Assets/Q16.1%20-%20Filter%20ftp%20(banner%20vsftpd%203.0.5).png)

**2. Memilih sesi malware dan membaca ukuran.** Dengan berpatokan pada nama aset `knights_payload.exe`, terlihat `RETR knights_payload.exe` berada pada sesi `198.51.100.7` (IP publik, konsisten sebagai penyerang). Ukuran dikonfirmasi dua kali: respons `213 524288` (jawaban `SIZE`) dan `150 Opening BINARY mode data connection for knights_payload.exe (524288 bytes)`. Mode pasif dinegosiasikan pada `227 Entering Passive Mode (198,51,100,7,156,64)` -> port data `156*256+64 = 40000`.

![Q16.2 - SIZE dan RETR](Assets/Q16.2%20-%20SIZE%20dan%20RETR%20knights_payload%20(524288).png)

### Temuan

| Item | Nilai |
|---|---|
| IP server FTP penyerang | `198.51.100.7` |
| Banner software FTP | `vsftpd 3.0.5` |
| Kredensial | `knights_agent` / `N4v1_s3cur3_2026` |
| Ukuran `knights_payload.exe` | `524288` byte (512 KB) |

### Validasi

![Q16.3 - Validasi nc 3403](Assets/Q16.3%20-%20Validasi%20nc%203403%20(flag).png)

Flag: `KOMJAR26{FTP_Th3ft_NcMqmDIgQwWba0tCS9ETbN0Aw}`

---

## Soal 17 - Unduhan Payload via HTTP C2

**Berkas:** `soal17_wired_http_c2.pcapng` | **Port validasi:** 3404

### Yang diminta
Nama domain (Host) sumber malware, IP server penyerang, nama berkas `.exe` malware, dan kode status HTTP.

### Langkah analisis

**1. Melihat seluruh transaksi HTTP.** Filter:
```
http
```
Di antara permintaan halaman biasa, terdapat `GET /navi_agent.exe` (frame 30) dari `10.7.1.50` ke `203.0.113.42`, dan respons `200 OK` (frame 31).

![Q17.1 - Filter http](Assets/Q17.1%20-%20Filter%20http%20(semua%20request).png)

**2. Menyaring langsung ke berkas eksekusi.** Filter:
```
http.request.uri contains ".exe"
```
Buka panel detail HTTP: `Host: wired-update.net`, Request URI `/navi_agent.exe`, Full request URI `http://wired-update.net/navi_agent.exe`. IP penyerang adalah Destination paket = `203.0.113.42`.

![Q17.2 - Filter uri exe](Assets/Q17.2%20-%20Filter%20uri%20exe%20(Host%20wired-update).png)

**3. Membuktikan payload.** Follow HTTP Stream menampilkan respons `HTTP/1.1 200 OK`, `Server: nginx/1.24.0`, `Content-Disposition: attachment; filename="navi_agent.exe"`, dan body diawali `MZ ... This program cannot be run in DOS mode` - header PE yang membuktikan berkas adalah executable Windows asli.

![Q17.3 - Follow Stream](Assets/Q17.3%20-%20Follow%20Stream%20(nginx,%20navi_agent%20MZ).png)

### Temuan

| Item | Nilai |
|---|---|
| Domain (Host) | `wired-update.net` |
| IP server penyerang | `203.0.113.42` |
| Nama berkas malware | `navi_agent.exe` |
| Kode status HTTP | `200` |

### Validasi

![Q17.4 - Validasi nc 3404](Assets/Q17.4%20-%20Validasi%20nc%203404%20(flag).png)

Flag: `KOMJAR26{Navi_C2_D0wnl04d_8pH20OIfPFX7DUbyIwVSDg46J}`

---

## Soal 18 - Transfer Malware via SMB

**Berkas:** `soal18_wired_smb_transfer.pcapng` | **Port validasi:** 3405

### Yang diminta
Nama protokol yang dieksploitasi, IP pengirim dan penerima, folder tujuan pada korban, dan nama berkas `.exe` malware.

### Langkah analisis

**1. Melihat rangkaian operasi SMB.** Filter:
```
smb2
```
Urutan operasi SMB2 menceritakan seluruh serangan: `Tree Connect ... \\10.7.1.50\ADMIN$` (share administratif yang memetakan `C:\Windows`), `Create Request File: System32\wired_trojan_payload.exe`, `Write Request ... File: System32\wired_trojan_payload.exe`, lalu `Close`. Pengirim (penulis berkas) adalah `10.7.3.100`, penerima (server yang menyimpan) adalah `10.7.1.50`. Relatif terhadap `ADMIN$`, folder tujuan adalah `System32` (yaitu `C:\Windows\System32`).

![Q18.1 - Filter smb2](Assets/Q18.1%20-%20Filter%20smb2%20(Tree%20Connect%20Create%20Write).png)

**2. Paket penulisan definitif.** Detail `Write Request` (frame 20) menampilkan Tree `\\10.7.1.50\ADMIN$`, Write Length 1028, nama berkas `System32\wired_trojan_payload.exe`, dan data payload (terlihat pola `WIRED_PR_EXPLOIT...` pada dump). Ini paket yang membuktikan malware benar-benar ditulis ke korban.

![Q18.2 - Write Request detail](Assets/Q18.2%20-%20Write%20Request%20detail%20(System32%20payload).png)

### Temuan

| Item | Nilai |
|---|---|
| Protokol dieksploitasi | `SMB2` |
| IP pengirim (penyerang) | `10.7.3.100` |
| IP penerima (korban) | `10.7.1.50` |
| Folder tujuan | `System32` (via share `ADMIN$` = `C:\Windows\System32`) |
| Nama berkas malware | `wired_trojan_payload.exe` |

### Validasi
Catatan: server menolak jawaban `smb` dan menerima `smb2`; folder diterima dalam bentuk `system32`.

![Q18.3 - Validasi nc 3405](Assets/Q18.3%20-%20Validasi%20nc%203405%20(flag).png)

Flag: `KOMJAR26{SMB_Tr4nsf3r_8FNyM96dawHLrhTQVQilBOfso}`

---

## Soal 19 - Email Pemerasan via SMTP

**Berkas:** `soal19_wired_smtp_threat.pcapng` | **Port validasi:** 3406

### Yang diminta
Email korban, kata sandi yang diklaim bocor, jenis malware, batas waktu (dalam hari), dan MailClientID.

### Langkah analisis

**1. Menyaring lalu lintas SMTP.** SMTP adalah plaintext (port 25). Filter:
```
smtp
```
Terdapat **beberapa sesi**. Sesi awal (`chisa@internal.wired -> mika@internal.wired`, subjek "Laporan FTP mingguan") dan sebuah spam yang diblokir (`550`) adalah decoy - bukan email pemerasan.

![Q19.1 - Filter smtp](Assets/Q19.1%20-%20Filter%20smtp%20(beberapa%20sesi).png)

**2. Memastikan decoy.** Follow TCP Stream pada stream 0 hanya berisi email internal biasa tanpa kata sandi/malware/tenggat - jadi bukan targetnya.

![Q19.2 - Follow Stream decoy](Assets/Q19.2%20-%20Follow%20Stream%20decoy%20(email%20benign).png)

**3. Menemukan email pemerasan.** Dengan menaikkan pemilih Stream, ditemukan **stream 6**: server `mail.protocol7.co.jp` (Postfix), `MAIL FROM:<attacker@darkwired.net>`, `RCPT TO:<victim@protocol7.co.jp>`. Body memuat: "is your password!" -> `pr0tocol_7_user`; "infected with my private ransomware" -> `ransomware`; "72 hours (3 days)" -> tenggat; `MailClientID: 7719980706`. Karena soal menanyakan tenggat **dalam hari**, jawabannya `3` (bukan 72).

![Q19.3 - Follow Stream extortion](Assets/Q19.3%20-%20Follow%20Stream%20extortion%20(stream%206).png)

### Temuan

| Item | Nilai |
|---|---|
| Email korban | `victim@protocol7.co.jp` |
| Password bocor | `pr0tocol_7_user` |
| Jenis malware | `ransomware` |
| Batas waktu (hari) | `3` |
| MailClientID | `7719980706` |

### Validasi

![Q19.4 - Validasi nc 3406](Assets/Q19.4%20-%20Validasi%20nc%203406%20(flag).png)

Flag: `KOMJAR26{SMTP_Ext0rt10n_JtlmvSmNyEeaRl2cGRxSBzlsl}`

---

## Soal 20 - Dekripsi Lalu Lintas TLS

**Berkas:** `wired_tls_decrypt.pcapng` + `keyslogfile.txt` | **Port validasi:** 3407

### Yang diminta
Versi TLS, domain SNI, IP server HTTPS, User-Agent, dan method + path HTTP tersembunyi.

### Konsep dua fase
Koneksi TLS punya dua fase: **handshake** (negosiasi, plaintext) dan **application data** (HTTP asli, terenkripsi). Versi TLS, SNI, dan IP server berada di fase handshake (langsung terbaca); User-Agent dan method/path berada di dalam data terenkripsi (perlu didekripsi).

### Langkah analisis

**1. Membaca handshake tanpa dekripsi.** Filter:
```
tls.handshake.type == 1
```
Ini mengisolasi **Client Hello** - satu-satunya pesan tempat SNI (dikirim klien) dapat berada. Buka **TLS -> Handshake -> Extensions -> server_name**: `example.com`. Destination paket = `93.184.216.34` (server HTTPS). Versi `TLS 1.2`: tidak ada ekstensi `supported_versions`, sehingga ini benar-benar TLS 1.2 (bukan TLS 1.3 yang menyembunyikan versi di ekstensi itu). Ekstensi ALPN `http/1.1` menandakan isi terenkripsi berupa HTTP/1.1 biasa.

![Q20.1 - Client Hello](Assets/Q20.1%20-%20Client%20Hello%20(SNI,%20versi,%20server%20IP).png)

**2. Memuat key-log untuk dekripsi.** User-Agent dan method/path adalah data HTTP terenkripsi; sebelum dekripsi Wireshark hanya menampilkan "Application Data". Karena panitia menyediakan `keyslogfile.txt` (format NSS `CLIENT_RANDOM`), muat kuncinya lewat **Edit -> Preferences -> Protocols -> TLS -> (Pre)-Master-Secret log filename** dan arahkan ke `keyslogfile.txt`.

![Q20.2 - Preferences TLS keylog](Assets/Q20.2%20-%20Preferences%20TLS%20keylog.png)

**3. Membaca HTTP hasil dekripsi.** Setelah kunci dimuat, Wireshark mendekripsi Application Data dan menambahkan lapisan HTTP, sehingga filter `http` kini cocok. Terlihat permintaan `HEAD / HTTP/1.1`, `Host: example.com`, `User-Agent: curl/7.62.0`, dijawab `HTTP/1.1 200 OK`. Method `HEAD` adalah bagian "tersembunyi" (tidak lazim dibanding GET/POST); path adalah `/`.

![Q20.3 - HTTP terdekripsi](Assets/Q20.3%20-%20HTTP%20terdekripsi%20(HEAD,%20curl).png)

### Temuan

| Item | Nilai |
|---|---|
| Versi TLS | `TLS 1.2` |
| Domain SNI | `example.com` |
| IP server HTTPS | `93.184.216.34` |
| User-Agent | `curl/7.62.0` |
| Method + path | `HEAD /` |

### Validasi

![Q20.4 - Validasi nc 3407](Assets/Q20.4%20-%20Validasi%20nc%203407%20(flag).png)

Flag: `KOMJAR26{TLS_D3crypt_vEaV4kZGjYMwW1GXXwN5OA6w9}`

---

## 3. Rekapitulasi Flag

| Soal | Topik | Port | Flag |
|---|---|---|---|
| 14 | HTTP Brute Force | 3401 | `KOMJAR26{W1r3d_Brut3_jlUNLCw7Q8uIgOkU3ap80Xz53}` |
| 15 | USB HID Keystroke | 3402 | `KOMJAR26{USB_K3ystr0k3_wa37QzOwnVRSO5Bz39SBgsJu8}` |
| 16 | FTP Credential Theft | 3403 | `KOMJAR26{FTP_Th3ft_NcMqmDIgQwWba0tCS9ETbN0Aw}` |
| 17 | HTTP C2 Retrieval | 3404 | `KOMJAR26{Navi_C2_D0wnl04d_8pH20OIfPFX7DUbyIwVSDg46J}` |
| 18 | SMB Lateral Transfer | 3405 | `KOMJAR26{SMB_Tr4nsf3r_8FNyM96dawHLrhTQVQilBOfso}` |
| 19 | SMTP Threat | 3406 | `KOMJAR26{SMTP_Ext0rt10n_JtlmvSmNyEeaRl2cGRxSBzlsl}` |
| 20 | TLS Decrypted Stream | 3407 | `KOMJAR26{TLS_D3crypt_vEaV4kZGjYMwW1GXXwN5OA6w9}` |

## 4. Kesimpulan

Ketujuh soal menunjukkan satu prinsip inti: setiap informasi berada pada satu lapisan protokol tertentu, sehingga analisis dilakukan dengan memetakan "apa yang dicari" ke "protokol pemiliknya", lalu memilih display filter yang tepat.

- **Protokol plaintext** (HTTP, FTP, SMB, SMTP) membocorkan kredensial, nama berkas, dan isi pesan secara langsung lewat display filter dan Follow Stream. Ini memperlihatkan bahaya protokol tanpa enkripsi.
- **Data terenkripsi** (TLS/HTTPS pada Soal 20) tidak terbaca tanpa kunci; dengan key-log yang bocor, seluruh isi HTTP kembali terlihat - menegaskan bahwa keamanan TLS bergantung pada kerahasiaan kuncinya.
- **Jebakan (decoy)** muncul di Soal 14 (login sah "Welcome Alice"), Soal 16 (sesi `laporan.pdf`), dan Soal 19 (email internal biasa). Kunci membedakannya adalah berpatokan pada aset yang ditanyakan (IP penyerang, `knights_payload`, isi email ancaman), bukan sesi pertama yang terlihat.
- **Ketelitian format** pada validator penting: perhatikan petunjuk `Format:` (`IP`, `string`, `int`, `user:pass`), termasuk sensitivitas huruf dan awalan `0x`, serta konversi satuan (Soal 19 meminta hari, bukan jam).

Seluruh tujuh flag berhasil diperoleh, sehingga fase forensik Modul 1 selesai dan tervalidasi.
