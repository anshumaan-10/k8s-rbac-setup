# Kubernetes RBAC Setup Guide

This guide details how to set up **Role-Based Access Control (RBAC)** in a Kubernetes cluster to efficiently manage permissions and control access to resources. RBAC is a fundamental security feature in Kubernetes that allows you to define granular access control for users and service accounts. Follow along to learn how to create **Roles**, **ClusterRoles**, **RoleBindings**, and **ClusterRoleBindings**.

---

## 📜 Prerequisites

Before starting, ensure:
- You have **kubectl** installed and configured.
- Access to a Kubernetes cluster with administrative privileges.

---

## 🛠️ Role: **Namespace-Scoped Permissions**

A **Role** provides permissions within a specific namespace. Below is an example YAML configuration for a Role named **pod-reader** that allows the user to:
- `get`
- `watch`
- `list`

**`role.yaml`**:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

### Apply the Role:
```bash
kubectl apply -f role.yaml
```

### Verify the Role:
```bash
kubectl get role -n default
```

---

## 🔗 RoleBinding: **Binding Users to a Role**

A **RoleBinding** associates a user with a specific Role within a namespace. In this example, the **pod-reader** Role is assigned to the user **Anshumaan**.

**`rolebinding.yaml`**:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: Anshumaan
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

### Apply the RoleBinding:
```bash
kubectl apply -f rolebinding.yaml
```

### Verify the RoleBinding:
```bash
kubectl get rolebinding -n default
```

### Test Permissions for **Anshumaan**:
```bash
kubectl auth can-i get pod --as Anshumaan -n default
```

---

## 🌐 ClusterRole: **Cluster-Wide Permissions**

A **ClusterRole** grants permissions that apply across all namespaces. Below, the **secret-reader** ClusterRole allows users to:
- `get`
- `watch`
- `list` secrets.

**`clusterrole.yaml`**:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: secret-reader
rules:
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get", "watch", "list"]
```

### Apply the ClusterRole:
```bash
kubectl apply -f clusterrole.yaml
```

### Verify the ClusterRole:
```bash
kubectl get clusterrole
```

---

## 🔗 RoleBinding (Namespace-Level): **Binding Users to a ClusterRole**

A **RoleBinding** can bind a ClusterRole to a specific namespace. Below, the **secret-reader** ClusterRole is assigned to **Anshu** within the `development` namespace.

**`rolebinding-namespace.yaml`**:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-secrets
  namespace: development
subjects:
- kind: User
  name: Anshu
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io
```

### Apply the RoleBinding:
```bash
kubectl apply -f rolebinding-namespace.yaml
```

### Verify the RoleBinding:
```bash
kubectl get rolebinding -n development
```

### Test Permissions for **Anshu**:
```bash
kubectl auth can-i get secret --as Anshu -n development
```

---

## 🌍 ClusterRoleBinding: **Global Access for Users**

A **ClusterRoleBinding** associates a ClusterRole with a user across all namespaces. Here, **Anshumaan** is bound to the **secret-reader** ClusterRole, granting global access to secrets.

**`clusterrolebinding.yaml`**:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: read-secrets-global
subjects:
- kind: User
  name: Anshumaan
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io
```

### Apply the ClusterRoleBinding:
```bash
kubectl apply -f clusterrolebinding.yaml
```

### Verify the ClusterRoleBinding:
```bash
kubectl get clusterrolebinding
```

### Test Permissions for **Anshumaan**:
```bash
kubectl auth can-i get secret --as Anshumaan -A
```

---

## 🎯 Best Practices and Additional Tips

1. **Follow the Principle of Least Privilege**: Always grant the minimum permissions required for a task.
2. **Organize Resources**:
   - Use namespaces to separate environments (e.g., `dev`, `staging`, `prod`).
   - Bind users and service accounts to specific namespaces.
3. **Test Permissions**:
   - Use `kubectl auth can-i` commands to verify access levels.
   - Regularly audit RBAC configurations to avoid privilege escalation.
4. **Leverage Service Accounts**:
   - For applications running inside the cluster, bind Roles to Service Accounts instead of individual users.

---

## 🛡️ Common Commands for RBAC Management

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
- **Debug a user’s permissions**:
  ```bash
  kubectl auth can-i <action> <resource> --as <user> [-n <namespace>]
  ```

---

## 🤝 Resources and References

- [Kubernetes RBAC Documentation](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
- [Kubernetes Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

By following this guide, you’ll ensure secure and manageable access controls within your Kubernetes cluster. Happy clustering! 🚀
