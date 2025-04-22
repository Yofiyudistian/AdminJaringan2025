### DNS (Domain Name System)

**Definisi:**

DNS adalah sistem yang menerjemahkan **nama domain** (seperti www.google.com) menjadi **alamat IP** (seperti 142.250.190.78) yang dapat dimengerti oleh komputer.

**Tujuan:**

- Memudahkan pengguna mengakses situs web tanpa harus menghafal IP address.
- Meningkatkan efisiensi komunikasi antar perangkat dalam jaringan.
- Menyediakan struktur hierarki untuk nama domain (seperti .com, .org, .id, dll).

Contoh sederhananya:
Daripada menghafal 172.217.0.46, kita cukup mengetik www.google.com di browser, dan DNS akan mencarikan IP-nya untuk kita.

### IP Forwarding

` `**Definisi:**

IP forwarding (juga dikenal sebagai **routing**) adalah proses di mana perangkat jaringan (seperti router) meneruskan paket data dari satu jaringan ke jaringan lain berdasarkan **alamat IP tujuan**.

**Tujuan:**

- Menghubungkan antar jaringan (misalnya antara jaringan lokal dan internet).
- Mengarahkan lalu lintas data ke jalur yang benar agar sampai ke tujuan.
- Mendukung komunikasi lintas subnet/jaringan.

Contoh: Jika kamu mengakses website dari laptop di rumahmu, router rumahmu akan melakukan IP forwarding untuk mengirim permintaan dari laptopmu ke internet, lalu mengembalikan hasilnya ke laptopmu.


### KONFIGURASI VM1 (No GUI)

Setting network adapter 1 menjadi bridge dan adapter 2 menjadi internet.

·  **Adapter 1 = Bridge:**
Agar VM1 bisa terhubung langsung ke jaringan fisik (misalnya internet rumah), seperti layaknya perangkat biasa.

·  **Adapter 2 = Internal:**
Digunakan untuk membuat jaringan lokal (LAN) antara VM1 dan VM2, sehingga mereka bisa saling berkomunikasi tanpa keluar ke internet.

