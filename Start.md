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
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=disabled/' /etc/selinux/config
sudo sed -i 's/^SELINUX=permissive$/SELINUX=disabled/' /etc/selinux/config
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
* IP 주소, Hostname 수정 필요 
sudo echo "{Master의 IP 주소} {Master의 Hostname}" | sudo tee -a /etc/hosts
sudo echo "{DB Server의 IP 주소} database.eyecloudai" | sudo tee -a /etc/hosts
# sudo echo "{Master의 IP 주소} registry.seculayer.com" | sudo tee -a /etc/hosts
```

### Make Directory
```shell
sudo mkdir -p /eyeCloudAI/app/ape/custom/cnvrtr
sudo mkdir -p /eyeCloudAI/data/processing/ape/jobs
sudo mkdir -p /eyeCloudAI/data/processing/ape/da
sudo mkdir -p /eyeCloudAI/data/processing/ape/errors
sudo mkdir -p /eyeCloudAI/data/processing/ape/results
sudo mkdir -p /eyeCloudAI/data/processing/ape/rtdetect
sudo mkdir -p /eyeCloudAI/data/processing/ape/temp
sudo mkdir -p /eyeCloudAI/data/processing/ape/division
sudo mkdir -p /eyeCloudAI/data/processing/ape/detects
sudo mkdir -p /eyeCloudAI/data/storage/ape/models
```

### Server Module Install
```shell
* IP 주소 수정 필요 
sudo kubeadm init --apiserver-advertise-address={kubernetes 마스터의IP}
sudo mkdir -p "${HOME}"/.kube
sudo cp -i /etc/kubernetes/admin.conf "${HOME}"/.kube/config
sudo chown "$(id -u)":"$(id -g)" "${HOME}"/.kube/config
```

### Node Setting
```shell
# Node Label (* Hostname 수정 필요)
kubectl label nodes {Master의 Hostname} deploy=true
kubectl label nodes {Master의 Hostname} app=true
kubectl label nodes {Master의 Hostname} gpushare=true
kubectl label nodes {DB Server의 Hostname} db=true
kubectl label nodes {Master의 Hostname} master=true

# Node Taint
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
```
#### Plugins
##### calico
```shell
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

##### gpushare
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
# Gpushare Image Pull
# docker pull seculayer/gpushare-scheduler-extender:2.0
# docker pull seculayer/gpushare-device-plugin:2.0

# Gpushare Start
kubectl apply -f device-plugin-ds.yaml
kubectl apply -f device-plugin-rbac.yaml
kubectl apply -f gpushare-schd-extender.yaml

# Gpushare Config Apply
sudo cp scheduler-policy-config.yaml /etc/kubernetes/scheduler-policy-config.yaml
sudo cp kube-scheduler.yaml /etc/kubernetes/manifests/kube-scheduler.yaml
```

<!-- - registry
<details>
<summary>registry-deploy.yaml</summary>

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: docker-registry
  labels:
    app: docker-registry
  namespace: [@namespace]
spec:
  replicas: 1
  selector:
    matchLabels:
      app: docker-registry
  template:
    metadata:
      labels:
        app: docker-registry
    spec:
      containers:
        - name: docker-registry
          image: registry:2.7.1
          ports:
            - containerPort: 5000
          volumeMounts:
            - name: docker-images
              mountPath: /var/lib/registry/docker/registry/v2
            - name: timezone-config
              mountPath: /etc/localtime
      volumes:
        - name: docker-images
          hostPath:
            path: [@app_dir]/data/storage/docker-images
            type: Directory
        - name: timezone-config
          hostPath:
            path: /usr/share/zoneinfo/Asia/Seoul
      nodeSelector:
        deploy: "true"
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: docker-registry
  name: docker-registry-svc
  namespace: [@namespace]
spec:
  type: NodePort
  selector:
    app: docker-registry
  #    release: docker-registry
  ports:
    - name: docker-registry-http
      port: 5000
      protocol: TCP
      targetPort: 5000
      nodePort: 31500
---
```
</details> -->

#### Namespace
<details>
<summary>ml-namespace.yaml</summary>

```yaml
---
apiVersion: v1
kind: Namespace
metadata:
  name: apeautoml
---
```
</details>

```shell
kubectl apply -f ml-namespace.yaml
```
#### DB Server
<details>
<summary>ml-db-deploy.yaml (* DB 폴더 경로 수정 필요)</summary>

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: apeautoml-db
  labels:
    app: apeautoml-db
  namespace: apeautoml
