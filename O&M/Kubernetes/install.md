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

master：

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

work：

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

## Calico

### 1. 安装operator

```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.29.2/manifests/tigera-operator.yaml
```

### 2. 下载YAML清单模板

```bash
curl https://raw.githubusercontent.com/projectcalico/calico/v3.29.2/manifests/custom-resources.yaml -O
```

**内容：**、

```yaml
# This section includes base Calico installation configuration.
# For more information, see: https://docs.tigera.io/calico/latest/reference/installation/api#operator.tigera.io/v1.Installation
apiVersion: operator.tigera.io/v1
kind: Installation
metadata:
  name: default
spec:
  # Configures Calico networking.
  calicoNetwork:
    ipPools:
    - name: default-ipv4-ippool
      blockSize: 26
      cidr: 172.168.0.0/16 ## 将cidr改为你在引导控制平面时设置的cidr
      encapsulation: VXLANCrossSubnet
      natOutgoing: Enabled
      nodeSelector: all()

---

# This section configures the Calico API server.
# For more information, see: https://docs.tigera.io/calico/latest/reference/installation/api#operator.tigera.io/v1.APIServer
apiVersion: operator.tigera.io/v1
kind: APIServer
metadata:
  name: default
spec: {}
```

### 安装calico

```bash
kubectl create -f custom-resources.yaml
```

### 验证

```bash
watch kubectl get pods -n calico-system
```

### 安装calicoctl作为kubectl的插件

```bash
cd /usr/local/bin/ && while [ ! -f kubectl-calico ]; do sudo curl -L https://github.com/projectcalico/calico/releases/download/v3.29.2/calicoctl-linux-amd64 -o kubectl-calico || sleep 5; done && sudo chmod +x kubectl-calico
```

