<div align="center">
  <h1 align="center">Debian 12 Server</h3>
  <p align="center">
    Panduan Latihan untuk membuat server debian 12
  </p>
</div>

## Daftar Isi

1. [Penginstalan Debian 12](#1-Persiapan-penginstalan-debian)
2. [File Server](#2-File-Server)
3. [Web Server](#3-Web-Server)
4. [Mail Server](#4-Mail-Server)

---

## 1. Persiapan penginstalan debian

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
[share]
   path = /home/share
   writable = yes
   create mask = 0777
   directory mask = 0777
   public = no
   guest ok = yes
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

## 3. Web Server

Panduan setup web server dengan Nginx & Apache di Debian 12.

### 3.1 Install Apache2

```bash
apt update
apt install -y apache2
```

### 3.2 Start Service

```bash
systemctl enable apache2
systemctl start apache2
systemctl status apache2
```

### 3.3 Cek Web Server

Buka browser:

```text
http://IP_SERVER
```

Jika muncul halaman Apache2 default berarti berhasil.

### 3.4 CTFd (Capture The Flag Platform)

Panduan install CTFd dari GitHub dengan Docker.

### 3.5 Persiapan Sistem

| Komponen | Minimal | Rekomendasi |
|----------|---------|------------|
| OS | Ubuntu 20.04+ / Debian 12 | Ubuntu 22.04 |
| RAM | 2 GB | 4 GB |
| CPU | 2 core | 4 core |
| Disk | 20 GB | 40 GB |
| Docker | 20.10+ | Latest |

### 3.6 Install Docker

```bash 
# 1. Update & install dependency
apt update
apt install -y ca-certificates curl gnupg lsb-release

# 2. Add Docker GPG key
install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
chmod a+r /etc/apt/keyrings/docker.gpg

# 3. Add Docker repository (DEBIAN, bukan Ubuntu)
echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/debian \
$(. /etc/os-release && echo $VERSION_CODENAME) stable" | \
tee /etc/apt/sources.list.d/docker.list > /dev/null

# 4. Install Docker Engine
apt update
apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# 5. Enable & start Docker
systemctl enable docker
systemctl start docker

# 6. (optional) add user ke docker group
usermod -aG docker admin
```

### 3.7 Install Git & Clone CTFd

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

### 3.8 Konfigurasi docker-compose.yml

```bash
# Edit docker-compose.yml
nano docker-compose.yml
```

Isi lengkap:

```yaml
services:
  ctfd:
    build: .
    restart: always
    ports:
      - "8000:8000"
    environment:
      - UPLOAD_FOLDER=/var/uploads
      - DATABASE_URL=mysql+pymysql://ctfd:ctfd@db/ctfd
      - REDIS_URL=redis://cache:6379
      - WORKERS=1
      - WORKER_TIMEOUT=600 #menambahkan ini
      - LOG_FOLDER=/var/log/CTFd
      - ACCESS_LOG=-
      - ERROR_LOG=-
      - REVERSE_PROXY=true
      - MAX_CONTENT_LENGTH=104857600 #100mb
    volumes:
      - .data/CTFd/logs:/var/log/CTFd
      - .data/CTFd/uploads:/var/uploads:rw
      - .:/opt/CTFd:ro
    depends_on:
      - db
    networks:
        default:
        internal:

  db:
    image: mariadb:10.11
    restart: always
    environment:
      - MARIADB_ROOT_PASSWORD=ctfd
      - MARIADB_USER=ctfd
      - MARIADB_PASSWORD=ctfd
      - MARIADB_DATABASE=ctfd
      - MARIADB_AUTO_UPGRADE=1
    volumes:
      - .data/mysql:/var/lib/mysql
    networks:
        internal:
    # This command is required to set important mariadb defaults
    command: [mysqld, --character-set-server=utf8mb4, --collation-server=utf8mb4_unicode_ci, --wait_timeout=28800, --log-warnings=0]

  cache:
    image: redis:4
    restart: always
    volumes:
    - .data/redis:/data
    networks:
        internal:

networks:
    default:
    internal:
        internal: true
```

### 3.9 Jalankan CTFd

pastikan berada di folder CTFd
```bash
cd /home/website/ctf/CTFd
```
jalankan dan liat statusnya
```bash 
# Pull & run
docker-compose up -d

# Cek status
docker-compose ps
```
untuk melihat log 
```bash 
# Lihat logs
docker-compose logs -f ctfd
```

### 3.10 Setup Pertama (Setup Wizard)

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

### 3.11 Konfigurasi Tambahan

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

### Perintah Penting

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

### Troubleshooting

| Problem | Solution |
|---------|----------|
| Can't access CTFd | Cek port: `docker-compose ps` |
| Upload Gagal | Tambah MAX_CONTENT_LENGTH |
| Error Database | Cek DATABASE_URL di .env |
| Slow Performance | Tambah WORKERS=2+ |
| 502 Error | Cek logs: `docker-compose logs nginx` |

### 3.12 Keamanan Dasar

```bash
# 1. Restart
docker-compose restart

# Batasi akses port
ufw allow 80/tcp
ufw allow 443/tcp
ufw enable
```

## 4. Mail Server

## 4.1 Persiapan Server

### Login sebagai root

```bash
su -
```

### Set hostname

```bash
hostnamectl set-hostname mail.local
```
nama bisa di ganti sesuai keinginan
lalu cek hasilnya
```bash
hostname  # cek hasilnya
```

## 2. Install Paket

### Apache, PHP, dan dependensinya

```bash
apt install apache2 mariadb-server php php-mysql php-intl php-mbstring php-xml php-zip php-curl unzip -y
```

### Postfix

```bash
apt install postfix -y
```

Saat proses install muncul dialog, pilih:
- **Internet Site**
- System mail name: `mail.local`

### Dovecot

```bash
apt install dovecot-core dovecot-imapd dovecot-pop3d -y
```

### Roundcube

```bash
apt install roundcube roundcube-mysql -y
```

Saat muncul dialog:
- Pilih **Yes** untuk dbconfig
- Isi password database sesuai keinginan kamu

---

## 3. Konfigurasi Postfix

Edit file konfigurasi utama Postfix:

```bash
nano /etc/postfix/main.cf
```

Pastikan isi berikut ada dan benar:

```ini
myhostname = mail.local
mydomain = local
myorigin = /etc/mailname

inet_interfaces = all
inet_protocols = ipv4

mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain

home_mailbox = Maildir/

smtpd_sasl_type = dovecot
smtpd_sasl_path = private/auth
smtpd_sasl_auth_enable = yes
smtpd_sasl_security_options = noanonymous

smtpd_recipient_restrictions =
  permit_mynetworks,
  permit_sasl_authenticated,
  reject_unauth_destination
```

Restart Postfix:

```bash
systemctl restart postfix
```

---

## 4. Konfigurasi Dovecot

### Set lokasi penyimpanan email (Maildir)

```bash
nano /etc/dovecot/conf.d/10-mail.conf
```

Cari baris `mail_location` dan ubah jadi:

```ini
mail_location = maildir:~/Maildir
```

### Aktifkan autentikasi sistem

```bash
nano /etc/dovecot/conf.d/10-auth.conf
```

Pastikan baris ini aktif (tidak ada tanda `#` di depannya):

```ini
!include auth-system.conf.ext
```

Dan baris ini dikomentari:

```ini
#!include auth-sql.conf.ext
```

### Hubungkan Dovecot ke Postfix lewat socket

```bash
nano /etc/dovecot/conf.d/10-master.conf
```

Cari bagian `unix_listener` dan pastikan isinya seperti ini:

```ini
unix_listener /var/spool/postfix/private/auth {
  mode = 0660
  user = postfix
  group = postfix
}
```

Restart Dovecot:

```bash
systemctl restart dovecot
```

---

## 5. Buat User Email

### Tambah user Linux

```bash
adduser user1
adduser user2
```

### Buat folder Maildir untuk masing-masing user

```bash
maildirmake.dovecot /home/user1/Maildir
maildirmake.dovecot /home/user2/Maildir
```

### Set kepemilikan folder

```bash
chown -R user1:user1 /home/user1/Maildir
chown -R user2:user2 /home/user2/Maildir
```

---

## 6. Konfigurasi Roundcube

### Edit file konfigurasi Roundcube

```bash
nano /etc/roundcube/config.inc.php
```

Sesuaikan bagian berikut:

```php
$config['imap_host'] = '127.0.0.1:143';
$config['smtp_host'] = '127.0.0.1:25';

$config['smtp_user'] = '%u';
$config['smtp_pass'] = '%p';

$config['mail_domain'] = 'mail.local';
$config['username_domain'] = 'mail.local';

$config['product_name'] = 'Roundcube Webmail';
$config['skin'] = 'elastic';
```

### Aktifkan konfigurasi Apache untuk Roundcube

```bash
ln -s /etc/roundcube/apache.conf /etc/apache2/conf-enabled/roundcube.conf
systemctl reload apache2
```

---

## 7. Testing

### Akses webmail di browser

Buka:

```
http://10.25.10.150/roundcube
```

Login dengan:

```
Username: user1@mail.local
Password: password_user1
```

### Coba kirim email

- **From:** user1@mail.local
- **To:** user2@mail.local
- **Subject:** TEST

Kalau email berhasil masuk ke inbox user2, berarti mail server sudah jalan dengan benar.

### Pantau log di server

Buka terminal dan jalankan perintah ini untuk melihat aktivitas email secara real-time:

```bash
tail -f /var/log/mail.log
```

---

## Selesai

Mail server sudah berhasil dibangun. Berikut ringkasan komponen yang dipakai:

| Komponen | Fungsi | Port |
|----------|--------|------|
| Postfix | Kirim email (SMTP) | 25 |
| Dovecot | Terima email (IMAP) | 143 |
| Roundcube | Webmail via browser | 80 |

---

*Ditulis sebagai dokumentasi praktikum jaringan.*
