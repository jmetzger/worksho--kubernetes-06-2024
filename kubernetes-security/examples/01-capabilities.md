# Capabilities 

```
# Remove cap_chown
CAP_CHOWN
```

```
apiVersion: v1
kind: Pod
metadata:
  name: capability-demo
spec:
  containers:
    - name: demo-container
      image: busybox
      command: ["sh", "-c", "sleep 3600"]
      securityContext:
        capabilities:
          drop:
            - NET_RAW
```

```
kubectl apply -f .
# ping does not work now in container
```
