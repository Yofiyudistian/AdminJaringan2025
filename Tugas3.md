# Instalasi NTP Client dan Samba di Linux

## A. Instalasi NTP Client

### 1. Install dan Konfigurasi NTP Client

Install NTP dengan perintah:

```bash
sudo apt install ntp
```

### 2. Buka File Konfigurasi NTP

```bash
sudo nano /etc/ntp.conf
```

### 3. Ubah Nama NTP Server ke Server Indonesia

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

### 4. Restart dan Aktifkan NTP

```bash
sudo systemctl restart ntp
sudo systemctl enable ntp
```

### 5. Verifikasi Waktu dengan Perintah

```bash
ntpq -p
```

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

#### Buat Folder untuk Public Share

```bash
sudo mkdir -p /srv/samba/publicAdmin
sudo chmod 777 /srv/samba/publicAdmin
sudo chown nobody:nogroup /srv/samba/publicAdmin
```

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

#### Restart dan Cek Status Samba

```bash
sudo systemctl restart smbd
sudo systemctl status smbd
```

#### Cek IP Address

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

```ini
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

#### Cek Shared Folder di Windows (Pastikan Satu Jaringan), Lalu Akses dengan IP yang Sesuai

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

---

### Referensi:
- [Server-World: Samba di Debian](https://www.server-world.info/en/note?os=Debian_12&p=samba&f=1)
- Package terkait: `samba`, `smbclient`, `cifs-tools`

