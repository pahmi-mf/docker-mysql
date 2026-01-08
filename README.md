# MySQL 8.4 Docker Server

Docker setup untuk MySQL 8.4 yang dapat diakses dari localhost untuk digunakan dengan Laravel, DBeaver, dan tools MySQL lainnya.

## 📋 Prasyarat

Sebelum memulai, pastikan Anda telah menginstall:

- [Docker](https://docs.docker.com/get-docker/) (versi 20.10 atau lebih baru)
- [Docker Compose](https://docs.docker.com/compose/install/) (versi 2.0 atau lebih baru)

Cek versi Docker dan Docker Compose:
```bash
docker --version
docker compose version
```

**Catatan**: Dokumentasi ini menggunakan format `docker compose` (Docker Compose V2). Jika Anda masih menggunakan Docker Compose V1 (standalone), ganti semua perintah `docker compose` menjadi `docker-compose` (dengan hyphen).

## 🚀 Instalasi dan Setup

### 1. Clone atau Download Project

Jika project ini ada di repository Git:

```bash
git clone <repository-url>
cd mysql-server
```

Atau jika Anda sudah memiliki project ini, pastikan Anda berada di direktori project:

```bash
cd /path/to/mysql-server
```

### 2. Setup Environment Variables

Copy file `.env.example` menjadi `.env`:

```bash
cp .env.example .env
```

Edit file `.env` dan sesuaikan konfigurasi:

```bash
nano .env
```

Atau menggunakan editor favorit Anda (vim, code, dll).

Sesuaikan nilai-nilai berikut:

```env
MYSQL_ROOT_PASSWORD=your_secure_password_here
MYSQL_DATABASE=
MYSQL_PORT=3307
```

**Penting**: 
- Ganti `your_secure_password_here` dengan password yang aman untuk root MySQL
- `MYSQL_DATABASE` bisa dikosongkan jika ingin membuat database nanti
- `MYSQL_PORT` default adalah 3307, ubah jika port tersebut sudah digunakan

### 3. Jalankan MySQL Container

**Pastikan file `.env` sudah dibuat dan `MYSQL_PORT` sudah diset dengan benar!**

Jika ada container lama yang masih berjalan, hapus terlebih dahulu:

```bash
docker compose down
```

Jalankan container dalam mode detached (background):

```bash
docker compose up -d
```

Perintah ini akan:
- Download image MySQL 8.4 (jika belum ada)
- Membuat dan menjalankan container MySQL
- Bind port dari `.env` (default: 3307) ke port 3306 di dalam container
- Membuat volume untuk data persistence
- Menjalankan health check

**Catatan**: Port yang digunakan akan mengikuti `MYSQL_PORT` di file `.env`. Jika file `.env` tidak ada atau variable tidak terset, akan menggunakan default 3307.

### 4. Cek Status Container

Pastikan container berjalan dengan baik:

```bash
docker compose ps
```

Output yang diharapkan:
```
NAME           IMAGE       COMMAND                  SERVICE   CREATED         STATUS                    PORTS
mysql-server   mysql:8.4   "docker-entrypoint.s…"   mysql     X seconds ago   Up X seconds (healthy)   0.0.0.0:3307->3306/tcp
```

Status harus menunjukkan `healthy` setelah beberapa saat.

### 5. View Logs (Optional)

Untuk melihat logs container:

```bash
docker compose logs -f mysql
```

Tekan `Ctrl+C` untuk keluar dari log viewer.

## 🔌 Koneksi ke MySQL

### Connection Details

- **Host**: `localhost` atau `127.0.0.1`
- **Port**: `3307` (default)
- **User**: `root`
- **Password**: Sesuai dengan `MYSQL_ROOT_PASSWORD` di file `.env`
- **Database**: Buat database baru sesuai kebutuhan

### Koneksi dari Laravel

Tambahkan konfigurasi berikut di file `.env` Laravel:

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3307
DB_DATABASE=nama_database_anda
DB_USERNAME=root
DB_PASSWORD=password_dari_docker_.env
```

### Koneksi dari DBeaver

1. Buka DBeaver
2. Klik **New Database Connection** → Pilih **MySQL**
3. Isi form:
   - **Host**: `localhost`
   - **Port**: `3307`
   - **Database**: (kosongkan atau nama database yang sudah dibuat)
   - **Username**: `root`
   - **Password**: Password dari file `.env`
4. Klik **Test Connection** untuk memastikan koneksi berhasil
5. Klik **Finish**

### Koneksi dari MySQL Client

```bash
mysql -h 127.0.0.1 -P 3307 -u root -p
```

## 🛑 Menghentikan Docker Container

### Stop Container (Container tetap ada, bisa di-start kembali)

```bash
docker compose stop
```

Container akan berhenti tetapi data tetap tersimpan. Untuk menjalankan lagi:

```bash
docker compose start
```

### Stop dan Hapus Container (Data tetap tersimpan)

```bash
docker compose down
```

Perintah ini akan:
- Stop container
- Hapus container
- Data MySQL tetap tersimpan di `./mysql-data`

### Stop dan Hapus Container + Hapus Semua Data

**⚠️ PERHATIAN**: Perintah ini akan menghapus semua data MySQL!

```bash
docker compose down -v
```

Perintah ini akan:
- Stop container
- Hapus container
- Hapus volume dan semua data di `./mysql-data`

Gunakan hanya jika Anda yakin ingin menghapus semua data.

### Restart Container

Jika ingin restart container:

```bash
docker compose restart
```

## 📦 Mengelola Container

### Start Container yang sudah di-stop

```bash
docker compose start
```

### Restart Container

```bash
docker compose restart
```

### Melihat Logs Container

```bash
# Logs real-time
docker compose logs -f mysql

# Logs terakhir 100 baris
docker compose logs --tail=100 mysql

# Logs dengan timestamp
docker compose logs -t mysql
```

### Masuk ke Container MySQL

```bash
docker compose exec mysql bash
```

### Cek Status Container

```bash
docker compose ps
```

## Backup & Restore

### Backup Database

```bash
# Backup semua database
docker compose exec mysql mysqldump -u root -p${MYSQL_ROOT_PASSWORD} --all-databases > backup_$(date +%Y%m%d_%H%M%S).sql

# Backup database spesifik
docker compose exec mysql mysqldump -u root -p${MYSQL_ROOT_PASSWORD} nama_database > backup_nama_database_$(date +%Y%m%d_%H%M%S).sql
```

### Restore Database

```bash
# Restore dari file backup
docker compose exec -T mysql mysql -u root -p${MYSQL_ROOT_PASSWORD} < backup_file.sql

# Restore database spesifik
docker compose exec -T mysql mysql -u root -p${MYSQL_ROOT_PASSWORD} nama_database < backup_file.sql
```

## Membuat Database & User Baru

Masuk ke MySQL container:

```bash
docker compose exec mysql mysql -u root -p${MYSQL_ROOT_PASSWORD}
```

Atau menggunakan MySQL client lokal:

```bash
mysql -h 127.0.0.1 -P 3307 -u root -p
```

Kemudian jalankan SQL:

```sql
CREATE DATABASE nama_database CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'nama_user'@'%' IDENTIFIED BY 'password_user';
GRANT ALL PRIVILEGES ON nama_database.* TO 'nama_user'@'%';
FLUSH PRIVILEGES;
```

## Troubleshooting

### Docker daemon tidak berjalan

Jika Anda mendapatkan error:
```
Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
```

**Solusi:**

1. **Linux**: Start Docker service
   ```bash
   sudo systemctl start docker
   sudo systemctl enable docker  # Untuk auto-start saat boot
   ```

2. **Cek status Docker service**
   ```bash
   sudo systemctl status docker
   ```

3. **Jika menggunakan Docker Desktop**: Pastikan Docker Desktop sudah running

4. **Jika masih error**: Tambahkan user ke docker group (untuk Linux)
   ```bash
   sudo usermod -aG docker $USER
   # Logout dan login kembali, atau:
   newgrp docker
   ```

### Container tidak bisa start

1. Cek apakah port 3307 sudah digunakan:
   ```bash
   sudo lsof -i :3307
   ```

2. Ubah port di file `.env` jika diperlukan:
   ```env
   MYSQL_PORT=3308
   ```

3. Cek logs untuk error:
   ```bash
   docker compose logs mysql
   ```

### Lupa password root

1. Stop container:
   ```bash
   docker compose stop
   ```

2. Hapus data volume (HATI-HATI, ini akan menghapus semua data):
   ```bash
   docker compose down -v
   ```

3. Edit file `.env` dengan password baru
4. Start ulang container:
   ```bash
   docker compose up -d
   ```

### Health Check Failed

Jika health check gagal, tunggu beberapa saat karena MySQL butuh waktu untuk inisialisasi. Atau cek logs:

```bash
docker compose logs -f mysql
```

## Persistensi Data

Data MySQL disimpan di directory `./mysql-data` di project root. Directory ini di-mount ke `/var/lib/mysql` di dalam container.

**Penting**: Directory `mysql-data` sudah di-ignore oleh git untuk menghindari commit data sensitif.

## Versi MySQL

MySQL versi 8.4 menggunakan official image dari Docker Hub. Untuk menggunakan versi spesifik, edit `docker compose.yml`:

```yaml
image: mysql:8.4.0
```

Lihat [MySQL Docker Hub](https://hub.docker.com/_/mysql) untuk versi yang tersedia.

## 📝 Quick Reference

### Perintah Penting

| Perintah | Deskripsi |
|----------|-----------|
| `docker compose up -d` | Jalankan MySQL container |
| `docker compose stop` | Stop container (data tetap tersimpan) |
| `docker compose start` | Start container yang sudah di-stop |
| `docker compose restart` | Restart container |
| `docker compose down` | Stop dan hapus container (data tetap) |
| `docker compose down -v` | Stop dan hapus container + semua data |
| `docker compose ps` | Lihat status container |
| `docker compose logs -f mysql` | Lihat logs real-time |
| `docker compose exec mysql mysql -u root -p` | Masuk ke MySQL CLI |

### Connection Details

- **Host**: `localhost` atau `127.0.0.1`
- **Port**: `3307` (default, atau sesuai `MYSQL_PORT` di `.env`)
- **User**: `root`
- **Password**: Sesuai `MYSQL_ROOT_PASSWORD` di `.env`

### File Penting

- `docker compose.yml` - Konfigurasi Docker Compose
- `.env` - Environment variables (buat dari `.env.example`)
- `.env.example` - Template environment variables
- `mysql-data/` - Directory untuk data persistence (auto-created)
