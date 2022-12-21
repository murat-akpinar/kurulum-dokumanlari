# Murat Akpınar Task

Created: December 15, 2022 8:38 PM
Etiket: Servisler
Status: Test

# 1 ) OpenSSH Kurulumu

Her şeyden önce repolarımızı güncelliyoruz.

```bash
sudo apt update
```

Open SSH’ı kurmak için;

```bash
sudo apt install openssh-server -y
```

Open SSH’ı kurduktan sonra “**systemctl**” komutu ile servisimizin başlatmak için “**start**”, servis her sistem yeniden başladığında otomatik olarak başlaması için “**enable**” ve servisimizi durumunu öğrenmek için “**status**” komutlarını kullanacağız.

```bash
sudo systemctl start ssh && sudo systemctl enable ssh && sudo systemctl status ssh
```

# 1.a ) OpenSSH Key

A ) İlk önce SSH bağlantısı için kullanacağımız “baglanti” adında bir kullanıcı oluşturalım.

```bash
useradd baglanti -m -s /bin/bash
```

B ) OpenSSH Key ile bağlantı kurmak için ilk önce kendi bilgisayarımız bağlantı isteği atacağımız bilgisayarda yapacağımız işlemler.

```bash
ssh-keygen -t rsa -b 4096
```

Bu komutu yazıktan sonra bir anahtar üretimine başlayacaktır.  Bizden bu key’i nereye kayıt edeceğinizi soracaktır buraya “/home/<kullanıcı-ismi>/.ssh/baglanti-key” ismini vereceğim. Ondan sonra bizden “passphrase” isteyecektir isterseniz bir parola belirleyebilirsiniz ama ben boş bırakacağım. Bu isimde iki dosya elde etmiş olacağız “baglanti-key ve baglanti-key.pub” 

<aside>
💡 Eğer “Enter file in which to save the key” bölümünü boş bıraksaydık kullanıcı home dizinin için de “.ssh” dizinine bu dosyaları “id_rsa ve id_rsa.pub“ olarak oluşturacaktı.

</aside>

Bu yarattığımız key’i bağlanacağımız sunucuya yüklemek için ssh-copy-id komutunu kullanacağız.

```bash
ssh-copy-id baglanti@192.168.1.200
```

Bu komutu yazdıktan sonra “baglanti” kullanıcısının parolasını isteyecek ve girdiğimizde ssh key’i sunucuya yüklemiş olacaktır.

# 1.b) OpenSSH Ayarları

SSH bağlantısı için sistemimizin ip bilgisi (192.168.1.5) ve bir kullanıcı ismi gereklidir. Bu yüzden openssh servisimizin güvenliği artırmak için /etc/ssh/ dizinine “sshd_config” dosyasında OpenSSH ayarlarını yapacağız.

C ) Sistemimizin daha güvenli hale getirmek için OpenSSH ile gerçekleşecek bağlantılar için;

Parola ile doğrulamayı kapatmamız gerekir ve ssh key bağlantısını aktif hale getirmemiz gerekir. Bunun için tercih ettiğiniz text editör ile “/etc/ssh/sshd_config” dosyasını elinizle düzeltebilirsiniz veya “sed” komutu ile değişiklikleri sağlayabilirsiniz.

```bash
sudo sed -i s/"#PasswordAuthentication yes"/"PasswordAuthentication no"/g /etc/ssh/sshd_config
sudo sed -i s/"#PubkeyAuthentication yes"/"PubkeyAuthentication yes"/g /etc/ssh/sshd_config
```

Bu yaptığımız değişikliklerin geçerli olması için servisimizi “restart” yapmamız gerekiyor.

```bash
sudo systemctl restart ssh && sudo systemctl status ssh
```

# 2 ) Apache Kurulumu

İlk önce repolarımızı tekrar bir güncelleyelim.

```bash
sudo apt update
```

Apache servisini kurmak için;

```bash
apt install apache2
```

Apache servisinin otomatik olarak başlaması ve durumunu kontrol etmek için;

```bash
sudo systemctl enable apache2 && sudo systemctl status apache2
```

# 3 ) Güvenlik Duvarı

İlk önce güvenlik duvarımızı çalışıp çalışmadığını kontrol ediyoruz.

```bash
ufw status
```

Eğer “inactive” ise “ ufw enable “ komutu ile aktif hale getiriyoruz. İlk önce sistemimize olan input ve output isteklerini engelliyoruz.

```bash
sudo ufw default deny incoming && sudo ufw default allow outgoing
```

Apache ve OpenSSH için güvenlik duvarı izinlerini veriyoruz. Yaptığımız değişiklikleri geçerli olması için "ufw enable” komutunu kullanıyoruz. 

```bash
sudo ufw allow ssh
sudo ufw allow http
ufw enable
```

Hangi portlara izin verdiğimizi görmek için “ ufw status “ komutunu kullanıyoruz.

```bash
$sudo ufw status
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
22/tcp (v6)                ALLOW       Anywhere (v6)
80/tcp (v6)                ALLOW       Anywhere (v6)
```

Ön tanımlı olarak SSH 22 portu, Apache 80 portunu kullanıyor.

