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