spec:
  replicas: 1
  selector:
    matchLabels:
      app: apeautoml-db
  template:
    metadata:
      labels:
        app: apeautoml-db
    spec:
      containers:
        - name: apeautoml-db
          image: seculayer/ape-db:1.0.0
          ports:
            - containerPort: 3306
          volumeMounts:
            - name: db-data
              mountPath: /var/lib/mysql
            - name: timezone-config
              mountPath: /etc/localtime
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mariadb-user
                  key: password
            - name: MYSQL_DATABASE
              value: APEAutoML
            - name: MYSQL_USER
              valueFrom:
                secretKeyRef:
                  name: mariadb-user
                  key: username
            - name: MYSQL_ROOT_HOST
              value: '%'
            - name: MYSQL_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mariadb-user
                  key: password
      volumes:
        - name: db-data
          hostPath:
            path: {DB 폴더 경로}
            type: Directory
        - name: timezone-config
          hostPath:
            path: /usr/share/zoneinfo/Asia/Seoul
      nodeSelector:
        db: "true"
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: apeautoml-db
  name:  apeautoml-db-svc
  namespace: apeautoml
spec:
  type: NodePort
  selector:
    app:  apeautoml-db
  ports:
    - name: apeautoml-db
      port: 3306
      protocol: TCP
      targetPort: 3306
      nodePort: 30306
---
```
</details>

```shell
# Create DB Config Folder(User Permission)(* DB 폴더 경로 수정 필요) 
mkdir -p {DB 폴더 경로}

# Create Secret
kubectl create secret generic mariadb-user --from-literal=username={DB User Name} --from-literal=password={DB Password} --namespace=apeautoml

# Image Pull
# docker pull seculayer/ape-db:1.0.0

# DB Start
kubectl apply -f ml-db-deploy.yaml
```

#### Configmap
<details>
<summary>APEFlow(* Master 서버 유저명, 비밀번호 수정 필요)</summary>

apeflow-conf.xml
```xml
<configuration>
    <property>
        <name>app_dir</name>
        <value>/eyeCloudAI</value>
        <description>Application root directory</description>
    </property>
    <property>
        <name>dir_processing</name>
        <value>/processing/ape</value>
        <description>None</description>
    </property>
    <property>
        <name>dir_temp</name>
        <value>/temp</value>
        <description>None</description>
    </property>
    <!-- log setting -->
    <property>
        <name>dir_log</name>
        <value>/eyeCloudAI/logs</value>
        <description>None</description>
    </property>
    <property>
        <name>log_name</name>
        <value>ApeFlow</value>
        <description>None</description>
    </property>
    <property>
        <name>log_level</name>
        <value>INFO</value>
        <description>[INFO, DEBUG, WARN, ERROR, CRITICAL]</description>
    </property>
    <property>
        <name>mrms_svc</name>
        <value>mrms-svc</value>
        <description>None</description>
    </property>
    <property>
        <name>mrms_sftp_port</name>
        <value>10022</value>
        <description>None</description>
    </property>
    <property>
        <name>mrms_rest_port</name>
        <value>9200</value>
        <description>None</description>
    </property>
    <property>
        <name>mrms_username</name>
        <value>{Master 서버 유저명(AES256 암호화)}</value>
    </property>
    <property>
        <name>mrms_password</name>
        <value>{Master 서버 유저 비밀번호(AES256 암호화)}</value>
    </property>
    <property>
        <name>batch_size</name>
        <value>1024</value>
    </property>
