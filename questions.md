# Complete Solutions for CKA Questions

# Question 1 - Create a new pod called web-pod with image busybox Allow the pod to be able to set system_time​. The container should sleep for 3200 seconds​

# Imperative Command:
kubectl run web-pod --image=busybox --command -- sleep 3200 --privileged

# Declarative Way:
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sleep", "3200"]
    securityContext:
      privileged: true

---

# Question 2 - Create a new deployment called myproject, with image nginx:1.16 and ​1 replica. Next upgrade the deployment to version 1.17 using rolling​ update​. Make sure that the version upgrade is recorded in the resource annotation.​

# Imperative Command:
kubectl create deployment myproject --image=nginx:1.16
kubectl set image deployment/myproject nginx=nginx:1.17 --record

# Declarative Way:
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

---

# Question 3 - Create a new deployment called my-deployment. Scale the deployment to 3 replicas.             ​
Make sure desired number of pod always running. 

# Imperative Command:
kubectl create deployment my-deployment --image=nginx
kubectl scale deployment my-deployment --replicas=3

# Declarative Way:
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

---

# Question 4 - Deploy a web-nginx pod using the nginx:1.17 image with the labels set to tier=web-app.​


# Imperative Command:
kubectl run web-nginx --image=nginx:1.17 --labels=tier=web-app

# Declarative Way:
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

---

# Question 5 - Create a static pod on node01 called static-pod with image nginx and you     have to make sure that it is recreated/restarted automatically in case ​of any failure happens


# Declarative Way (Place on node01 at `/etc/kubernetes/manifests/static-pod.yaml`):
apiVersion: v1
kind: Pod
metadata:
  name: static-pod
spec:
  containers:
  - name: nginx
    image: nginx

---

# Question 6 - Create a pod called pod-multi with two containers, as given below:​
Container 1 - name: container1, image: nginx​
Container2 - name: container2, image: busybox, command: sleep 4800​

# Declarative Way:
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

---

# Question 7 - Create a pod called test-pod in "custom" namespace belonging to the test environment (env=test) and backend tier (tier=backend).​ image: nginx:1.17​

# Declarative Way:
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

---

# Question 8 -  Get the node node01 in JSON format and store it in a file at ​ ./node-info.json​

# Command:
kubectl get node node01 -o json > ./node-info.json

---

# Question 9 -  Use JSON PATH query to retrieve the oslmages of all the nodes and store it in a file “all-nodes-os-info.txt” at root location.​ 
Note: The osImage are under the nodeInfo section under status of each node.​


# Command:
kubectl get nodes -o jsonpath='{range .items[*]}{.status.nodeInfo.osImage}{"\n"}{end}' > /root/all-nodes-os-info.txt

---

# Question 10 -  Create a Persistent Volume with the given specification.​
       Volume Name: pv-demo​
             Storage:100Mi​
Access modes: ReadWriteMany​
  Host Path: /pv/host-data​

# Declarative Way:
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

---

# Question 11 -Worker Node “node01” not responding, Debug the issue and fix it.​

# Steps:
1. SSH into the node: `ssh user@node01`
2. Check kubelet status: `systemctl status kubelet`
3. Restart kubelet: `systemctl restart kubelet`
4. Verify node status: `kubectl get nodes`

---

# Question 12 -  Upgrade the Cluster (Master and worker Node) from 1.18.0 to 1.19.0. Make sure to first drain both Node and make it available after upgrade.​

# Commands:
kubectl drain <node> --ignore-daemonsets --delete-local-data
apt-get update && apt-get install -y kubeadm=1.19.0-00
kubeadm upgrade plan
kubeadm upgrade apply v1.19.0
kubectl uncordon <node>

---

# Question 13 - Take a backup of the ETCD database and save it to “/opt/etcd-backup.db” . Also restore the ETCD database from the backup​

# Backup:
ETCDCTL_API=3 etcdctl snapshot save /opt/etcd-backup.db --endpoints=<etcd-endpoint>

# Restore:
ETCDCTL_API=3 etcdctl snapshot restore /opt/etcd-backup.db --endpoints=<etcd-endpoint>

---

# Question 14 - Create a new user “anshu”. Grant him access to the cluster. User “ajeet” should have permission to create, list, get, update and delete pods. The private key exists at location:​
/root/ajeet/.key and csr at /root/anshu.csr
# Steps:
openssl genrsa -out /root/ajeet.key 2048
openssl req -new -key /root/ajeet.key -out /root/anshu.csr
kubectl config set-credentials anshu --client-certificate=/root/ajeet.csr --client-key=/root/anshu.key
kubectl create rolebinding anshu-rolebinding --user=anshu --clusterrole=edit --namespace=default

---

# Question 15 - Create a Nginx pod dns-resolver using image nginx, expose it internally with a aservice called dns-resolver-service.
Check if service name is resolvable from within the cluster.
Use the image busybox:1.28 for dns lookup
Save the result in /root/nginx.svc

# Declarative Way:
apiVersion: v1
kind: Pod
metadata:
  name: dns-resolver
spec:
  containers:
  - name: nginx
    image: nginx
---
apiVersion: v1
kind: Service
metadata:
  name: dns-resolver-service
spec:
  selector:
    app: dns-resolver
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
---

# Command to Test DNS:
kubectl run test-dns --image=busybox:1.28 --restart=Never --command -- nslookup dns-resolver-service > /root/nginx.svc

---

# Question 16 - A pod “appychip” (image=nginx) in default namespace is not running.​ Find the problem and fix it and make it running.​

# Steps:
kubectl describe pod appychip
# Identify issues (e.g., ImagePullError, missing resources) and fix them.
kubectl edit pod appychip

---

# Question 17 - Create a ReplicaSet (Name: appychip, Image: nginx:1.18, Replica: 4)​
There is already a Pod running in a cluster.​
Make sure that the total count of pods running in the cluster is not more than 4​

# Declarative Way:
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: appychip
spec:
  replicas: 4
  selector:
    matchLabels:
      app: appychip
  template:
    metadata:
      labels:
        app: appychip
    spec:
      containers:
      - name: nginx
        image: nginx:1.18

---

# Question 18 - Create a Network Policy named "appychip" in default namespace​
There should be two types, ingress and egress. ​
The ingress should block traffic from an IP range of your choice except some other IP range. Should also have namespace and pod selector.​
Ports for ingress policy should be 6379​
For Egress, it should allow traffic to an IP range of your choice on 5978 port.​

# Declarative Way:
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: appychip
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: appychip
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - ipBlock:
        cidr: 192.168.1.0/24
      except:
        - 192.168.1.100/32
    - namespaceSelector:
        matchLabels:
          project: appychip
    - podSelector:
        matchLabels:
          role: backend
    ports:
    - protocol: TCP
      port: 6379
  egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/24
    ports:
    - protocol: TCP
      port: 5978
