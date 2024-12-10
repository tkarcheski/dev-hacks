To use Kubernetes with the `kubernetes-jammy` repository on Ubuntu 24.04 LTS (Codename: Noble), follow these steps:

---

### 1. **Remove Existing Kubernetes Repository**
First, clean up any existing Kubernetes repository configurations:
```bash
sudo rm /etc/apt/sources.list.d/kubernetes.list
```

---

### 2. **Add the `kubernetes-jammy` Repository**
The `kubernetes-jammy` repository is compatible with recent Ubuntu LTS versions:

```bash
sudo tee /etc/apt/sources.list.d/kubernetes.list <<EOF
deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-jammy main
EOF
```

---

### 3. **Add the Kubernetes Keyring**
Add the secure keyring required for the Kubernetes repository:

```bash
sudo apt-get install -y curl apt-transport-https
curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /usr/share/keyrings/kubernetes-archive-keyring.gpg
```

---

### 4. **Update and Install Kubernetes Components**
Update the system package index and install Kubernetes components:
```bash
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

- The `hold` command ensures that the installed versions of these components are not automatically updated by the package manager.

---

### 5. **Verify Installation**
Confirm the installation of Kubernetes components:
```bash
kubeadm version
kubectl version --client
kubelet --version
```

---

### 6. **Initialize Kubernetes Cluster**
For a single-node cluster, initialize Kubernetes with a pod network CIDR:
```bash
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
```

---

### 7. **Set Up `kubectl` Configuration**
After initialization, set up `kubectl` for the current user:
```bash
mkdir -p $HOME/.kube
sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

---

### 8. **Install a Pod Network**
Install a pod network add-on like Calico:
```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

---

### 9. **Join Additional Nodes (Optional)**
For multi-node clusters, use the `kubeadm join` command generated during the initialization step to add worker nodes.

---

### Troubleshooting Tips
- If you encounter any issues during installation, verify your architecture with `uname -m` (ensure it is `x86_64`).
- Ensure your system has at least 2 CPUs and 2GB of RAM for a minimal setup.

---

This setup should work seamlessly on Ubuntu 24.04 LTS with the `kubernetes-jammy` repository.
