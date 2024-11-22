# Kubernetes RBAC Setup Guide  

This guide demonstrates how to configure **Role-Based Access Control (RBAC)** in a Kubernetes cluster to provide fine-grained access to resources based on user roles. We’ll set up access for multiple users (`Anshumaan`, `Ashutosh`, `Sai Krishna`, and `Khagendra Thapa`) based on organizational roles such as **Admin**, **Editor**, and **Viewer**.  

---

## 🛠️ Objectives  

- Create roles with different access levels.  
- Assign broader permissions to an **Admin**, moderate permissions to an **Editor**, and read-only permissions to a **Viewer**.  
- Demonstrate namespace-specific and cluster-wide access control.  

---

## 🌐 User Roles  

1. **Anshumaan**: **Admin**  
   - Has full access to all resources in the cluster.  
2. **Ashutosh**: **Editor**  
   - Can view and edit resources in a specific namespace.  
3. **Sai Krishna**: **Viewer**  
   - Can only view resources in a specific namespace.  
4. **Khagendra Thapa**: **Viewer**  
   - Cluster-wide read-only access.  

---

## 🔐 Configuration  

### 1. **ClusterRole for Admin**  
The **Admin** has full access across the cluster.  

**`admin-clusterrole.yaml`**:  
```yaml  
apiVersion: rbac.authorization.k8s.io/v1  
kind: ClusterRole  
metadata:  
  name: cluster-admin-role  
rules:  
- apiGroups: ["*"]  
  resources: ["*"]  
  verbs: ["*"]  
```  

**ClusterRoleBinding for Anshumaan (Admin)**:  
**`admin-clusterrolebinding.yaml`**:  
```yaml  
apiVersion: rbac.authorization.k8s.io/v1  
kind: ClusterRoleBinding  
metadata:  
  name: admin-access-global  
subjects:  
- kind: User  
  name: Anshumaan  
  apiGroup: rbac.authorization.k8s.io  
roleRef:  
  kind: ClusterRole  
  name: cluster-admin-role  
  apiGroup: rbac.authorization.k8s.io  
```  

### Apply Commands:  
```bash  
kubectl apply -f admin-clusterrole.yaml  
kubectl apply -f admin-clusterrolebinding.yaml  
```  

---

### 2. **Role for Editor (Namespace-Specific)**  
The **Editor** has access to modify resources in the `development` namespace.  

**`editor-role.yaml`**:  
```yaml  
apiVersion: rbac.authorization.k8s.io/v1  
kind: Role  
metadata:  
  namespace: development  
  name: editor-role  
rules:  
- apiGroups: [""]  
  resources: ["pods", "services", "deployments"]  
  verbs: ["get", "list", "create", "update", "delete"]  
```  

**RoleBinding for Ashutosh (Editor)**:  
**`editor-rolebinding.yaml`**:  
```yaml  
apiVersion: rbac.authorization.k8s.io/v1  
kind: RoleBinding  
metadata:  
  name: editor-access  
  namespace: development  
subjects:  
- kind: User  
  name: Ashutosh  
  apiGroup: rbac.authorization.k8s.io  
roleRef:  
  kind: Role  
  name: editor-role  
  apiGroup: rbac.authorization.k8s.io  
```  

### Apply Commands:  
```bash  
kubectl apply -f editor-role.yaml  
kubectl apply -f editor-rolebinding.yaml  
```  

---

### 3. **Role for Viewer (Namespace-Specific)**  
The **Viewer** can only view resources in the `staging` namespace.  

**`viewer-role.yaml`**:  
```yaml  
apiVersion: rbac.authorization.k8s.io/v1  
kind: Role  
metadata:  
  namespace: staging  
  name: viewer-role  
rules:  
- apiGroups: [""]  
  resources: ["pods", "services", "deployments"]  
  verbs: ["get", "list"]  
```  

