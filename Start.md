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
### Register Hostname
```shell
sudo echo "{Master의 IP 주소} {Master의 Hostname}" | sudo tee -a /etc/hosts
sudo echo "{DB Server의 IP 주소} {DB Server의 Hostname}" | sudo tee -a /etc/hosts
```

### Server Module Install
```shell
sudo kubeadm init --apiserver-advertise-address={kubernetes 마스터의IP}
sudo mkdir -p "${HOME}"/.kube
sudo cp -i /etc/kubernetes/admin.conf "${HOME}"/.kube/config
sudo chown "$(id -u)":"$(id -g)" "${HOME}"/.kube/config
```

### Node Setting
```shell
# Node Label
kubectl label nodes {Master의 Hostname} deploy=true
kubectl label nodes {Master의 Hostname} app=true
kubectl label nodes {Master의 Hostname} gpushare=true
kubectl label nodes {DB Server의 Hostname} db=true
kubectl label nodes {Master의 Hostname} master=true

# Node Taint
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
```
#### Plugins
####### calico
```shell
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

####### gpushare
<details>
<summary>device-plugin-ds.yaml</summary>

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: gpushare-device-plugin-ds
  namespace: kube-system
spec:
  selector:
    matchLabels:
      component: gpushare-device-plugin
      app: gpushare
      name: gpushare-device-plugin-ds
  template:
    metadata:
      annotations:
        scheduler.alpha.kubernetes.io/critical-pod: ""
      labels:
        component: gpushare-device-plugin
        app: gpushare
        name: gpushare-device-plugin-ds
    spec:
      serviceAccount: gpushare-device-plugin
      hostNetwork: true
      nodeSelector:
        gpushare: "true"
      containers:
        - image: seculayer/gpushare-device-plugin:2.0
          name: gpushare
          # Make this pod as Guaranteed pod which will never be evicted because of node's resource consumption.
          command:
            - gpushare-device-plugin-v2
            - -logtostderr
            - --v=5
            - --memory-unit=MiB
          resources:
            limits:
              memory: "300Mi"
              cpu: "1"
            requests:
              memory: "300Mi"
              cpu: "1"
          env:
            - name: KUBECONFIG
              value: /etc/kubernetes/kubelet.conf
            - name: NODE_NAME
              valueFrom:
                fieldRef:
                  fieldPath: spec.nodeName
          securityContext:
            allowPrivilegeEscalation: false
            capabilities:
              drop: ["ALL"]
          volumeMounts:
            - name: timezone-config
              mountPath: /etc/localtime
            - name: device-plugin
              mountPath: /var/lib/kubelet/device-plugins
      volumes:
        - name: timezone-config
          hostPath:
            path: /usr/share/zoneinfo/Asia/Seoul
        - name: device-plugin
          hostPath:
            path: /var/lib/kubelet/device-plugins
```
</details>

<details>
<summary>device-plugin-rbac.yaml</summary>

```yaml
# rbac.yaml
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: gpushare-device-plugin
rules:
  - apiGroups:
      - ""
    resources:
      - nodes
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - ""
    resources:
      - events
    verbs:
      - create
      - patch
  - apiGroups:
      - ""
    resources:
      - pods
    verbs:
      - update
      - patch
      - get
      - list
      - watch
  - apiGroups:
      - ""
    resources:
      - nodes/status
    verbs:
      - patch
      - update
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: gpushare-device-plugin
  namespace: kube-system
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: gpushare-device-plugin
  namespace: kube-system
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: gpushare-device-plugin
subjects:
  - kind: ServiceAccount
    name: gpushare-device-plugin
    namespace: kube-system
```
</details>

<details>
<summary>gpushare-schd-extender.yaml</summary>

