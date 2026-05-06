<div align="center">
  <h1 align="center">Debian 12 Server</h3>
  <p align="center">
    Panduan Latihan untuk membuat server debian 12
  </p>
</div>

## Daftar Isi

1. [Penginstalan Debian 12](#1.Persiapan-penginstalan-debian)
2. [File Server](#2.File-Server)
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
## 2.File Server

Panduan ini membahas cara setup file server dengan Samba di Debian 12.

### 2.1 Install Samba

```bash
su -
apt update
apt install -y samba smbclient cifs-utils
```

### 2.2 Buat Folder File Server

```bash
mkdir -p /home/nanzz/fileserver/dokumen
mkdir -p /home/nanzz/fileserver/foto
mkdir -p /home/nanzz/fileserver/video
mkdir -p /home/nanzz/fileserver/backup
chown -R nobody:nogroup /home/nanzz/fileserver
chmod -R 777 /home/nanzz/fileserver
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

[NanzFileServer]
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

### 2.7 Troubleshooting

```bash
systemctl status smbd
ss -tlnp | grep -E ":445|:139"
tail -f /var/log/samba/log.smbd
systemctl restart smbd
```

### 2.8 Perintah Penting

| Perintah | Fungsi |
|---------|-------|
| `testparm` | Test konfigurasi |
| `smbpasswd -a user` | Tambah user |
| `smbclient -L localhost` | List share |

## 3.Web Server