**RoleBinding for Sai Krishna (Viewer)**:  
**`viewer-rolebinding.yaml`**:  
```yaml  
apiVersion: rbac.authorization.k8s.io/v1  
kind: RoleBinding  
metadata:  
  name: viewer-access  
  namespace: staging  
subjects:  
- kind: User  
  name: Sai Krishna  
  apiGroup: rbac.authorization.k8s.io  
roleRef:  
  kind: Role  
  name: viewer-role  
  apiGroup: rbac.authorization.k8s.io  
```  

### Apply Commands:  
```bash  
kubectl apply -f viewer-role.yaml  
kubectl apply -f viewer-rolebinding.yaml  
```  

---

### 4. **ClusterRole for Global Viewer**  
The **Global Viewer** can read all resources across all namespaces.  

**`global-viewer-clusterrole.yaml`**:  
```yaml  
apiVersion: rbac.authorization.k8s.io/v1  
kind: ClusterRole  
metadata:  
  name: global-viewer-role  
rules:  
- apiGroups: ["*"]  
  resources: ["*"]  
  verbs: ["get", "list"]  
```  

**ClusterRoleBinding for Khagendra Thapa (Global Viewer)**:  
**`global-viewer-clusterrolebinding.yaml`**:  
```yaml  
apiVersion: rbac.authorization.k8s.io/v1  
kind: ClusterRoleBinding  
metadata:  
  name: global-viewer-access  
subjects:  
- kind: User  
  name: Khagendra Thapa  
  apiGroup: rbac.authorization.k8s.io  
roleRef:  
  kind: ClusterRole  
  name: global-viewer-role  
  apiGroup: rbac.authorization.k8s.io  
```  

### Apply Commands:  
```bash  
kubectl apply -f global-viewer-clusterrole.yaml  
kubectl apply -f global-viewer-clusterrolebinding.yaml  
```  

---

## 🔍 Testing Permissions  

- **Check permissions for `Anshumaan` (Admin)**:  
  ```bash  
  kubectl auth can-i create deployment --as Anshumaan -A  
  ```  

- **Check permissions for `Ashutosh` (Editor)**:  
  ```bash  
  kubectl auth can-i update pod --as Ashutosh -n development  
  ```  

- **Check permissions for `Sai Krishna` (Viewer)**:  
  ```bash  
  kubectl auth can-i delete service --as Sai Krishna -n staging  
  ```  

- **Check permissions for `Khagendra Thapa` (Global Viewer)**:  
  ```bash  
  kubectl auth can-i list pod --as Khagendra Thapa -A  
  ```  

---

## 🛡️ Best Practices  

1. **Use Namespaces to Segregate Environments**:  
   - Assign specific roles per environment (e.g., `dev`, `staging`, `prod`).  

2. **Grant Minimum Privileges**:  
   - Follow the principle of least privilege for all users and service accounts.  

3. **Audit Access Regularly**:  
   - Use `kubectl describe rolebinding` or `kubectl describe clusterrolebinding` to review access assignments.  

4. **Use Service Accounts for Applications**:  
   - Bind applications to service accounts instead of individual users.  

---

## 🎯 Common Commands for RBAC  

- **List all Roles in a namespace**:  
  ```bash  
  kubectl get role -n <namespace>  
  ```  

- **List all ClusterRoles**:  
  ```bash  
  kubectl get clusterrole  
  ```  

- **List all RoleBindings in a namespace**:  
  ```bash  
  kubectl get rolebinding -n <namespace>  
  ```  

- **List all ClusterRoleBindings**:  
  ```bash  
  kubectl get clusterrolebinding  
  ```  

- **Test user access**:  
  ```bash  
  kubectl auth can-i <action> <resource> --as <user> [-n <namespace>]   
  ```

---

## 🤝 Resources and References

- [Kubernetes RBAC Documentation](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
- [Kubernetes Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

By following this guide, you’ll ensure secure and manageable access controls within your Kubernetes cluster. Happy clustering! 🚀