# 4 ) Domain Ayarları

İlk önce  /var/www dizinin içine “bugday ve ozgurstaj2002” için iki ayrı dizin oluşturuyoruz.

```bash
mkdir /var/www/html/bugday.org
mkdir /var/www/html/ozgurstaj2022.com
```

Bu dizinleri oluşturduk fakat bunu sahiplikleri ve gurupları “root” oldu bunları apache2 servisinin kullanıcısına vereceğim(www-data).

```bash
sudo chown www-data:www-data bugday.org
sudo chown www-data:www-data ozgurstaj2022.com
```

Sahiplikleri ve guruplarını ayarladık şimdi bu iki dizinin izinlerini vereceğiz

```bash
sudo chmod -R 755 /var/www

# Owner için 7 = Okuma, Yazma Çalıştırma
# Group için 5 = Okuma ve Çalıştırma
# Other için 5 = Okuma ve Çalıştırma
```

# 4.a ) Virtual Host Ayarları

/etc/apache2/sites-available/000-default.conf dosyasını bugday ve ozgurstaj2022 için değiştireceğiz bu yüzden “000-default.conf” dosyasının bugday.org.conf ve ozgurstaj2022.com.conf adında kopyalarını oluşturacağız.

```bash
sudo cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/bugday.org.conf
sudo cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/ozgurstaj2022.com.conf
```

Tercih ettiğiniz text editör ile bugday.org ve buğday.org için bu bölümü içine ekliyoruz. Türkçe karakterli domain için punycode converter ile buğday.org için “xn—bugday-l1a.org” elde ediyoruz

```bash

<VirtualHost *:80>

        ServerAdmin admin@bugday.org
        ServerName bugday.org
        ServerAlias bugday.org
        DocumentRoot /var/www/html/bugday.org

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

<VirtualHost *:80>

        ServerAdmin admin@bugday.org
        ServerName xn--buday-l1a.org
        ServerAlias buğday.org
        DocumentRoot /var/www/html/bugday.org

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Tercih ettiğiniz text editör ile ozgurstaj2022.com ve www.ozgurstaj2022.com için bu bölümü içine ekliyoruz.

```bash

<VirtualHost *:80>

        ServerAdmin admin@ozgurstaj2022.com
        ServerName ozgurstaj2022.com
        ServerAlias ozgurstaj2022.com
        DocumentRoot /var/www/html/ozgurstaj2022.com

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

<VirtualHost *:80>

        ServerAdmin admin@ozgurstaj2022.com
        ServerName www.ozgurstaj2022.com
        ServerAlias wwwozgurstaj2022.com
        DocumentRoot /var/www/html/ozgurstaj2022.com

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Oluşturduğumuz bu config dosyalarını etkinleştirmek için için “a2ensite" komutunu kullanacağız.

```bash
sudo a2ensite bugday.org.conf
sudo a2ensite ozgurstaj2022.com.conf
```

Yaptığımız değişiklikleri servisi tekrar başlatmadan sadece dosyalarda yaptığımız değişikliklerin geçerli olması için “reload” kullanıyoruz.

```bash
sudo systemctl reload apache2 && sudo systemctl status apache2
```

# 5 ) Özgurstaj2022.com

ilk önce repolarımızı güncelliyoruz.

```bash
sudo apt update
```

“ Kullanıcılarımın kişisel verilerini toplamayacağım “ 100 kere yazmak için sistemimize PHP paketini yüklüyoruz.

```bash
sudo apt install php -y
```

Tercih ettiğimiz text editörü ile ozgurstaj2022 dizinin içine “index.php” dosyasını oluşturuyoruz

“vim /var/www/html/ozgurstaj2022.com/index.php”

```bash

<html>
  <head>
    <meta charset="utf-8">
    <title>ozgur staj 2022</title>
  </head>
  <body>

    <?php
      for ($i=1; $i<=100; $i++) 
        {
         echo "Kullanıcılarımın kişisel verilerini toplamayacağım" . "</br>";
        }
    ?>
</body>
</html>
```

# 5.a ) Parola Koruması

ilk önce “yonetim” adında bir dizin oluşturuyoruz.

```bash
mkdir /var/www/html/ozgurstaj2022.com/yonetim
```

“htpasswd” komutu ile bir kullanıcı oluşturacağız bu komutu yazıktan sonra bizden bir parola belirlememizi isteyecektir.

```bash
htpasswd -c /etc/apache2/.htpasswd isim
```

Sonrasında “/etc/apache2/sites-available/000-default.conf” dosyasına bu bölümü ekliyoruz.

```bash
...
      <Directory "/var/www/html/ozgurstaj2022.com/yonetim">
       AuthType Basic
       AuthName "Restricted Content"
       AuthUserFile /etc/apache2/.htpasswd
       Require valid-user
      </Directory>
...
```

Yaptığımız değişikliklerin geçerli olması için tekrar servisimizi yeniden başlatıyoruz.

```bash
sudo systemctl reload apache2 && systemctl status apache2
```

# 6) bugday.org

