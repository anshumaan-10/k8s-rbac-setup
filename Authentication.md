Setting Up Basic Authentication in Kubernetes
Important Notes:

Basic authentication was deprecated in Kubernetes v1.19 and is no longer available in later releases.
It is not recommended for production environments due to security risks. This tutorial is solely for learning purposes.
Step 1: Create User Details File
Create a file to store the user credentials. This file will be mounted into the kube-apiserver.

Path: /tmp/users/user-details.csv
Contents of the File:

password123,user1,u0001
password123,user2,u0002
password123,user3,u0003
password123,user4,u0004
password123,user5,u0005
Step 2: Edit the Kube-APIServer Static Pod Configuration
The kube-apiserver configuration file in a kubeadm setup is located at:
/etc/kubernetes/manifests/kube-apiserver.yaml.

Modify this file to include the basic-auth file and mount the directory where the user details file is stored.

Add Volume and Volume Mounts:
apiVersion: v1
kind: Pod
metadata:
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
      <content-hidden>
    image: k8s.gcr.io/kube-apiserver-amd64:v1.11.3
    name: kube-apiserver
    volumeMounts:
    - mountPath: /tmp/users
      name: usr-details
      readOnly: true
  volumes:
  - hostPath:
      path: /tmp/users
      type: DirectoryOrCreate
    name: usr-details
Add Basic Authentication Flag:
Update the command section of the file to include the --basic-auth-file flag:
- --authorization-mode=Node,RBAC
<content-hidden>
- --basic-auth-file=/tmp/users/user-details.csv
Step 3: Create Roles and Role Bindings
For users to access specific Kubernetes resources, define appropriate roles and role bindings.

Role Definition:

Create a role that allows reading pods in the default namespace.

---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
Role Binding:

Bind the pod-reader role to the user (e.g., user1).

---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: user1 # Name is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
Step 4: Authenticate Using Basic Authentication
To access the Kubernetes API, use curl with the username and password from the user details file.

curl -v -k https://localhost:6443/api/v1/pods -u "user1:password123"
Example Output
Upon successful authentication, the API server will return a list of pods in the default namespace:

{
  "kind": "PodList",
  "apiVersion": "v1",
  "metadata": {
    "selfLink": "/api/v1/namespaces/default/pods",
    "resourceVersion": "12345"
  },
  "items": [
    {
      "metadata": {
        "name": "nginx-64f497f8fd-krkg6",
        "namespace": "default",
        "labels": {
          "pod-template-hash": "2090539498",
          "run": "nginx"
        }
      }
    }
  ]
}
Conclusion
While basic authentication is easy to set up for testing and learning, it is not secure and should never be used in production. Use more secure and scalable options such as certificates, OAuth, or external identity providers.
