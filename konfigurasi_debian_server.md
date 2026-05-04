# Konfigurasi Server Berbasis Debian 12

## Daftar Isi
0. [Instalasi Debian 12](#0-instalasi-debian-12)
1. [Konfigurasi File Server](#1-konfigurasi-file-server-samba)
2. [Konfigurasi Web Server](#2-konfigurasi-web-server)

---

## 0. Instalasi Debian 12

### 0.1 Download ISO Debian 12
Unduh ISO Debian 12 dari situs resmi:
```bash
# Via CLI (jika sudah ada sistem)
wget https://cdimage.debian.org/debian-cd/12.0.0/amd64/iso-dvd/debian-12.0.0-amd64-DVD-1.iso
```

atau download dari:
- https://www.debian.org/distrib/
- https://cdimage.debian.org/debian-cd/12.0.0/amd64/iso-dvd/

### 0.2 Buat Bootable USB
```bash
# Menggunakan dd (linux)
sudo dd if=debian-12.0.0-amd64-DVD-1.iso of=/dev/sdX bs=4M status=progress

# Atau menggunakan Ventoy
# Download Ventoy dari https://ventoy.net
```

### 0.3 Proses Instalasi

#### Langkah 1: Boot dari USB
- Atur boot priority di BIOS ke USB
- Pilih "Debian 12" dari menu boot

#### Langkah 2: Pilih Bahasa
- Pilih bahasa: English
- Pilih lokasi: Indonesia atau United States

#### Langkah 3: Konfigurasi Jaringan
- Isi hostname: debian-server
- Isi domain: (kosongkan jika tidak punya)

#### Langkah 4: Buat User
- Buat user biasa (bukan root)
- Contoh: nanzz

#### Langkah 5: Partisi Disk
Pilih opsi:
- **Guided - use entire disk** (untuk satu sistem)
- **Guided - use entire disk and set up LVM** (jika mau LVM)
- **Manual** (untuk custom partitioning)

#### Langkah 6: Konfirmasi
- Review konfigurasi partisi
- Konfirmasi penulisan ke disk
- Tunggu proses instalasi selesai

### 0.4 Install Desktop Environment (Optional)

#### Instalasi Desktop KDE Plasma
```bash
apt-get install kde-plasma-desktop
```

#### Instalasi Desktop GNOME
```bash
apt-get install gnome-core
```

#### Instalasi Desktop XFCE (Ringan)
```bash
apt-get install xfce4 xfce4-goodies
```

### 0.5 Update Sistem Pertama Kali
```bash
# Update list package
apt-get update

# Upgrade semua package
apt-get upgrade

# Upgrade distribusi
apt-get dist-upgrade
```

### 0.6 Install Package Dasar
```bash
# Install useful utilities
apt-get install -y \
    vim \
    curl \
    wget \
    git \
    htop \
    net-tools \
    unzip \
    tar \
    gzip \
    build-essential \
    software-properties-common
```

### 0.7 Konfigurasi Network (Static IP)

#### Edit file konfigurasi
```bash
vim /etc/network/interfaces
```

#### Untuk Static IP
```bash
# Sebelum (DHCP)
# iface eth0 inet dhcp

# Sesudah (Static)
auto eth0
iface eth0 inet static
    address 192.168.1.100
    netmask 255.255.255.0
    gateway 192.168.1.1
    dns-nameservers 8.8.8.8 8.8.4.4
```

#### Restart network
```bash
# Untuk Debian 12 (systemd)
systemctl restart networking

# Atau restart interface tertentu
ip link set eth0 down
ip link set eth0 up
```

### 0.8 Konfigurasi SSH

#### Install OpenSSH Server
```bash
apt-get install -y openssh-server
```

#### Edit Konfigurasi SSH
```bash
vim /etc/ssh/sshd_config
```

#### Ubah port (opsional)
```bash
# Ubah port default 22 ke port lain
Port 2222
```

#### Restart SSH
```bash
systemctl restart ssh
systemctl enable ssh
```

#### Test SSH
```bash
ssh user@192.168.1.100 -p 2222
```

### 0.9 Aktifkan Repo

#### Repo Utama (sudah aktifdefault)
```
deb http://deb.debian.org/debian bookworm main
deb http://deb.debian.org/debian bookworm updates
```

#### Tambahkan Repo Security
```bash
# Buka sources.list
vim /etc/apt/sources.list
```

Tambahkan:
```
deb http://security.debian.org/debian-security bookworm-security main
```

#### Update setelah mengubah repo
```bash
apt-get update
```

---

## 1. Konfigurasi File Server (Samba)

### 1.1 Install Samba
```bash
apt-get update
apt-get install -y samba
```

### 1.2 Cek Status Samba
```bash
systemctl status smbd
# Atau
service smbd status
```

### 1.3 Konfigurasi Samba
File: `/etc/samba/smb.conf`

Tambahkan konfigurasi share di akhir file:
```ini
[NanzFileServer]
   path = /home/nanzz/fileserver
   browseable = yes
   read only = no
   guest ok = yes
   force user = nobody
```

### 1.4 Test Konfigurasi
```bash
testparm
```

### 1.5 Restart Samba
```bash
systemctl restart smbd
# Atau
service smbd restart
```

### 1.6 Akses File Server

#### Dari Windows
```
\\192.168.1.xxx\NanzFileServer
# atau
\\localhost\NanzFileServer
```

#### Dari Linux
```bash
# Mount menggunakan smbclient
smbclient //localhost/NanzFileServer -U guest

# Mount sebagai filesystem
mount -t cifs //localhost/NanzFileServer /mnt -o guest
```

#### Cek Daftar Share
```bash
smbclient -L localhost -U guest
```

### 1.7 Perintah Berguna Samba

#### Manajemen Service
```bash
systemctl start smbd    # Mulai Samba
systemctl stop smbd     # Hentikan Samba
systemctl restart smbd # Restart Samba
systemctl status smbd  # Cek status
```

#### Cek Konfigurasi
```bash
testparm               # Test konfigurasi
testparm -s            # Test konfigurasi (silent)
```

#### List Share
```bash
smbclient -L localhost -U guest
```

---

## 2. Konfigurasi Web Server

### 2.1 Persiapan Lingkungan

#### Cek File Backup
```bash
ls -la /home/nanzz/website/
# Menemukan: ctf_manual_20260401_144346.tar.gz
```

#### Ekstrak File Backup
```bash
tar -xzf ctf_manual_20260401_144346.tar.gz -C /home/nanzz/website/
```

### 2.2 Install Nginx dan Docker

#### Install Nginx
```bash
apt-get update
apt-get install -y nginx
```

#### Install Docker
```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
```

#### Verifikasi Installation
```bash
nginx -v        # Nginx 1.22.1
docker --version # Docker 29.4.1
docker compose version # Docker Compose v5.1.3
```

### 2.3 Build Docker Containers

#### Build CTFd Image
```bash
cd /home/nanzz/website/ctf/CTFd
docker compose build
```
Proses ini akan mendownload dependencies dan build CTFd image (butuh beberapa menit).

#### Jalankan Containers
```bash
docker compose up -d
```

#### Cek Status Containers
```bash
docker ps
```
Output yang diharapkan:
```
CONTAINER ID   IMAGE           PORTS
ctfd-nginx-1  nginx:stable    0.0.0.0:8080->80/tcp
ctfd-ctfd-1   ctfd-ctfd       0.0.0.0:8000->8000/tcp
ctfd-db-1     mariadb:10.11    3306/tcp
ctfd-cache-1  redis:4         6379/tcp
```

### 2.4 Konfigurasi Nginx (Local Only)

#### Edit Config Nginx
File: `/home/nanzz/website/ctf/CTFd/conf/nginx/http.conf`

Ubah port listen:
```nginx
# Sebelum: listen 8000;
# Sesudah: listen 80;
```

#### Edit Docker Compose
File: `/home/nanzz/website/ctf/CTFd/docker-compose.yml`

Ubah port mapping agar hanya lokal:
```yaml
# Sebelum: - 8080:80
# Sesudah: - 127.0.0.1:8000:80
```

#### Restart Nginx
```bash
docker compose restart nginx
```

### 2.5 Akses Web Server

| Service | Port | URL |
|---------|------|-----|
| CTFd App | 8000 | http://127.0.0.1:8000 |

---

## Lampiran A: Perintah Berguna

### Manajemen Containers
```bash
docker compose start      # Mulai semua containers
docker compose stop     # Hentikan semua containers
docker compose restart   # Restart semua containers
docker compose down    # Hentikan dan hapus containers
```

### Manajemen Nginx
```bash
nginx -t            # Test konfigurasi
nginx -s reload     # Reload konfigurasi
systemctl status nginx  # Cek status
systemctl restart nginx  # Restart nginx
```

### Manajemen Service
```bash
systemctl enable nginx   # Aktifkan saat boot
systemctl disable nginx # Nonaktifkan saat boot
```

---

## Lampiran B: Troubleshooting

### Container tidak mau start
```bash
docker logs ctfd-ctfd-1
# Periksa error message
```

### Nginx tidak bisa start
```bash
nginx -t
# Periksa error konfigurasi
```

### Samba tidak bisa diakses
```bash
systemctl status smbd
testparm
```

###Reload Konfigurasi
```bash
docker compose restart nginx
# Atau
systemctl restart smbd
nginx -s reload
```

### Cek Port yang Digunakan
```bash
ss -tuln | grep :80
netstat -tuln | grep :445
```

---

## Lampiran C: Informasi Sistem

| Service | Software | Port | Lokasi |
|---------|----------|------|--------|
| Web Server | Nginx 1.22.1 | 8000 | CTFd Docker |
| File Server | Samba 4.17.12 | 445 (SMB) | /home/nanzz/fileserver |
| Database | MariaDB 10.11 | 3306 | Docker |
| Cache | Redis 4 | 6379 | Docker |

### Credential
- **Samba**: Tidak ada username (guest)
- **CTFd Database**: user=ctfd, password=ctfd

---

## Sumber Daya
- Debian Official: https://www.debian.org/
- Dokumentasi Debian: https://www.debian.org/doc/
- Dokumentasi Nginx: https://nginx.org/en/docs/
- Dokumentasi CTFd: https://docs.ctfd.io/
- Dokumentasi Samba: https://www.samba.org/samba/docs/
- Wiki Samba: https://wiki.samba.org/

---

*Document dibuat: 4 Mei 2026*
*Terakhir diperbarui: 4 Mei 2026*