Wordpress için ilk önce bize veri tabanı gerekiyor bunun için mariadb kullanacağım. Bunun için mariadb repolarını sistemimize ekliyoruz ve repolarımızı güncelliyoruz.

Eğer curl yüklü değilse “sudo apt install curl ile yükleyebiliriz”.

```bash
sudo curl -LsS https://r.mariadb.com/downloads/mariadb_repo_setup | sudo bash
sudo apt update
```

mariadb için kurmamız gereken mariadb-server, client ve bacup paketlerini kuracağız.  Ayrıca php-mysql kurmamız gerekiyor mysql ile php kendi aralarında iletişim kurabilmesi için gerekiyor

```bash
sudo apt install mariadb-server mariadb-client mariadb-backup php-mysql -y
```

Mariadb çalıştığını kontrol etmek ve sistemimiz yeniden başladığında otomatik olarak çalışması için “enable” ve “status” kullanacağız.

```bash
sudo systemctl enable mariadb && sudo systemctl status mariadb
```

# 6.a ) Mariadb Ayarları

Yüklediğimiz mariadb varsayılan olarak bazı veri tabanları ve kullanıcı ayarları ile geliyor bunları düzenlemek için "mariadb-secure-installation” kullanıyoruz.

```bash
mariadb-secure-installation

```

Burada test database ve anonymous useri siliyoruz.

- Switch to unix_socket authentication = N
- Change the root password = N
- Remove anonymous users? = Y
- Disallow root login remotely = Y
- Remove test database and access to it? = Y
- Reload privilege tables now? = Y

# 7 ) Mariadb Veri Tabanı - Kullanıcı Oluşturma

Wordpress tablolarının tutulacağı ve bağlantı kuracağı bir kullanıcı yaratmamız gerekiyor. Bunun için “mariadb” yazıp veri tabanı içine giriyoruz burada en önemli şey yazdığımız sorguların sonuna mutlaka “;” noktalı virgül koymayı unutmayın.

İlk önce databse oluşturuyoruz.

```bash
CREATE DATABASE bugday_db;
```

Oluşturduğumuz database kontrol edebilmek için veri tabanını listeliyoruz.

```bash
SHOW DATABASES;
```

Kullanıcı oluşturmak için ilk önce onun için bir parola vereceğiz parolamız “dbpassw0d” olacak. Bu parolayı “hash” şeklinde alacağız.

```bash
SELECT PASSWORD('dbpassw0rd');
# *3AA1FAA1EA7983922FDB92DE6F199972B231227A oluşturduğumuz hash
```

Veri tabanı kullanıcımız bugday_db_user olacak oluşturduğumuz parola hash ile oluşturacağız.

```bash
CREATE USER bugday_db_user@localhost IDENTIFIED BY PASSWORD '*3AA1FAA1EA7983922FDB92DE6F199972B231227A';
```

Oluşturduğumuz kullanıcıyı kontrol edebilmek için veri tabanını kullanıcılarını listeliyoruz.

```bash
SELECT host,user FROM mysql.user;
```

Kullanıcı ve database oluşturduk şimdi oluşturduğumuz kullanıcıyı yetkilendireceğiz vereceğiz.

```bash
GRANT ALL PRIVILEGES ON bugday_db.* TO bugday_db_user@localhost IDENTIFIED BY PASSWORD '*3AA1FAA1EA7983922FDB92DE6F199972B231227A';
```

Bu işlemden sonra mariadb’nin izinler için tutuğu ön belliği tazeliyoruz.

```bash
FLUSH PRIVILEGES;
```

Yaptıklarımızı kontrol etmek için

```bash
SHOW GRANTS FOR bugday_db_user@localhost;
```

# 8 ) Wordpress Kurulumu

Wordpress için gerekli php paketlerini kuruyoruz;

```bash
sudo apt install php-gd php-zip php-fpm -y
```

Apache rewrite enable hali getiriyoruz çünkü sitemiz de oluşturacağımız yazıların url kısımlarını daha okunabilir bir hale getiriyor.

```bash
a2enmod rewrite
sudo systemctl restart apache2
```

İlk önce wordpress kurulum dosyasını indiriyoruz.

```bash
wget https://wordpress.org/latest.tar.gz
```

İndirdikten sonra tar ile dosyası çıkatıyoruz.

```bash
tar -xzf latest.tar.gz
```

Çıkardığımız wordpress dosyalarını “/var/www/html/bugday.org” içerisine kopyalıyoruz.

```bash
cp -R wordpress/* /var/www/html/bugday.org
```

Kurulum sayfasına geçmeden önce bu dosyaları “root” olarak taşıdığımız için dosya ve dizin sahiplikleri değişti bunları tekrar ayarlamamız gerekiyor.

```bash
sudo chown -R www-data:www-data /var/www/html/bugday.org

```

# 9 ) Bugday.org Wordpress Ayarları

Sunucumuzun ip ile tarayıcımızdan girdiğimizde wordpress kurulum sayfasında önceden belirlediğimiz veri tabanı kullanıcı ismi ve tablo isimlerini yazıyoruz.

Yada /var/www/html/wp-config.php dosyasını elimizle düzenleyebiliriz.