</configuration>
```
</details>

<details>
<summary>DA(* Master 서버 유저명, 비밀번호 수정 필요)</summary>

da-conf.xml
```xml
<configuration>
    <!-- app path setting -->
    <property>
        <name>dir_data_root</name>
        <value>/eyeCloudAI/data</value>
        <description>Data root directory</description>
    </property>
    <!-- log setting -->
    <property>
        <name>dir_log</name>
        <value>/eyeCloudAI/logs</value>
        <description>None</description>
    </property>
    <property>
        <name>log_name</name>
        <value>DataAnalyzer</value>
        <description>None</description>
    </property>
    <property>
        <name>log_level</name>
        <value>INFO</value>
        <description>[INFO, DEBUG, WARN, ERROR, CRITICAL]</description>
    </property>
    <!-- hosts -->
    <property>
        <name>storage_svc</name>
        <value>mrms-svc</value>
    </property>
    <property>
        <name>storage_sftp_port</name>
        <value>10022</value>
    </property>
    <property>
        <name>storage_ssh_username</name>
        <value>{Master 서버 유저명(AES256 암호화)}</value>
    </property>
    <property>
        <name>storage_ssh_password</name>
        <value>{Master 서버 유저 비밀번호(AES256 암호화)}</value>
    </property>
    <property>
        <name>mrms_svc</name>
        <value>mrms-svc</value>
    </property>
    <property>
        <name>mrms_sftp_port</name>
        <value>10022</value>
    </property>
    <property>
        <name>mrms_rest_port</name>
        <value>9200</value>
    </property>
    <property>
        <name>mrms_ssh_username</name>
        <value>{Master 서버 유저명(AES256 암호화)}</value>
    </property>
    <property>
        <name>mrms_ssh_password</name>
        <value>{Master 서버 유저 비밀번호(AES256 암호화)}</value>
    </property>
    <property>
        <name>text_distribute_instances</name>
        <value>100000</value>
    </property>
    <property>
        <name>image_distribute_instances</name>
        <value>1024</value>
        <description>MB</description>
    </property>
    <property>
        <name>worker_waiting_timeout</name>
        <value>86400</value>
        <description>seconds</description>
    </property>
    <property>
        <name>write_cycle_history</name>
        <value>False</value>
        <description>True or False</description>
    </property>
</configuration>
```
</details>

<details>
<summary>DPRS(* Master 서버 유저명, 비밀번호 수정 필요)</summary>

dprs-conf.xml
```xml
<configuration>
    <!-- app path setting -->
    <property>
        <name>dir_data_root</name>
        <value>/eyeCloudAI/data</value>
        <description>Data root directory</description>
    </property>
    <property>
        <name>ape.features.dir</name>
        <value>/processing/ape/division</value>
    </property>
    <!-- log setting -->
    <property>
        <name>dir_log</name>
        <value>/eyeCloudAI/logs</value>
        <description>None</description>
    </property>
    <property>
        <name>log_name</name>
        <value>DataProcessRecommender</value>
        <description>None</description>
    </property>
    <property>
        <name>log_level</name>
        <value>INFO</value>
        <description>[INFO, DEBUG, WARN, ERROR, CRITICAL]</description>
    </property>
    <!-- hosts -->
    <property>
        <name>mrms_svc</name>
        <value>mrms-svc</value>
    </property>
    <property>
        <name>mrms_sftp_port</name>
        <value>10022</value>
    </property>
    <property>
        <name>mrms_rest_port</name>
        <value>9200</value>
    </property>
    <property>
        <name>mrms_username</name>
        <value>{Master 서버 유저명(AES256 암호화)}</value>
    </property>
    <property>
        <name>mrms_password</name>
        <value>{Master 서버 유저 비밀번호(AES256 암호화)}</value>
    </property>
    <!-- recommend -->
    <property>
        <name>rcmd_min_count</name>
        <value>1</value>
    </property>
    <property>
        <name>rcmd_max_count</name>
        <value>2</value>
    </property>
</configuration>

```
</details>

<details>
<summary>HPRS(* Master 서버 유저명, 비밀번호 수정 필요)</summary>

hprs-conf.xml
```xml
<configuration>
    <!-- app path setting -->
    <property>
        <name>dir_data_root</name>
        <value>/eyeCloudAI/data</value>
        <description>Data root directory</description>
    </property>
    <!-- log setting -->
    <property>
        <name>dir_log</name>
        <value>/eyeCloudAI/logs</value>
        <description>None</description>
    </property>
    <property>
        <name>log_name</name>
        <value>HyperParameterRecommender</value>
        <description>None</description>
    </property>
    <property>
        <name>log_level</name>
        <value>INFO</value>
        <description>[INFO, DEBUG, WARN, ERROR, CRITICAL]</description>
    </property>
    <!-- hosts -->
    <property>
        <name>mrms_svc</name>
        <value>mrms-svc</value>
    </property>
    <property>
        <name>mrms_sftp_port</name>
        <value>10022</value>
    </property>
    <property>
        <name>mrms_rest_port</name>
        <value>9200</value>
    </property>
    <property>
        <name>mrms_username</name>
        <value>{Master 서버 유저명(AES256 암호화)}</value>
    </property>
    <property>
        <name>mrms_password</name>
        <value>{Master 서버 유저 비밀번호(AES256 암호화)}</value>
    </property>
    <!-- recommend -->
    <property>
        <name>rcmd_min_count</name>
        <value>1</value>
    </property>
    <property>
        <name>rcmd_max_count</name>
        <value>2</value>
    </property>
