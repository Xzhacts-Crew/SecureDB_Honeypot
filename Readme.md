# Secure Database Server MYSQL and OPENSSH
## 
#
| Nama  | Nim | Role |
| ----- | --- | ---- |
| Indra Bagas Pratama (1stind)   | 22.83.0859  | System Security Administrator |
| Tri Kustiyani (chocojace) | 23.83.0859  | Build Service Database |
| Deny Ardiansyah (ardiansyahpotter) | 23.83.0909  | Build Service SSH SFTP |
| M. Aidhil Fitrah (AshXyzz) | 23.83.0841 | Secure & Build Honeypot |
| M. Akbar Baihaqqy | 23.83.0866  | Secure & Build Honeypot |
| Fatihul Faqih H. | 23.83.0848  | Secure Honeypot |

## Deskripsi
- Secure Database server Mysql PhpMyadmin
- Secure File Transfer Protocol OpenSSH
- Secure Shell with Honeypot
- Pembatasan Hak akses akses file dan folder dan Autentifikasi login
- Secure Port with Firewall UFW, PortKnocking, 
- Konfigutasi ip table untuk menghindariserangan DDOS dengan rule Iptables
- Menonaktifkan listing folder dan menghapus versi dan port apache
- Instalasi Fail2ban untuk memblok lalu lintas jaringan serangan brute-force dan serangan berbasis pola.
- Instalasi Rkhunter untuk mengecek root backdoor yg ada pada sistem

## Requirements
- Ubuntu Server 22.04
- Storange minimal 40 GB
- RAM 4 GB ++

## Skenario
Kami memiliki 2 virtual mesin (VM), dengan sistem operasi ubuntu server
1. VM1 (Superadmin) adalah virtual mesin yang memiliki service database server dan SFTP server
2. VM2 (hnypt) adalah virtual mesin sebagai honeypots dari server utama yang diinstalasi IPS IDS untuk bisa mengamati log bila terjadi serangan
3. Ketika VM1 diserang secara remote ssh port 22, VM1 akan dikonfigurasi dengan Iptables untuk membelokkan serangan ke VM2. Dan ketika serangan berhasil dialihkan makan penyerang hanya akan terjebak didalam VM2 dan segala log aktivitasnya akan tercatat pada file log  

## Topologi
* VM1 IP Address 192.168.10.11/24
* VM2 IP Address 192.168.10.12/24
* VM1 dan VM2 terhubung secara lokal dan dikonfigurasi untuk loadbalancing Haproxy

