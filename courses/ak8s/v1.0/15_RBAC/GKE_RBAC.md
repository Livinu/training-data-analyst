# [Implementing Role-Based Access Control With Kubernetes Engine](https://googlepluralsight.qwiklabs.com/focuses/40226)

Overview
In this lab, you will create namespaces within a GKE cluster, and then use role-based access control to permit a non-admin user to work with Pods in a specific namespace.

Objectives
In this lab, you learn how to perform the following tasks:

Create namespaces for users to control access to cluster resources
Create roles and rolebindings to control access within a namespace

## Task 1: Create namespaces for users to access cluster resources
Username 2 currently has access to the project, but only possesses the Viewer role, which makes all resources in the project visible, but read-only.

### Create a GKE cluster
In Cloud Shell

  ```bash
  export my_zone=us-central1-a
export my_cluster=standard-cluster-1

source <(kubectl completion bash)
```


####  Create VPC native GKE cluster
```bash
gcloud container clusters create $my_cluster \
   --num-nodes 3 --enable-ip-alias --zone $my_zone
   ```
   Configure access to cluster for kubectl
   
   ```bash
   gcloud container clusters get-credentials $my_cluster --zone $my_zone
```


Clone repo verify the versions on clone
```bash
git clone https://github.com/GoogleCloudPlatformTraining/training-data-analyst
```
cd to the folder
```bash
cd ~/training-data-analyst/courses/ak8s/15_RBAC/
```
#### Create a Namespace
verify existing namespaces
```bash
kubectl get namespaces
```
Create a production namespace with the given yaml file: my-namespace.yaml
```bash
kubectl create -f ./my-namespace.yaml
kubectl get namespaces
```

View details of the given namespace production
```bash
kubectl describe namespaces production
```

#### Create Ressource (pod) in the namespace
```bash
kubectl apply -f ./my-pod.yaml --namespace=production
```
```bash
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    name: nginx
  namespace: production
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
    
```
    
```bash
    kubectl apply -f ./my-pod.yaml --namespace=production
 ```
```bash
    kubectl get pods
  ```
## Task 2. About Roles and RoleBindings
    
   In this task you will create a sample custom role, and then create a RoleBinding that grants Username 2 the editor role in the production namespace.

The role is defined in the pod-reader-role.yaml file that is provided for you. This manifest defines a role called pod-reader that provides create, get, list & watch permission for Pod objects in the production namespace. Note that this role cannot delete Pods.
   
   
   ## Create a custom role
    
```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: production
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["create", "get", "list", "watch"]

```
  To grant the Username 1 account cluster-admin privileges, run the following command, replacing [USERNAME_1_EMAIL] with the email address of the Username 1 account:
  
```bash
kubectl create clusterrolebinding cluster-admin-binding --clusterrole cluster-admin --user [USERNAME_1_EMAIL]
```

```bash
kubectl apply -f pod-reader-role.yaml
```

```bash
kubectl apply -f pod-reader-role.yaml
```
```bash
kubectl get roles --namespace production
```

## Create a RoleBinding

The role is used to assign privileges, but by itself it does nothing. The role must be bound to a user and an object, which is done in the RoleBinding.

The username2-editor-binding.yaml manifest file creates a RoleBinding called username2-editor for the second lab user to the pod-reader role you created earlier. That role can create and view Pods but cannot delete them.

```yaml
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: username2-editor
  namespace: production
subjects:
- kind: User
  name: [USERNAME_2_EMAIL]
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```



```bash
export USER2=[USERNAME_2_EMAIL]
```


```bash
sed -i "s/\[USERNAME_2_EMAIL\]/${USER2}/" username2-editor-binding.yaml

to verify use: cat username2-editor-binding.yaml
```
Test Access
Now you will test whether Username 2 can create a Pod in the production namespace by using Username 2 to create a Pod using the manifest file production-pod.yaml. This manifest deploys a simple Pod with a single nginx container

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: production-pod
  labels:
    name: production-pod
  namespace: production
spec:
  containers:
  - name: production-pod
    image: nginx
    ports:
    - containerPort: 8080
```
Because this is a separate user account you need to prepare the Cloud Shell environment a second time so that you have access to the Cluster and the sample files from the lab repository. do as above until get credetials

```bash
gcloud container clusters get-credentials $my_cluster --zone $my_zone
```

```bash
git clone https://github.com/GoogleCloudPlatformTraining/training-data-analyst
```


```bash
cd ~/training-data-analyst/courses/ak8s/15_RBAC/
kubectl apply -f ./production-pod.yaml
```


```bash
kubectl apply -f username2-editor-binding.yaml
```


```bash
kubectl get rolebinding
```

In the Cloud Shell for Username 1, execute the following command with the production namespace specified:

```bash
kubectl get rolebinding --namespace production
```
Switch back to the Username 2 GCP Console tab.
```bash
kubectl apply -f ./production-pod.yaml
```
    
   ```bash
kubectl get pods --namespace production
```
Verify that only the specific RBAC permissions granted by the pod-reader role are in effect for Username 2 by attempting to delete the production-pod.
```bash
kubectl delete pod production-pod --namespace production
```
```bash

```



