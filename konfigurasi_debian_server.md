# Konfigurasi Server iRedMail Debian

## 1. Pengechekan Service iRedMail

### Service yang Berjalan
```
amavis.service       - Interface between MTA and virus scanner/content filters
dovecot.service   - Dovecot IMAP/POP3 email server
iredadmin.service - iRedAdmin daemon service
iredapd.service  - iRedAPD (A simple postfix policy server)
postfix@-.service - Postfix Mail Transport Agent
```

### Port yang Aktif
| Port | Service |
|------|--------|
| 25   | SMTP   |
| 587  | Submission |
| 993  | IMAPS |
| 995  | POP3S |
| 80   | Web Server |
| 443  | HTTPS |

---

## 2. Masalah: Redirect HTTP ke HTTPS

### Masalah
- Saat pertama kali diakses, semua request HTTP (port 80) otomatis di-redirect ke HTTPS (port 443)
- Tidak bisa akses webmail via HTTP karena self-signed certificate menyebabkan warning di browser
- Port 80 dipegang oleh Apache, bentrok dengan Nginx

### Solusi
1. Matikan Apache, gunakan Nginx sebagai web server utama
2. Hapus konfigurasi redirect HTTP ke HTTPS
3. Nonaktifkan `force_https` di Roundcube
4. Konfigurasi ulang agar bisa akses via HTTP

---

## 3. Langkah Konfigurasi

### 3.1 Matikan Apache dan Aktifkan Nginx
```bash
# Stop dan disable Apache
systemctl stop apache2
systemctl disable apache2

# Start Nginx
systemctl start nginx
```

### 3.2 Hapus SSL Config
```bash
rm /etc/nginx/sites-enabled/00-default-ssl.conf
nginx -t && service nginx reload
```

### 3.3 Edit Konfigurasi Nginx (HTTP)
File: `/etc/nginx/sites-enabled/00-default.conf`

```nginx
server {
    listen 80;
    listen [::]:80;

    server_name _;

    # ACME challenge
    location ~* ^/.well-known/acme-challenge/ {
        root /opt/www/well_known;
        try_files $uri =404;
        allow all;
    }

    # iRedAdmin static files
    location ~ ^/iredadmin/static/(.*) {
        alias /opt/www/iredadmin/static/$1;
    }

    # iRedAdmin
    location ~ ^/iredadmin(.*) {
        rewrite ^/iredadmin(/.*)$ $1 break;
        include uwsgi_params;
        uwsgi_pass 127.0.0.1:7791;
        uwsgi_param UWSGI_CHDIR /opt/www/iredadmin;
        uwsgi_param UWSGI_SCRIPT iredadmin;
        uwsgi_param SCRIPT_NAME /iredadmin;
    }

    location = /iredadmin {
        rewrite ^ /iredadmin/;
    }

    # Roundcube as subfolder /mail
    location ~ ^/(mail|)(.*\.php)$ {
        include /etc/nginx/templates/fastcgi_php.tmpl;
        fastcgi_param SCRIPT_FILENAME /opt/www/roundcubemail/$2;
    }

    location ~ ^/(mail|/)(.*) {
        alias /opt/www/roundcubemail/$2;
        index index.php;
    }

    location = /mail {
        return 301 /mail/;
    }

    # Root redirects to mail
    location = / {
        return 301 /mail/;
    }
}
```

### 3.4 Update Roundcube Template
File: `/etc/nginx/templates/roundcube.tmpl`

```nginx
# Roundcube as subfolder on HTTP
location ~ ^/(mail|)(.*\.php)$ {
    include /etc/nginx/templates/fastcgi_php.tmpl;
    fastcgi_param SCRIPT_FILENAME /opt/www/roundcubemail/$2;
}

location ~ ^/(mail|/)(.*) {
    alias /opt/www/roundcubemail/$2;
    index index.php;
}

location = /mail {
    return 301 /mail/;
}
```

### 3.5 Nonaktifkan force_https di Roundcube
File: `/opt/www/roundcubemail/config/config.inc.php`

```php
// Sebelum:
// $config['force_https'] = true;

// Sesudah:
// $config['force_https'] = true;
```
Ubah menjadi comment (nonaktifkan).

### 3.6 Disable SOGo Template (Optional)
```bash
# Kalau ada error dengan sogo template, disable sementara:
mv /etc/nginx/templates/sogo.tmpl /etc/nginx/templates/sogo.tmpl.bak
# Atau buat file kosong
echo "# SOGo disabled" > /etc/nginx/templates/sogo.tmpl
```

### 3.7 Reload Konfigurasi
```bash
nginx -t && service nginx reload
```

---

## 4. URL Akses

| Service | URL |
|--------|-----|
| Webmail (Roundcube) | http://10.25.10.51/mail/ |
| iRedAdmin | http://10.25.10.51/iredadmin/ |

---

## 5. Informasi Tambahan

### Service Login Default
- **iRedAdmin**: admin / `password` (sesuaikan setelah install)
- **Roundcube**: Gunakan akun email yang dibuat

### Port Service
- SMTP: 25, 587
- IMAP: 143, 993
- POP3: 110, 995
- Web: 80, 443

### Direktori Penting
- Webmail: `/opt/www/roundcubemail`
- iRedAdmin: `/opt/www/iredadmin`
- Konfigurasi Nginx: `/etc/nginx/`
- Log: `/var/log/nginx/`

---

## 6. Troubleshooting

### Jika Webmail Tidak Muncul
```bash
# Cek status service
systemctl status nginx
systemctl status dovecot
systemctl status postfix

# Cek port listening
ss -tlnp | grep :80

# Test localhost
curl -s http://localhost/mail/
```

### Jika 404
```bash
# Cek root path Roundcube
ls /opt/www/roundcubemail/public_html/

# Reload nginx
service nginx reload
```

---

*Dokumen ini dibuat pada: 5 Mei 2026*