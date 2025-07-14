# 🚀 Kubernetes & AKS Assignment

This repo demonstrates Kubernetes concepts and Azure Kubernetes Service (AKS) features through hands-on YAML configurations and commands.

---

## 📦 1. Deploy ReplicaSet, ReplicationController, and Deployment

### ➤ ReplicationController
```yaml
# replicationcontroller.yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: my-rc
spec:
  replicas: 2
  selector:
    app: rc-demo
  template:
    metadata:
      labels:
        app: rc-demo
    spec:
      containers:
      - name: nginx
        image: nginx
```
### ➤ ReplicaSet
```yaml
# replicaset.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: my-rs
spec:
  replicas: 2
  selector:
    matchLabels:
      app: rs-demo
  template:
    metadata:
      labels:
        app: rs-demo
    spec:
      containers:
      - name: nginx
        image: nginx
```

### ➤ Deployment
```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: deploy-demo
  template:
    metadata:
      labels:
        app: deploy-demo
    spec:
      containers:
      - name: nginx
        image: nginx:latest
```
### 🛠️ Apply:
```bash
kubectl apply -f replicationcontroller.yaml
kubectl apply -f replicaset.yaml
kubectl apply -f deployment.yaml
```

## 🌐 2. Kubernetes Service Types
### ➤ ClusterIP
```yaml

# clusterip.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-clusterip
spec:
  type: ClusterIP
  selector:
    app: deploy-demo
  ports:
    - port: 80
      targetPort: 80
```
### ➤ NodePort
```yaml

# nodeport.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nodeport
spec:
  type: NodePort
  selector:
    app: deploy-demo
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30036
```
### ➤ LoadBalancer (for AKS)
```yaml
# loadbalancer.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-loadbalancer
spec:
  type: LoadBalancer
  selector:
    app: deploy-demo
  ports:
    - port: 80
      targetPort: 80
```
## 💾 3. Persistent Volume & Claim
### ➤ PersistentVolume
```yaml
# pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /data/pv
```
### ➤ PersistentVolumeClaim
```yaml
# pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```
### ➤ Pod using PVC
```yaml

# volume-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-using-pvc
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - mountPath: "/usr/share/nginx/html"
      name: my-storage
  volumes:
  - name: my-storage
    persistentVolumeClaim:
      claimName: my-pvc
```
## ☸️ 4. Create & Manage AKS Cluster
```bash
# Create AKS Cluster (Azure CLI)
az aks create --resource-group myResourceGroup --name myAKSCluster --node-count 2 --enable-addons monitoring --generate-ssh-keys

# Get credentials
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster

# Check nodes
kubectl get nodes
```
## 💚 5. Liveness & Readiness Probes
### ➤ Probe Example
```yaml

# probe-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: probe-pod
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
    livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 10
      periodSeconds: 5
    readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5
```
## ⚖️ 6. Taints and Tolerations
### ➤ Taint a Node
```bash

kubectl taint nodes <node-name> key=value:NoSchedule
```
### ➤ Toleration in Pod
```yaml

# toleration-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: toleration-pod
spec:
  tolerations:
  - key: "key"
    operator: "Equal"
    value: "value"
    effect: "NoSchedule"
  containers:
  - name: nginx
    image: nginx
```
## 🔄 7. Horizontal Pod Autoscaler (HPA)
### ➤ Install Metrics Server
```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```
### ➤ Apply Deployment
```yaml

# autoscale-deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hpa-deploy
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hpa-demo
  template:
    metadata:
      labels:
        app: hpa-demo
    spec:
      containers:
      - name: nginx
        image: nginx
        resources:
          requests:
            cpu: 100m
          limits:
            cpu: 200m
```
### ➤ Create HPA
```bash

kubectl autoscale deployment hpa-deploy --cpu-percent=50 --min=1 --max=5
kubectl get hpa
```
