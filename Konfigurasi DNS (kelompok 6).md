# Membuat PC Menjadi Server DNS Lokal dengan BIND di Linux

Mata Kuliah : Workshop Administrasi Jaringan

Dosen Pengampu : Dr. Ferry Astika Saputra ST, M.Sc

# Kelompok 6

Anggota :

1. Yofi Yudistian (3123600006)

2. Muhammad Zaidan Zhafiz Satrianto (3123600021)

3. Muhammad Felda Hibatullah (3123600023)

# Tujuan

DNS (Domain Name System) adalah seperti "buku telepon" internet: ia menerjemahkan nama seperti kelompok6.home menjadi alamat IP seperti 192.168.6.10, yang bisa dimengerti oleh komputer. Dalam panduan ini, kamu membuat sebuah PC menjadi server DNS lokal menggunakan aplikasi BIND di Linux.

# Topologi

![image](https://github.com/user-attachments/assets/2f36d468-5c75-4052-bb5a-5330d7ab00e2)

Menunjukkan sebuah server utama (master) dengan IP 10.252.109.10/24 yang terhubung ke beberapa router, masing-masing mengarah ke jaringan domain lokal kelompok 2, 3, dan 6 dengan subnet IP 192.168.x.0/24. Setiap domain memiliki router tersendiri dan beberapa node/PC di dalamnya. Topologi ini menggambarkan sistem jaringan tersegmentasi yang dikendalikan dari satu titik pusat, kemungkinan untuk kebutuhan DNS lokal atau simulasi jaringan antar kelompok.

# 1. Konfigurasi File named.conf

Tambahkan baris berikut:

```bash
include "/etc/bind/named.conf.internal-zones";
```

![image](https://github.com/user-attachments/assets/1246220f-97f2-4f04-90c4-6ac96a6a7747)


- Baris ini menghubungkan file utama konfigurasi DNS (named.conf) ke file tambahan (named.conf.internal-zones).
- Fungsinya seperti daftar isi, tempat kamu menambahkan bab baru yang berisi informasi detail tentang zona DNS, seperti kelompok6.home.

# 2. Mengatur File Zona dan Konfigurasi named.conf.options

Isi File /etc/bind/named.conf.options:

```bash
acl internal-network {
 192.168.6.0/24;
};

options {
 directory "/var/cache/bind";
 forwarders {
 10.10.10.1;
 };
 allow-query { localhost; internal-network; };
 allow-transfer { localhost; internal-network; };
 recursion yes;
 dnssec-validation auto;
 listen-on-v6 { any; };
};
```

![image](https://github.com/user-attachments/assets/ededa4a1-f299-4ae3-a47b-b7d9f9d88f04)

- acl internal-network: menentukan siapa yang boleh mengakses DNS.
- forwarders: meneruskan pertanyaan DNS ke server eksternal jika tidak tahu jawabannya.
- allow-query, allow-transfer: membatasi akses ke DNS.
- recursion yes: DNS dapat mencari ke DNS lain jika tidak tahu jawabannya.

# 3. Membuat File Zona kelompok6.home.lan

Contoh isi file zona:

```bash
@ IN SOA kelompok6.home. root.kelompok6.home. (
 2025042401 ;Serial
 3600 ;Refresh
 1800 ;Retry
 604800 ;Expire
 86400 ) ;Minimum TTL

 IN NS ns.kelompok6.home.
 IN A 192.168.6.10
 IN MX 10 ns.kelompok6.home.

ns IN A 192.168.6.10
www IN CNAME ns
```

![image](https://github.com/user-attachments/assets/25057206-10cc-43ca-bb3b-a3d14e7b29b7)

- SOA: informasi utama tentang domain.
- NS: nameserver dari domain ini.
- A: mencocokkan nama ke alamat IP.
- MX: pengatur jalur email.
- CNAME: alias (contoh: www mengarah ke ns).

# 4. Membuat File Reverse DNS

Perintah: nano 192.168.6.db

```bash
@ IN SOA kelompok6.home. root.kelompok6.home. (
 2025042401 ;Serial
 3600 ;Refresh
 1800 ;Retry
 604800 ;Expire
 86400 ) ;Minimum TTL

 IN NS ns.kelompok6.home.

10 IN PTR ns.kelompok6.home.
10 IN PTR www.kelompok6.home.
```

![image](https://github.com/user-attachments/assets/5b62288c-78c4-47cc-a049-5eed73e6d70e)

- SOA: mendefinisikan server utama untuk zona ini serta metadata seperti TTL dan versi serial.
- NS: Mendefinisikan nameserver utama zona ini.
- PTR: menyatakan bahwa alamat IP akan dikenali sebagai nama domain tertentu jika ada permintaan reverse lookup.

# 5. Validasi File Zona dengan named-checkzone

Perintah:

```bash
named-checkzone kelompok6.home ./kelompok6.home.lan
```

Hasil:
zone kelompok6.home/IN: loaded serial 2025042401
OK

![image](https://github.com/user-attachments/assets/9b1b144e-7484-47b7-a5e7-d9d39ce2cb44)

- Memastikan file zona valid dan tidak ada kesalahan penulisan.

# 7. Pengujian DNS dengan dig

Perintah:

```bash
dig kelompok6.home
```

![image](https://github.com/user-attachments/assets/0fd82780-b43f-47fb-a4d2-a4f92a7dc385)


- Perintah dig digunakan untuk menguji apakah DNS server bisa menjawab pertanyaan.
- Hasil menunjukkan DNS berfungsi dengan baik.

# 8. Menampilkan Web dengan Domain

- Menginstal web server di PC dengan IP 192.168.6.10 (misalnya Apache):

```bash
 sudo apt update
 sudo apt install apache2
```

- Letakkan file website di direktori: /var/www/html/
- Buat halaman

```bash
nano index.html
```

- Restart web server

```bash
sudo systemctl restart apache2
```

- Akses web di browser:
 - Lokal: 192.168.6.10

![image](https://github.com/user-attachments/assets/0fbeb090-397f-4c17-9b28-74ec9e16f004)

 - Klien lain: 192.168.2.10

![image](https://github.com/user-attachments/assets/615487c2-5cb6-40c3-808b-3616e847c557)
