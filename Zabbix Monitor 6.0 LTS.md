# Zabbix Monitor 6.0 LTS

Created: November 26, 2022 2:55 PM
Etiket: Network Monitor
Kaynak: https://www.zabbix.com/download
Status: Complete 🙌

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

**A ) ilk önce paket yöneticimize zabbix’in repolarını ekliyoruz.** 

```bash
wget https://repo.zabbix.com/zabbix/6.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.0-4%2Bubuntu20.04_all.deb
dpkg -i zabbix-release_6.0-4+ubuntu20.04_all.deb
apt update
```

<aside>
✨ apt update yaptığınızda;
” [https://repo.zabbix.com/](https://repo.zabbix.com/zabbix-agent2-plugins/1/ubuntu) “ gibi ifadeler görmüyorsanız bu repo eklenmemiş demektir.

</aside>

**B ) Zabbix çalışması için gerekli paketlerin  kurulumu**;

```bash
apt -y install zabbix-server-pgsql zabbix-frontend-php php7.4-pgsql zabbix-apache-conf zabbix-sql-scripts zabbix-agent
```

**C )** **Veri tabanı ayarları**

```bash

sudo -u postgres createuser --pwprompt zabbix # Bir paralo belirliyoruz 'passw0rd'
sudo -u postgres createdb -O zabbix zabbix

# Zabbix-sql ile PostgresSQL eşliyoruz

zcat /usr/share/zabbix-sql-scripts/postgresql/server.sql.gz | sudo -u zabbix psql zabbix
```

**D )** **Zabbix Server Konfigürasyonu**

Hangi text editör kullanıyorsanız onunla “ /etc/zabbix/zabbix_server.conf ” dosyasını düzenlememiz gerekiyor.  “ # DBPassword= ” olan ifadeyi öncellikle başında ki “ # ” kaldırıp yorum satırı olmaktan çıkarıyoruz sonra daha önce belirlediğimiz parolayı yazıyoruz.

```bash
DBPassword=passw0rd
```

**E )** **Servisleri Başlatma**

```bash
systemctl restart zabbix-server zabbix-agent apache2
systemctl enable zabbix-server zabbix-agent apache2
# enable aslında otomatik başlat. Sistem kapanıp açıldığında otomatik olarak zabbix başlayacak.
```

F ) **Zabbix Ayarları**

Bundan sonra artık sunucumuzun ip ile tarayıcımızdan giriş yapabiliriz.

localhost/zabbix adresine girdiğimizde bizi “Wellcome to Zabbix 6.0 “ ekran karşılayacaktır.

Configure DB Connection adımında “password” kısmına veri tabanı için verdiğimiz parolayı giriyoruz. “passw0rd“ 

![alt text](https://i.hizliresim.com/7hr6zdf.png)

Eğer hiç bir sorun oluşmazsa “Congratulations! You have successfully installed Zabbix fronted” diyecektir.

G ) **Zabbix Giriş**

Varsayılan olarak bu bilgiler ile zabbix paneline giriş yapıyoruz.

Kullanıcı adı : Admin

Parolası : zabbix

<aside>
✨ ilk işiniz Administration/Users bölümünden Admin için varsayılan parolayı değiştirmelisiniz.

</aside>

![alt text](https://i.hizliresim.com/4o8ar6y.png)

![alt text](https://i.hizliresim.com/ear95h5.png)
