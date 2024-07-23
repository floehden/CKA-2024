# Tasks for Day 21 of CKA Course by Piyush
In Day 21 we had to generate a SSL certificate for the usage in K8s. In the following Tasks it is explained step by step how to do it.


## 1. Generate a PKI private key and CSR and name it as learner.key and learner.csr
```
openssl genrsa -out learner.key 2048
```
```
openssl req -new -key learner.key -out learner.csr -subj "/CN=learner"
```

## 2. Create a CertificateSigningRequest for learner and set the expiration date to 1 week
```
touch csr.yaml
```
[The basic signing request](https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/#create-certificatessigningrequest)
```
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: myuser
spec:
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1Z6Q0NBVDhDQVFBd0VqRVFNQTRHQTFVRUF3d0hiR1ZoY201bGNqQ0NBU0l3RFFZSktvWklodmNOQVFFQgpCUUFEZ2dFUEFEQ0NBUW9DZ2dFQkFPQ0pBZnd1M3lKZkFTUlpjUUQraFI0cjlNMEJXRituZWduTUwxNDhNdXlSClAxRmZzZFRoN2dwQTlkYjROemNSNGJEVGFVZXlmcjVOTmdsRkFHeWZyZFlab05IajZxSHNkVzRhbVhWUHpYSmMKNURYVWxVekhwYUJhU1FCamJQVGd5QkNteklicjlYQ0k3Q0NrUWhCb0cyUFdRM0pmQ2FKYms1WG1zUittalFCWgpvZG9ScHl4S0R5SlJPdko1ZGl3M1E2eTViczJNK1d5R0UvVzVxZCthaUVWaS9aUHNBM0FHR011bXJhamdYK2p5CkZIUnB5a1AwaHV3SklwS3B5WUVXaElGSzNJcXAzT3ZpOWZhN1BGOExvZDg5ME42cmx2OGtINFFzOHdSNWlaQTMKYWgyQnBuWHplM3hkRnlIT2dJcTNXRkhTWUpSZmU2RmNUNFFRYVdJL0Z5Y0NBd0VBQWFBQU1BMEdDU3FHU0liMwpEUUVCQ3dVQUE0SUJBUUI2UWdNZVdLais2QlRlTVNWSXZjZGFYOEM5V3kxL3krOHBYb3Y1K1JKbS9leTRwbTV5CmdUZGhoOWVwQ0xPTk9WOXN6dE50aHM2ZDhmMDRCY0ZRRWJ1enJqTXhyeEVFMExGUFRrREpKUWVqaFNvc0NNdEIKcHcwUGtBR3JQWFZpRXlSckVsOUFLRFZmQVZWQnh0bXVuVTdFd3B3MUNlQjFNVEdqTFhWbzBSRitmc2pTKzByNgpmd1FOOFVacEZXVlhnd0VhTjlwOWZrck5SSTdBdmdxYk5CKy8xTzZUWTBPUTJ1c3hobGFoa1F1SitGODQrN0QzCkJ4QVJpMlhRb1hILytpVWRzWlZ1WGJDVWZnQXp3dnZKQTNKWm9OclMzcWlFWi81SjdwU0tjM1hTcWI5WWtyNUMKZjhmSGg0L1l1QS9teFlCL2VUWXJZQWEycDRla1FZVkZ1bER3Ci0tLS0tRU5EIENFUlRJRklDQVRFIFJFUVVFU1QtLS0tLQo=
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 604800  # seven day
  usages:
  - client auth
```



## 3. Make sure to use the encoded value of csr in the request field
```
cat learner.csr | base64 | tr -d "\n"
```
```
kubectl apply -f csr.yaml
```

## 4. Approve the csr
```
kubectl certificate approve learner
```
```
CreationTimestamp:   Mon, 22 Jul 2024 15:34:44 +0200
Requesting User:     kubernetes-admin
Signer:              kubernetes.io/kube-apiserver-client
Requested Duration:  7d
Status:              Approved,Issued
```

## 5. Retrieve the certificate from the CSR
```
kubectl describe csr/learner
```
The certification string is in the field certificate

## 6. Export the issued certificate from the CertificateSigningRequest to a yaml
kubectl get csr/learner -o yaml

## 7. Redirect the certificate value to learner.crt file after decoding it
```
kubectl get csr learner -o jsonpath='{.status.certificate}'| base64 -d > learning.crt
```
