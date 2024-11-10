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

## Conclusion

RBAC roles in Kubernetes help manage user and service account permissions efficiently. By defining roles like **Admin**, **General**, and **Viewer**, you can control access to resources in the cluster. Understanding how to create roles, bind them to service accounts, and troubleshoot common issues will ensure smooth cluster operation and security.

