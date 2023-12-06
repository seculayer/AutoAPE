# AutoAPE 시작하기
> 추후 업데이트 예정입니다.

## 준비
* Nvidia GPU가 탑재된 워크스테이션 또는 서버
### GPU Support
* CUDA Computing Capability >= 3.5


### Dependencies
* docker >= 19.03.12
* kubernetes >= 1.22.4
* Nvidia GPU Driver >= 440.82


### 설치 및 테스트 완료 OS
* Linux
    * CentOS == 7.8.2003
    * Oracle Linux == 8.5

## Install Guide
```shell
sudo systemctl stop firewalld
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=disabled$/' /etc/selinux/config
sudo sed -i 's/^SELINUX=permissive/SELINUX=disabled$/' /etc/selinux/config
sudo swapoff -a
sudo sed -ri '/\sswap\s/s/^#?/#/' /etc/fstab
```
### Required packages Install
```shell
sudo yum install -y yum-utils device-mapper-persistent-data lvm2 sshpass net-tools iproute-tc wget
wget https://raw.githubusercontent.com/seculayer/AutoAPE-parent/main/conf/k8s.conf
sudo cp ./k8s.conf /etc/sysctl.d/
sudo sysctl --system
```
### Docker & Kubernetes Install
```shell
DOCKER_VERSION=19.03.15-3.el8
sudo yum config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install -y docker-ce-"${DOCKER_VERSION}"
sudo mkdir /etc/docker

# GPU Version
wget https://raw.githubusercontent.com/seculayer/AutoAPE-parent/main/conf/daemon-gpu.json



wget https://raw.githubusercontent.com/seculayer/AutoAPE-parent/main/conf/daemon.json
sudo cp daemon.json /etc/docker/daemon.json

sudo systemctl daemon-reload
sudo systemctl restart docker
sudo chown -R "${USER}":"${USER}" /eyeCloudAI/data
sudo usermod -aG docker "${USER}"
sudo chmod 666 /var/run/docker.sock
sudo systemctl enable docker

sudo yum config-manager --add-repo=https://raw.githubusercontent.com/seculayer/AutoAPE-parent/main/conf/kubernetes.repo
sudo yum install -y kubelet-"${KUBE_VERSION}" kubeadm-"${KUBE_VERSION}" kubectl-"${KUBE_VERSION}" --disableexcludes=kubernetes
sudo systemctl enable kubelet
kubeadm config images pull
```
### Server Module Install
```shell
sudo kubeadm init --apiserver-advertise-address={kubernetes 마스터의IP}
```
#### Plugins
```shell
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```
#### DB Server
#### MRMS

