### **Question 1: Create a new pod called `web-pod` with image `busybox`. Allow the pod to set system time. The container should sleep for 3200 seconds.**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command:
    - sleep
    - "3200"
    securityContext:
      allowPrivilegeEscalation: true
      capabilities:
        add: ["SYS_TIME"]
```

Run the following command to create the pod:
```bash
kubectl apply -f web-pod.yaml
```

---

### **Question 2: Create a new deployment called `myproject`, with image `nginx:1.16` and 1 replica. Next, upgrade the deployment to version `1.17` using rolling update. Ensure that the version upgrade is recorded in the resource annotation.**

1. **Create Deployment (myproject)**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myproject
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myproject
  template:
    metadata:
      labels:
        app: myproject
    spec:
      containers:
      - name: nginx
        image: nginx:1.16
```

Apply it:
```bash
kubectl apply -f myproject-deployment.yaml
```

2. **Upgrade Deployment to Version 1.17**

To upgrade to version `nginx:1.17`, run:
```bash
kubectl set image deployment/myproject nginx=nginx:1.17 --record
```

---

### **Question 3: Create a new deployment called `my-deployment`. Scale the deployment to 3 replicas. Ensure the desired number of pods are always running.**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-deployment
  template:
    metadata:
      labels:
        app: my-deployment
    spec:
      containers:
      - name: nginx
        image: nginx
```

Apply it:
```bash
kubectl apply -f my-deployment.yaml
```

---

### **Question 4: Deploy a web-nginx pod using the `nginx:1.17` image with the labels set to `tier=web-app`.**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-nginx
  labels:
    tier: web-app
spec:
  containers:
  - name: nginx
    image: nginx:1.17
```

Apply it:
```bash
kubectl apply -f web-nginx.yaml
```

---

### **Question 5: Create a static pod on `node01` called `static-pod` with image `nginx` and make sure it is recreated/restarted automatically in case of any failure.**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: static-pod
  namespace: default
spec:
  containers:
  - name: nginx
    image: nginx
  nodeName: node01
  restartPolicy: Always
```

Apply it:
```bash
kubectl apply -f static-pod.yaml
```

---

### **Question 6: Create a pod called `pod-multi` with two containers:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-multi
spec:
  containers:
  - name: container1
    image: nginx
  - name: container2
    image: busybox
    command: ["sleep", "4800"]
```

Apply it:
```bash
kubectl apply -f pod-multi.yaml
```

---

### **Question 7: Create a pod called `test-pod` in `custom` namespace belonging to the test environment (`env=test`) and backend tier (`tier=backend`). Image: `nginx:1.17`.**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
  namespace: custom
  labels:
    env: test
    tier: backend
spec:
  containers:
  - name: nginx
    image: nginx:1.17
```

Apply it:
```bash
kubectl apply -f test-pod.yaml
```

---

### **Question 8: Get the node `node01` in JSON format and store it in a file at `./node-info.json`.**

```bash
kubectl get node node01 -o json > ./node-info.json
```

---

### **Question 9: Use JSON PATH query to retrieve the `osImage` of all the nodes and store it in a file `all-nodes-os-info.txt` at the root location.**

```bash
kubectl get nodes -o=jsonpath='{.items[*].status.nodeInfo.osImage}' > /all-nodes-os-info.txt
```

---

### **Question 10: Create a Persistent Volume with the given specification.**

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-demo
spec:
  capacity:
    storage: 100Mi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: /pv/host-data
    type: Directory
```

Apply it:
```bash
kubectl apply -f pv-demo.yaml
```

---

### **Question 11: Worker Node `node01` not responding, debug the issue and fix it.**

1. **Check node status:**
   ```bash
   kubectl get nodes
   ```

2. **Get detailed node information:**
   ```bash
   kubectl describe node node01
   ```

3. **Check for logs and issues in Kubelet on the node:**
   ```bash
   journalctl -u kubelet -f
   ```

4. **Check for disk or resource issues, restart the Kubelet service:**
   ```bash
   systemctl restart kubelet
   ```

5. **Check if the node is scheduling pods properly:**
   ```bash
   kubectl get pods --all-namespaces -o wide | grep node01
   ```

---

### **Question 12: Upgrade the Cluster (Master and Worker Node) from `1.18.0` to `1.19.0`. Make sure to first drain both nodes and make them available after the upgrade.**

1. **Drain the master node:**
   ```bash
   kubectl drain master-node --ignore-daemonsets --delete-local-data
   ```

2. **Upgrade Kubernetes on Master Node:**
   ```bash
   sudo apt-get update && sudo apt-get install -y kubeadm=1.19.0-00
   sudo kubeadm upgrade plan
   sudo kubeadm upgrade apply v1.19.0
   ```

3. **Upgrade kubelet and kubectl:**
   ```bash
   sudo apt-get install -y kubelet=1.19.0-00 kubectl=1.19.0-00
   sudo systemctl restart kubelet
   ```

4. **Uncordon the master node:**
   ```bash
   kubectl uncordon master-node
   ```

5. **Drain and upgrade the worker node `node01`:**
   ```bash
   kubectl drain node01 --ignore-daemonsets --delete-local-data
   sudo apt-get update && sudo apt-get install -y kubeadm=1.19.0-00
   sudo kubeadm upgrade apply v1.19.0
   sudo apt-get install -y kubelet=1.19.0-00 kubectl=1.19.0-00
   sudo systemctl restart kubelet
   kubectl uncordon node01
   ```

---

### **Question 13: Take a backup of the ETCD database and save it to `/opt/etcd-backup.db`. Also restore the ETCD database from the backup.**

1. **Take a backup:**
   ```bash
   ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
   --cacert=/etc/kubernetes/pki/etcd/ca.crt \
   --cert=/etc/kubernetes/pki/etcd/server.crt \
   --key=/etc/kubernetes/pki/etcd/server.key \
   snapshot save /opt/etcd-backup.db
   ```

2. **Restore the backup:**
   ```bash
   ETCDCTL_API=3 etcdctl --data-dir="/var/lib/etcd-backup" \
   --endpoints=https://127.0.0.1:2379 \
   --cacert=/etc/kubernetes/pki/etcd/ca.crt \
   --cert=/etc/kubernetes/pki/etcd/server.crt \
   --key=/etc/kubernetes/pki/etcd/server.key \
   snapshot restore /opt/etcd-backup.db
   ```

3. **Update `etcd.yaml` configuration for the restored backup:**

   ```yaml
   --data-dir="/var/lib/etcd-backup"
   ```

   Ensure the volume mount and path are correctly specified in `/etc/kubernetes/manifests/etcd.yaml`.

--- 

These steps should help you resolve all the questions!
