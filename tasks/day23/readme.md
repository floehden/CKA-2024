# Days 23 - Kubernetes RBAC Explained - Role Based Access Control Kubernetes

In this [session](https://www.youtube.com/watch?v=uGcDt7iNFkE) we learned about the role based access control in Kubernetes.
We learned, how to give roles to a user and restrict the access of the user, the he can only do certain things, like only get pods
For further informations look at [this](https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/).


For this we used command like 
```
kubectl auth can-i get pod --as adam
```
to get to know what we can do with the user adam (if we don't use --as adam we see it as the current user) or also 
```
kubectl auth whoami
```
to get to know what user we are and in which group we are like in this example
```
ATTRIBUTE   VALUE
Username    kubernetes-admin
Groups      [kubeadm:cluster-admins system:authenticated]
```

There are two different types of API groups in Kubernetes
* Core group - Pods and other core components of Kubernetes are in this ggroup, that's why there is just the v1 as the apiVersion
* Name group - In this example the Role is in the name group, that's why me have as apiVersion: rbac.authorization.k8s.io/v1 in the yaml

To apply the role in the role.yaml use 
```
kubectl apply -f role.yaml
```
and to see the roles use
```
kubectl get role
```
Now we only have created to role, with binding.yaml (a RoleBinding) we apply this role to a user
```
kubectl apply -f binding.yaml
```
In the description of the rolebinding read-pods we can see, that adam is now a user of the Role pod-reader
```
kubectl describe rolebinding read-pods

Name:         read-pods
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  Role
  Name:  pod-reader
Subjects:
  Kind  Name  Namespace
  ----  ----  ---------
  User  adam  
```
Now adam can also see pods which we can see with 
```
kubectl auth can-i get pod --as adam

yes
```
To count all the roles in the cluster use 
```
k get roles -A --no-headers | wc -l

12
```
To change the user we need to set the user in kubectl with this
```
kubectl config set-credentials adam --client-key=../day21/adam.key --client-certificate=../day21/adam.crt --embed-certs=true
```
We need the name of the user, the key and the certificate of the client
We also need to set a context for the user with 
```
kubectl config set-context adam cluster=kind-cka-course --user=adam

CURRENT   NAME              CLUSTER           AUTHINFO          NAMESPACE
          adam              kind-cka-course   adam              
*         kind-cka-course   kind-cka-course   kind-cka-course   
```
So far we are not logged in with adam. TO do that, we need to switch the context with 
```
kubectl config use-context adam
```


## Tasks
The tasks are the following

### Switch back to the original context with admin permissions.
For this first get the Contexts with
```
kubectl config get-contexts
```
Use the one named as the cluster itself, in my case: kind-cka-course
```
kubectl config use-context kind-cka-course
```

### Create a new Role named pod-reader. The Role should grant permissions to get, watch and list Pods.
The role.yaml looks like this
```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the API group
  resources: ["pods"] # type of ressource to grant access
  verbs: ["get", "watch", "list"] # what we can do with this ressource
```
Apply it!

### Create a new RoleBinding named read-pods. Map the user krishna to the Role named pod-reader.
The binding.yaml that the user krishna is in the role pod-reader looks like this
```
apiVersion: rbac.authorization.k8s.io/v1
# This role binding allows "jane" to read pods in the "default" namespace.
# You need to already have a Role named "pod-reader" in that namespace.
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
# You can specify more than one "subject"
- kind: User
  name: krishna # "name" is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  # "roleRef" specifies the binding to a Role / ClusterRole
  kind: Role #this must be Role or ClusterRole
  name: pod-reader # this must match the name of the Role or ClusterRole you wish to bind to
  apiGroup: rbac.authorization.k8s.io
```

### Make sure that both objects have been created properly.
To see the role use
```
kubectl get role
```
And you should see the pod-reader role
With the following you can see, that the RoleBinding read-pods added the role pod-reader to Krishna
```
k describe rolebinding read-pods

Name:         read-pods
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  Role
  Name:  pod-reader
Subjects:
  Kind  Name     Namespace
  ----  ----     ---------
  User  krishna  
```

### Switch to the context named krishna.
```
kubectl config use-context krishna
```


### Create a new Pod named nginx with the image nginx. What would you expect to happen?
To create an nginx pod we use
```
kubectl run nginx --image=nginx:latest
```
I expect it to fail, beucase the Role has only the right to watch, list and get Pods
And it failed, because we cannot create the resource pods
```
Error from server (Forbidden): pods is forbidden: User "krishna" cannot create resource "pods" in API group "" in the namespace "default"
```


### List the Pods in the namespace. What would you expect to happen?
I expect it to get through, because listing is allowed in the Role pod-reader
```
kubectl get pods

kubectl get pods -A # Did not work

Error from server (Forbidden): pods is forbidden: User "krishna" cannot list resource "pods" in API group "" at the cluster scope
```

### Create a new deploymened named nginx-deploy. What would you expect to happen?
The deployment is in deploy.yaml and I expect it to fail, becasue we are not allowed to do anything with the resource deployment
```
kubectl apply -f deploy.yaml

Error from server (Forbidden): error when retrieving current configuration of:
Resource: "apps/v1, Resource=deployments", GroupVersionKind: "apps/v1, Kind=Deployment"
Name: "nginx-deploy", Namespace: "default"
from server for: "deploy.yaml": deployments.apps "nginx-deploy" is forbidden: User "krishna" cannot get resource "deployments" in API group "apps" in the namespace "default"
```