## Installation Service
* Configure Apache
* Configure Database Server Mysql
* Configure PhpMyadmin
* Configure OpenSSH 
[Follow this link](https://github.com/1stind/Server-DB_SFTP_Mail)

## Secure Apache and PhpMyadmin
Step 1: Update Repo

```sh
apt update && upgrade -y
```
Step 2: Membuat autentifikasi pada website apache
```sh
htpasswd -c /etc/apache2/.htpasswd superadmin
```
Step 3: Menonaktifkan listing folder apache
```sh
a2dismod --force autoindex
```
```sh
Module autoindex already disabled
```
Step 3: Menonaktifkan listing folder apache
```sh
/etc/apache2/conf-available/security.conf
```
```sh
#Nonaktifkan server signature
#ServerSignature Off
#ServerSignature On
```
* foto1
* foto2 

Step 3: Mengubah sub folder default PhpMyadmin
```sh
nano /etc/apache2/conf-available/phpmyadmin.conf
```
```sh
# phpMyAdmin default Apache configuration

Alias /superdb /usr/share/phpmyadmin

<Directory /usr/share/phpmyadmin>
    AuthType Basic
    AuthName "Restricted Content"
    AuthUserFile /etc/apache2/.htpasswd
    Require valid-user
```
```sh
systemctl restart apache2
```
* Video Gif aset1

### Secure SFTP OpenSSH-Server
Step 1: Konfigurasi
* Ubah port default
* Disable root login
* Batasi akses 
* 
```sh
nano /etc/ssh/sshd_config
```
```sh
Include /etc/ssh/sshd_config.d/*.conf

Port 2222 #ubah port default
#AddressFamily any
#ListenAddress 0.0.0.0

# Authentication:
#LoginGraceTime 2m
PasswordAuthentication yes
PermitRootLogin no #menonaktifkan akses melalui root
#StrictModes yes
MaxAuthTries 3 #membatasi maksimal kesalahan akses sebanyak 3x
#MaxSessions 10

ChrootDirectory none #disable akses direktori root saat SFTP
Subsystem   sftp     internal-sftp #membuat akses internal hanya pada folder user
AllowUsers superftp@192.168.10.11 superadmin@192.168.56.5
#membuat batasan akses SFTP hanya untuk user superftp pada ip 192.168.10.5
#membuat batasan jalur akses SSH hanya melalui user superadmin pada ip 192.168.10.5
```
* Video gif aset2

## Konfigurasi Firewall UFW
Step 1: Aktifkan UFW
```sh
enable ufw
```
```sh
ufw status verbose
```
Step 2: Deny port yang tidak digunakan
```sh
ufw default deny incoming
```
```sh
ufw default allow outgoing
```
```sh
ufw deny 22
```
```sh
ufw allow 2222
```
* lakukan deny pada port yang terbuka dan tidak digunakan
```sh
ufw allow from 192.168.10.0/24 
```
* hanya mengijinkan akses port dari ip yang terdaftar pada UFW


Step 4: Install Php
```sh
apt-get install php php-mysql libapache2-mod-php php-cli php-cgi php-gd mysql-server mysql-client zip -y
```
Step 5: Install PhpMyadmin

```sh
apt install phpmyadmin -y
```
* configure phpmyadmin pada pilihan apache2 dan lighttpd, sesuaikan pada service web server yang digunakan
* configure database with dbconfig-common. y

Step 5: hak akses direktori phpmyadmin

```sh
 ln -s /etc/phpmyadmin/apache.conf /etc/apache2/conf-available/phpmyadmin.conf
```
```sh
a2enconf phpmyadmin.conf
```
```sh
systemctl restart apache2
```
Step 6: Testing

```sh
http:192.168.0.0/phpmyadmin
```
* ganti ip 192.168.0.0 sesuai ip pada ubuntu yang digunakan

### Secure Shell & Secure File Transfer Protocol With OpenSSH
Step 1: Install openssh-server
```sh
apt install openssh-server
```
Step 2: Konfigurasi openssh-server
```sh
nano /etc/ssh/sshd.conf
```
* 
*
*

Step 3: Testing remote SSH
```sh
ssh root@192.168.0.0
```
Step 4: Testing SFTP
```sh
sftp://192.168.0.0:22
```
* 
*

### Configure DHCP Server
Step 1: Install ISC DHCP Server
```sh
apt install isc-dhcp-server
```
Step 2: Konfigurasi 
* tentukan dulu network interfaces dan network address yang nantinya akan dibuatkan dhcp server
```sh
nano /etc/default/isc-dhcp-server
```
* 

```sh
nano /etc/dhcp/dhcpd.conf
```
Step 3: Testing
```sh
systemctl restart isc-dhcp-server
```
* 


### Configure Mail Server Roundcube Webmail
Step 1: Download roundcube
```sh
wget https://github.com/roundcube/roundcubemail/releases/download/1.5.2/roundcubemail-1.5.2-complete.tar.gz
```
Step 2: Install
```sh
apt install bind9 dnsutils postfix dovecot-imapd apache2 php mariadb-server openssl composer php-{net-smtp,mysql,gd,xml,mbstring,intl,zip,json,pear,bz2,gmp,imap,imagick,auth-sasl,mail-mime,net-ldap3,net-sieve,curl} libapache2-mod-php curl -y
```
* configure console Postfix akan muncul, pertama Kita pilih Internet Site
* isikan domain mail name

Step 3: Konfigurasi DNS
* copy db.local dan db.127 untuk membuat zone forward & reverse

```sh
cp /etc/bind/db.local db.id
```
```sh
cp /etc/bind/db.127 db.192
```
Step 3: Testing
```sh
systemctl restart isc-dhcp-server
```
Step 3: Testing
```sh
systemctl restart isc-dhcp-server
```
Step 3: Testing
```sh
systemctl restart isc-dhcp-server
```
Step 3: Testing
```sh
systemctl restart isc-dhcp-server
```

> Note: `--capt-add=SYS-ADMIN` is required for PDF rendering.

Verify the deployment by navigating to your server address in
your preferred browser.

```sh
127.0.0.1:8000
```

## Honeypot

Download Suricata
```sh
wget https://www.openinfosecfoundation.org/download/suricata-6.0.8.tar.gz
```

Extract file
```sh
tar xzf suricata-6.0.8.tar.gz
```

Ke directory dan configure
```sh
cd suricata-6.0.8
./configure --enable-nfqueue --prefix=/usr --sysconfdir=/etc --localstatedir=/var
```

Install suricata
```sh
make
make install-full
```

Konfigurasi suricata
```sh
nano /etc/suricata/suricata.yaml
```

Verify file konfigurasi
```sh
suricata -T -c /etc/suricata/suricata.yaml -v
```

Jalankan suricata secara manual
```sh
suricata -D -c /etc/suricata/suricata.yaml -i eth0
```

Selanjutnya, kembali ke server Suricata dan periksa file log Suricata
```sh
tail -f /var/log/suricata/fast.log
```

Contoh Output nya akan seperti ini:
```sh
10/18/2022-14:01:38.569298  [**] [1:2210008:2] SURICATA STREAM 3way handshake SYN resend different seq on SYN recv [**] [Classification: Generic Protocol Command Decode] [Priority: 3] {TCP} 157.32.37.21:59188 -> 209.23.10.188:80
10/18/2022-14:01:38.569304  [**] [1:2210004:2] SURICATA STREAM 3way handshake SYNACK resend with different ack [**] [Classification: Generic Protocol Command Decode] [Priority: 3] {TCP} 209.23.10.188:80 -> 157.32.37.21:59188
10/18/2022-14:01:38.569649  [**] [1:2210008:2] SURICATA STREAM 3way handshake SYN resend different seq on SYN recv [**] [Classification: Generic Protocol Command Decode] [Priority: 3] {TCP} 157.32.37.21:53343 -> 209.23.10.188:80
10/18/2022-14:01:38.569655  [**] [1:2210004:2] SURICATA STREAM 3way handshake SYNACK resend with different ack [**] [Classification: Generic Protocol Command Decode] [Priority: 3] {TCP} 209.23.10.188:80 -> 157.32.37.21:53343
10/18/2022-14:01:38.570762  [**] [1:2210008:2] SURICATA STREAM 3way handshake SYN resend different seq on SYN recv [**] [Classification: Generic Protocol Command Decode] [Priority: 3] {TCP} 157.32.37.21:62070 -> 209.23.10.188:80
10/18/2022-14:01:38.570770  [**] [1:2210004:2] SURICATA STREAM 3way handshake SYNACK resend with different ack [**] [Classification: Generic Protocol Command Decode] [Priority: 3] {TCP} 209.23.10.188:80 -> 157.32.37.21:62070
10/18/2022-14:01:38.571748  [**] [1:2210008:2] SURICATA STREAM 3way handshake SYN resend different seq on SYN recv [**] [Classification: Generic Protocol Command Decode] [Priority: 3] {TCP} 157.32.37.21:5001 -> 209.23.10.188:80
```
>Note : Contoh tersebut jika ada yang menyerang menggunakan DDoS

**Sekian Terima Kasih**





   [dill]: <https://github.com/joemccann/dillinger>
   [git-repo-url]: <https://github.com/joemccann/dillinger.git>
   [john gruber]: <http://daringfireball.net>
   [df1]: <http://daringfireball.net/projects/markdown/>
   [markdown-it]: <https://github.com/markdown-it/markdown-it>
   [Ace Editor]: <http://ace.ajax.org>
   [node.js]: <http://nodejs.org>
   [Twitter Bootstrap]: <http://twitter.github.com/bootstrap/>
   [jQuery]: <http://jquery.com>
   [@tjholowaychuk]: <http://twitter.com/tjholowaychuk>
   [express]: <http://expressjs.com>
   [AngularJS]: <http://angularjs.org>
   [Gulp]: <http://gulpjs.com>

   [PlDb]: <https://github.com/joemccann/dillinger/tree/master/plugins/dropbox/README.md>
   [PlGh]: <https://github.com/joemccann/dillinger/tree/master/plugins/github/README.md>
   [PlGd]: <https://github.com/joemccann/dillinger/tree/master/plugins/googledrive/README.md>
   [PlOd]: <https://github.com/joemccann/dillinger/tree/master/plugins/onedrive/README.md>
   [PlMe]: <https://github.com/joemccann/dillinger/tree/master/plugins/medium/README.md>
   [PlGa]: <https://github.com/RahulHP/dillinger/blob/master/plugins/googleanalytics/README.md>
