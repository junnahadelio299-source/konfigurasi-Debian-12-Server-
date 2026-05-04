# Konfigurasi Server Berbasis Debian 12

## Daftar Isi
1. [Konfigurasi File Server](#1-konfigurasi-file-server-samba)
2. [Konfigurasi Web Server](#2-konfigurasi-web-server)

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
- Dokumentasi Nginx: https://nginx.org/en/docs/
- Dokumentasi CTFd: https://docs.ctfd.io/
- Dokumentasi Samba: https://www.samba.org/samba/docs/
- Wiki Samba: https://wiki.samba.org/

---

*Document dibuat: 4 Mei 2026*
*Terakhir diperbarui: 4 Mei 2026*