```yaml
# rbac.yaml
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: gpushare-schd-extender
rules:
  - apiGroups:
      - ""
    resources:
      - nodes
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - ""
    resources:
      - events
    verbs:
      - create
      - patch
  - apiGroups:
      - ""
    resources:
      - pods
    verbs:
      - update
      - patch
      - get
      - list
      - watch
  - apiGroups:
      - ""
    resources:
      - bindings
      - pods/binding
    verbs:
      - create
  - apiGroups:
      - ""
    resources:
      - configmaps
    verbs:
      - get
      - list
      - watch
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: gpushare-schd-extender
  namespace: kube-system
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: gpushare-schd-extender
  namespace: kube-system
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: gpushare-schd-extender
subjects:
  - kind: ServiceAccount
    name: gpushare-schd-extender
    namespace: kube-system

# deployment yaml
---
kind: Deployment
apiVersion: apps/v1
metadata:
  name: gpushare-schd-extender
  namespace: kube-system
spec:
  replicas: 1
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: gpushare
      component: gpushare-schd-extender
  template:
    metadata:
      labels:
        app: gpushare
        component: gpushare-schd-extender
      annotations:
        scheduler.alpha.kubernetes.io/critical-pod: ''
    spec:
      hostNetwork: true
      tolerations:
        - effect: NoSchedule
          operator: Exists
          key: node-role.kubernetes.io/control-plane
        - effect: NoSchedule
          operator: Exists
          key: node.cloudprovider.kubernetes.io/uninitialized
      nodeSelector:
        node-role.kubernetes.io/control-plane: ""
      serviceAccount: gpushare-schd-extender
      containers:
        - name: gpushare-schd-extender
          image: seculayer/gpushare-scheduler-extender:2.0
          env:
            - name: LOG_LEVEL
              value: debug
            - name: PORT
              value: "12345"
          volumeMounts:
            - name: timezone-config
              mountPath: /etc/localtime
      volumes:
        - name: timezone-config
          hostPath:
            path: /usr/share/zoneinfo/Asia/Seoul

# service.yaml
---
apiVersion: v1
kind: Service
metadata:
  name: gpushare-schd-extender
  namespace: kube-system
  labels:
    app: gpushare
    component: gpushare-schd-extender
spec:
  type: NodePort
  ports:
    - port: 12345
      name: http
      targetPort: 12345
      nodePort: 32766
  selector:
    # select app=ingress-nginx pods
    app: gpushare
    component: gpushare-schd-extender

```
</details>

<details>
<summary>kube-scheduler.yaml</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    component: kube-scheduler
    tier: control-plane
  name: kube-scheduler
  namespace: kube-system
spec:
  containers:
    - command:
        - kube-scheduler
        - --authentication-kubeconfig=/etc/kubernetes/scheduler.conf
        - --authorization-kubeconfig=/etc/kubernetes/scheduler.conf
        - --bind-address=127.0.0.1
        - --kubeconfig=/etc/kubernetes/scheduler.conf
        - --leader-elect=true
        - --config=/etc/kubernetes/scheduler-policy-config.yaml
      image: registry.k8s.io/kube-scheduler:v1.22.4
      imagePullPolicy: IfNotPresent
      livenessProbe:
        failureThreshold: 8
        httpGet:
          host: 127.0.0.1
          path: /healthz
          port: 10259
          scheme: HTTPS
        initialDelaySeconds: 10
        periodSeconds: 10
        timeoutSeconds: 15
      name: kube-scheduler
      resources:
        requests:
          cpu: 100m
      startupProbe:
        failureThreshold: 24
        httpGet:
          host: 127.0.0.1
          path: /healthz
          port: 10259
          scheme: HTTPS
        initialDelaySeconds: 10
        periodSeconds: 10
        timeoutSeconds: 15
      volumeMounts:
        - mountPath: /etc/kubernetes/scheduler.conf
          name: kubeconfig
          readOnly: true
        - mountPath: /etc/kubernetes/scheduler-policy-config.yaml
          name: scheduler-policy-config
          readOnly: true
  hostNetwork: true
  priorityClassName: system-node-critical
  securityContext:
    seccompProfile:
      type: RuntimeDefault
  volumes:
    - hostPath:
        path: /etc/kubernetes/scheduler.conf
        type: FileOrCreate
      name: kubeconfig
    - hostPath:
        path: /etc/kubernetes/scheduler-policy-config.yaml
        type: FileOrCreate
      name: scheduler-policy-config
status: {}
```
</details>

<details>
<summary>scheduler-policy-config.yaml</summary>

```yaml
---
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
clientConnection:
  kubeconfig: /etc/kubernetes/scheduler.conf
extenders:
  - urlPrefix: "http://127.0.0.1:32766/gpushare-scheduler"
    filterVerb: filter
    bindVerb: bind
    enableHTTPS: false
    nodeCacheCapable: true
    managedResources:
      - name: seculayer.com/gpu-mem
        ignoredByScheduler: false
    ignorable: false
```
</details>

```shell

```

#### DB Server
#### MRMS

## 접은 제목
접은 내용
