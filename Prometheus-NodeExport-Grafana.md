# Prometheus & Grafana

Created: November 30, 2022 3:18 PM
Etiket: Network Monitor
Kaynak: https://www.youtube.com/watch?v=4WWW2ZLEg74
OS: Ubuntu
Status: Test

# Grafana Nedir

Grafana; alınan tüm bilgileri görselleştirerek istenilen şekle sokar ve takip edilmesini ve okunmasını daha kolay hale getirir. Tablo şeklinde gördüğümüz tüm bilgiler, node_exporter tarafından elde ettiğimiz bilgilerdir.

# Doküman

[https://prometheus.io/download/](https://prometheus.io/download/) sitesinden hangi versiyonu indirmek istiyorsanız onu seçmeniz gerekmektedir, burada önemli nokta seçtiğiniz versiyon ***LTS*** (Long Term Support) uzun dönem desteği olan bir versiyonu seçmeniz sizin yararınıza olacaktır. Çünkü daha stabil ve güncelleme almaya devam edecek olduğunu belirtir.

Kuracağımız versiyon;

- Prometheus Versiyon : **2.37.4 Linux**
- Exporter Versiyon : **Node Exporter 1.5.0**
- Grafana Versiyon : **9.3.1 Enterprise(Standalone)**
[https://grafana.com/grafana/download](https://grafana.com/grafana/download)
- OS Distro : **Ubuntu 20.04**

# Kurulum

İlk olarak sistemimizi güncelliyoruz

```bash
apt update && apt upgrade -y
```

## 1 ) Prometheus Kurulumu

[https://prometheus.io/download/](https://prometheus.io/download/) sitesinden **Prometheus 2.37.4 Linux LTS** sürümünü indirmemiz gerekiyor. Seçtiğimiz sürümün üstüne sağ tıklayıp “Bağlantı Adresini Kopyala” dediğimizde indirme linkini elde edeceğiz.

```bash
curl -L -O https://github.com/prometheus/prometheus/releases/download/v2.37.4/prometheus-2.37.4.linux-amd64.tar.gz
```

İndirdiğimiz dosyayı tar komutu ile “/usr/local/bin” çıkartıyoruz. Sonra prometheus dizininin içine taşıyoruz.

```bash
tar xvfz prometheus-2.37.4.linux-amd64.tar.gz -C /usr/local/bin
mv /usr/local/bin/prometheus-2.37.4.linux-amd64 /usr/local/bin/prometheus
```

prometheus.yml dosyasında ki localhost bölümünü kendi sanal makinamın ip ile değiştiriyorum.

```bash
sed -i s/"localhost:9090"/"192.168.1.200:9090"/g /usr/local/bin/prometheus/prometheus.yml

```

/etc/systemd/system/ dizinin içine “ prometheus.service ” dosyasını oluşturuyoruz. Böylece sistem prometheus servisini tanıyabilecek.

```bash
cat << EOF > /etc/systemd/system/prometheus.service
[Unit]
Description=Prometheus Service
After=network.target
[Service]
Type=simple
ExecStart=/usr/local/bin/prometheus/prometheus --config.file=/usr/local/bin/prometheus/prometheus.yml
[Install]
WantedBy=multi-user.target
EOF
```

Güvenlik duvarından 9090 portuna izin veriyoruz.

```bash
ufw allow 9090/tcp && ufw status
```

Prometheus servisini sistem açılışında otomatik olarak başlaması için “ enable “. Servisi başlatmak için “ start “. Servisin durumunu görmek için “ status “ komutunu kullanıyoruz.

```bash
systemctl enable prometheus && systemctl start prometheus && systemctl status prometheus
```

Eğer bir problem olmazsa “ systemctl status prometheus “ komutu ile kontrol edebilir ve ya tarayıcımızdan 192.168.1.200:9090 adresine gittiğimizde menü çubuğundan Status altında target bölümünden de bakabiliriz.

![Untitled](Prometheus%20&%20Grafana%20e970e32886fe466082d0031432a8342f/Untitled.png)

## 2 ) Node_Exporter Kurulumu

[https://prometheus.io/download/](https://prometheus.io/download/) sitesinden node_exporter-1.5.0.linux-amd sürümünü indirmemiz gerekiyor. Seçtiğimiz sürümün üstüne sağ tıklayıp “Bağlantı Adresini Kopyala” dediğimizde indirme linkini elde edeceğiz.

```bash
curl -L -O https://github.com/prometheus/node_exporter/releases/download/v1.5.0/node_exporter-1.5.0.linux-amd64.tar.gz
```

İndirdiğimiz dosyayı tar komutu ile “/usr/local/bin” çıkartıyoruz. Sonra prometheus dizininin içine taşıyoruz.

```bash
tar -xvzf node_exporter-1.5.0.linux-amd64.tar.gz -C /usr/local/bin/
mv /usr/local/bin/node_exporter-1.5.0.linux-amd64 /usr/local/bin/node_exporter
```

/etc/systemd/system/ dizinin içine “ node-exporter.service ” dosyasını oluşturuyoruz. Böylece sistem node-exporter servisini tanıyabilecek.

```bash
cat << EOF > /etc/systemd/system/node-exporter.service
[Unit]
Description=Prometheus Node Exporter Service
After=network.target
[Service]
Type=simple
ExecStart=/usr/local/bin/node_exporter/node_exporter
[Install]
WantedBy=multi-user.target
EOF
```

/usr/local/bin/prometheus/prometheus.yml dosyasının içini tercih etiğiniz text editör ile node_exporter’i ekliyoruz.

```bash
cat << EOF >> /usr/local/bin/prometheus/prometheus.yml

  - job_name: "node_exporter"
    static_configs:
      - targets: ["192.168.1.200:9100"]
EOF
```

Güvenlik duvarından 9090 portuna izin veriyoruz.

```bash
ufw allow 9100/tcp && ufw status
```

node-exporter servisini sistem açılışında otomatik olarak başlaması için “ enable “. Servisi başlatmak için “ start “. Servisin durumunu görmek için “ status “ komutunu kullanıyoruz.

```bash
systemctl enable node-exporter && systemctl start node-exporter && systemctl status node-exporter
```

Tekrar “http://192.168.1.200:9090/targets?search=” adresine gittiğimizde prometheus ve node_exporteri görebileceğiz.

![Untitled](Prometheus%20&%20Grafana%20e970e32886fe466082d0031432a8342f/Untitled%201.png)

# 3 ) Grafana Kurulumu

İlk önce grana repolarını ekliyoruz

```bash
curl https://packages.grafana.com/gpg.key | sudo apt-key add -
add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
```

Sonra repomuzu güncelliyoruz ve grafana paketlerini kontrol edeceğiz

```bash
apt update -y

```

![Untitled](Prometheus%20&%20Grafana%20e970e32886fe466082d0031432a8342f/Untitled%202.png)

Burada “packages.grafana” eklendiğini görebiliyoruz repomuz eklendi ve güncellendi. Şimdi kurabiliriz.

```bash
apt install grafana -y
```

Kurulumdan sonra daemon-reload ile sistem dosyalarını yeniden yüklüyoruz

```bash
systemctl daemon-reload
```

Güvenlik duvarından 3000 portuna izin veriyoruz.

```bash
ufw allow 3000/tcp && ufw status
```

Grafana servislerini başlatıyoruz ve sunucu yeniden başladığında otomatik olarak grafana servisinin başlaması için “ enable “ yazıyoruz ve status ile servisin durumunu kontrol ediyoruz

```bash
systemctl start grafana-server && systemctl enable grafana-server && systemctl status grafana-server
```

# 3.a ) Grafana Giriş

Username : admin

password : admin

“ http://192.168.1.200:3000 “ adresine gittiğimizde bizi grafana login ekranı karşılayacak ön tanımlı isim ve parola ile giriş yaptıktan sonra direkt olarak yeni bir şifre oluşturmanızı isteyecektir.

![Untitled](Prometheus%20&%20Grafana%20e970e32886fe466082d0031432a8342f/Untitled%203.png)

# 3.b ) Grafana Ayarları

Sol menüden “ Configuration ” sekmesinden “ Data Sources “ bölümüne geliyoruz.

![Untitled](Prometheus%20&%20Grafana%20e970e32886fe466082d0031432a8342f/Untitled%204.png)

Add data source tıkladığımızda  “ Prometheus ” seçiyoruz.

![Untitled](Prometheus%20&%20Grafana%20e970e32886fe466082d0031432a8342f/Untitled%205.png)

Bu bölümde sunucumuzun ip adresini (http://192.168.1.200:9090) ve prometheus portunu ekliyoruz. Save & test diyoruz. 

![Untitled](Prometheus%20&%20Grafana%20e970e32886fe466082d0031432a8342f/Untitled%206.png)

Sonra “ Dashboards ” menüsünün altında “ Import “  sayfasına tıklıyoruz.

![Untitled](Prometheus%20&%20Grafana%20e970e32886fe466082d0031432a8342f/Untitled%207.png)

Import via grafana.com bölümüne “ 14513 “ yazıyor ve load diyoruz.

Bu eklediğimiz numaralar aslında önceden oluşturulmuş Dashboard grafik şemalarıdır. Daha fazlası için “ [https://grafana.com/grafana/dashboards/](https://grafana.com/grafana/dashboards/) “ adresinden bakabilirsiniz.

![Untitled](Prometheus%20&%20Grafana%20e970e32886fe466082d0031432a8342f/Untitled%208.png)

Premetheus seçip import diyoruz.

![Untitled](Prometheus%20&%20Grafana%20e970e32886fe466082d0031432a8342f/Untitled%209.png)

Artık sistemimizin durumunu grafana üzerinden takip edip kontrol edebiliriz.