</configuration>

```
</details>

<details>
<summary>MARS(* Master 서버 유저명, 비밀번호 수정 필요)</summary>

mars-conf.xml
```xml
<configuration>
    <!-- app path setting -->
    <property>
        <name>dir_data_root</name>
        <value>/eyeCloudAI/data</value>
        <description>Data root directory</description>
    </property>
    <property>
        <name>ape.features.dir</name>
        <value>/processing/ape/division</value>
    </property>
    <!-- log setting -->
    <property>
        <name>dir_log</name>
        <value>/eyeCloudAI/logs</value>
        <description>None</description>
    </property>
    <property>
        <name>log_name</name>
        <value>MLAlgRecommender</value>
        <description>None</description>
    </property>
    <property>
        <name>log_level</name>
        <value>INFO</value>
        <description>[INFO, DEBUG, WARN, ERROR, CRITICAL]</description>
    </property>
    <!-- hosts -->
    <property>
        <name>mrms_svc</name>
        <value>mrms-svc</value>
    </property>
    <property>
        <name>mrms_sftp_port</name>
        <value>10022</value>
    </property>
    <property>
        <name>mrms_rest_port</name>
        <value>9200</value>
    </property>
    <property>
        <name>mrms_username</name>
        <value>{Master 서버 유저명(AES256 암호화)}</value>
    </property>
    <property>
        <name>mrms_password</name>
        <value>{Master 서버 유저 비밀번호(AES256 암호화)}</value>
    </property>
    <!-- recommend -->
    <property>
        <name>rcmd_min_count</name>
        <value>1</value>
    </property>
    <property>
        <name>rcmd_max_count</name>
        <value>2</value>
    </property>
</configuration>

```
</details>

<details>
<summary>MLPS(* Master 서버 유저명, 비밀번호 수정 필요)</summary>

mlps-conf.xml
```xml
<configuration>
    <!-- app path setting -->
    <property>
        <name>mlps_version</name>
        <value>3.0.0</value>
        <description>None</description>
    </property>
    <property>
        <name>app_dir</name>
        <value>/eyeCloudAI</value>
        <description>Application root directory</description>
    </property>
    <property>
        <name>app_modules</name>
        <value>/app/ape/mlps</value>
        <description>None</description>
    </property>
    <!-- data path setting -->
    <property>
        <name>dir_resources</name>
        <value>/resources</value>
        <description>None</description>
    </property>
    <property>
        <name>dir_data_root</name>
        <value>/data</value>
        <description>None</description>
    </property>
    <property>
        <name>dir_processing</name>
        <value>/processing/ape</value>
        <description>None</description>
    </property>
    <property>
        <name>dir_job</name>
        <value>/jobs</value>
        <description>None</description>
    </property>
    <property>
        <name>dir_storage</name>
        <value>/storage/ape</value>
        <description>None</description>
    </property>
    <property>
        <name>dir_result</name>
        <value>/results</value>
        <description>None</description>
    </property>
    <property>
        <name>dir_temp</name>
        <value>/temp</value>
        <description>None</description>
    </property>
    <property>
        <name>dir_error</name>
        <value>/errors</value>
        <description>None</description>
    </property>
    <!-- log setting -->
    <property>
        <name>log_dir</name>
        <value>/logs</value>
        <description>None</description>
    </property>
    <property>
        <name>log_name</name>
        <value>MLProcessingServer</value>
        <description>None</description>
    </property>
    <property>
        <name>log_level</name>
        <value>INFO</value>
        <description>[INFO, DEBUG, WARN, ERROR, CRITICAL]</description>
    </property>
    <!-- rest url setting -->
    <property>
        <name>mrms_svc</name>
        <value>mrms-svc</value>
        <description>None</description>
    </property>
    <property>
        <name>mrms_sftp_port</name>
        <value>10022</value>
        <description>None</description>
    </property>
    <property>
        <name>mrms_rest_port</name>
        <value>9200</value>
        <description>None</description>
    </property>
    <property>
        <name>mrms_username</name>
        <value>{Master 서버 유저명(AES256 암호화)}</value>
    </property>
    <property>
        <name>mrms_password</name>
        <value>{Master 서버 유저 비밀번호(AES256 암호화)}</value>
    </property>
    <property>
        <name>etls_wating_time</name>
        <value>5</value>
        <description>Seconds</description>
    </property>
    <property>
        <name>cvt_data</name>
        <value>false</value>
        <description>true or false</description>
    </property>
