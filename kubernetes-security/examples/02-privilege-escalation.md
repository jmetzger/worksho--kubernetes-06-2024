# AllowPrivilegesEscalation (Example) 

## Prerequisites 

  * We create a little image, that holds a script that can escalate

## Step 1: (needs NOT to be done, because image already exists on dockertrainereu/escalate:0.1)

```
# on your local client docker needs to run
sudo snap install docker --classic 
```

```
sudo su -
cd
mkdir escalate
cd escalate
vi Dockerfile
```

```
FROM alpine:3.20 AS builder
WORKDIR /build
RUN cat > escalate.c <<EOF
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <errno.h>

int main(void) {
    // Escalate to root
    setreuid(0, 0); setregid(0, 0);

    // Spawn a shell
    char* const argv[] = {"/bin/sh", NULL};
    char* const environ[] = {"PATH=/bin:/sbin:/usr/bin:/usr/sbin", NULL};
    if (-1 == execve("/bin/sh", argv, environ)) {
        printf("Unable to execve /bin/sh, errno %d\n", errno);
    }
}
EOF
RUN cat /build/escalate.c
RUN apk add --no-cache gcc musl-dev
RUN gcc escalate.c -Wall -o escalate

FROM alpine:3.20 AS runner
WORKDIR /app
COPY --from=builder /build/escalate ./escalate
RUN chown root:root ./escalate && chmod +s ./escalate
RUN adduser app-user --uid 1000 --system --disabled-password --no-create-home
USER app-user
ENTRYPOINT ["sh", "-c", "echo Application running && sleep infinity"]
```

```
docker build -t dockertrainereu/escalate:0.1 . 
docker login
docker push dockertrainereu/escalate:0.1
```

## Step 2: Do this as normal user (Test with allowPrivilegeEscalation : false) 

  * HINT: allowPrivilegeEscalation: true is the default 

```
cd
mkdir -p manifests
cd manifests
mkdir test-escalation
cd test-escalation
```

```
vi 01-escalate.yml
```

```
apiVersion: v1
kind: Pod
metadata:
  name: escalation-test
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 1000
  containers:
  - name: my-app
    image: dockertrainereu/escalate:0.1
    securityContext:
      allowPrivilegeEscalation: true
```

```
kubectl apply -f .
kubectl exec -it escalation-test -- sh
```

```
id
# non-root user 
./escalate
id
# now we are root 
```

## Step 3: Do this as normal user (Test with allowPrivilegeEscalation : true) 

````
# in pod manifest change AllowPrivilegeEscalation: false
```

```
kubectl delete -f .
kubectl apply -f .
kubectl exec -it test-escalation -- sh
```

```
id
# non-root user
./escalate
id
# still non-root user
```

## Reference:

  * https://blog.christophetd.fr/stop-worrying-about-allowprivilegeescalation/
