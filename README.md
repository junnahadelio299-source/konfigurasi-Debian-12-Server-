<div align="center">
  <h1 align="center">Debian 12 Server</h3>
  <p align="center">
    Panduan Latihan untuk membuat server debian 12
  </p>
</div>

## Daftar Isi

1. [Penginstalan Debian 12](#1.Persiapan-penginstalan-debian)
2. [File Server](#2-File-Server)
3. [Web Server](#3.Web-Server)
4. [Mail Server](#4.Mail-Server)

---

## 1.Persiapan penginstalan debian

### 1.1 Sistem Minimal yang Dibutuhkan

| Komponen | Spesifikasi Minimal | Spesifikasi Rekomendasi |
|----------|-------------------|------------------------|
| Prosesor | Intel Core 2 Duo / AMD equivalent | Intel Core i3 / AMD Ryzen 3 |
| RAM | 512 MB | 2 GB atau lebih |
| Disk | 5 GB | 20 GB atau lebih |
| Koneksi | Internet stabil | Internet stabil (untuk update) |

### 1.2 Persiapan Sebelum Instalasi

#### A. Media Instalasi
1. **Unduh ISO Debian 12**
   - Kunjungi: https://www.debian.org/distrib/
   - Pilih versi `debian-12-x.x-amd64-DVD-1.iso` (DVD) atau `debian-12-x.x-amd64-netinst.iso` (netinstall)
   - Ukuran: ~3GB (netinst) atau ~7GB (DVD full)

2. **Buat Bootable USB**
   - Gunakan tool seperti Rufus, Etcher, atau dd command:
   ```bash
   # Contoh dengan dd
   sudo dd if=debian-12-x.x-amd64-netinst.iso of=/dev/sdX bs=4M status=progress
   ```

#### B. Backup Data
- Backup semua data penting sebelum instalasi
- Jika dual-boot, backup tabel partisi

#### C. Informasi yang Diperlukan
- `Hostname` untuk server
- `Password root` yang kuat
- `Partisi` yang diinginkan:
  - `/` (root) - ~10-20 GB
  - `swap` - sama dengan RAM (jika RAM < 8GB)
  - `/home` - untuk data user
  - `/var` - untuk log dan database
  - `/tmp` - untuk file sementara

### 1.3 Langkah-Langkah Instalasi Debian 12

#### 1. Booting
1. Colokkan USB bootable ke komputer
2. Masuk ke BIOS/UEFI (biasanya tombol F2, F12, Del, atau Esc)
3. Atur boot priority ke USB first
4. Simpan dan restart

#### 2. Pilih Bahasa dan Lokasi
- Pilih bahasa: `English`
- Pilih lokasi: `Indonesia` atau `United States`
- Pilih layout keyboard: `English (US)` atau `Indonesian`

#### 3. Konfigurasi Jaringan (Jika Menggunakan DHCP)
- Jika menggunakan IP statis, pilih `Configure network manually`
- Isikan:
  - IP Address: `192.168.1.x`
  - Netmask: `255.255.255.0`
  - Gateway: `192.168.1.1`
  - DNS Server: `8.8.8.8` (Google) atau `1.1.1.1` (Cloudflare)

#### 4. Pengaturan Hostname dan Domain
- Hostname: `debian12`
- Domain: `local` (atau nama domain Anda)

#### 5. Pengaturan Password Root
- Masukkan password root yang kuat
- Minimal 8 karakter dengan kombinasi huruf besar, kecil, angka, dan simbol

#### 6. Buat User Biasa
- Nama lengkap: `admin`
- Username: `admin`
- Password: `xxxxxx`

#### 7. Partisi Disk
Pilih salah satu metode:

**Opsi A: Guided - Use entire disk**
- Pilih disk yang akan digunakan
- Pilih `All files in one partition` ( Beginners)
- Atau `Separate /home, /var, /tmp` (Advanced)

**Opsi B: Manual**
- Buat partisi sendiri sesuai kebutuhan

#### 8. Tunggu Instalasi Selesai
- Tunggu hingga proses copying selesai
- Proses ini memakan waktu 5-15 menit

#### 9. Konfigurasi Apt Mirror
- Pilih mirror terdekat:
  - Indonesia: `http://kartolo.sby.datautama.net.id/debian/`
  - Atau gunakan mirror default

#### 10. Apakah Anda ingin berpartisipasi dalam survey?
- Pilih `No` atau `Yes` sesuai kebutuhan

#### 11. Selesai
- Restart komputer
- Login dengan user yang dibuat

### 1.4 Hal yang Harus Dilakukan Setelah Instalasi

```bash
# Update sistem
sudo apt update && sudo apt upgrade -y

# Install sudo (jika belum ada)
su -
apt install sudo

# Tambahkan user ke grup sudo
usermod -aG sudo admin

# Install OpenSSH Server
sudo apt install openssh-server

# Konfigurasi SSH
sudo nano /etc/ssh/sshd_config
# Ubah port default (opsional)
# Larang login root: PermitRootLogin no

# Restart SSH
sudo systemctl restart ssh

# Atur firewall dasar
sudo apt install ufw
sudo ufw allow OpenSSH
sudo ufw enable

# Set timezone
sudo timedatectl set-timezone Asia/Jakarta

# Install useful tools
sudo apt install curl wget git htop net-tools
```
## 2. File Server

Panduan ini membahas cara setup file server dengan Samba di Debian 12.

### 2.1 Install Samba

```bash
su -
apt update
apt install -y samba smbclient cifs-utils
```

### 2.2 Buat Folder File Server

```bash
mkdir -p /home/share
```
lalu beri izin agar semua orang bisa mengaksesnya
```bash 
chown -R nobody:nogroup /home/share
chmod -R 777 /home/share
```

### 2.3 Konfigurasi Samba

```bash
cp /etc/samba/smb.conf /etc/samba/smb.conf.bak
nano /etc/samba/smb.conf
```

Isi konfigurasi:

```ini
[global]
   workgroup = WORKGROUP
   server string = File Server
   netbios name = FILESERVER
   security = user
   map to guest = bad user
   create mask = 0775
   directory mask = 0775

[share]
   path = /home/nanzz/fileserver
   browseable = yes
   read only = no
   guest ok = yes
   force user = nobody
```

### 2.4 Aktifkan Service

```bash
systemctl enable smbd nmbd
systemctl start smbd nmbd
systemctl restart smbd
```

### 2.5 Tambah User (Opsional)

```bash
useradd -M -s /sbin/nologin namauser
smbpasswd -a namauser
smbpasswd -e namauser
pdbedit -L
```

### 2.6 Cara Mengakses

| OS | Cara |
|----|------|
| Windows | `\\10.25.10.51` |
| Mac | `smb://10.25.10.51` |
| Linux | `smb://10.25.10.51` |

untuk android bisa menggunakan aplikasi file manager bawaan atau yang mendukung jaringan lan seperti Cx File explorer

### 2.7 Troubleshooting

untuk mengecek apakah service samba berjalan
```bash
systemctl status smbd
```

untuk mengecek apakah port samba bisa diakses 

```bash 
ss -tlnp | grep -E ":445|:139"
```
untuk mengecek log file
```bash 
tail -f /var/log/samba/log.smbd
```
untuk memulai ulang service samba 
```bash 
systemctl restart smbd
```

### 2.8 Perintah Penting

| Perintah | Fungsi |
|---------|-------|
| `testparm` | Test konfigurasi |
| `smbpasswd -a user` | Tambah user |
| `smbclient -L localhost` | List share |

## 3.Web Server

Panduan setup web server dengan Nginx & Apache di Debian 12.

### 3.1 Install Nginx

```bash
su -
apt update
apt install -y nginx
```

### 3.2 Konfigurasi Nginx

```bash
nano /etc/nginx/nginx.conf
```

### 3.3 Start Service

```bash
systemctl enable nginx
systemctl start nginx
systemctl status nginx
```

### 3.4 Buat Virtual Host

```bash
mkdir -p /var/www/example.com
nano /etc/nginx/sites-available/example.com
ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/
nginx -t
systemctl restart nginx
```

## 4.CTFd (Capture The Flag Platform)

Panduan install CTFd dari GitHub dengan Docker.

### 4.1 Persiapan Sistem

| Komponen | Minimal | Rekomendasi |
|----------|---------|------------|
| OS | Ubuntu 20.04+ / Debian 12 | Ubuntu 22.04 |
| RAM | 2 GB | 4 GB |
| CPU | 2 core | 4 core |
| Disk | 20 GB | 40 GB |
| Docker | 20.10+ | Latest |

### 4.2 Install Docker

```bash
# Update & install dependencies
apt update
apt install -y ca-certificates curl gnupg lsb-release

# Add Docker GPG key
install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
chmod a+r /etc/apt/keyrings/docker.gpg

# Add Docker repository
echo "deb [arch=$(dpkg --print-architecture) https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker
apt update
apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Enable & start Docker
systemctl enable docker
systemctl start docker

# Add user to docker group
usermod -aG docker admin
```

### 4.3 Install Git & Clone CTFd

```bash
# Install git
apt install -y git

# Clone CTFd dari GitHub
cd /opt
git clone https://github.com/CTFd/CTFd.git
cd CTFd

# Checkout versi stabil (opsional)
git checkout 3.9.0  # atau versi lain
```

### 4.4 Konfigurasi Environment

```bash
# Buat file .env
nano .env
```

Isi dengan:

```env
UPLOAD_FOLDER=/var/uploads
DATABASE_URL=mysql+pymysql://ctfd:ctfd_password@db/ctfd
REDIS_URL=redis://cache:6379
WORKERS=1
WORKER_TIMEOUT=600
LOG_FOLDER=/var/log/CTFd
ACCESS_LOG=-
ERROR_LOG=-
REVERSE_PROXY=true
MAX_CONTENT_LENGTH=104857600
SECRET_KEY=generate_secret_key_anda
```

**Buat SECRET_KEY:**
```bash
openssl rand -hex(32)
# Hasilkan: abcd1234... (copy hasil ini ke SECRET_KEY)
```

### 4.5 Konfigurasi docker-compose.yml

```bash
# Edit docker-compose.yml
nano docker-compose.yml
```

Isi lengkap:

```yaml
version: '3'

services:
  ctfd:
    image: ctfd/ctfd
    restart: always
    ports:
      - "8000:8000"
    environment:
      - UPLOAD_FOLDER=/var/uploads
      - DATABASE_URL=mysql+pymysql://ctfd:ctfd_password@db/ctfd
      - REDIS_URL=redis://cache:6379
      - WORKERS=1
      - WORKER_TIMEOUT=600
      - LOG_FOLDER=/var/log/CTFd
      - ACCESS_LOG=-
      - ERROR_LOG=-
      - REVERSE_PROXY=true
      - MAX_CONTENT_LENGTH=104857600
      - SECRET_KEY=YOUR_SECRET_KEY_HERE
    volumes:
      - ctfd_data:/var/uploads
      - ctfd_logs:/var/log/CTFd
    depends_on:
      - db
      - cache
    networks:
      - internal

  nginx:
    image: nginx:stable-alpine
    restart: always
    ports:
      - "80:80"
    volumes:
      - ./conf/nginx:/etc/nginx/conf.d
    depends_on:
      - ctfd
    networks:
      - internal

  db:
    image: mariadb:10.11
    restart: always
    environment:
      - MARIADB_ROOT_PASSWORD=root_password
      - MARIADB_USER=ctfd
      - MARIADB_PASSWORD=ctfd_password
      - MARIADB_DATABASE=ctfd
    volumes:
      - db_data:/var/lib/mysql
    networks:
      - internal
    command: [mysqld, --character-set-server=utf8mb4, --collation-server=utf8mb4_unicode_ci, --wait_timeout=28800, --log-warnings=0]

  cache:
    image: redis:4
    restart: always
    networks:
      - internal

networks:
  internal:
    driver: bridge

volumes:
  ctfd_data:
  ctfd_logs:
  db_data:
```

### 4.6 Buat Folder Config Nginx

```bash
mkdir -p conf/nginx
nano conf/nginx/http.conf
```

Isi:

```nginx
events {
    worker_connections 1024;
}

http {
    upstream app_servers {
        server ctfd:8000;
    }

    server {
        listen 80;
        server_name _;
        
        client_max_body_size 100M;
        gzip on;

        location / {
            proxy_pass http://app_servers;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Host $server_name;
        }

        location /events {
            proxy_pass http://app_servers;
            proxy_http_version 1.1;
            proxy_set_header Connection "";
            chunked_transfer_encoding off;
            proxy_buffering off;
            proxy_cache off;
        }
    }
}
```

### 4.7 Jalankan CTFd

```bash
# Pastikan di folder CTFd
cd /opt/CTFd

# Pull & run
docker-compose up -d

# Cek status
docker-compose ps

# Lihat logs
docker-compose logs -f ctfd
```

### 4.8 Setup Pertama (Setup Wizard)

1. Buka browser ke `http://IP_SERVER:8000` atau `http://IP_SERVER:80`

2. Klik **"Setup"**

3. Isi form:
   | Field | Nilai |
   |-------|-------|
   | Admin Email | admin@ctfd.local |
   | Password | (password kuat) |
   | Server Name | CTFd Anda |
   | Admin Team | Admin |

4. Klik **Submit**

### 4.9 Konfigurasi Tambahan

#### Upload Size (100MB)
```yaml
# docker-compose.yml
MAX_CONTENT_LENGTH=104857600

# conf/nginx/http.conf
client_max_body_size 100M;
```

#### Email Notification
```env
# .env
MAIL_ENABLED=true
MAIL_SERVER=smtp.gmail.com
MAIL_PORT=587
MAIL_USE_TLS=true
MAIL_USERNAME=email@gmail.com
MAIL_PASSWORD=app_password
```

#### HTTPS/SSL
```bash
# Install certbot
apt install certbot python3-certbot-nginx

# Get SSL
certbot --nginx -d domain.com

# Auto renew
certbot renew --dry-run
```

### 4.10 Perintah Penting

```bash
# Start CTFd
docker-compose up -d

# Stop CTFd
docker-compose down

# Restart
docker-compose restart ctfd

# Lihat logs
docker-compose logs -f

# Update CTFd
git fetch origin
git checkout 3.10.0
docker-compose pull
docker-compose up -d

# Backup database
docker-compose exec db mysqldump -u root -p ctfd > backup_ctfd.sql

# Restore database
docker-compose exec -T db mysql -u root -p ctfd < backup_ctfd.sql
```

### 4.11 Troubleshooting

| Problem | Solution |
|---------|----------|
| Can't access CTFd | Cek port: `docker-compose ps` |
| Upload Gagal | Tambah MAX_CONTENT_LENGTH |
| Error Database | Cek DATABASE_URL di .env |
| Slow Performance | Tambah WORKERS=2+ |
| 502 Error | Cek logs: `docker-compose logs nginx` |

### 4.12 Keamanan Dasar

```bash
# Ganti Secrets
# 1. Generate new SECRET_KEY
openssl rand -hex(32)

# 2. Update .env
nano .env

# 3. Restart
docker-compose restart

# Batasi akses port
ufw allow 80/tcp
ufw allow 443/tcp
ufw enable
```

