# Jarkom-Modul-2-C06-2022

## Anggota Kelompok

1. 5025201050 - Elshe Erviana Angely
2. 5025201051 - Muhammad Fath Mushaffa Azhar
3. 5025201076 - Raul Ilma Rajasa

## Soal 1

WISE akan dijadikan sebagai DNS Master, Berlint akan dijadikan DNS Slave, dan Eden akan digunakan sebagai Web Server. Terdapat 2 Client yaitu SSS, dan Garden. Semua node terhubung pada router Ostania, sehingga dapat mengakses internet (1).

Untuk itu, perlu dilakukan pembuatan topologi dengan router Ostania serta node-node yang terdiri dari Wise (DNS Master), SSS dan Garden (Client), Berlint (DNS Slave), dan Eden (Web Server), yaitu sebagai berikut:

![topologi-c06](https://cdn.discordapp.com/attachments/855800698602913792/1035874328861999176/unknown.png)
Dari situ, buat konfigurasi jaringan untuk setiap router dan node-node yang ada (klik kanan pada node/router -> configure -> edit network configuration), dengan isi sebagai berikut:

- Ostania:

```
auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet static
	address 192.182.1.1
	netmask 255.255.255.0

auto eth2
iface eth2 inet static
	address 192.182.2.1
	netmask 255.255.255.0

auto eth3
iface eth3 inet static
	address 192.182.3.1
	netmask 255.255.255.0
```

- WISE:

```
auto eth0
iface eth0 inet static
	address 192.182.1.2
	netmask 255.255.255.0
	gateway 192.182.1.1
```

- SSS:

```
auto eth0
iface eth0 inet static
	address 192.182.2.2
	netmask 255.255.255.0
	gateway 192.182.2.1
```

- Garden:

```
auto eth0
iface eth0 inet static
	address 192.182.2.3
	netmask 255.255.255.0
	gateway 192.182.2.1
```

- Berlint:

```
auto eth0
iface eth0 inet static
	address 192.182.3.3
	netmask 255.255.255.0
	gateway 192.182.3.1
```

- Eden:

```
auto eth0
iface eth0 inet static
	address 192.182.3.2
	netmask 255.255.255.0
	gateway 192.182.3.1
```

Setelah itu, masukkan `iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.182.0.0/16` pada router Ostania tepatnya di file .bashrc serta jalankan perintah `echo nameserver 192.168.122.1 > /etc/resolv.conf` pada console node selain Ostania.
Lalu testing command `ping google.com` pada salah satu node untuk mengetes apakah node tersebut sudah terkoneksi internet atau belum:
![testing-no1-c06](https://cdn.discordapp.com/attachments/855800698602913792/1035878842885226546/unknown.png)

## Soal 2

Untuk mempermudah mendapatkan informasi mengenai misi dari Handler, bantulah Loid membuat website utama dengan akses wise.yyy.com dengan alias www.wise.yyy.com pada folder wise (2).

Untuk itu, open console pada WISE dan Berlint untuk menginstalasi bind9 karena kedua node akan dijadikan DNS Server sebagai Master dan Slave dengan memasukkan

```
apt-get update
apt-get install bind9 -y`
```

pada file .bashrc. Serta tambahkan konfigurasi dns utils pada client (SSS dan Garden) dengan memasukkan pada file .bashrc sebagai berikut:

```
apt-get update
apt-get install dnsutils -y
```

Setelah itu, setup data file bind pada wise seperti berikut:

```
mkdir /etc/bind/wise
cp /etc/bind/db.local /etc/bind/wise/wise.C06.com
nano /etc/bind/wise/wise.C06.com
```

Konfigurasi data file bind wise seperti berikut:

```
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     wise.C06.com. root.wise.C06.com. (
                     2022102601         ; Serial
                         604800         ; Refresh
86400	    ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      wise.C06.com.
@       IN      A       192.182.1.2       ; IP WISE
www     IN      CNAME   wise.C06.com.
eden    IN      A       192.182.3.2       ; IP Eden
www.eden        IN      CNAME   eden.wise.C06.com.
ns1     IN      A       192.182.3.3       ; IP Berlint

```

![WISE-Konfigurasi](https://cdn.discordapp.com/attachments/855800698602913792/1035895601017126962/unknown.png)
Lakukan konfigurasi zonasi juga dengan `nano /etc/bind/named.conf.local`:

```
zone "wise.C06.com" {
  type master;
  file "/etc/bind/wise/wise.C06.com";
};
```

![WISE-NameConf](https://cdn.discordapp.com/attachments/855800698602913792/1035896519213199440/unknown.png)

Setelah itu, restart bind pada wise dengan command `service bind9 restart`
Pada client SSS dan Garden, terapkan nameserver sebagai berikut pada `/etc/resolv.conf`:

```
#nameserver 192.168.122.1 # IP Utama
nameserver 192.182.1.2 # IP WISE
nameserver 192.182.3.3 # IP Berlint
```

Lakukan testing dengan memasukkan command berikut:

```
ping wise.C06.com
ping www.wise.C06.com
host -t CNAME www.wise.C06.com
```

![Testing-2](https://cdn.discordapp.com/attachments/855800698602913792/1035903660883976202/unknown.png)

## Soal 3

Setelah itu ia juga ingin membuat subdomain eden.wise.yyy.com dengan alias www.eden.wise.yyy.com yang diatur DNS-nya di WISE dan mengarah ke Eden (3).

Untuk itu, tambahkan konfigurasi data file bind wise seperti berikut:

```
eden    IN      A       192.182.3.2       ; IP Eden
www.eden        IN      CNAME   eden.wise.C06.com.
```

![WISE-Konfigurasi](https://cdn.discordapp.com/attachments/855800698602913792/1035906102027632721/unknown.png)

Setelah itu, restart bind pada wise dengan command `service bind9 restart`

Lakukan testing pada client (SSS atau Garden) dengan memasukkan command berikut:

```
ping eden.wise.C06.com
ping www.eden.wise.C06.com
host -t CNAME www.eden.wise.C06.com
```

![Testing-3](https://cdn.discordapp.com/attachments/855800698602913792/1035906945816743947/unknown.png)

Untuk nomor 8-17 semua konfigurasi akan dilakukan di **Eden**.
Langkah yang perlu dilakukan di **Eden** adalah:
```sh
apt-get update
apt-get install apache2 php unzip wget -y
```
Selain itu di client perlu dilakukan instalasi `lynx` untuk membuka website melalui console.
```sh
apt-get install lynx
``` 
## Soal 4

Buat juga reverse domain untuk domain utama (4)

Untuk itu, setup data file bind untuk reverse domain pada wise seperti berikut:

```
cp /etc/bind/db.local /etc/bind/wise/1.182.192.in-addr.arpa
nano /etc/bind/wise/1.182.192.in-addr.arpa
```

Konfigurasi data file bind wise seperti berikut:

```
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     wise.C06.com. root.wise.C06.com. (
                     2022102601         ; Serial
                         604800         ; Refresh
                        86400           ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
1.182.192.in-addr.arpa.       IN      NS      wise.C06.com.
2       IN      PTR     wise.C06.com.
```

![WISE-Konfigurasi](https://cdn.discordapp.com/attachments/855800698602913792/1035908043453513808/unknown.png)
Tambahkan konfigurasi zonasi juga dengan menambahkan pada `nano /etc/bind/named.conf.local` WISE:

```
zone "1.182.192.in-addr.arpa" {
  type master;
  file "/etc/bind/wise/1.182.192.in-addr.arpa";
};
```

![WISE-NameConf-Reverse](https://cdn.discordapp.com/attachments/855800698602913792/1035908784134029314/unknown.png)

Setelah itu, restart bind pada wise dengan command `service bind9 restart`

Lakukan testing pada client (SSS atau Garden) dengan memasukkan command berikut:

```
host -t PTR "192.182.1.2"
```

![Testing-4](https://cdn.discordapp.com/attachments/855800698602913792/1035909198342529154/unknown.png)

## Soal 5

Agar dapat tetap dihubungi jika server WISE bermasalah, buatlah juga Berlint sebagai DNS Slave untuk domain utama (5).

Untuk itu, ubah konfigurasi zonasi wise.c06.com pada `nano /etc/bind/named.conf.local` WISE seperti berikut:

```
zone "wise.C06.com" {
  type master;
  notify yes;
  also-notify { 192.182.3.3; };
  allow-transfer { 192.182.3.3; };
  file "/etc/bind/wise/wise.C06.com";
};
```

![WISE-NameConf-5](https://cdn.discordapp.com/attachments/855800698602913792/1035912516112498748/unknown.png)

Selain itu, tambah juga konfigurasi zonasi wise.c06.com pada `nano /etc/bind/named.conf.local` **Berlint** seperti berikut:

```
zone "wise.C06.com" {
   type slave;
   masters { 192.182.1.2; };
   file "/var/lib/bind/wise.C096.com";
};
```

Setelah itu, restart bind pada Berlint dengan command `service bind9 restart`

Untuk membuktikan pengujian benar, stop bind di wise terlebih dahulu dengan command `service bind9 stop`. Selanjutnya, tinggal melakukan testing pada client (SSS atau Garden) dengan memasukkan command berikut:

```
ping wise.C06.com
ping www.wise.C06.com
ping eden.wise.C06.com
ping www.eden.wise.C06.com
host -t CNAME www.eden.wise.C06.com
host -t PTR "192.182.1.2"
```

![Testing-5](https://cdn.discordapp.com/attachments/855800698602913792/1035913657185484820/unknown.png)

## Soal 8
Pada nomor 8 diminta untuk melakukan konfigurasi webserver pada alamat `www.wise.C06.com` dengan `DocumentRoot` pada `/var/www/wise.C06.com`. <br />
Untuk itu, untuk mendapatkan resource maka perlu dilakukan `wget` dengan perintah
```sh

```
Setelah itu dilakukan konfigurasi pada `/etc/apache2/sites-available` dengan menambahkan file `wise.C06.com.conf`. Konfigurasi yang dilakukan adalah sebagai berikut:
```
<VirtualHost *:80>
    ServerName wise.C06.com
    ServerAlias www.wise.C06.com
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/wise.C06.com
</VirtualHost>
```
Selanjutnya dilakukan 
```
a2ensite wise.C06.com
```
untuk melakukan enable pada site. <br />
Setelah site di-enable, apache2 direload dengan perintah:
```
service apache2 reload
```
Testing dilakukan di client dengan mengetikkan
```
lynx wise.C06.com
```

## Soal 9
Pada soal 9, dibutuhkan agar url www.wise.C06.com/index.php/home dapat menjadi menjadi www.wise.C06.com/home. <br />
Untuk itu di folder `/var/www/wise.C06.com` ditambahkan file `.htaccess` yang berisi:
```
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^([^\.]+)$ index.php/$1 [NC,L]
```
Kemudian pada konfigurasi site di `/etc/apache2/sites-available/wise.C06.com` ditambahkan konfigurasi sebagai berikut:
```
<Directory /var/www/wise.C06.com>
    Options +FollowSymLinks -Multiviews
    AllowOverride All
</Directory>
```
Karena diperlukan modul rewrite untuk memanipulasi url, maka perlu menjalankan perintah
```
a2enmod rewrite
```

## Soal 10
Pada soal 10 subdomain `eden.wise.C06.com` diperlukan penyimpanan asset pada `/var/www/eden.wise.C06.com`. <br />
Untuk itu, untuk mendapatkan resource maka perlu dilakukan `wget` dengan perintah
```sh

```
Setelah itu dilakukan konfigurasi di `/etc/apache2/sites-available/eden.wise.C06.com` sebagai berikut:
```
<VirtualHost *:80>
    ServerName eden.wise.C06.com
    ServerAlias www.eden.wise.C06.com
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/eden.wise.C06.com
</VirtualHost>
```
Selanjutnya situs perlu di-enable dengan command
```
a2ensite eden.wise.C06.com
```

## Soal 11
Pada soal 11, diminta untuk melakukan directory listing pada folder `/public`. Untuk melakukan directory listing, pada `/etc/apache2/sites-available/eden.wise.C06.com.conf` ditambahkan konfigurasi:
```
<Directory /var/www/eden.wise.C06.com/public>
        Options +Indexes
</Directory>
```
Lalu, karena hanya ingin melakukan directory listing tanpa dapat mengakses file di dalamnya maka ditambahkan:
```
<Directory /var/www/eden.wise.C06.com/public/*>
        Options -Indexes
</Directory>
```

## Soal 12
Karena ingin menggunakan custom error file, maka pada konfigurasi di `/etc/apache2/sites-available/eden.wise.C06.com` ditambahkan:
```
ErrorDocument 404 /error/404.html
```

## Soal 13
Agar url `www.eden.wise.C06.com/public/js` bisa diakses dengan membuka `www.eden.wise.C06.com/js` diperlukan directory alias pada konfigurasi `/etc/apache2/sites-available/eden.wise.C06.com`
```
Alias "/js" "/var/www/eden.wise.C06.com/public/js"
```
Karena sebelumnya direktori `js` tidak dapat diakses maka perlu ditambahkan:
```
<Directory /var/www/eden.wise.C06.com/public/js>
        Options +Indexes
</Directory>
```
## Soal 14
Pada soal 14 site `www.strix.operation.wise.C06.com` hanya bisa diakses dengan port 15000 dan 15500.
Untuk itu perlu dilakukan konfigurasi pada `/etc/apache2/ports.conf` dengan menambahkan:
```
Listen 15500
Listen 15000
```
Sehingga konfigurasi situsnya menjadi:
```
<VirtualHost *:15000 *:15500>
    ServerName strix.operation.wise.C06.com
    ServerAlias www.strix.operation.wise.C06.com
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/strix.operation.wise.C06.com
</VirtualHost>
```
## Soal 15
Untuk membuat autentikasi, pada folder `/etc/apache2` ditambahkan file `.htpasswd` dengan perintah
```
touch .httpasswd
```
Untuk membuat basic auth dilakukan perintah:
```
htpasswd -nb Twilight opStrix > /etc/apache2/.htpasswd
```
Kemudian pada konfigurasi site di `/etc/apache2/sites-available/strix.operation.wise.C06.com.conf` ditambahkan:
```
<Directory /var/www/strix.operation.wise.C06.com>
    AuthType Basic
    AuthName "Restricted Files"
    AuthUserFile /etc/apache2/.htpasswd
    Require valid-user
</Directory>
```
## Soal 17
Setiap gambar dengan substring `eden` akan dialihkan ke `eden.png` sehingga perlu ditambahkan file `.htaccess` pada `/var/www/eden.wise.C06.com/public/images/` sebagai berikut:
```
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule (.*)eden(.*)(png|jpg) http://eden.wise.C06.com/public/images/eden.png
```
Kemudian pada `/etc/apache2/sites-available/eden.wise.C06.com` perlu ditambahkan konfigurasi agar direktory imagesnya bisa diakses:
```
<Directory /var/www/eden.wise.C06.com/public/images>
    Options +FollowSymLinks -Multiviews +Indexes
    AllowOverride All
</Directory>
```
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

Sehingga saat IP Eden diakses (192.182.3.2) akan melakukan lookup pada situs http://wise.C06.com <br /> ![lookup](https://media.discordapp.net/attachments/964890423946543124/1035901332206387240/using.png) <br />
Hasilnya: <br /> ![hasil](https://media.discordapp.net/attachments/964890423946543124/1035901332206387240/using.png) <br />

**Sehingga untuk Revisi Nomor 16 SELESAI**
