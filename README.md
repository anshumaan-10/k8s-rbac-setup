Here's a sample README to guide you in setting up RBAC (Role-Based Access Control) for different types of roles in Kubernetes, such as **Admin**, **General**, and **Other Roles** (like Viewer, Developer, etc.).

---

# Kubernetes RBAC Setup

This document provides a guide to setting up Kubernetes Role-Based Access Control (RBAC) for different roles in your Kubernetes cluster. RBAC allows you to define fine-grained access policies for users or service accounts to control what actions they can perform on resources in the cluster.

## Table of Contents

1. [Introduction](#introduction)
2. [Role Definitions](#role-definitions)
   - [Admin Role](#admin-role)
   - [General Role](#general-role)
   - [Other Roles](#other-roles)
3. [RBAC Resources](#rbac-resources)
4. [Applying RBAC to Users](#applying-rbac-to-users)
5. [Troubleshooting](#troubleshooting)

## Introduction

RBAC in Kubernetes enables cluster administrators to grant or restrict access to resources based on roles. Each role is associated with a set of permissions that define what actions a user or service account can take on cluster resources.

In this guide, we’ll create 4 different roles:
- **Admin**
- **General**
- **Viewer**
- **Developer**

## Role Definitions

### Admin Role

The **Admin** role grants full access to all Kubernetes resources in the cluster. Users with this role can manage all resources and perform any action within the cluster.

#### YAML for Admin Role

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  # Admin Role Name
  # ClusterRole enables access to resources across the entire cluster.
  # Modify if specific namespace-level access is needed.
  name: cluster-admin
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
```

This role allows full access across all namespaces, including control over the RBAC system itself.

### General Role

The **General** role grants permissions for day-to-day operations, such as viewing and editing resources within namespaces but with more restricted access compared to the Admin role.

#### YAML for General Role

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: general-user
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
```

The **General** role has more limited permissions than the **Admin** role and is ideal for users who need to manage and interact with applications but not the cluster's infrastructure or RBAC resources.

### Other Roles

#### Viewer Role

The **Viewer** role is designed for users who only need read-only access to resources in the cluster.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: viewer
rules:
  - apiGroups: [""]
    resources: ["pods", "services", "deployments", "configmaps"]
    verbs: ["list", "get"]
  - apiGroups: ["apps"]
    resources: ["deployments"]
    verbs: ["list", "get"]
```

This role only allows the user to view resources, without any permissions to modify them.

#### Developer Role

The **Developer** role provides a set of permissions needed to deploy and manage applications, but does not allow administrative access to cluster-wide resources.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: developer
rules:
  - apiGroups: [""]
    resources: ["pods", "services", "deployments", "configmaps"]
    verbs: ["create", "get", "update", "delete"]
  - apiGroups: ["apps"]
    resources: ["deployments", "replicasets"]
    verbs: ["create", "get", "update", "delete"]
```

This role allows developers to manage application resources like deployments and services but does not provide access to cluster management tasks.

## RBAC Resources

### RoleBinding

Once the roles are created, you'll need to bind them to users, service accounts, or groups using **RoleBindings** or **ClusterRoleBindings**. These resources define which users or service accounts are assigned specific roles.

#### Example: Assigning Admin Role

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-binding
subjects:
  - kind: User
    name: "admin-user"
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
```

This binds the **cluster-admin** role to a user named **admin-user**.

### Applying RBAC to Users

Once the roles and bindings are defined, you can apply them using `kubectl`:

```bash
kubectl apply -f admin-role.yaml
kubectl apply -f general-role.yaml
kubectl apply -f viewer-role.yaml
kubectl apply -f developer-role.yaml
kubectl apply -f admin-binding.yaml
```

## Troubleshooting

- **Access Denied Errors**: If a user is denied access to a resource, check the RBAC bindings and ensure the correct roles are assigned to the user or service account.
  
- **Review Role Permissions**: You can always review the permissions granted by a role using the following command:

  ```bash
  kubectl describe clusterrole <role-name>
  ```

- **Role Conflicts**: Ensure no conflicting roles are assigned to the same user. For example, if a user is bound to both **cluster-admin** and **viewer**, the more restrictive role will not apply.

## Conclusion

With RBAC, you can implement precise access control in your Kubernetes cluster, ensuring that users and service accounts only have access to the resources they need to do their job. By defining roles such as **Admin**, **General**, **Viewer**, and **Developer**, you can provide the right level of access to ensure security and compliance in your cluster.