![image](https://github.com/user-attachments/assets/8cf86e80-bfab-4cf9-a14b-7d3ab11041ba)
![image](https://github.com/user-attachments/assets/e2f71e79-18f3-4674-841c-650a7f01e6c4)


Install requirement

Sudo apt install bind9 bind9utils

·  **BIND9**: Software DNS yang paling umum di Linux.

·  **bind9utils**: Tools tambahan untuk cek konfigurasi DNS.

![image](https://github.com/user-attachments/assets/d367f74b-067e-4eb0-ad0a-8538753e582a)


Sudo apt install iptables iptables-persistent

- **iptables**: Untuk mengatur firewall dan routing NAT.
- **iptables-persistent**: Menyimpan aturan iptables agar tetap aktif setelah reboot.

![image](https://github.com/user-attachments/assets/106fc567-6943-4f60-90e0-31d4c30aeb22)


Konfigurasi ip addres

Sudo nano /etc/networking/interfaces. 

Digunakan untuk menetapkan alamat IP statis pada VM1 (adapter internal).

![image](https://github.com/user-attachments/assets/945ab0a8-0815-45fa-9f23-75b5036afbb9)


Restart network

sudo systemctl restart networking

![image](https://github.com/user-attachments/assets/acd5e008-ccea-435d-94ac-9e2cde53ef4e)


Dns configuratioin

Sudo nano /etc/bind/named.conf

Tambahkan include "/etc/bind/named.conf.external-zones"

![image](https://github.com/user-attachments/assets/9ea9164b-e18a-422e-b5ce-9537aedb41cb)


Konfigurasi file named.conf.options

Tambahkan 

allow-query { any; };

allow-transfer { any; }; 

recursion yes;

·  **allow-query**: Mengizinkan semua klien untuk meminta DNS.

·  **allow-transfer**: Mengizinkan transfer zona (berguna untuk slave DNS, jika ada).

·  **recursion**: Mengaktifkan pencarian DNS berantai (jika VM1 tidak tahu, dia cari tahu ke DNS lain).

![image](https://github.com/user-attachments/assets/f08725c9-c5a6-45bc-a37d-51979a7d962f)


Konfigurasi file named.conf.external-zones

Sudo /etc/bind/named.conf.external-zones

Di sini kita mendefinisikan dua **zone DNS**:

- Satu untuk domain kelompok6.com.
- Satu lagi untuk **reverse DNS** dari IP 192.168.200.x.

![image](https://github.com/user-attachments/assets/cf290274-33cf-41e8-b4f7-e61ab72a30a3)


Sudo nano /etc/bind/kelompok6.com

Digunakan sebagai Zona DNS untuk domain.

![image](https://github.com/user-attachments/assets/5e99dfcc-0b51-4aeb-ae8a-f8572a9d603d)


sudo nano 1.200.168.192.db

digunakan sebagai Zona reverse DNS untuk IP.

![image](https://github.com/user-attachments/assets/675018b7-1687-49ad-b2d3-6c2c63fa5299)


named-checkzone [nama\_zone] [file\_konfigurasi\_zone]

named-checkzone kelompok6.com /etc/bind/kelompok6.com

namaed-checkzone 1.200.168.192.in-addr.arpa ./1.200.168.192.db

Untuk memastikan tidak ada error penulisan di file zona

![image](https://github.com/user-attachments/assets/7ea8dc0e-8069-4bae-84d6-a635f04d89b6)



restart named.service

DirEstart agar konfigurasi dns aktif

![image](https://github.com/user-attachments/assets/a3d5fe0d-47b8-4948-8ad7-0891a529d01d)


set ip forwarding

echo 1 > /proc/sys/net/ipv4/ip\_forward

Mengaktifkan kemampuan VM1 untuk meneruskan paket antar jaringan (dari internal ke luar).

![image](https://github.com/user-attachments/assets/127b61ac-5b1c-4477-9ab7-93f710032b12)


Atur NAT di iptables:

sudo iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE

NAT agar VM2 bisa akses internet via VM1.

sudo iptables -A FORWARD -i enp0s8 -o enp0s3 -j ACCEPT 

sudo iptables -A FORWARD -i enp0s3 -o enp0s8 -j ACCEPT

Mengizinkan lalu lintas antar adapter internal dan eksternal.

![image](https://github.com/user-attachments/assets/7bf19d2e-7dee-4f02-8cd5-3afce915ceb3)


sudo iptables-save

Menyimpan aturan iptables agar tidak hilang setelah reboot.

![image](https://github.com/user-attachments/assets/bfee91ac-a7ae-47d1-8be7-b3044a21f310)


### konfigurasi vm2 (gui)

ubah jadi networknya jadi internal network dan diwired juga disetting seperti gambar dibawah. Agar VM2 bisa terhubung dengan VM1 melalui adapter internal.

Atur IP static seperti:

- IP: 192.168.200.10
- Netmask: 255.255.255.0
- Gateway: 192.168.200.1 (IP VM1)
- DNS: 192.168.200.1, 1.1.1.1 (VM1 sebagai DNS Server)

![image](https://github.com/user-attachments/assets/122a5ea4-d4c5-4310-8523-238192dca91e)
![image](https://github.com/user-attachments/assets/85475f70-3573-4ce5-b83f-b7ca541a2c82)


ping vm1

ping 192.168.200.1

- Untuk memastikan koneksi jaringan antar VM berhasil.

![image](https://github.com/user-attachments/assets/52a533e5-1d09-440c-876b-3a9e9ec9e2c7)


Cek DNS vm1

Dig www.kelompok6.com.

Dig –x 192.168.200.1

Nslookup www.kelompok6.com.

Untuk memastikan DNS dari VM1 bekerja dan bisa menjawab permintaan nama domain. 

![image](https://github.com/user-attachments/assets/b56675de-41fd-4b4a-bae6-5b17c8d74e1f)
![image](https://github.com/user-attachments/assets/9c221c6f-587d-4f3f-bf7e-8a637f0402a9)



