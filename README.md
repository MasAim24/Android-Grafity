Google Antigravity CLI (agy) on Termux (Android)
Panduan dan skrip instalasi resmi untuk menjalankan Google Antigravity CLI (agy) di lingkungan Termux Android secara seamless tanpa perlu selalu masuk manual ke dalam kontainer Linux.

📌 Latar Belakang Masalah
Saat menginstal agy langsung di Termux menggunakan perintah standar:
curl -fsSL https://antigravity.google/cli/install.sh | bash


Instalasi akan gagal dengan pesan error:
bash: line 236: /data/data/com.termux/files/home/.local/bin/agy: cannot execute: required file not found


Mengapa ini terjadi?
Bionic libc vs GNU glibc: Biner agy dikompilasi untuk distro Linux standar berbasis glibc (membutuhkan dynamic linker /lib/ld-linux-aarch64.so.1).
Android Kernel: Termux berjalan di atas pustaka C Android (Bionic libc), sehingga kernel menolak mengeksekusi biner yang memerlukan loader glibc.
Solusinya
Menggunakan PRoot Distro (Ubuntu) sebagai runtime glibc, kemudian membuat skrip pembungkus (transparent wrapper) di $PREFIX/bin/agy agar CLI dapat dipanggil langsung dari shell utama Termux dan tetap mengenali folder kerja aktif ($PWD).
⚡ Cara Cepat (One-Line Installer)
Jalankan perintah berikut langsung di terminal Termux:
curl -fsSL https://raw.githubusercontent.com/MasAim24/Android-Grafity/main/install.sh | bash


Setelah selesai, uji instalasi:
agy --version


🛠️ Langkah Manual (Step-by-Step)
Jika Anda ingin mengonfigurasinya langkah demi langkah, ikuti instruksi di bawah ini:
1. Update Termux & Pasang PRoot Distro
Pastikan paket Termux berada dalam kondisi mutakhir:
pkg update -y && pkg install -y proot-distro


2. Pasang Kontainer Ubuntu
Pasang rootfs Ubuntu resmi:
proot-distro install ubuntu


3. Masuk ke Ubuntu & Pasang Dependensi
Masuk ke sesi root Ubuntu:
proot-distro login ubuntu


Di dalam terminal Ubuntu (root@localhost:~#), jalankan:
apt update && apt install -y curl ca-certificates


4. Eksekusi Installer agy
Jalankan skrip instalasi resmi di dalam Ubuntu:
curl -fsSL https://antigravity.google/cli/install.sh | bash


Pastikan biner terpasang di /root/.local/bin/agy, lalu keluar dari Ubuntu:
exit


5. Buat Wrapper Script di Termux
Kembali ke shell utama Termux (~ $). Buat file wrapper di $PREFIX/bin/agy:
cat << 'EOF' > "$PREFIX/bin/agy"
#!/data/data/com.termux/files/usr/bin/bash
# Teruskan perintah ke PRoot Ubuntu dengan mempertahankan direktori kerja aktif ($PWD)
proot-distro login ubuntu --bind "$PWD" --work-dir "$PWD" -- /root/.local/bin/agy "$@"
EOF


Berikan hak akses eksekusi:
chmod +x "$PREFIX/bin/agy"


6. Verifikasi
Uji perintah langsung dari Termux:
agy --version


⚠️ Troubleshooting & Catatan Penting
Error attempted to run proot-distro in a proot session:
Penyebab: Perintah proot-distro dijalankan saat Anda masih berada di dalam Ubuntu atau di dalam sesi termux-chroot.
Solusi: Ketik exit untuk keluar dari kontainer terlebih dahulu sebelum membuat atau memanggil wrapper Termux.
Akses File & Direktori Kerja:
Wrapper ini menggunakan flag --bind "$PWD" --work-dir "$PWD". Artinya, jika Anda menjalankan agy di folder ~/project-saya, perintah tersebut akan otomatis membaca dan membuat file di folder tersebut.
Untuk mengizinkan akses ke galeri/penyimpanan internal HP, jalankan termux-setup-storage di Termux terlebih dahulu.

📄 Lisensi
MIT License - Bebas digunakan dan dimodifikasi untuk kebutuhan komunitas pengembang Android/Termux.
