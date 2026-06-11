# Panduan Deploy Laravel Inertia React TS di Hostinger (Versi Terbaru)

Dokumentasi ini dirancang khusus untuk pemula yang ingin men-deploy project Laravel + Inertia + React + TypeScript ke hosting Hostinger. Panduan ini mencakup setup Git otomatis, aktivasi environment Node.js, dan penanganan error umum.

---

## 📋 Daftar Isi

1. [Prasyarat](#prasyarat)
2. [Persiapan Project di Lokal](#persiapan-project-di-lokal)
3. [Deploy Otomatis via Git Hostinger](#deploy-otomatis-via-git-hostinger)
4. [Setup Environment Server (Node.js & NPM)](#setup-environment-server-nodejs--npm)
5. [Konfigurasi Pasca-Deploy & Fix Error 403](#konfigurasi-pasca-deploy--fix-error-403)
6. [Troubleshooting Umum](#troubleshooting-umum)

---

## Prasyarat

Sebelum memulai, pastikan Anda memiliki:

- ✅ **Akun Hostinger** dengan paket yang mendukung akses Terminal/SSH dan Git.
- ✅ **Repository GitHub** yang berisi project Laravel siap deploy.
- ✅ **Domain aktif** yang sudah terhubung ke hosting (contoh: `danzcode.site`).
- ✅ Pemahaman dasar terminal/command line.

---

## Persiapan Project di Lokal

Lakukan langkah ini di komputer Anda sebelum push ke GitHub agar server tidak perlu melakukan build berat.

### 1. Build Assets untuk Production

Hostinger shared hosting memiliki resource terbatas. Sebaiknya build aset React/TS di laptop, lalu upload hasilnya.

```bash
# Install dependencies
npm install

# Build production (menghasilkan folder public/build/)
npm run build
```

### 2. Commit dan Push ke GitHub

Pastikan folder public/build/ tidak ter-ignore oleh .gitignore jika Anda memilih strategi build lokal.

```bash
git add .
git commit -m "Siap deploy ke Hostinger"
git push origin main
```

## Deploy Otomatis via Git Hostinger

Fitur baru Hostinger memungkinkan deployment semudah Vercel/Netlify.

### Langkah 1: Masuk ke Menu Git

1. Login ke hPanel.
2. Pilih website target (danzcode.site).
3. Di sidebar kiri, cari menu Website > Tingkat Lanjut > Git.

### Langkah 2: Hubungkan GitHub

Anda akan melihat halaman "Deploy dari GitHub":

1. Klik tombol hitam "Lanjutkan dengan GitHub".
2. Authorize akses Hostinger ke akun GitHub Anda.
3. Pilih repository project Laravel Anda dari daftar (misal: projectDataSantri).
4. Pilih branch main atau master.
5. Klik Import/Deploy.

> 💡 Catatan: Proses ini akan meng-clone seluruh repository Anda ke dalam folder public_html di server. Tunggu hingga status berubah menjadi "Success".

## Setup Environment Server (Node.js & NPM)

Secara default, terminal Hostinger tidak mengenali perintah npm atau node. Anda harus mengaktifkannya secara manual.

### Opsi A: Menggunakan NVM (Direkomendasikan)

Cara ini paling stabil dan fleksibel.

1. Buka Terminal di hPanel (Menu: Tingkat Lanjut > Terminal).
2. Install NVM:

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
```

3. Aktifkan NVM (reload terminal):

```bash
source ~/.bashrc
# Jika error, coba: source ~/.bash_profile
```

4. Install Node.js (pilih versi LTS, misal 20):

```bash
nvm install 20
nvm use 20
nvm alias default 20
```

5. Verifikasi instalasi NVM dan Node.js:

```bash
node -v
npm -v
```

### Opsi B: Menggunakan Package Manager Bawaan Hostinger

Jika Anda tidak ingin menggunakan NVM, Anda bisa menggunakan package manager yang sudah terinstall di server (biasanya Node 14).

```bash
npm install
```

### Opsi B: Menggunakan Fitur Node.js hPanel

Jika NVM gagal, gunakan fitur bawaan:

1. Menu Tingkat Lanjut > Node.js.
2. Buat aplikasi baru (pilih versi 18, mode Production).
3. Salin command source /home/u.../nodevenv/.../bin/activate yang muncul.
4. Paste command tersebut di Terminal untuk mengaktifkan npm sementara.

## Konfigurasi Pasca-Deploy & Fix Error 403

Error 403 Forbidden terjadi karena web server mencoba mengakses root project, padahal Laravel mewajibkan akses ke folder public/. Karena menu "Document Root" sering tersembunyi di Hostinger versi baru, kita gunakan metode Symlink.

### Langkah 1: Akses Terminal

Masuk ke direktori domain Anda:

```bash
cd /home/u611762746/domains/danzcode.site/
```

### Langkah 2: Buat Symlink (Jembatan Folder)

Perintah ini akan membuat public_html menjadi "pintu masuk" yang langsung mengarah ke folder public milik Laravel.

```bash
# 1. Rename folder public_html bawaan hostinger (sebagai backup)
mv public_html public_html_backup

# 2. Buat symlink baru yang menunjuk ke folder public Laravel
ln -s public_html_backup/public public_html

# 3. Verifikasi (harus muncul panah ->)
ls -la public_html
```

Output yang diharapkan:

```bash
public_html -> public_html_backup/public
```

### Langkah 3: Setup File .env & Database

1. Buat file .env:

```bash
cp .env.example .env
```

2. Edit .env dengan kredensial Hostinger Anda:

```bash
APP_NAME=Laravel
APP_ENV=production
APP_KEY=
APP_DEBUG=false
APP_URL=https://danzcode.site

LOG_CHANNEL=stack
LOG_DEPRECATIONS=null
LOG_LEVEL=warning

DB_CONNECTION=mysql
DB_HOST=[IP_ADDRESS]
DB_PORT=3306
DB_DATABASE=db_name_anda
DB_USERNAME=db_user_anda
DB_PASSWORD=password_database_anda

BROADCAST_DRIVER=log
CACHE_DRIVER=file
QUEUE_CONNECTION=sync
SESSION_DRIVER=file
MEMCACHED_HOST=[IP_ADDRESS]
```

3. Import Database:

- Buka hPanel > Databases > phpMyAdmin (atau Database Manager).
- Pilih database yang sudah Anda buat.
- Pilih menu Import > Pilih file .sql dari komputer Anda > Go.

### Langkah 4: Install Dependencies & Run Migrations

Setelah NVM aktif (lihat Opsi A di atas):

```bash
# 1. Install dependencies (menghasilkan folder vendor/)
# Tambahkan --no-dev jika tidak perlu package development
npm install

# 2. Jalankan Migration
php artisan migrate --force

# 3. (Opsional) Import data awal
# php artisan db:seed --class=SomeSeederSeeder
```

### Langkah 5: Set Permission yang Benar

```bash
chmod -R 755 storage/
chmod -R 755 bootstrap/cache/
chown -R u611762746:u611762746 storage/
```

### Langkah 6: Cleanup (Optional)

Jika Anda menggunakan storage:publish, Anda perlu membuat symlink untuk storage dan public:

```bash
# 1. Hapus symlink public jika masih ada (agar tidak bentrok)
rm -f public

# 2. Buat symlink ke storage (jika menggunakan storage:link)
# Pastikan folder storage/app/public sudah ada dan diisi file
ln -s storage/app/public public_html/storage
```

## Troubleshooting Umum

| Masalah                     | Penyebab                             | Solusi                                             |
| --------------------------- | ------------------------------------ | -------------------------------------------------- |
| 403 Forbidden               | Web server salah membaca folder root | Lakukan langkah Symlink di atas                    |
| npm: command not found      | Node.js belum diaktifkan             | Lakukan Setup Environment Server                   |
| 500 Internal Server Error   | Permission atau .env salah           | Cek storage/logs/laravel.log                       |
| Vite manifest not found     | Asset belum di-build                 | Jalankan npm run build di lokal lalu push ulang    |
| Database Connection Refused | Kredensial DB salah                  | Pastikan user DB sudah dibuat di hPanel > Database |

```bash
npm: command not found
Node.js belum diaktifkan
Lakukan Setup Environment Server
500 Internal Server Error
Permission atau .env salah
Cek storage/logs/laravel.log
Vite manifest not found
Asset belum di-build
Jalankan npm run build di lokal lalu push ulang
Database Connection Refused
Kredensial DB salah
Pastikan user DB sudah dibuat di hPanel > Database
```

### ⚠️ Penting untuk Pemula

Jangan jalankan npm run dev di server. Gunakan hanya npm run build.
Setiap kali update kode React/TS, build di lokal dulu (npm run build), lalu push ke GitHub. Biarkan Hostinger menarik update via Git.
Simpan file .env dengan aman. Jangan pernah commit .env ke GitHub. Gunakan .env.example sebagai template.

### 💡 Penjelasan Tambahan untuk Anda:

1.  **Mengapa Symlink?**
    Di Hostinger versi terbaru, pengaturan "Document Root" seringkali disembunyikan di balik beberapa layer menu atau bahkan dihilangkan untuk paket tertentu. Metode `ln -s` (symlink) adalah cara "hacky" tapi sangat efektif dan permanen. Ini memberitahu sistem operasi: _"Hei, kalau ada yang cari folder `public_html`, sebenarnya arahkan mereka ke folder `public` di dalamnya."_

2.  **Kenapa Build di Lokal?**
    Shared hosting biasanya punya RAM terbatas (seringkali cuma 512MB - 1GB). Proses `npm install` dan `npm run build` untuk project React+TS bisa memakan RAM >1GB dan menyebabkan proses terbunuh (Killed/OOM). Dengan build di laptop, server hanya bertugas menyajikan file statis yang sudah jadi.

3.  **Struktur Folder Setelah Symlink:**
    Sebelum symlink: `danzcode.site/public_html/app`, `danzcode.site/public_html/public/index.php`
    Setelah symlink: Ketika browser minta `danzcode.site/index.php`, server membaca `public_html` -> diteruskan ke `public_html_backup/public/index.php`. Inilah yang membuat Laravel berjalan benar.

4.  **Jika terdapat error 403, coba buat symlink:**
    ```bash
    cd domains/danzcode.site/
    mv public_html public_html_old
    ln -s public_html_old/public public_html
    ```
Dokumentasi di atas sudah saya susun agar alurnya logis: **Persiapan -> Deploy -> Fix Environment -> Fix Routing -> Final Check**. Semoga membantu!
