# [Day 24](https://www.youtube.com/watch?v=DswQe7shSa4) - Kubernetes RBAC Continued - Clusterrole and Clusterrole Binding

In the [previous](https://www.youtube.com/watch?v=uGcDt7iNFkE) day we learned about how we can attach roles to groups or users. These roles are namespace scoped.
Namespace scoped resources are: pods, deployments, rs or also services
Cluster scoped resources are: namespaces, nodes and others

We can also Cluster Roles. These can for example list node, watch nodes and get nodes.
Similar to Role and RoleBindungs, we can bind cluster roles to user and/or groups with ClusterRoleBindings.

There is also a speciality with Roles. As longs as we don't apply any namespace in the role, it applies to all namespaces, so it's at cluster level.

To get all the resources at namespace level type in:
```
kubectl api-resources --namespaced=true
```
When we put --namespaced=false we can see all the resources at cluster level.

To create a cluster role we can do it iteratively and declaritive
```
kubectl create clusterrole NAME --verb=verb --resource=resource.group
```
will do it interatively. NAME is the Name of the ClusterRole like node-reader, verb ist wha we wanna do like list,watch,get and resource.group is the resource we wanna do something with like a node

to see all the roles we can use
```
kubectl get clusterroles
```
and we can also see our new clusterrole in the list

To attach this ClusterRole, we need a ClusterRoleBinding
```
kubectl create clusterrolebinding read-nodes --clusterrole=node-reader --user=krishna
```
Now we see what nodes we have in the cluster with the user krishna. For groups we need to use --group=<sample-group>. This is also the case for RoleBindings.

## [Tasks](https://github.com/piyushsachdeva/CKA-2024/blob/main/Resources/Day24/task.md)
The Tasks for were to recreate the commands and try it for some other resources, that we can see with 
```
kubectl api-resources --namespaced=false
```