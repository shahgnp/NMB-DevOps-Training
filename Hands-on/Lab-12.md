# Kubernetes Hands-On Lab  
## ConfigMaps and Secrets

## Lab Objectives

By the end of this lab, students will be able to:

1. Create a **ConfigMap**
2. Create a **Secret**
3. Use them as **Environment Variables**
4. Mount them as **Volumes inside a Pod**
5. Verify the configuration inside a running container

---

# Lab Prerequisites

Make sure students have:

- A running **Kubernetes cluster** 
- `kubectl` installed
- Access to a terminal

Verify the cluster is working:

```bash
kubectl get nodes
```

You should see at least **one node in Ready state**.

---

# 1 Create a ConfigMap

Create a file called:

```
configmap.yaml
```

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: production
  APP_VERSION: "1.0"
  DATABASE_HOST: mysql-service
```

### Apply the ConfigMap

```bash
kubectl apply -f configmap.yaml
```

### Verify ConfigMap

```bash
kubectl get configmap
```

---

# 2 Create a Secret

We will store **database credentials**.

First encode the values.

```bash
echo -n "admin" | base64
```

Now encode password:

```bash
echo -n "password123" | base64
```

---

### Create Secret YAML

Create a file:

```
secret.yaml
```

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  username: YWRtaW4=
  password: cGFzc3dvcmQxMjM=
```

---

### Apply the Secret

```bash
kubectl apply -f secret.yaml
```

Verify:

```bash
kubectl get secrets
```
---

# 3 Use ConfigMap as Environment Variables

Create a pod that loads values from the ConfigMap.

Create file:

```
configmap-env-pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-env-pod
spec:
  containers:
  - name: nginx-container
    image: nginx
    envFrom:
    - configMapRef:
        name: app-config
```

---

### Deploy the Pod

```bash
kubectl apply -f configmap-env-pod.yaml
```

Check pod status:

```bash
kubectl get pods
```

---

### Verify Environment Variables

Open a shell inside the pod.

```bash
kubectl exec -it configmap-env-pod -- env
```

---

# 4 Use Secret as Environment Variables

Create another pod.

```
secret-env-pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-pod
spec:
  containers:
  - name: nginx-container
    image: nginx
    env:
    - name: DB_USERNAME
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: username

    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: password
```

---

### Deploy Pod

```bash
kubectl apply -f secret-env-pod.yaml
```

---

### Verify Secret Values

```bash
kubectl exec -it secret-env-pod -- env
```
---

# 5 Mount ConfigMap as a Volume

Instead of environment variables, we can mount configuration as **files**.

Create:

```
configmap-volume-pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-volume-pod
spec:
  containers:
  - name: nginx-container
    image: nginx
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config

  volumes:
  - name: config-volume
    configMap:
      name: app-config
```

---

### Deploy Pod

```bash
kubectl apply -f configmap-volume-pod.yaml
```

---

### Check Mounted Files

```bash
kubectl exec -it configmap-volume-pod -- ls /etc/config
```

Expected:

```
APP_ENV
APP_VERSION
DATABASE_HOST
```

Check file contents:

```bash
kubectl exec -it configmap-volume-pod -- cat /etc/config/APP_ENV
```

---

# 6 Mount Secret as a Volume

Create:

```
secret-volume-pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-volume-pod
spec:
  containers:
  - name: nginx-container
    image: nginx
    volumeMounts:
    - name: secret-volume
      mountPath: /etc/secret

  volumes:
  - name: secret-volume
    secret:
      secretName: db-secret
```

---

### Deploy Pod

```bash
kubectl apply -f secret-volume-pod.yaml
```

---

### Verify Secret Files

```bash
kubectl exec -it secret-volume-pod -- ls /etc/secret
```

Check contents:

```bash
kubectl exec -it secret-volume-pod -- cat /etc/secret/username
```

# 7 Cleanup Resources

After the lab:

```bash
kubectl delete pod configmap-env-pod
kubectl delete pod secret-env-pod
kubectl delete pod configmap-volume-pod
kubectl delete pod secret-volume-pod

kubectl delete configmap app-config
kubectl delete secret db-secret
```

---
