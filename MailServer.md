# Membangun Mail Server dengan Postfix + Dovecot + Roundcube

Panduan ini ngebahas cara setup mail server dari nol di Debian/Ubuntu Server. Kita bakal pake tiga komponen utama:

- **Postfix** — buat ngirim email (SMTP)
- **Dovecot** — buat nerima & ngambil email (IMAP/POP3)
- **Roundcube** — tampilan webmail biar bisa akses email lewat browser

> **Environment yang dipakai:**
> OS: Debian / Ubuntu Server
> IP Server: `10.222.11.178`

---

## Daftar Isi

1. [Persiapan Server](#1-persiapan-server)
2. [Install Paket](#2-install-paket)
3. [Konfigurasi Postfix](#3-konfigurasi-postfix)
4. [Konfigurasi Dovecot](#4-konfigurasi-dovecot)
5. [Buat User Email](#5-buat-user-email)
6. [Konfigurasi Roundcube](#6-konfigurasi-roundcube)
7. [Testing](#7-testing)

---

## 1. Persiapan Server

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

### Set IP static

Buka file konfigurasi network:

```bash
nano /etc/network/interfaces
```

Isi seperti ini (sesuaikan dengan jaringan kamu):

```ini
auto ens33
iface ens33 inet static
  address 10.25.10.150
  netmask 255.255.255.0
  gateway 10.25.10.1
  dns-nameservers 8.8.8.8
```

Restart network setelah disimpan:

```bash
systemctl restart networking
```

### Update sistem

```bash
apt update && apt upgrade -y
```

---

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
