# Zabbix Monitor 6.0 LTS

Created: November 26, 2022 2:55 PM
Etiket: Network Monitor
Kaynak: https://www.zabbix.com/download
Status: Test

# Zabbix Nedir

Alexei Vladishev tarafından geliştirilen açık kaynaklı bir izleme yazılımdır. Bu yazılım sayesinde donanımları, servislerin durumlarını takip edebilir ve sorunları tespit edebilirsiniz.

# Doküman

[https://www.zabbix.com/download](https://www.zabbix.com/download) sitesinden hangi versiyonu indirmek istiyorsanız onu seçmeniz gerekmektedir, burada önemli nokta seçtiğiniz versiyon ***LTS*** (Long Term Support) uzun dönem desteği olan bir versiyonu seçmeniz sizin yararınıza olacaktır. Çünkü daha stabil ve güncelleme almaya devam edecek olduğunu belirtir.

Kurmak istediğiniz sürüme karar verdiğinizde site size repo adresini ve izlemeniz gereken adımları sunuyor olacak.

Kuracağımız versiyon;

- Zabbix versiyon : **6.0 LTS**
- OS Distro : **Ubuntu 20.04**
- Zabbix Component : **Server, Frontend, Agent**
- Database : ****PostgreSQL {Eğer postgresql kurulu değilse bu adresten kurabilirsiniz -**** [https://www.postgresql.org/download/linux/ubuntu/](https://www.postgresql.org/download/linux/ubuntu/)****}****
- Web Server: **Apache**

# Kurulum

**A )** ilk önce paket yöneticimize zabbix’in repolarını ekliyoruz. 

```bash
wget https://repo.zabbix.com/zabbix/6.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.0-4%2Bubuntu20.04_all.deb
dpkg -i zabbix-release_6.0-4+ubuntu20.04_all.deb
apt update
```

<aside>
✨ apt update yaptığınızda;
” [https://repo.zabbix.com/](https://repo.zabbix.com/zabbix-agent2-plugins/1/ubuntu) “ gibi ifadeler görmüyorsanız bu repo eklenmemiş demektir.

</aside>

**B ) Z**abbix çalışması için gerekli paketlerin  kurulumu;

```bash
apt -y install zabbix-server-pgsql zabbix-frontend-php php7.4-pgsql zabbix-apache-conf zabbix-sql-scripts zabbix-agent
```

**C )** Veri tabanı ayarları

```bash

sudo -u postgres createuser --pwprompt zabbix # Bir paralo belirliyoruz 'passw0rd'
sudo -u postgres createdb -O zabbix zabbix

# Zabbix-sql ile PostgresSQL eşliyoruz

zcat /usr/share/zabbix-sql-scripts/postgresql/server.sql.gz | sudo -u zabbix psql zabbix
```

**D )** Zabbix Server Konfigürasyonu

Hangi text editör kullanıyorsanız onunla “ /etc/zabbix/zabbix_server.conf ” dosyasını düzenlememiz gerekiyor.  “ # DBPassword= ” olan ifadeyi öncellikle başında ki “ # ” kaldırıp yorum satırı olmaktan çıkarıyoruz sonra daha önce belirlediğimiz parolayı yazıyoruz.

```bash
DBPassword=passw0rd
```

**E )** Servisleri Başlatma

```bash
systemctl restart zabbix-server zabbix-agent apache2
systemctl enable zabbix-server zabbix-agent apache2
# enable aslında otomatik başlat. Sistem kapanıp açıldığında otomatik olarak zabbix başlayacak.
```

Bundan sonra artık sunucumuzun ip ile tarayıcımızdan giriş yapabiliriz.

localhost/zabbix adresine girdiğimizde bizi bu ekran karşılayacaktır.

![Untitled](Untitled.png)

F ) Zabbix Ayarları

Burada ki password kısmına önceden oluşturduğumuz “ passw0rd “ yazıyoruz

![Untitled](Untitled%201.png)

Eğer hiç bir sorun oluşmazsa bu ekranı görmüş olacaksınız.

![Untitled](Untitled%202.png)

G ) Zabbix Giriş

Varsayılan olarak bu bilgiler ile giriş yapıyoruz.

Kullanıcı adı : Admin

Parolası : zabbix

![Untitled](Untitled%203.png)

<aside>
✨ ilk işiniz Administration/Users bölümünden Admin için varsayılan parolayı değiştirmelisiniz.

</aside>

![Untitled](Untitled%204.png)

![Untitled](Untitled%205.png)