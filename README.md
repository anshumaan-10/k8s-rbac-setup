Here’s the expanded version of the README that includes troubleshooting steps, how to view the roles and service accounts, and additional commands related to role management in Kubernetes.

---

# Kubernetes RBAC Setup for Admin, General, and Viewer Roles

This document guides you through setting up **RBAC (Role-Based Access Control)** for three types of roles in Kubernetes: **Admin**, **General**, and **Viewer**. It includes details on role creation, troubleshooting, viewing roles, and commands related to managing service accounts and roles.

## Table of Contents

1. [Introduction](#introduction)
2. [RBAC Roles Definitions](#rbac-roles-definitions)
   - [Admin Role](#admin-role)
   - [General Role](#general-role)
   - [Viewer Role](#viewer-role)
3. [RBAC Resources](#rbac-resources)
4. [Applying RBAC to Users](#applying-rbac-to-users)
5. [Viewing Roles and Service Accounts](#viewing-roles-and-service-accounts)
6. [Troubleshooting RBAC Issues](#troubleshooting-rbac-issues)
7. [Conclusion](#conclusion)

## Introduction

RBAC (Role-Based Access Control) in Kubernetes helps control access to resources based on roles assigned to users, service accounts, or groups. This guide demonstrates how to configure three roles: **Admin**, **General**, and **Viewer**, along with instructions on how to troubleshoot, view, and manage the associated **ServiceAccount**, **ClusterRole**, and **ClusterRoleBinding**.

## RBAC Roles Definitions

### Admin Role

The **Admin** role provides full access to all Kubernetes resources. It allows full management of resources like **pods**, **services**, **deployments**, **configmaps**, **secrets**, **roles**, and **cluster roles**.

#### YAML for Admin Role

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-service-account
  namespace: default
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: admin-cluster-role
rules:
  - apiGroups: [""]
    resources: ["pods", "services", "deployments", "configmaps", "secrets"]
    verbs: ["*"]
  - apiGroups: ["apps"]
    resources: ["deployments", "replicasets", "statefulsets"]
    verbs: ["*"]
  - apiGroups: [""]
    resources: ["nodes", "namespaces", "persistentvolumes"]
    verbs: ["*"]
  - apiGroups: ["rbac.authorization.k8s.io"]
    resources: ["roles", "rolebindings", "clusterroles", "clusterrolebindings"]
    verbs: ["*"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-cluster-role-binding
subjects:
  - kind: ServiceAccount
    name: admin-service-account
    namespace: default
roleRef:
  kind: ClusterRole
  name: admin-cluster-role
  apiGroup: rbac.authorization.k8s.io
```

### General Role

The **General** role has permissions for day-to-day operations within the cluster. It allows creating and updating resources like **pods**, **services**, and **deployments**, but with more limited access compared to the **Admin** role.

#### YAML for General Role

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: general-service-account
  namespace: default
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: general-cluster-role
rules:
  - apiGroups: [""]
    resources: ["pods", "services", "deployments"]
    verbs: ["list", "get", "create", "update"]
  - apiGroups: ["apps"]
    resources: ["deployments", "statefulsets"]
    verbs: ["list", "get", "create", "update"]
  - apiGroups: [""]
    resources: ["configmaps", "secrets"]
    verbs: ["list", "get", "create"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: general-cluster-role-binding
subjects:
  - kind: ServiceAccount
    name: general-service-account
    namespace: default
roleRef:
  kind: ClusterRole
  name: general-cluster-role
  apiGroup: rbac.authorization.k8s.io
```

### Viewer Role

The **Viewer** role provides read-only access to view resources in the cluster. Users with this role cannot modify resources.

#### YAML for Viewer Role

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: viewer-service-account
  namespace: default
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: viewer-cluster-role
rules:
  - apiGroups: [""]
    resources: ["pods", "services", "deployments", "configmaps"]
    verbs: ["list", "get"]
  - apiGroups: ["apps"]
    resources: ["deployments"]
    verbs: ["list", "get"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: viewer-cluster-role-binding
subjects:
  - kind: ServiceAccount
    name: viewer-service-account
    namespace: default
roleRef:
  kind: ClusterRole
  name: viewer-cluster-role
  apiGroup: rbac.authorization.k8s.io
```

## RBAC Resources

### ServiceAccount

A **ServiceAccount** is an identity for applications running inside the Kubernetes cluster. It can be used to grant specific permissions and interact with the Kubernetes API server.

### ClusterRole

A **ClusterRole** defines a set of permissions that apply across the entire cluster. It is a higher-level role than a **Role**, which is namespace-specific.

### ClusterRoleBinding

A **ClusterRoleBinding** associates a **ClusterRole** with a user or service account. It grants the permissions defined in the **ClusterRole** to the specified service account.

## Applying RBAC to Users

To apply these roles and bindings to your Kubernetes cluster, use the following `kubectl` commands:

```bash
kubectl apply -f admin-role.yaml
kubectl apply -f general-role.yaml
kubectl apply -f viewer-role.yaml
kubectl apply -f admin-cluster-role-binding.yaml
kubectl apply -f general-cluster-role-binding.yaml
kubectl apply -f viewer-cluster-role-binding.yaml
```

## Viewing Roles and Service Accounts

### View All Roles

To view the roles defined in your cluster, use the following command:

```bash
kubectl get clusterroles
```

This will display all **ClusterRole** resources in the cluster.

### View All Service Accounts

To list the service accounts in the default namespace, use:

```bash
kubectl get serviceaccounts -n default
```

You can also list service accounts in a specific namespace by changing `default` to the desired namespace.

### View ClusterRoleBindings

To view all **ClusterRoleBinding** resources, use the command:

```bash
kubectl get clusterrolebindings
```

### Describe Specific Role

To get more details about a specific **ClusterRole** (e.g., `admin-cluster-role`), use:

```bash
kubectl describe clusterrole admin-cluster-role
```

This will display detailed information about the role, including the permissions and resources it grants.

### Describe Specific Service Account

To get detailed information about a service account (e.g., `admin-service-account`), use:

```bash
kubectl describe serviceaccount admin-service-account -n default
```

This will show information about the service account, including the tokens associated with it.

### Describe ClusterRoleBinding

To describe a specific **ClusterRoleBinding** (e.g., `admin-cluster-role-binding`), use:

```bash
kubectl describe clusterrolebinding admin-cluster-role-binding
```

## Troubleshooting RBAC Issues

RBAC issues can arise when a service account or user does not have the expected permissions. Here are common troubleshooting steps:

### 1. **Check Permissions for Service Account**

If a service account is unable to access certain resources, verify that the service account has the correct **ClusterRoleBinding** or **RoleBinding**. You can do this by checking the output of:

```bash
kubectl describe serviceaccount <service-account-name> -n <namespace>
```

### 2. **Check Effective Permissions**

You can check the effective permissions a user or service account has by using the `kubectl auth can-i` command:

```bash
kubectl auth can-i <verb> <resource> --as=system:serviceaccount:<namespace>:<service-account-name>
```

For example, to check if a service account can list pods:

```bash
kubectl auth can-i list pods --as=system:serviceaccount:default:admin-service-account
```

This will return `yes` if the service account has the permission, or `no` if it does not.

### 3. **Check RoleBinding or ClusterRoleBinding**

If a service account is not granted the expected permissions, verify that it is properly bound to the correct **ClusterRole** or **Role**. Use the following commands to inspect bindings:

```bash
kubectl describe clusterrolebinding <binding-name>
kubectl describe rolebinding <binding-name> -n <namespace>
```

### 4. **Review ClusterRole Rules**

Ensure that the **ClusterRole** includes the necessary resources and verbs (actions like `get`, `list`,

 `create`, `delete`). Review the role with:

```bash
kubectl describe clusterrole <role-name>
```

### 5. **Audit Logs**

Check the Kubernetes API server's audit logs to see if any access attempts are being denied due to RBAC issues. These logs may provide additional context on why access is being blocked.

Here’s an expanded section to include the creation of tokens for the three roles, configuring the `kubeconfig.yml` with certificate and token authentication details, and spinning up VMs to test the access.

---

## Creating Tokens for Service Accounts

To allow service accounts to authenticate with Kubernetes, you need to create a token for each service account. These tokens can then be used for authentication in your `kubeconfig.yml` file.

### 1. **Create Tokens for Admin, General, and Viewer Service Accounts**

Run the following commands to create a token for each service account:

```bash
# Create token for admin service account
kubectl create token admin-service-account -n default

# Create token for general service account
kubectl create token general-service-account -n default

# Create token for viewer service account
kubectl create token viewer-service-account -n default
```

This will output tokens that you can use for each service account. Keep these tokens secure as they will allow access to the Kubernetes cluster.

## Configuring `kubeconfig.yml` with Certificate and Token Authentication

### 2. **Update `kubeconfig.yml` with Certificate Authentication**

To authenticate with the Kubernetes cluster, you can use either **certificate-based authentication** or **token-based authentication**. Below is an example of how you would configure both for the Admin, General, and Viewer roles.

#### Example `kubeconfig.yml` File

```yaml
apiVersion: v1
kind: Config
clusters:
- name: kubernetes-cluster
  cluster:
    certificate-authority-data: <base64-ca-cert>
    server: https://<master-node-ip>:6443
users:
- name: admin-user
  user:
    token: <admin-token>
- name: general-user
  user:
    token: <general-token>
- name: viewer-user
  user:
    token: <viewer-token>
contexts:
- name: admin-context
  context:
    cluster: kubernetes-cluster
    user: admin-user
- name: general-context
  context:
    cluster: kubernetes-cluster
    user: general-user
- name: viewer-context
  context:
    cluster: kubernetes-cluster
    user: viewer-user
current-context: admin-context
```

Replace the following placeholders:
- `<base64-ca-cert>`: The base64-encoded certificate authority (CA) certificate. You can get the CA certificate from your Kubernetes master node.
- `<master-node-ip>`: The IP address of your Kubernetes master node.
- `<admin-token>`, `<general-token>`, `<viewer-token>`: The tokens you generated for each service account.

### 3. **Obtain the CA Certificate**

To get the CA certificate for your Kubernetes cluster, run the following command on the master node:

```bash
cat /etc/kubernetes/pki/ca.crt | base64 -w 0
```

This will provide the base64-encoded CA certificate, which you can then paste into the `kubeconfig.yml` file.

## Spinning up Virtual Machines (VMs) and Testing Access

### 4. **Spin up 3 VMs for Each Role**

To test the access for each service account, you can spin up three virtual machines (VMs). Each VM will simulate a user with one of the roles.

You can use any cloud provider or virtual machine manager (such as **AWS EC2**, **Google Cloud Compute Engine**, or **VirtualBox**) to create the VMs. Here’s an example of how to create a VM using Google Cloud Platform (GCP):

```bash
# Spin up an Admin VM
gcloud compute instances create admin-vm --image-family=debian-11 --image-project=debian-cloud --zone=us-central1-a

# Spin up a General VM
gcloud compute instances create general-vm --image-family=debian-11 --image-project=debian-cloud --zone=us-central1-a

# Spin up a Viewer VM
gcloud compute instances create viewer-vm --image-family=debian-11 --image-project=debian-cloud --zone=us-central1-a
```

### 5. **Configure Access for Each VM**

Once the VMs are created, connect to each VM and copy the respective `kubeconfig.yml` file onto each instance.

#### Admin VM

On the **Admin VM**, you would copy the `kubeconfig.yml` file with the `admin-token` and set it as the KUBECONFIG environment variable:

```bash
export KUBECONFIG=/path/to/admin-kubeconfig.yml
kubectl get pods --all-namespaces
```

This VM should be able to list all pods and resources across namespaces.

#### General VM

On the **General VM**, copy the `kubeconfig.yml` with the `general-token`:

```bash
export KUBECONFIG=/path/to/general-kubeconfig.yml
kubectl get pods --all-namespaces
```

This VM should be able to view pods but may not be able to modify them.

#### Viewer VM

On the **Viewer VM**, copy the `kubeconfig.yml` with the `viewer-token`:

```bash
export KUBECONFIG=/path/to/viewer-kubeconfig.yml
kubectl get pods --all-namespaces
```

This VM should only be able to view resources and should not have access to create, update, or delete them.

### 6. **Verify Access**

To verify that the access control is working as expected:

- On the **Admin VM**, you should be able to run full commands to access, create, and delete resources.
- On the **General VM**, you should be able to view and modify resources like **pods** and **deployments**, but not **roles** or **cluster roles**.
- On the **Viewer VM**, you should only be able to list resources but not make any modifications.

### 7. **Troubleshoot Permissions**

If any of the VMs encounter issues accessing resources, follow these troubleshooting steps:

- **Check the Kubeconfig**: Ensure the correct token is being used for each service account.
- **Review RBAC Policies**: Ensure the service accounts are properly bound to the correct roles and role bindings.
- **Check Logs**: Review the Kubernetes logs for any access denials related to the service accounts.

---

## Conclusion

By following this guide, you have created three distinct RBAC roles for Admin, General, and Viewer access in Kubernetes. You have configured `kubeconfig.yml` with both certificate and token-based authentication, and spin up VMs to test the access for each role. The troubleshooting section helps ensure that permissions are correctly set and gives you tools to debug any access issues.


