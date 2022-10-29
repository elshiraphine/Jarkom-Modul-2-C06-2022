# Jarkom-Modul-2-C06-2022

## Revisi

### Nomor 14 dan 15

Karena config pada apache2 nya sudah benar. Untuk melakukan testing dilakukan perubahan pada DNS yang ada pada Berlint (Slave) dan WISE (DNS Masters).
Perubahan adalah sebagai berikut:
1. Pada `/etc/bind/named.conf.local` di **WISE (DNS Masters)**
   ```conf
   zone "wise.C06.com" {
        type master;
        file "/etc/bind/wise/wise.C06.com";
        // IP Berlint
        notify yes;
        also-notify { 192.182.3.3; };
        allow-transfer { 192.182.3.3; };
    };
   ```
2. Pada `/etc/bind/wise/wise.C06.com` di **WISE (DNS Masters)** ditambahkan
   ```conf
    ns1             IN      A       192.182.3.3     ; IP Berlint
    operation       IN      NS      ns1
   ```
3. Pada `/etc/bind/named.conf.local` di **Berlint (Slave)**
   ```conf
    zone "wise.C06.com" {
        type slave;
        masters { 192.182.1.2; };
        file "/var/lib/bind/wise.C06.com";
    };

    zone "operation.wise.C06.com" {
        type master;
        file "/etc/bind/operation/operation.wise.C06.com";
    };
   ```
4. Pada `/etc/bind/operation/operation.wise.C06.com` di **Berlint (Slave)** ditambahkan
   ```conf
    @               IN      NS      operation.wise.C06.com.
    @               IN      A       192.182.3.2             ; IP Eden
    www             IN      CNAME   operation.wise.C06.com.
    strix           IN      A       192.182.3.2             ; IP Eden
    www.strix       IN      CNAME   strix.operation.wise.C06.com.
   ```

Kemudian dilakukan testing dengan konfigurasi apache2 yang tidak diubah, dihasilkan:
![akses-ke-port-15500](https://media.discordapp.net/attachments/964890423946543124/1035901390238785546/unknown.png) <br />

Untuk membuka web tersebut perlu melakukan autentikasi dengan password, sehingga:
![auth](https://media.discordapp.net/attachments/964890423946543124/1035901412720250940/unknown.png) <br />

**Sehingga untuk Revisi Nomor 14 dan 15 SELESAI**

### Nomor 16
Pada nomor 16, kita perlu menambahkan file `.htaccess` di `/var/www/html` berisi:
```conf
RewriteEngine On
RewriteBase /
RewriteCond %{HTTP_HOST} ^192\.182\.3\.2$
RewriteRule ^(.*)$ http://wise.C06.com/$1 [L,R=301]
```

Kemudian di file `/etc/apache2/sites-available/000-default.conf` ditambahkan
```conf
<Directory /var/www/html>
    Options +FollowSymLinks -Multiviews
    AllowOverride All
</Directory>
```
Sehingga saat IP Eden diakses (192.182.3.2) akan melakukan lookup pada situs http://wise.C06.com <br />
![lookup](https://media.discordapp.net/attachments/964890423946543124/1035901332206387240/using.png) <br />
Hasilnya: <br />
![hasil](https://media.discordapp.net/attachments/964890423946543124/1035901332206387240/using.png) <br />
**Sehingga untuk Revisi Nomor 16 SELESAI**