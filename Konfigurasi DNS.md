Membuat PC Menjadi Server DNS Lokal dengan BIND di Linux

# Pendahuluan

DNS (Domain Name System) adalah seperti "buku telepon" internet: ia menerjemahkan nama seperti kelompok6.home menjadi alamat IP seperti 192.168.6.10, yang bisa dimengerti oleh komputer. Dalam panduan ini, kamu membuat sebuah PC menjadi server DNS lokal menggunakan aplikasi BIND di Linux.

# 1. Konfigurasi File named.conf

Tambahkan baris berikut:

![C:\Users\ALDA\AppData\Local\Packages\5319275A.WhatsAppDesktop_cv1g1gvanyjgm\TempState\55C3E298C9CD3D8EA2665241B6061629\WhatsApp Image 2025-04-29 at 12.47.43_ff5ba655.jpg](data:image/jpeg;base64...)

include "/etc/bind/named.conf.internal-zones";

Penjelasan:

- Baris ini menghubungkan file utama konfigurasi DNS (named.conf) ke file tambahan (named.conf.internal-zones).

- Fungsinya seperti daftar isi, tempat kamu menambahkan bab baru yang berisi informasi detail tentang zona DNS, seperti kelompok6.home.

# 2. Mengatur File Zona dan Konfigurasi named.conf.options

Isi File /etc/bind/named.conf.options:

![C:\Users\ALDA\AppData\Local\Packages\5319275A.WhatsAppDesktop_cv1g1gvanyjgm\TempState\1482F6B491F631405A4581F8F44FD726\WhatsApp Image 2025-04-29 at 13.06.47_a624fc9f.jpg](data:image/jpeg;base64...)

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

Penjelasan:

- acl internal-network: menentukan siapa yang boleh mengakses DNS.

- forwarders: meneruskan pertanyaan DNS ke server eksternal jika tidak tahu jawabannya.

- allow-query, allow-transfer: membatasi akses ke DNS.

- recursion yes: DNS dapat mencari ke DNS lain jika tidak tahu jawabannya.

# 3. Membuat File Zona kelompok6.home.lan

Contoh isi file zona:

![C:\Users\ALDA\AppData\Local\Packages\5319275A.WhatsAppDesktop_cv1g1gvanyjgm\TempState\7C0B5EFDB64885A42DE0C26584E839C3\WhatsApp Image 2025-04-29 at 12.51.31_936a8722.jpg](data:image/jpeg;base64...)

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

Penjelasan:

- SOA: informasi utama tentang domain.

- NS: nameserver dari domain ini.

- A: mencocokkan nama ke alamat IP.

- MX: pengatur jalur email.

- CNAME: alias (contoh: www mengarah ke ns).

# 4. Membuat File Reverse DNS

Perintah: nano 192.168.6.db

![C:\Users\ALDA\AppData\Local\Packages\5319275A.WhatsAppDesktop_cv1g1gvanyjgm\TempState\9DE6D14FFF9806D4BCD1EF555BE766CD\WhatsApp Image 2025-04-29 at 16.23.42_ff310889.jpg](data:image/jpeg;base64...)

@ IN SOA kelompok6.home. root.kelompok6.home. (
 2025042401 ;Serial
 3600 ;Refresh
 1800 ;Retry
 604800 ;Expire
 86400 ) ;Minimum TTL

 IN NS ns.kelompok6.home.

10 IN PTR ns.kelompok6.home.
10 IN PTR www.kelompok6.home.

Penjelasan:

- SOA: (Start of Authority) mendefinisikan server utama untuk zona ini serta metadata seperti TTL dan versi serial.

- NS: Mendefinisikan nameserver utama zona ini.

- 10 IN PTR ns.kelompok6.home.: menyatakan bahwa IP 192.168.6.10 dikenali sebagai ns.kelompok6.home. saat reverse lookup.

- 10 IN PTR www.kelompok6.home.: menunjuk ke alias www.kelompok6.home. untuk IP yang sama.

# 5. Validasi File Zona dengan named-checkzone

![C:\Users\ALDA\AppData\Local\Packages\5319275A.WhatsAppDesktop_cv1g1gvanyjgm\TempState\463F02F62D6DE5EF16542D530B144FA4\WhatsApp Image 2025-04-29 at 13.00.52_a5d4d1d8.jpg](data:image/jpeg;base64...)

Perintah:
named-checkzone kelompok6.home ./kelompok6.home.lan

Hasil:
zone kelompok6.home/IN: loaded serial 2025042401
OK

Penjelasan: Memastikan file zona valid dan tidak ada kesalahan penulisan.

# 6. Pengujian DNS dengan dig

![C:\Users\ALDA\AppData\Local\Packages\5319275A.WhatsAppDesktop_cv1g1gvanyjgm\TempState\BB3EB41E9E943362D9D5200F7E5BB84D\WhatsApp Image 2025-04-29 at 13.03.07_86c3c7e0.jpg](data:image/jpeg;base64...)

Perintah:
dig kelompok6.home

Hasil:
kelompok6.home. 86400 IN A 192.168.6.10

Penjelasan: Perintah dig digunakan untuk menguji apakah DNS server bisa menjawab pertanyaan. Hasil menunjukkan DNS berfungsi dengan baik.

# Kesimpulan

- Menyiapkan dan mengonfigurasi DNS lokal menggunakan BIND di Linux.

- Membuat file zona untuk domain kelompok6.home.

- Melakukan pengecekan dan pengujian.

Sekarang perangkat di jaringan lokal bisa menggunakan nama domain seperti kelompok6.home tanpa harus mengingat alamat IP-nya.
