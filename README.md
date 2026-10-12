<div align="center">

# 🔐 sultrakey

**File `.env` terenkripsi dan launcher yang menjalankan aplikasi apa pun dengan nilai yang sudah dibuka.**

Node.js · PHP · CodeIgniter · Spring Boot · Go · Rust · apa saja yang membaca environment variable

[![Release](https://img.shields.io/github/v/release/banksultra/sultrakey-releases?color=blue)](https://github.com/banksultra/sultrakey-releases/releases/latest)
[![License: MIT](https://img.shields.io/badge/license-MIT-yellow.svg)](LICENSE)
[![Encryption: age](https://img.shields.io/badge/encryption-age%20X25519-green)](https://age-encryption.org)
[![Platforms](https://img.shields.io/badge/platform-Linux%20%7C%20Windows%20%7C%20macOS-lightgrey)](https://github.com/banksultra/sultrakey-releases/releases/latest)

<br>

<img src="images/Screenshot.png" alt="Layar setup sultrakey" width="800">

<br>

[Pasang](#pasang) ·
[Mulai cepat](#mulai-cepat) ·
[Template](#template-envexample) ·
[Perintah](#perintah) ·
[Per bahasa](#cara-pakai-per-bahasa) ·
[pm2](#pm2) ·
[Docker](#docker) ·
[Update](#update)

</div>

---

Repository ini hanya berisi file rilis. Source code-nya privat.

## Kenapa sultrakey

| | |
| --- | --- |
| 📦 **Satu binary statis** | Tanpa dependensi. Jalan di Rocky Linux 8/9, Ubuntu, CentOS 7, serta Windows dan macOS. |
| 🛡️ **Enkripsi standar** | [age](https://age-encryption.org) X25519, tanpa kriptografi buatan sendiri. |
| 🔌 **Aplikasi tidak diubah** | Aplikasi membaca environment variable seperti biasa. |

## Pasang

Windows (PowerShell, tidak perlu Administrator):

```powershell
powershell -ExecutionPolicy Bypass -c "irm https://github.com/banksultra/sultrakey-releases/releases/latest/download/install.ps1 | iex"
```

Linux / macOS:

```sh
curl -fsSL https://github.com/banksultra/sultrakey-releases/releases/latest/download/install.sh | sh        # laptop
curl -fsSL https://github.com/banksultra/sultrakey-releases/releases/latest/download/install.sh | sudo sh   # server → /usr/bin
```

Versi tertentu: `... | SULTRAKEY_VERSION=0.6.1 sh` (Windows: jalankan `$env:SULTRAKEY_VERSION = '0.6.1'` dulu).
Rilis dimulai dari 0.6.1.

Cek hasilnya: `sultrakey --version`.

## Mulai cepat

1. Repo aplikasi punya `.env.example` (semua variabel, nilainya kosong atau default yang bukan rahasia).
2. Tambahkan `.env` dan `.env.lock` ke `.gitignore`. Jangan pakai pola `.env*`, karena `.env.example` ikut terabaikan.
3. `sultrakey init <app>` membuat kunci dan `.env` dari template.
4. `sultrakey setup` menanyakan variabel yang masih kosong, lalu menyimpannya terenkripsi.
5. `sultrakey check` memastikan semua variabel terisi dan bisa dibuka.
6. `sultrakey run -- <perintah>` membuka nilai-nilainya lalu menjalankan aplikasi.

Windows:

```powershell
cd C:\projects\contoh-app
sultrakey init contoh-app
sultrakey setup
sultrakey run -- npm run dev
```

Linux / macOS (membuat kunci butuh sudo, laptop juga; kunci disimpan di `/etc/sultrakey/`):

```sh
sudo sultrakey init contoh-app --owner $USER
sultrakey setup
sultrakey run -- npm run dev
```

Nama aplikasi: huruf kecil, angka, dan tanda strip.

## Template (`.env.example`)

```env
# Port aplikasi
# @plain
PORT=8899
# Host Redis, tanpa http:// dan port. Contoh: 10.10.1.20
REDIS_HOST=
# Password dari admin Redis. Kosongkan bila tidak ada.
# @optional
REDIS_PASSWORD=
```

- Komentar di atas variabel menjadi keterangannya di layar `setup`.
- Tanpa anotasi, variabel dianggap **rahasia dan wajib**: dienkripsi, dan harus terisi sebelum `run`.
- Nama variabel hanya boleh huruf, angka, dan `_`, serta tidak boleh diawali angka. Nama bertitik seperti
  `database.default.hostname` (gaya CodeIgniter) tidak bisa dipakai. Tulis `DB_HOST`.
- Anotasi ditulis di komentar tepat di atas variabel:

| Anotasi       | Akibat                                                                          |
| ------------- | ------------------------------------------------------------------------------- |
| `# @plain`    | Disimpan apa adanya, tidak dienkripsi. Untuk port, level log, dan non-rahasia.  |
| `# @optional` | Boleh kosong. Aplikasi menerima nilai kosong.                                   |
| `# @masking`  | Diketik sebagai `*` dan dimasukkan dua kali. Untuk rahasia bernama biasa (`DATABASE_URL`). |

- Nilai di template menjadi default untuk variabel biasa. Untuk variabel rahasia, nilainya diabaikan dan harus diketik.
- Kosongkan nilai contoh seperti `SMTP_HOST=smtp.example.com`. Kalau tidak, contoh itu tersimpan sebagai nilai asli.
- Nilai panjang (sertifikat): di `setup` ketik `@/path/ke/file`, atau jalankan `sultrakey set KEY --file <path>`.

## Perintah

| Perintah                            | Fungsi                                                                      |
| ----------------------------------- | --------------------------------------------------------------------------- |
| `init <app> [--owner user[:group]]` | Membuat kunci (tidak pernah menimpa) dan `.env`. Bisa mengenkripsi `.env` lama yang polos. |
| `setup`                             | Mengisi variabel kosong dalam satu layar. F1 = bantuan, Esc = batal.        |
| `set <KEY> [--stdin \| --file F]`   | Mengganti satu nilai. Nilai tidak pernah diambil dari argumen.              |
| `list`                              | Menampilkan nama variabel dan statusnya. Nilai tidak pernah ditampilkan.    |
| `check`                             | Memeriksa semua terisi, bisa dibuka, dan kunci cocok.                       |
| `run [--env F] -- <cmd> [args]`     | Menjalankan `check`, lalu menjalankan aplikasi dengan nilainya.             |
| `install` / `update` / `uninstall`  | Mengelola binary sultrakey sendiri.                                         |

Opsi global: `--env` (bawaan `./.env`), `--template` (bawaan `./.env.example`), `--key-file`.

Kunci dicari berurutan di:

1. `--key-file`
2. `SULTRAKEY_KEY_FILE`
3. `$CREDENTIALS_DIRECTORY/sultrakey.key` (systemd `LoadCredential=`)
4. `/etc/sultrakey/<app>.key` (Windows: `%APPDATA%\sultrakey\<app>.key`)

Kode exit: `0` berhasil, `64` salah pemakaian, `78` masalah konfigurasi, `1` lainnya.
Kode `78` dipakai pm2 dan systemd untuk berhenti me-restart aplikasi.

## Cara kerja `run`

- Aplikasi **tidak perlu diubah**. Ia membaca environment variable seperti biasa.
- Linux/macOS: sultrakey mengganti dirinya dengan proses aplikasi (`exec`). PID, sinyal stop, dan exit code langsung ke aplikasi.
- Windows: aplikasi jalan sebagai proses anak. Ctrl+C sampai ke aplikasi. Bila terminal ditutup paksa, aplikasi ikut berhenti.
- Nilai dari `.env` menang atas environment yang sudah ada. Bila bentrok, muncul peringatan berisi nama variabel saja.
- Variabel `@optional` yang kosong dikirim sebagai string kosong.
- Variabel berawalan `SULTRAKEY_` tidak diteruskan ke aplikasi.

## Cara pakai per bahasa

Pola yang sama untuk semua: tulis `sultrakey run --` di depan perintah start aplikasi.

### Node.js

```sh
sultrakey run -- node dist/main.js
sultrakey run -- npm run dev
```

```js
const host = process.env.REDIS_HOST;
```

- Pakai `dotenv` biasa (tanpa `override: true`) boleh. Dengan `override: true`, aplikasi membaca teks `enc:...`.
- SvelteKit (adapter-node): `import { env } from '$env/dynamic/private'`, lalu `env.REDIS_HOST`. Jangan pakai `$env/static/*`.
- Vite `import.meta.env` dan Next.js `NEXT_PUBLIC_*` dibaca saat build. Nilainya masuk ke hasil build dan bisa sampai ke browser. Jangan taruh rahasia di sana.

### PHP

Variabel masuk lewat environment, jadi baca dengan `getenv()`.

```sh
sultrakey run -- php artisan serve     # Laravel
sultrakey run -- php -S 0.0.0.0:8000 -t public
```

```php
$host = getenv('REDIS_HOST');
```

- `$_ENV` hanya terisi bila `variables_order` di `php.ini` memuat `E` (contoh: `"EGPCS"`). `getenv()` selalu bisa.
- PHP-FPM: jalankan lewat sultrakey, dan matikan `clear_env`:

  ```sh
  sultrakey run -- php-fpm -F
  ```

  ```ini
  ; pool php-fpm (www.conf)
  clear_env = no
  ```

  Tanpa `clear_env = no`, PHP-FPM membuang semua variabel dan aplikasi tidak menerima apa pun.
- Apache `mod_php` dan nginx + FPM yang sudah jalan sebagai service tidak lewat sultrakey. Bungkus prosesnya
  (php-fpm) seperti di atas, atau jalankan lewat systemd/Docker.

### CodeIgniter 4

```sh
sultrakey run -- php spark serve
sultrakey run -- php-fpm -F        # produksi, dengan clear_env = no
```

Template `.env.example` memakai nama bergaya underscore, bukan titik:

```env
# @plain
CI_ENVIRONMENT=production
DB_HOST=
DB_NAME=
DB_USER=
DB_PASS=
```

Baca dengan `env()` di `app/Config/Database.php` (isi di constructor):

```php
public function __construct()
{
    parent::__construct();
    $this->default['hostname'] = env('DB_HOST');
    $this->default['database'] = env('DB_NAME');
    $this->default['username'] = env('DB_USER');
    $this->default['password'] = env('DB_PASS');
}
```

- CI4 juga membaca file `.env` miliknya sendiri. File itu adalah `.env` dari sultrakey, jadi isinya `enc:...`.
  CI4 tidak menimpa variabel yang sudah ada di environment, jadi nilai dari sultrakey yang dipakai. Aman selama aplikasi
  dijalankan lewat `sultrakey run`.
- Jangan jalankan CI4 tanpa sultrakey memakai `.env` ini. Aplikasi akan membaca `enc:...` sebagai nilainya.

### Spring Boot

```sh
sultrakey run -- java -jar app.jar
sultrakey run -- mvnw.cmd spring-boot:run     # Windows
sultrakey run -- ./mvnw spring-boot:run       # Linux/macOS
```

```properties
# application.properties
spring.data.redis.host=${REDIS_HOST}
spring.data.redis.password=${REDIS_PASSWORD:}
```

- Spring Boot otomatis membaca environment variable. `REDIS_HOST` juga cocok dengan `spring.data.redis.host` lewat relaxed binding.
- `${REDIS_PASSWORD:}` = nilai bawaan kosong, cocok untuk variabel `@optional`.

### Golang

```sh
sultrakey run -- go run .
sultrakey run -- ./app
```

```go
host := os.Getenv("REDIS_HOST")
```

### Rust

```sh
sultrakey run -- cargo run
sultrakey run -- ./target/release/app
```

```rust
let host = std::env::var("REDIS_HOST")?;
```

Untuk `cargo run`, nilai ikut ke aplikasi. Untuk memeriksa tanpa menjalankan aplikasi: `sultrakey check`.

## pm2

Syarat: Linux, aplikasi jalan sebagai user sendiri (bukan root), `sultrakey` terpasang di `/usr/bin` (pasang dengan `sudo`).

Simpan sebagai `ecosystem.config.js` di folder aplikasi:

```js
module.exports = {
  apps: [{
    name: 'contoh-app',
    cwd: '/opt/contoh-app',
    script: '/usr/bin/sultrakey',
    args: ['run', '--', '/usr/bin/node', 'dist/main.js'],
    interpreter: 'none',
    exec_mode: 'fork',
    instances: 1,
    autorestart: true,
    stop_exit_codes: [78],
    max_restarts: 10,
    restart_delay: 3000,
  }]
};
```

| Baris                               | Alasan                                                                 |
| ----------------------------------- | ---------------------------------------------------------------------- |
| `script: '/usr/bin/sultrakey'`      | pm2 menjalankan sultrakey, bukan node langsung.                        |
| `interpreter: 'none'`               | sultrakey bukan file JavaScript. Tanpa ini pm2 gagal menjalankannya.   |
| `exec_mode: 'fork'`, `instances: 1` | Mode cluster tidak bisa dipakai.                                       |
| `cwd`                               | Folder tempat `.env` berada.                                           |
| `stop_exit_codes: [78]`             | Kalau konfigurasi salah (password kosong, kunci salah), pm2 berhenti, bukan restart terus. |

Jalankan sebagai user aplikasi:

```sh
sudo -u contoh-app -H pm2 start /opt/contoh-app/ecosystem.config.js
sudo -u contoh-app -H pm2 logs contoh-app --lines 50
```

Nyala otomatis setelah reboot:

```sh
sudo pm2 startup systemd -u contoh-app --hp /home/contoh-app
sudo -u contoh-app -H pm2 save
```

Contoh untuk bahasa lain: ganti isi `args` setelah `'--'`, misalnya `['run', '--', '/usr/bin/java', '-jar', 'app.jar']`
atau `['run', '--', './app']`.

Jangan lakukan:

- **Jangan** `sultrakey run -- pm2 start ...`. pm2 menyimpan nilai yang sudah dibuka, dan `pm2 save` menulisnya polos ke `~/.pm2/dump.pm2`.
- **Jangan** menaruh password di blok `env:` file pm2.
- **Jangan** memakai `exec_mode: 'cluster'`.
- **Jangan** menjalankan pm2 sebagai root untuk aplikasi ini.

## Docker

Kunci dan `.env` tetap di server dan di-mount saat container jalan. Keduanya tidak pernah masuk image.

### 1. Siapkan di server (host)

Buat user dengan uid tetap, lalu buat kunci atas nama user itu:

```sh
sudo useradd --system --create-home --uid 1001 --shell /sbin/nologin contoh-app
cd /opt/contoh-app
sudo sultrakey init contoh-app --owner contoh-app
sudo sultrakey setup
```

### 2. Masukkan binary ke image

Unduh `sultrakey-x86_64-unknown-linux-musl` dari [rilis terbaru](https://github.com/banksultra/sultrakey-releases/releases/latest)
(ARM: `sultrakey-aarch64-unknown-linux-musl`), taruh di folder build. Binary statis, jalan di Alpine, Debian, Ubuntu, dan distroless.

`Dockerfile` (contoh Node.js):

```dockerfile
FROM node:22-alpine
WORKDIR /app

COPY sultrakey-x86_64-unknown-linux-musl /usr/local/bin/sultrakey
RUN chmod 755 /usr/local/bin/sultrakey

COPY . .
RUN npm ci --omit=dev

USER 1001
ENTRYPOINT ["/usr/local/bin/sultrakey", "run", "--"]
CMD ["node", "dist/main.js"]
```

Untuk bahasa lain, ganti `FROM` dan `CMD`. Misalnya `CMD ["java", "-jar", "app.jar"]`, `CMD ["./app"]`,
atau `CMD ["php-fpm", "-F"]` (dengan `clear_env = no`).

`.dockerignore`:

```
.env
.env.lock
```

`ENTRYPOINT` harus berbentuk array seperti di atas. Bentuk teks biasa menjalankan shell di depan, sehingga sinyal stop tidak sampai ke aplikasi.

### 3. Jalankan

`docker-compose.yml`:

```yaml
services:
  contoh-app:
    image: contoh-app:latest
    init: true
    user: "1001:1001"
    working_dir: /app
    volumes:
      - /opt/contoh-app/.env:/app/.env:ro
      - /etc/sultrakey/contoh-app.key:/etc/sultrakey/contoh-app.key:ro
    restart: "on-failure:5"
```

```sh
docker compose up -d
docker compose logs -f contoh-app
```

Atau dengan `docker run`:

```sh
docker run -d --name contoh-app --init --user 1001:1001 -w /app \
  -v /opt/contoh-app/.env:/app/.env:ro \
  -v /etc/sultrakey/contoh-app.key:/etc/sultrakey/contoh-app.key:ro \
  --restart on-failure:5 \
  contoh-app:latest
```

Periksa dari dalam container:

```sh
docker compose run --rm --entrypoint /usr/local/bin/sultrakey contoh-app check
```

Hasil yang benar: `✓ All good`.

Mengganti password: jalankan `sudo sultrakey set KEY` di **host** (folder `/opt/contoh-app`), lalu `docker compose restart contoh-app`.

### Masalah yang sering muncul

| Gejala                                         | Penyebab dan perbaikan                                                                                  |
| ---------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| `No permission to read key file`               | uid container beda dengan pemilik file kunci. Samakan uid, atau `sudo chown 1001 /etc/sultrakey/contoh-app.key`. |
| Sama, padahal uid sudah sama (Rocky/RHEL)      | SELinux. Tambahkan `,z` di akhir volume: `...:/app/.env:ro,z`.                                          |
| `Key file ... is too open`                     | `sudo chmod 400 /etc/sultrakey/contoh-app.key`                                                          |
| `File .env not found`                          | Volume `.env` belum dipasang, atau `working_dir` salah.                                                 |
| Container restart lalu berhenti                | Konfigurasi salah (exit 78). Lihat `docker compose logs` dan ikuti pesan `Fix:`.                        |

Docker tidak bisa mengecualikan exit code 78, jadi `on-failure:5` dipakai agar restart tidak berjalan tanpa henti.

## Update

```sh
sultrakey update --check     # hanya cek; exit 1 bila ada versi baru
sultrakey update             # server: sudo sultrakey update
```

Terpasang versi 0.6.0 atau lebih lama? Jalankan perintah pasang di atas sekali. Kunci dan file `.env` tidak disentuh.

## Pasang manual (server tanpa internet)

1. Dari [rilis terbaru](https://github.com/banksultra/sultrakey-releases/releases/latest), unduh binary untuk server
   (`sultrakey-x86_64-unknown-linux-musl`, ARM: `sultrakey-aarch64-unknown-linux-musl`) dan `SHA256SUMS`.
2. Salin keduanya ke server, lalu cek binary. Harus tercetak `OK`:

   ```sh
   grep 'sultrakey-x86_64-unknown-linux-musl$' SHA256SUMS | sha256sum -c -
   ```

3. Pasang:

   ```sh
   chmod +x sultrakey-x86_64-unknown-linux-musl
   sudo ./sultrakey-x86_64-unknown-linux-musl install
   ```

## Darurat: membuka nilai tanpa sultrakey

Butuh CLI `age`.

```sh
echo '<teks base64 setelah enc:>' | base64 -d | age -d -i /etc/sultrakey/<app>.key
```

Kehilangan kunci berarti semua nilai harus diisi ulang. Simpan cadangan kunci di tempat offline.

## File di setiap rilis

| File                        | Fungsi                         |
| --------------------------- | ------------------------------ |
| `sultrakey-<target>[.exe]`  | binary, satu per sistem        |
| `SHA256SUMS`                | checksum semua file            |
| `manifest.json`             | dibaca oleh `sultrakey update` |
| `install.sh`, `install.ps1` | skrip pasang di atas           |

## Lisensi

[MIT](LICENSE) © 2026 Rede
