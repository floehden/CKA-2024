# [Day 25](https://www.youtube.com/watch?v=k2iCq7IlMKM) - Kubernetes Service Account - RBAC Continued

In this session we learned about services account in Kubernetes.
There are different types of users in Kubernetes
* human users - a human user using kubectl or other API Services
* services accounts - these accounts are for the interaction between services and applications or even monitoring and automation

They can be used the same and put in groups or get roles via RoleBindings.
There is always the default Service Account, but always use it with cautions.

### Create a Service Accounts
Creating it via commandline works like this
```
kubectl create sa <sa-name>
```
We can also add a secret to it to make it a long living account
```
apiVersion: v1
kind: Secret
metadata:
  name: build-robot-secret
  annotations:
    kubernetes.io/service-account.name: <sa-name>
type: kubernetes.io/service-account-token
```

### Add permissions to the Service Account
Look up if the service account can get pods
```
kubectl auth can-i get pods --as build-sa
```
Add role for getting pods
```
kubectl create role build-role --verb=get,list,watch --resource=pod
kubectl create rolebinding rb --role=build-role --user=build-sa
```
Now we can get pods with the service account





For Further informations go to [this](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/).

## [Tasks](https://github.com/piyushsachdeva/CKA-2024/tree/main/Resources/Day25)

### [Add ImagePullSecret to a service account](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#add-imagepullsecrets-to-a-service-account)
ImagePullSecrets are necessary to authenticate and pull Container Images from private container registries.

We can create this secret with
```
kubectl create secret docker-registry GHCR-secret --docker-server=ghcr.io \
        --docker-username=<github-username> --docker-password=<github-accesstoken> \
        --docker-email=<github-email>
```
and add the secret to the service account 
```
kubectl patch serviceaccount build-sa -p '{"imagePullSecrets": [{"name": "GHCR-secret"}]}'
```

Test
```
kubectl run ghnginx --image=ghcr.io/linuxserver/nginx:1.26.1-r0-ls283 --as build-sa
```
