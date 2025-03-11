# Instalasi NTP Client dan Samba di Linux

## A. Instalasi NTP Client

### 1. Install dan Konfigurasi NTP Client

Install NTP dengan perintah:

```bash
sudo apt install ntp
```
![image](https://github.com/user-attachments/assets/0c257675-0c1e-4933-a2cc-5af1a8fa4871)


### Buka File Konfigurasi NTP

```bash
sudo nano /etc/ntp.conf
```
![image](https://github.com/user-attachments/assets/aa4f8d9f-e896-479c-b4db-9aff5bf1055f)


### 2. Ubah Nama NTP Server ke Server Indonesia

Dari:

```bash
#pool 0.ubuntu.pool.ntp.org iburst
#pool 1.ubuntu.pool.ntp.org iburst
#pool 2.ubuntu.pool.ntp.org iburst
#pool 3.ubuntu.pool.ntp.org iburst
```

Menjadi:

```bash
server 0.id.pool.ntp.org
server 1.id.pool.ntp.org
server 2.id.pool.ntp.org
server 3.id.pool.ntp.org
```
![image](https://github.com/user-attachments/assets/cf0239c5-d534-4470-a75e-026cd1a99b10)


### Restart dan Aktifkan NTP

```bash
sudo systemctl restart ntp
sudo systemctl enable ntp
```

### Verifikasi Waktu dengan Perintah

```bash
ntpq -p
```
![image](https://github.com/user-attachments/assets/c9384b76-1e5f-49fd-86bc-56cbbb5e1069)


#### Referensi:
- [Server-World: NTP di Debian](https://www.server-world.info/en/note?os=Debian_12&p=ntp&f=1)
- [NTP Pool Server Indonesia](https://www.ntppool.org/en/zone/id)

---

## B. Instalasi dan Konfigurasi Samba

### 1. Membuat Public Shared Folder

#### Install Samba dan Paket Terkait

```bash
sudo apt install -y samba smbclient cifs-utils
```

- `samba` untuk berbagi file di Ubuntu
- `smbclient` untuk mengakses folder bersama dari sistem lain
- `cifs-utils` untuk memount folder SMB dari Windows atau server lain
![image](https://github.com/user-attachments/assets/d3dde38b-3330-40e0-b50f-e06903b67d72)


#### Buat Folder untuk Public Share

```bash
sudo mkdir -p /srv/samba/publicAdmin
sudo chmod 777 /srv/samba/publicAdmin
sudo chown nobody:nogroup /srv/samba/publicAdmin
```
![image](https://github.com/user-attachments/assets/f78d1558-922f-4ee8-9964-b98ab4e87885)


#### Edit File Konfigurasi Samba

```bash
sudo nano /etc/samba/smb.conf
```

Tambahkan konfigurasi di akhir file:

```ini
[Public] 
path = /srv/samba/publicAdmin 
browseable = yes 
writable = yes 
guest ok = yes 
create mask = 0777 
directory mask = 0777
```
![image](https://github.com/user-attachments/assets/fc4b143c-1bd4-4dcb-8693-42f527e9e676)


#### Restart dan Cek Status Samba

```bash
sudo systemctl restart smbd
sudo systemctl status smbd
```
![image](https://github.com/user-attachments/assets/ad05884b-1354-4e0a-aec6-93bb78685405)


#### Untuk linux cek IP Address

```bash
ip a
```

#### Cek Folder yang Di-share

```bash
smbclient -L //192.168.188.94/Public -N
```

---

### 2. Membuat Limited Shared Folder

#### Buat Folder dan Tentukan Pemilik

```bash
sudo mkdir -p /srv/samba/limited
sudo chmod 770 /srv/samba/limited
sudo chown nobody:sambashare /srv/samba/limited
```

#### Tambah User di Samba

```bash
sudo useradd -M -s /sbin/nologin smbuser
sudo passwd smbuser
sudo smbpasswd -a smbuser
sudo smbpasswd -e smbuser
sudo usermod -aG sambashare smbuser
```

#### Edit File Konfigurasi Samba

```bash
sudo nano /etc/samba/smb.conf
```

Tambahkan konfigurasi berikut:

```
[Limited]
   path = /srv/samba/limited
   browseable = no
   writable = yes
   valid users = smbuser
   create mask = 0770
   directory mask = 0770
```

#### Restart Samba

```bash
sudo systemctl restart smbd
```

#### Akses Limited Shared Folder dari Linux

```bash
smbclient //192.168.188.94/Limited -U smbuser
```

---

### 3. Akses ke Folder Share dari CLI Client

#### Cek IP Address

```bash
ip a
```
![image](https://github.com/user-attachments/assets/fe3894d0-2bdd-4be4-82a4-6f2543f73c79)


#### Cek network di Windows (Pastikan Satu Jaringan), Lalu Akses dengan IP yang Sesuai
![image](https://github.com/user-attachments/assets/4b67dcdb-2497-4ff1-af90-4b1d9a1ef3f2)


#### Akses dari Linux

```bash
smbclient -L //192.168.188.94/Public -N
```

#### Hasilnya Akan Terlihat Seperti Ini:

```
Sharename    Type    Comment
---------    ----    -------
print$       Disk    Printer Drivers
Public       Disk
IPC$         IPC     IPC Service (Samba 4.17.12-Debian)
nobody       Disk    Home Directories
SMB1 disabled -- no workgroup available
```

#### Akses Folder Public

```bash
smbclient //192.168.188.94/Public -N
```

#### Lihat Isi Folder

```bash
smb: \> ls
  .                               D       0  Mon Mar 10 14:36:02 2025
  ..                              D       0  Mon Mar 10 14:33:53 2025
  tes                             N      11  Mon Mar 10 14:35:59 2025

  19480400 blocks of size 1024. 12861968 blocks available
```
### Hasilnya
![image](https://github.com/user-attachments/assets/937f489c-c165-46f7-acc9-c64fa83ee699)

---

### Referensi:
- [Server-World: Samba di Debian](https://www.server-world.info/en/note?os=Debian_12&p=samba&f=1)
- Package terkait: `samba`, `smbclient`, `cifs-tools`

## Administrasi Sistem Debian 12
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
APT (**Advanced Package Tool**) adalah alat utama untuk mengelola paket di Debian. Beberapa perintah dasar:
```sh
apt update        # Memperbarui metadata repositori
apt install <paket>  # Menginstal paket
apt upgrade       # Memperbarui paket yang terinstal
apt remove <paket>  # Menghapus paket
apt autoremove    # Menghapus paket yang tidak diperlukan
```

## Software: Manajer Paket Sederhana
**Software** adalah antarmuka grafis sederhana untuk mengelola aplikasi di Debian. Fitur utamanya meliputi:
- Pencarian dan instalasi aplikasi.
- Pembaruan sistem.
- Manajemen repositori.

## Discover: Manajer Paket KDE
**Discover** adalah manajer paket untuk lingkungan desktop KDE. Memungkinkan pengguna untuk mencari, menginstal, dan memperbarui aplikasi dengan antarmuka yang intuitif.

## Synaptic: Manajer Paket Komprehensif
**Synaptic** adalah antarmuka grafis yang lebih detail untuk mengelola paket. Ini menampilkan semua paket yang tersedia, termasuk library dan dependensi. Fitur utamanya meliputi:
- Pencarian paket.
- Instalasi dan penghapusan paket.
- Pembersihan paket yang tidak diperlukan.

## Membersihkan Sistem
Beberapa cara untuk membersihkan sistem:
```sh
apt clean                 # Membersihkan cache paket
apt autoremove --purge    # Menghapus paket yang tidak diperlukan beserta file konfigurasinya
deborphan                 # Menemukan dan menghapus paket yang tidak terpakai
```

## Menginstal Paket Eksternal `.deb`
Paket `.deb` dapat diinstal menggunakan:
- **GDebi**: Antarmuka grafis untuk menginstal paket `.deb` dengan manajemen dependensi.
- **dpkg**: Alat terminal untuk menginstal paket `.deb`, tetapi tidak mengelola dependensi secara otomatis:
  ```sh
  dpkg -i nama_paket.deb
  apt -f install  # Menginstal dependensi yang hilang
  ```

## Menginstal Aplikasi Flatpak
Flatpak adalah sistem virtualisasi untuk menjalankan aplikasi dalam lingkungan terisolasi (**sandbox**). Beberapa langkah untuk menggunakan Flatpak:
```sh
apt install flatpak  # Instal Flatpak
flatpak remote-add flathub https://flathub.org/repo/flathub.flatpakrepo  # Tambahkan repositori Flathub
```
Kelola Flatpak melalui terminal atau manajer paket grafis seperti **Software (Gnome)** dan **Discover (KDE)**.

