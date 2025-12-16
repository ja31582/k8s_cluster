
### przygotowanie maszyny hosta

# install kvm
sudo apt update
sudo apt install -y qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virtinst virt-manager

## kvm check
lsmod | grep kvm

## virtualization check (> 0 – OK)
egrep -c '(vmx|svm)' /proc/cpuinfo

## turn on service
sudo systemctl enable --now libvirtd

## add yourself to group
sudo usermod -aG libvirt $(whoami)
sudo usermod -aG libvirt-qemu $(whoami)

# kvm network conf
sudo nano /etc/netplan/01-netcfg.yaml

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eno1:
      dhcp4: no
  bridges:
    br0:
      interfaces: [eno1]
      dhcp4: yes

```
sudo chmod 600 /etc/netplan/01-netcfg.yaml
sudo chown root:root /etc/netplan/01-netcfg.yaml
sudo netplan apply

# download iso
cd /var/lib/libvirt/images/
sudo wget https://releases.ubuntu.com/jammy/ubuntu-22.04.5-live-server-amd64.iso

virt-install \
  --name k8s-master \
  --ram 1024 \
  --vcpus 2 \
  --disk path=/mnt/kvm/images/k8s-master.qcow2,size=40 \
  --os-variant ubuntu22.04 \
  --cdrom /mnt/kvm/iso/ubuntu-22.04.5-live-server-amd64.iso \
  --network bridge=br0 \
  --graphics none

virt-install \
  --name k8s-worker1 \
  --ram 1536 \
  --vcpus 2 \
  --disk path=/var/lib/libvirt/images/k8s-worker1.qcow2,size=40 \
  --os-variant ubuntu22.04 \
  --cdrom /var/lib/libvirt/images/ubuntu-22.04.5-live-server-amd64.iso \
  --network bridge=br0 \
  --graphics none

virt-install \
  --name k8s-worker2 \
  --ram 1536 \
  --vcpus 2 \
  --disk path=/var/lib/libvirt/images/ubuntu-22.04.5-live-server-amd64.iso	 \
  --os-variant ubuntu22.04 \
  --cdrom /var/lib/libvirt/images/ubuntu-22.04.iso \
  --network bridge=br0 \
  --graphics none

 ---


# Pobierz najnowszą wersję (sprawdź najnowszą tutaj: https://github.com/argoproj/argo-cd/releases)
VERSION=$(curl --silent "https://api.github.com/repos/argoproj/argo-cd/releases/latest" | grep '"tag_name":' | sed -E 's/.*"v([^"]+)".*/\1/')
curl -sSL -o argocd https://github.com/argoproj/argo-cd/releases/download/v$VERSION/argocd-linux-amd64
chmod +x argocd
sudo mv argocd /usr/local/bin/

# sprawdzenie wersji
argocd version

# logowanie do ArgoCD
argocd login 10.0.0.80 --username admin --password kaLIst01! --insecure

# dodanie repozytorium
argocd repo add https://github.com/uzytkownik/nazwarepo.git

#sprawdzenie dodanych repozytoriów
argocd repo list

---
example-

repo: k8s_cluster
├─ metal-lb/
├─ grafana/
├─ nginx/

#utworzenie aplikacji ArgoCD dla MetalLB
argocd app create loadbalancer-app \
  --repo https://github.com/ja31582/k8s_cluster.git \
  --path ./metal-lb \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace metallb-system \
  --sync-policy automated

#utworzenie aplikacji ArgoCD dla Grafana
argocd app create grafana-app \
  --repo https://github.com/ja31582/k8s_cluster.git \
  --path ./grafana \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace monitoring \
  --sync-policy automated

argocd app create k8s-dashboard-app \
  --repo https://github.com/ja31582/k8s_cluster.git \
  --path ./k8s-dashboard \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace kubernetes-dashboard \
  --sync-policy automated  

argocd app create test-lb-nginx-app \
  --repo https://github.com/ja31582/k8s_cluster.git \
  --path ./test-lb-nginx \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace test-lb-nginx \
  --sync-policy automated    

argocd app create longhorn-app \
  --repo https://github.com/ja31582/k8s_cluster.git \
  --path ./longhorn \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace longhorn-system \
  --sync-policy automated    

argocd app create jfrog-app \
  --repo https://github.com/ja31582/k8s_cluster.git \
  --path ./jfrog-artifactory \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace jfrog \
  --sync-policy automated     

---
#listowanie aplikacji
argocd app list

#sprawdzenie statusu aplikacji
argocd app get loadbalancer-app









1. install Metal_LB (load ballancer)
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.15.3/config/manifests/metallb-native.yaml


#konfiguracja puli adresowej IP oraz reklamy L2 dla MetalLB