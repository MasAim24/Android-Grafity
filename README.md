#!/data/data/com.termux/files/usr/bin/bash
set -eecho "==> [1/5] Memeriksa dependensi Termux..."
if [ -z "$PREFIX" ]; then
echo "[!] Error: Skrip ini harus dijalankan di dalam lingkungan Termux Android."
exit 1
fipkg update -y
pkg install -y proot-distro curl ca-certificatesecho "==> [2/5] Memeriksa instalasi Ubuntu di proot-distro..."
if ! proot-distro list | grep -q "ubuntu"; then
echo "--> Memasang rootfs Ubuntu..."
proot-distro install ubuntu
else
echo "--> Ubuntu sudah terpasang, melanjutkan..."
fiecho "==> [3/5] Memasang dependensi di dalam kontainer Ubuntu..."
proot-distro login ubuntu -- apt update -y
proot-distro login ubuntu -- apt install -y curl ca-certificatesecho "==> [4/5] Menginstal Google Antigravity CLI (agy)..."
proot-distro login ubuntu -- bash -c "curl -fsSL https://antigravity.google/cli/install.sh | bash"echo "==> [5/5] Membuat wrapper script di Termux..."
cat << 'EOF' > "$PREFIX/bin/agy"
#!/data/data/com.termux/files/usr/bin/bash
proot-distro login ubuntu --bind "$PWD" --work-dir "$PWD" -- /root/.local/bin/agy "$@"
EOFchmod +x "$PREFIX/bin/agy"echo ""
echo "="
echo " 🎉 Instalasi Berhasil!"
echo "="
echo "Verifikasi versi instalasi:"
agy --version
echo ""
echo "Ketik 'agy --help' untuk mulai menggunakan."