</configuration>

```
</details>

<details>
<summary>MRMS(* DB User Name, Password 수정 필요)</summary>

mrms-conf.xml
```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl"  href="configuration.xsl"?>
<configuration>
  <property>
    <name>ape.job.dir</name>
    <value>/eyeCloudAI/data/processing/ape/jobs</value>
  </property>
  <property>
    <name>ape.da.dir</name>
    <value>/eyeCloudAI/data/processing/ape/da</value>
  </property>
  <property>
    <name>ape.model.dir</name>
    <value>/eyeCloudAI/data/storage/ape</value>
  </property>
  <property>
    <name>kubernetes.log.debug.yn</name>
    <value>N</value>
  </property>
  <property>
    <name>use.learning.schedule</name>
    <value>true</value>
  </property>
  <property>
    <name>use.inference.schedule</name>
    <value>true</value>
  </property>
  <property>
    <name>use.data.analysis.schedule</name>
    <value>true</value>
  </property>
  <property>
    <name>kube.pod.cpu.limit</name>
    <value>1200</value>
    <description>m</description>
  </property>
  <property>
    <name>kube.pod.memory.limit</name>
    <value>0</value>
    <description>Gi</description>
  </property>
  <property>
    <name>kube.pod.cpu.limit.ratio</name>
    <value>0.5</value>
    <description>0-1 float value</description>
  </property>
  <property>
    <name>gpu.mem.unit</name>
    <value>MiB</value>
    <description>MiB or GiB</description>
  </property>
  <property>
    <name>gpu_mem_limit_inference</name>
    <value>1169</value>
    <description>Integer(MiB or GiB)</description>
  </property>
  <property>
    <name>gpu_mem_limit_learn</name>
    <value>2048</value>
    <description>Integer(MiB or GiB)</description>
  </property>
  <property>
    <name>global_steps</name>
    <value>50</value>
    <description>epoch</description>
  </property>
  <property>
    <name>remove_ttl_time</name>
    <value>648000</value>
    <description>seconds(integer)</description>
  </property>
  <property>
    <name>jobfile.remove.boolean</name>
    <value>false</value>
    <description>true/false</description>
  </property>
  <property>
    <name>registry.url</name>
    <value>seculayer</value>
    <description>Registry URL</description>
  </property>
  <property>
    <name>da.tag</name>
    <value>4.0-2023.4</value>
    <description>DA Module Version</description>
  </property>
  <property>
    <name>dprs.tag</name>
    <value>4.0-2023.3</value>
    <description>DPRS Module Version</description>
  </property>
  <property>
    <name>mars.tag</name>
    <value>4.0-2023.4</value>
    <description>MARS Module Version</description>
  </property>
  <property>
    <name>hprs.tag</name>
    <value>4.0-2023.3</value>
    <description>HPRS Module Version</description>
  </property>
  <property>
    <name>mlps.tag</name>
    <value>4.0-2023.4</value>
    <description>MLPS Module Version</description>
  </property>
  <property>
    <name>xai.tag</name>
    <value>4.0-2023.4</value>
    <description>XAI Module Version</description>
  </property>
</configuration>
```
db.properties
```properties
jdbc.driver=org.mariadb.jdbc.Driver
jdbc.url=jdbc:mariadb://apeautoml-db-svc:3306/APEAutoML?autoReconnect=true
jdbc.username={DB User Name}
jdbc.password={DB Password}
jdbc.poolMaxActive=20
jdbc.poolMaxIdle=10
jdbc.poolTimeToWait=5000
```

log4j2.properties
```properties
status = error
dest = err
name = eyeCloudAI mrms log

property.logDir = /eyeCloudAI/logs/
property.appname = mrms

appenders = console, rolling

appender.console.type = Console
appender.console.name = STDOUT
appender.console.layout.type = PatternLayout
appender.console.layout.pattern = %d{ISO8601} %p %t %c{1} - %m%n

appender.rolling.type = RollingFile
appender.rolling.name = LogToRollingFile
appender.rolling.fileName = ${logDir}/${appname}.log
appender.rolling.filePattern = ${logDir}/${appname}-%d{yyyy-MM-dd}.log.zip
appender.rolling.layout.type = PatternLayout
appender.rolling.layout.pattern = %d{ISO8601} %p %t %c{1} - %m%n
appender.rolling.policies.type = Policies
appender.rolling.policies.time.type = TimeBasedTriggeringPolicy
appender.rolling.policies.time.interval = 1

