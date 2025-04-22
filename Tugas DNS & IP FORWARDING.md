# Konfigurasi DNS dan IP Forwarding Menggunakan VM

## 📘 DNS (Domain Name System)

### Definisi:
DNS adalah sistem yang menerjemahkan nama domain (seperti `www.google.com`) menjadi alamat IP (seperti `142.250.190.78`) yang dapat dimengerti oleh komputer.

### Tujuan:
- Memudahkan pengguna mengakses situs web tanpa harus menghafal IP address.
- Meningkatkan efisiensi komunikasi antar perangkat dalam jaringan.
- Menyediakan struktur hierarki untuk nama domain (seperti `.com`, `.org`, `.id`, dll).

### Contoh:
Daripada menghafal `172.217.0.46`, kita cukup mengetik `www.google.com` di browser, dan DNS akan mencarikan IP-nya untuk kita.

---

## 🌐 IP Forwarding

### Definisi:
IP forwarding (juga dikenal sebagai routing) adalah proses di mana perangkat jaringan (seperti router) meneruskan paket data dari satu jaringan ke jaringan lain berdasarkan alamat IP tujuan.

### Tujuan:
- Menghubungkan antar jaringan (misalnya antara jaringan lokal dan internet).
- Mengarahkan lalu lintas data ke jalur yang benar agar sampai ke tujuan.
- Mendukung komunikasi lintas subnet/jaringan.

### Contoh:
Jika kamu mengakses website dari laptop di rumahmu, router rumahmu akan melakukan IP forwarding untuk mengirim permintaan dari laptopmu ke internet, lalu mengembalikan hasilnya ke laptopmu.

---

## ⚙️ KONFIGURASI VM1 (No GUI)

### 🔌 Setting Network
- **Adapter 1 = Bridge**: Agar VM1 bisa terhubung langsung ke jaringan fisik (misalnya internet rumah), seperti layaknya perangkat biasa.
- **Adapter 2 = Internal**: Digunakan untuk membuat jaringan lokal (LAN) antara VM1 dan VM2, sehingga mereka bisa saling berkomunikasi tanpa keluar ke internet.

---

### 📦 Install Requirement
```bash
sudo apt install bind9 bind9utils
