# **WORKSHOP ADMINISTRASI JARINGAN**

**NAMA:** YOFI YUDISTIAN  
**NRP:** 3123600010  
**KELAS:** 2 D4 TEKNIK INFORMATIKA A  

#
A. Instalasi NTP Client

1. Install dan Konfigurasi NTP Client

Menginstall dan mengkonfigurasi NTP client agar host memiliki waktu yang sinkron dengan NTP server di Indonesia.

Langkah-langkah:

Install NTP dengan perintah:

sudo apt install ntp

Buka file konfigurasi NTP:

sudo nano /etc/ntp.conf

Ubah konfigurasi default dengan server NTP Indonesia dari https://www.ntppool.org/en/zone/id:

server 0.id.pool.ntp.org
server 1.id.pool.ntp.org
server 2.id.pool.ntp.org
server 3.id.pool.ntp.org

Restart & aktifkan NTP dengan perintah:

sudo systemctl restart ntp
sudo systemctl enable ntp

Verifikasi waktu dengan perintah:

ntpq -p

Referensi:

Server World - NTP

NTP Pool

B. Instalasi dan Konfigurasi Samba

1. Membuat Public Shared Folder

Membuat folder publik yang bisa diakses melalui Windows dan Linux Client via file manager.

Langkah-langkah:

Install Samba dan dependensinya:

sudo apt install -y samba smbclient cifs-utils

Buat folder untuk shared public:

sudo mkdir -p /srv/samba/publicAdmin
sudo chmod 777 /srv/samba/publicAdmin
sudo chown nobody:nogroup /srv/samba/publicAdmin

Edit konfigurasi Samba:

sudo nano /etc/samba/smb.conf

Tambahkan di akhir file:

[Public] 
path = /srv/samba/publicAdmin
browseable = yes
writable = yes
guest ok = yes
create mask = 0777
directory mask = 0777

Restart Samba dan cek statusnya:

sudo systemctl restart smbd
sudo systemctl status smbd

Untuk Linux, buat file untuk uji coba dengan:

nano tes

Cek IP address dengan:

ip a

Verifikasi dengan:

smbclient -L //192.168.188.94/Public -N

2. Membuat Limited Shared Folder

Membuat folder dengan akses terbatas hanya untuk user tertentu.

Langkah-langkah:

Buat folder dan set pemilik:

sudo mkdir -p /srv/samba/limited
sudo chmod 770 /srv/samba/limited
sudo chown nobody:sambashare /srv/samba/limited

Tambahkan user Samba:

sudo useradd -M -s /sbin/nologin smbuser
sudo passwd smbuser
sudo smbpasswd -a smbuser
sudo smbpasswd -e smbuser
sudo usermod -aG sambashare smbuser

Edit konfigurasi Samba:

sudo nano /etc/samba/smb.conf

Tambahkan:

[Limited]
path = /srv/samba/limited
browseable = no
writable = yes
valid users = smbuser
create mask = 0770
directory mask = 0770

Restart Samba:

sudo systemctl restart smbd

Coba akses folder dari Linux dengan:

smbclient //192.168.188.94/Limited -U smbuser

Referensi:Server World - Samba

# Administrasi Sistem Debian 12

## Sumber Perangkat Lunak
Debian menggunakan repositori untuk mendistribusikan aplikasi. Alamat repositori disimpan dalam file `/etc/apt/sources.list`. File ini dapat diedit menggunakan perintah seperti `apt edit-sources` atau editor teks seperti `nano`.

- **`deb`**: Repositori biner (perangkat lunak yang sudah dikompilasi).
- **`deb-src`**: Repositori sumber (kode program untuk dikompilasi).
- **`bookworm`**: Nama versi Debian 12 yang saat ini stabil.

Repositori Debian dibagi menjadi beberapa bagian:
- **`main`**: Perangkat lunak bebas yang memenuhi DFSG (Debian Free Software Guidelines).
- **`non-free-firmware`**: Firmware non-bebas yang disertakan sejak Debian 12.
- **`contrib`**: Perangkat lunak bebas dengan dependensi non-bebas.
- **`non-free`**: Perangkat lunak yang tidak memenuhi DFSG.

## Paket Backport
Backport adalah mekanisme untuk membawa versi terbaru aplikasi dari repositori pengembangan ke versi stabil. Repositori backport tidak diaktifkan secara default, tetapi aman digunakan.

## Memodifikasi Repositori
Anda dapat memodifikasi file `/etc/apt/sources.list` untuk menambahkan repositori non-bebas atau backport. Perubahan ini dapat dilakukan melalui terminal atau menggunakan manajer paket grafis seperti **Synaptic**.

## APT di Terminal
APT (**Advanced P