loggers = seculayer, jetty, ibatis, quartz

logger.seculayer.name = com.seculayer
logger.seculayer.level = info
logger.seculayer.additivity = false
logger.seculayer.appenderRef.rolling.ref = LogToRollingFile
logger.seculayer.appenderRef.stdout.ref = STDOUT

logger.jetty.name = org.eclipse.jetty
logger.jetty.level = warn
logger.jetty.additivity = false
logger.jetty.appenderRef.stdout.ref = STDOUT

logger.ibatis.name = org.apache.ibatis
logger.ibatis.level = warn
logger.ibatis.additivity = false
logger.ibatis.appenderRef.stdout.ref = STDOUT

logger.quartz.name = org.quartz
logger.quartz.level = warn
logger.quartz.additivity = false
logger.quartz.appenderRef.stdout.ref = STDOUT

logger.apache.name = org.apache.http
logger.apache.level = warn
logger.apache.additivity = false
logger.apache.appenderRef.stdout.ref = STDOUT

rootLogger.level = info
rootLogger.appenderRef.stdout.ref = STDOUT

```
</details>

```shell
# Create Configmap
kubectl create configmap apeflow-conf --from-file=apeflow-conf.xml -n apeautoml
kubectl create configmap da-conf --from-file=da-conf.xml -n apeautoml
kubectl create configmap dprs-conf --from-file=dprs-conf.xml -n apeautoml
kubectl create configmap mars-conf --from-file=mars-conf.xml -n apeautoml
kubectl create configmap hprs-conf --from-file=hprs-conf.xml -n apeautoml
kubectl create configmap mars-conf --from-file=mars-conf.xml -n apeautoml
kubectl create configmap mlps-conf --from-file=mlps-conf.xml -n apeautoml
kubectl create configmap mrms-conf --from-file=mrms-conf.xml --from-file=db.properties --from-file=log4j2.properties -n apeautoml
```

#### MRMS
<details>
<summary>mrms-deploy.yaml</summary>

```yaml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: mrms-user
  namespace: apeautoml
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: mrms-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: mrms-user
    namespace: apeautoml
---
kind: Service
apiVersion: v1
metadata:
  name: mrms-svc
  namespace: apeautoml
spec:
  type: NodePort
  ports:
    - name: "ssh"
      port: 10022
      targetPort: 9100
      nodePort: 30022
    - name: "mrms-servlet"
      port: 9200
      targetPort: 9200
      nodePort: 31920
  selector:
    app: mrms
status:
  loadBalancer: {}
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mrms
  labels:
    app: mrms
  namespace: apeautoml
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mrms
  template:
    metadata:
      labels:
        app: mrms
    spec:
      serviceAccountName: mrms-user
      containers:
        - name: mrms
          image: seculayer/automl-mrms:4.0-2023.4
          imagePullPolicy: Always
          ports:
            - containerPort: 9200
          command: ["/bin/bash", "./mrms.sh"]
          env:
            - name: TZ
              value: Asia/Seoul
          volumeMounts:
            - name: timezone-config
              mountPath: /etc/localtime
            - name: automl-processing
              mountPath: /eyeCloudAI/data/processing
            - name: automl-storage
              mountPath: /eyeCloudAI/data/storage/ape
            - name: mrms-conf-v
              mountPath: /eyeCloudAI/app/ape/mrms/conf

        - name: mrms-sftp
          image: seculayer/sftp-server:1.0.0
          ports:
            - containerPort: 9100
          command: ["/entrypoint", "eyecloudai:eyecloudai!23:1000"]
          volumeMounts:
            - name: timezone-config
              mountPath: /etc/localtime
            - name: automl-processing
              mountPath: /eyeCloudAI/data/processing
            - name: automl-storage
              mountPath: /eyeCloudAI/data/storage/ape

      volumes:
        - name: automl-processing
          hostPath:
            path: /eyeCloudAI/data/processing
        - name: automl-storage
          hostPath:
            path: /eyeCloudAI/data/storage/ape
        - name: timezone-config
          hostPath:
            path: /usr/share/zoneinfo/Asia/Seoul
        - name: mrms-conf-v
          configMap:
            name: mrms-conf
      nodeSelector:
        app: "true"
        master: "true"
---
```
</details>

```shell
# MRMS Start
kubectl apply -f mrms-deploy.yaml
```
