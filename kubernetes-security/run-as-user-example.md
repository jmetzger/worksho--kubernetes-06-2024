# Example runasuser 

```
cd
mkdir -p manifests
cd manifests
mkdir -p security
cd security
```

```
vi 01-pod.yml 
```

```
apiVersion: v1
kind: Pod
metadata:
  name: security-nginx 
spec:
  securityContext:
    runAsUser: 10000
  containers:
  - name: web
    image: nginx:1.22
 ```

```
kubectl apply -f .
```
