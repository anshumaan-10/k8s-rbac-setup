For the **Certified Kubernetes Administrator (CKA)** exam, you need a thorough grasp of core **kubectl commands** related to **Pods, ReplicaSets, Deployments, Services, Namespaces**, and much more. Here's an exam-focused **all-in-one** cheat sheet of the most essential commands:

---

### **General Kubernetes Commands**
1. **Cluster Info**:  
   ```bash
   kubectl cluster-info
   ```
2. **Node List**:  
   ```bash
   kubectl get nodes
   ```
3. **Context Management**:  
   ```bash
   kubectl config view
   kubectl config use-context <context-name>
   ```

---

### **Pods**
1. **Create Pod from YAML**:  
   ```bash
   kubectl apply -f pod.yaml
   ```
2. **List Pods**:  
   ```bash
   kubectl get pods -o wide
   ```
3. **View Pod Logs**:  
   ```bash
   kubectl logs <pod-name>
   ```
4. **Execute Command in Pod**:  
   ```bash
   kubectl exec -it <pod-name> -- <command>
   ```
5. **Get Pod YAML**:  
   ```bash
   kubectl get pod <pod-name> -o yaml > pod.yaml
   ```

---

### **ReplicaSets**
1. **Create ReplicaSet**:  
   ```bash
   kubectl apply -f replicaset.yaml
   ```
2. **List ReplicaSets**:  
   ```bash
   kubectl get rs
   ```
3. **Scale ReplicaSet**:  
   ```bash
   kubectl scale rs <rs-name> --replicas=<number>
   ```

---

### **Deployments**
1. **Create Deployment**:  
   ```bash
   kubectl create deployment <name> --image=<image-name>
   ```
2. **Apply Deployment YAML**:  
   ```bash
   kubectl apply -f deployment.yaml
   ```
3. **Update Image in Deployment**:  
   ```bash
   kubectl set image deployment/<name> <container-name>=<new-image>
   ```
4. **Rollout Status**:  
   ```bash
   kubectl rollout status deployment <deployment-name>
   ```
5. **Rollback Deployment**:  
   ```bash
   kubectl rollout undo deployment <deployment-name>
   ```
6. **Delete Deployment**:  
   ```bash
   kubectl delete deployment <deployment-name>
   ```

---

### **Services**
1. **Create Service YAML**:  
   ```bash
   kubectl apply -f service.yaml
   ```
2. **Expose Deployment as Service**:  
   ```bash
   kubectl expose deployment <name> --type=<service-type> --port=<port>
   ```
3. **List Services**:  
   ```bash
   kubectl get svc
   ```

---

### **Namespaces**
1. **List Namespaces**:  
   ```bash
   kubectl get namespaces
   ```
2. **Create Namespace**:  
   ```bash
   kubectl create namespace <namespace-name>
   ```
3. **Switch Namespace**:  
   ```bash
   kubectl config set-context --current --namespace=<namespace-name>
   ```

---

### **ConfigMaps and Secrets**
1. **Create ConfigMap**:  
   ```bash
   kubectl create configmap <name> --from-literal=key=value
   ```
2. **List ConfigMaps**:  
   ```bash
   kubectl get configmaps
   ```
3. **Create Secret**:  
   ```bash
   kubectl create secret generic <name> --from-literal=key=value
   ```

---

### **Persistent Volumes and Claims**
1. **Create PersistentVolume**:  
   ```bash
   kubectl apply -f pv.yaml
   ```
2. **Create PersistentVolumeClaim**:  
   ```bash
   kubectl apply -f pvc.yaml
   ```

---

### **Troubleshooting**
1. **Describe Resource**:  
   ```bash
   kubectl describe <resource> <name>
   ```
2. **View Events**:  
   ```bash
   kubectl get events
   ```
3. **Debug Pod**:  
   ```bash
   kubectl debug pod/<pod-name> --image=<debug-image>
   ```

---

### **Other Useful Commands**
1. **Dry Run and YAML Output**:  
   ```bash
   kubectl run <name> --image=<image> --dry-run=client -o yaml > pod.yaml
   ```
2. **Top Resources (Metrics)**:  
   ```bash
   kubectl top pod
   kubectl top node
   ```
3. **Apply Imperative Manifest**:  
   ```bash
   kubectl apply -f <file>.yaml
   ```

---

This cheat sheet is tailored for **CKA exam scenarios**, emphasizing practical tasks and troubleshooting. Let me know if you’d like a deeper dive into any specific area!
