## How to install k8s

### 1. 添加Host解析

```bash
echo 127.0.0.1  $(hostname) | sudo  tee -a  /etc/hosts
```

---

### 2. 安装containerd

```bash
sudo dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
sudo dnf update -y && sudo dnf install -y containerd.io
```

---

### 3. 配置`systemd` cgroup驱动

````bash
containerd config default | sudo tee /etc/containerd/config.toml
````
**更改配置文件：**
结合 runc 使用 systemd cgroup 驱动，在 /etc/containerd/config.toml 中设置：

````ini
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true ## 原本是false
````

**重启containerd**

```bash
sudo systemctl enable --now containerd 
```

---

### 3. 开启端口转发

```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
EOF

# 应用 sysctl 参数而不重新启动
sudo sysctl --system
```

---

### 4. 配置conatinerd代理

```bash
sudo mkdir -p /etc/systemd/system/containerd.service.d/
cat << EOF | sudo tee /etc/systemd/system/containerd.service.d/http-proxy.conf
[Service]
Environment="HTTP_PROXY=http://ip:port"
Environment="HTTPS_PROXY=http://ip:port"
Environment="NO_PROXY=localhost,127.0.0.1,.yourdomain.com"
EOF
```

---

### 5. 关闭selinux

```bash
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
```

---

### 6. 开放防火墙

**master：**

```bash
# 开放 Kubernetes API 服务器端口
sudo firewall-cmd --add-port=6443/tcp --permanent
# 开放 etcd 服务器客户端 API 端口
sudo firewall-cmd --add-port=2379-2380/tcp --permanent
# 开放 kubelet API 端口
sudo firewall-cmd --add-port=10250/tcp --permanent
# 开放 kube-scheduler 端口
sudo firewall-cmd --add-port=10259/tcp --permanent
# 开放 kube-controller-manager 端口
sudo firewall-cmd --add-port=10257/tcp --permanent
# 应用更改
sudo firewall-cmd --reload
```

**work：**

```bash
# 开放 kubelet API 端口
sudo firewall-cmd --add-port=10250/tcp --permanent

# 开放 kube-proxy 端口
sudo firewall-cmd --add-port=10256/tcp --permanent

# 开放 NodePort Services 端口范围
sudo firewall-cmd --add-port=30000-32767/tcp --permanent

# 应用更改
sudo firewall-cmd --reload
```

---

### 7. 安装kubelet、kubeadm、kubectl

```bash
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.32/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.32/rpm/repodata/repomd.xml.key
exclude=kubelet kubeadm kubectl cri-tools kubernetes-cni
EOF
sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
```

---

### 8. 创建控制平面

```bash
sudo kubeadm config images pull
sudo kubeadm init --pod-network-cidr=172.168.0.0/16
```

---

## cilium Network
