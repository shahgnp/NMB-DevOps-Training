# Persistent Volumes (PV) and Persistent Volume Claims (PVC) in Kubernetes

In Kubernetes, **Persistent Volumes (PV)** and **Persistent Volume Claims (PVC)** provide **persistent storage** for applications running in pods. Normally, data stored inside a pod is lost when the pod is deleted or restarted. PVs and PVCs allow data to **persist beyond the lifecycle of a pod**.

---

# 1. Persistent Volume (PV)

A **Persistent Volume (PV)** is a **piece of storage in the Kubernetes cluster** that has been **provisioned by an administrator or dynamically created**.

Think of a **PV as the actual storage resource** (like a disk).

## Key Points

* Exists **independently of pods**
* Represents real storage in the cluster
* Can be backed by different storage systems such as:

  * NFS
  * Cloud disks (AWS EBS, GCE Persistent Disk)
  * Local storage
* Has defined **capacity and access modes**

## Example PV YAML

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /data/my-storage
```

### Explanation

* `5Gi` storage available
* `ReadWriteOnce` → Only **one node can write**
* Storage located at `/data/my-storage` on the node

---

# 2. Persistent Volume Claim (PVC)

A **Persistent Volume Claim (PVC)** is a **request for storage by a user or application**.

Think of **PVC as asking Kubernetes for storage**.

## Key Points

* Pods **do not use PV directly**
* Pods **request storage using PVC**
* Kubernetes **binds the PVC to a suitable PV**

## Example PVC YAML

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
```

### Explanation

The application requests:

* `2Gi` storage
* `ReadWriteOnce` access

Kubernetes finds a **matching PV with enough capacity** and binds it to the PVC.

---

# 3. How PV and PVC Work Together

Storage workflow in Kubernetes:

1. **Administrator creates a PV**

Example:
`PV → 5GB storage available`

2. **User/Application creates a PVC**

Example:
`PVC → request 2GB`

3. **Kubernetes binds them**

`PVC → connected to PV`

4. **Pod uses PVC**

The pod mounts the storage through the PVC.

---

# 4. Using PVC Inside a Pod

Example Pod configuration:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: myapp
    image: nginx
    volumeMounts:
    - mountPath: "/data"
      name: storage
  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: my-pvc
```

Now the container can **read and write data** at:

```
/data
```

The stored data **remains even if the pod is deleted or recreated**.

---

# 5. Access Modes

| Access Mode         | Meaning                       |
| ------------------- | ----------------------------- |
| ReadWriteOnce (RWO) | One node can read/write       |
| ReadOnlyMany (ROX)  | Multiple nodes can read only  |
| ReadWriteMany (RWX) | Multiple nodes can read/write |

---

# Secrets and ConfigMaps in Kubernetes

In Kubernetes, **ConfigMaps** and **Secrets** are used to **store configuration data separately from application code**. This allows applications to be more flexible, secure, and easier to manage.

Both are used to **inject configuration into Pods**, but they are designed for **different types of data**.

---

# 1. ConfigMaps

A **ConfigMap** is used to store **non-sensitive configuration data** in key-value pairs.

Examples of data stored in ConfigMaps:

* Application configuration
* Environment variables
* Command-line arguments
* Configuration files
* URLs or service endpoints

## Features

* Stores **plain text configuration data**
* Decouples configuration from container images
* Can be consumed by pods as:

  * Environment variables
  * Command-line arguments
  * Configuration files in volumes

---

## Example ConfigMap YAML

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: production
  APP_DEBUG: "false"
  DATABASE_URL: mysql-service
```

This ConfigMap stores three configuration values.

---

## Using ConfigMap as Environment Variables

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-pod
spec:
  containers:
  - name: app-container
    image: nginx
    envFrom:
    - configMapRef:
        name: app-config
```

The container will now have environment variables:

```
APP_ENV=production
APP_DEBUG=false
DATABASE_URL=mysql-service
```

---

## Using ConfigMap as a Volume

```yaml
volumes:
- name: config-volume
  configMap:
    name: app-config
```

The ConfigMap values will appear as **files inside the container**.

---

# 2. Secrets

A **Secret** is used to store **sensitive information securely**.

Examples:

* Database passwords
* API keys
* SSH keys
* Tokens
* TLS certificates

Secrets are **base64 encoded** and handled more securely by Kubernetes.

---

## Example Secret YAML

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  username: YWRtaW4=
  password: cGFzc3dvcmQ=
```

Here:

* `username` and `password` are **Base64 encoded values**.

Example encoding:

```
echo -n "admin" | base64
```

---

## Using Secret in a Pod

### As Environment Variables

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-pod
spec:
  containers:
  - name: app
    image: nginx
    env:
    - name: DB_USERNAME
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: username
```

The container receives:

```
DB_USERNAME=admin
```

---

### As Volume

```yaml
volumes:
- name: secret-volume
  secret:
    secretName: db-secret
```

The secret values will be **mounted as files inside the container**.

---

# 3. ConfigMap vs Secret

| Feature   | ConfigMap                        | Secret                           |
| --------- | -------------------------------- | -------------------------------- |
| Purpose   | Store configuration data         | Store sensitive data             |
| Security  | Stored as plain text             | Base64 encoded                   |
| Use cases | App configs, URLs, flags         | Passwords, tokens, keys          |
| Access    | Environment variables or volumes | Environment variables or volumes |

---

