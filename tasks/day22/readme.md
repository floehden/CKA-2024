# Day 22 - Kubernetes Authentication and Authorization

Today we had the session about Authentication and Authorization in Kubernetes. 
With Authentication we tell the cluster who we are. This is handled by key encryption and certificates, what we already learned in the past sessions.
Only through this Authorization can work in the first place, because Authorization is about what we can do, so what permissions a user has in the cluster.
There are different types of authorization
* [ABAC](https://kubernetes.io/docs/reference/access-authn-authz/abac/) (Attribute based authorization)
 * The API Server has a policy file with the users and all the permissions. 
* [RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) (Role based authorization)
 * permissions are added to a role and the user gets a role, this is the standart way to do it, also for most IAMs
* [NODE](https://kubernetes.io/docs/reference/access-authn-authz/node/) (Node based authorization)
 * Node authorization is a special-purpose authorization mode that specifically authorizes API requests made by kubelets.
* [Webhook](https://kubernetes.io/docs/reference/access-authn-authz/webhook/)
 * gives the user management to a third party like OpenPolicy or other tools


## Tasks
The [tasks](https://github.com/piyushsachdeva/CKA-2024/blob/main/Resources/Day22/task.md) today were to rerun the commands of the [video](https://www.youtube.com/watch?v=P0bogYEyfeI) and get to know where files and configs necessary for this topic are.
Like accesing the config of the API Server to see what type of authentication and authorization it is started with

For this we go into the control-plane docker container of kind
```
docker exec -it <container-name> bash
```
Than we look into the API-Server Config file
```
cat /etc/kubernetes/manifests/kube-apiserver.yaml
```
or to get the line directly 
```
cat /etc/kubernetes/manifests/kube-apiserver.yaml  | grep authorization
```
In this case it shows 
```
    - --authorization-mode=Node,RBAC
```

also we looked up, where all the keys and certificates are stored with
```
ls /etc/kubernetes/pki
```