Works on Debian bookworm
> with kubeadm

### Basic install
```sh
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
```

### ip forward

```sh
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
EOF

# Apply sysctl params without reboot
sudo /sbin/sysctl --system

# check if config set
/sbin/sysctl net.ipv4.ip_forward
```

### Install docker

```sh
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

```

```sh
sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

```sh
sudo groupadd docker
sduo usermod -aG docker $USER

newgrp docker
docker run hello-world
```

```sh

containerd config default | sed 's/SystemdCgroup = false/SystemdCgroup = true /' | sed 's/sandbox_iamge = "registry.k8s.io\/pause:3.6"/sandbox_image = "registry.k8s.io\/pause:3.9"/' | sudo tee /etc/containerd/config.toml

```

### swapoff

```sh
sudo /sbin/swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

```sh
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list


sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

```sh
sudo kubeadm init --pod-network-cidr=10.200.0.0/16

kubeadm list token

openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex

# from worker node
sudo kubeadm join 192.168.1.15:6443 --token  <token> --discovery-token-ca-cert-hash <sha-hash>
```


### Flannel setup

```sh
wget https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml

# change the cidr address and deploy

kubectl apply -f kube-flanne.yml
```

### Reference

| **CIDR Notation**              | **Network Part**               | **Host Part**                              | **Total Usable IPs**                  |
|---------------------------------|--------------------------------|--------------------------------------------|---------------------------------------|
| `/24` (e.g., `192.168.1.0/24`)  | `192.168.1` (24 bits)          | Last octet (8 bits) for hosts (e.g., `0-255`) | 254 (since 2 addresses are reserved)  |
| `/16` (e.g., `192.168.0.0/16`)  | `192.168` (16 bits)            | Last two octets (16 bits) for hosts (e.g., `0.0-255.255`) | 65,534 (since 2 addresses are reserved) |

