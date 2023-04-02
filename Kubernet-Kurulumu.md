# Kubernet Kurulumu

İlk önce bize iki tane biri master biri worker olacak şekilde birer makine lazım. Virtualbox üzerine iki tane ubuntu 20.04 LTS kurdum. 
Eğer  güvenlik duvarı portları ile uğraşmak istemezsenis, komple güvenlik duvarını "sudo systemctl stop ufw && sudo systemctl disable ufw" kullanabilirsiniz.

Bu adımları her iki sistemde uyguluyoruz.

#### Bunu master olarak kullacağımız sisteme yazıyoruz. ####

```bash
sudo hostnamectl set-hostname master-node
exec bash
```
---
#### Bunu worker olacak diğer sistemimize yazıyoruz. ####

```bash
sudo hostnamectl set-hostname worker-node
exec bash
```
---
#### **master-node** için güvenlik duvarı ayarları ####

```bash
sudo systemctl enable ufw && sudo systemctl start ufw
sudo ufw enable

sudo ufw allow 22/tcp
sudo ufw allow 6443/tcp
sudo ufw allow 2379/tcp
sudo ufw allow 2380/tcp
sudo ufw allow 10248/tcp
sudo ufw allow 10250/tcp
sudo ufw allow 10251/tcp
sudo ufw allow 10252/tcp
sudo ufw allow 10255/tcp
sudo ufw allow 10250/tcp
sudo ufw reload
```
---

#### **worker-node** için güvenlik duvarı ayarları ####

```bash
sudo systemctl enable ufw && sudo systemctl start ufw
sudo ufw enable

sudo ufw allow 22/tcp
sudo ufw allow 10248/tcp
sudo ufw allow 10250/tcp
sudo ufw allow 30000:32767/tcp
sudo ufw reload
```
---

#### Her iki sistem için : swap alanlarını kapatıyoruz ####
```bash
sudo swapoff -a
sudo sed -i s/"[/]swap"/"#swap"/g /etc/fstab
```
---
#### her iki sistem için : iptables bridged ayarları ####
```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF
```
```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
```
```bash
sudo sysctl --system
```

---
#### her iki sistem için : containerd kurulumu ####

```bash
sudo modprobe overlay
sudo modprobe br_netfilter
```
```bash
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF
```
```bash
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
```
```bash
sudo sysctl --system
```

---
#### her iki sistem için : ilk önce repoları güncelliyoruz ve upgrade yapıyoruz. Unutmayın her iki sistemde uyguluyoruz bunları. ####

```bash
sudo apt-get update && sudo apt-get upgrade -y
```

```bash
sudo apt-get install containerd -y
```

---
#### her iki sistem için : containerd kurulduktan sonra bir dizin oluşturuyoruz ####

```bash
sudo mkdir -p /etc/containerd
```

---
#### her iki sistem için : root kullanıcısına geçiyoruz ve bu adımı uyguluyoruz. Bu işlemi uyguladıktan sonra root kullanıcısından çıkın. 
Normal kullanıcıya geçmeyi unutmayın! ####

```bash
su -i
containerd config default | tee /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd
exit
```

---
#### her iki sistem için : kubeadm kurulumu için repolarımıza kubernet ekliyoruz. ####

```bash
sudo apt-get install -y apt-transport-https ca-certificates curl
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

---
#### her iki sistem için : sonra tekrar repoları güncelleyip yükleme işlemine geçiyoruz. ####

```bash
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
```

---
#### her iki sistem için : isteğe bağlı olarak bu paketlerin güncellemerlini kapataibliriz. ####

```bash
sudo apt-mark hold kubelet kubeadm kubectl
```
---

## Buraya dikakt buradan sonra ki adımlar sadece master-node için uygulanacak adımlardır. ##
```bash
sudo kubeadm config images pull
```

---
Ondan sonra sisteme master nod olarak ekliyoruz. “192.168.1.200” Bu kısma kendi master-node makinenizin ip adresini girmeniz gerekiyor.
```bash
sudo kubeadm init --pod-network-cidr=192.168.0.0/16 --apiserver-advertise-address=192.168.1.200 --control-plane-endpoint=192.168.1.200
```

---
Bu işlemi ardından eğer kurulum başarılı olduysa bu ekran sizi karşılayacak. 
```bash
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of control-plane nodes by copying certificate authorities
and service account keys on each node and then running the following as root:

  kubeadm join 192.168.1.200:6443 --token 3qr4l6.41gcohu09570pu2b \
        --discovery-token-ca-cert-hash sha256:c61dd1a8c531adc26e5a84d066cc02a320ce9f90069ba4d502428abe07acc43a \
        --control-plane

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.1.200:6443 --token 3qr4l6.41gcohu09570pu2b \
        --discovery-token-ca-cert-hash sha256:c61dd1a8c531adc26e5a84d066cc02a320ce9f90069ba4d502428abe07acc43a
```

---
Buradan sonra gene master-node sisteminde bu adımları uygulacağız.
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

---
Bu komutları **master-node** çalıştıyoruz.
```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/tigera-operator.yaml
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/custom-resources.yaml
```

---
Burada diğer önemli olan “kubeadm join” kısmı bu kısmı **worker-node** yazıyoruz.
```bash
sudo kubeadm join 192.168.1.200:6443 --token 3qr4l6.41gcohu09570pu2b \
        --discovery-token-ca-cert-hash sha256:c61dd1a8c531adc26e5a84d066cc02a320ce9f90069ba4d502428abe07acc43a
```

---
Eğer join tokeninizi kaybederseniz tekrar üretmeniz için bu komutu kullanabilirsiniz.
```
sudo kubeadm token create --print-join-command
```

---
Nodları görmek için "kubectl get nodes" komutu kullanıyoruz ve bu çıktıyı alıyorsak her şey yolunda.

```bash
user@master-node:~$ kubectl get nodes
NAME          STATUS   ROLES           AGE    VERSION
master-node   Ready    control-plane   101m   v1.26.3
worker-node   Ready    <none>          94m    v1.26.3
```

---
# **ÖNEMLİ NOT:** #

“kubectl get nodes” yazdığınızda böyle bir hata alıyorsanız.

“The connection to the server 192.168.1.200:6443 was refused - did you specify the right host or port?”

Bu hatanın sebebi sanırım swap alanı ile ilgili. bu hatayı alırsanız yapanız sırasıyla yapmanız gerekenler bu adımlardır.

```bash
sudo -i
swapoff -a
exit
strace -eopenat kubectl version
sudo sed -i s/"[/]swap"/"#swap"/g /etc/fstab
```
kaynak : [https://discuss.kubernetes.io/t/the-connection-to-the-server-host-6443-was-refused-did-you-specify-the-right-host-or-port/552/5](https://discuss.kubernetes.io/t/the-connection-to-the-server-host-6443-was-refused-did-you-specify-the-right-host-or-port/552/5)

---
