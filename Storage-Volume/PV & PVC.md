## 1. PersistentVolume (PV)

**Definition:**  
A **PersistentVolume (PV)** is a storage resource available in a Kubernetes cluster for storing persistent data.

- Data survives Pod deletion/recreation.
- Can be backed by AWS EBS, NFS, Azure Disk, etc.
- Usually managed/provisioned by the cluster administrator or dynamically by a StorageClass.

### Example PV
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

  persistentVolumeReclaimPolicy: Retain

  hostPath:
    path: /mnt/data
```

```shell
kubectl apply -f my-pv.yaml
kubectl get pv
```

---
## 2. PersistentVolumeClaim (PVC)

**Definition:**  
A **PersistentVolumeClaim (PVC)** is a request by an application/user for persistent storage.

Example:
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

```shell
kubectl apply -f my-pvc.yaml
kubectl get pvc
```

---
## 3. PV vs PVC

| PV                                  | PVC                         |
| ----------------------------------- | --------------------------- |
| Storage resource                    | Storage request             |
| Provides storage                    | Requests storage            |
| Created/provisioned by admin/system | Created by application/user |
| Example: 5Gi                        | Example: request 2Gi        |

**Remember:**
> **PV = Storage**  
> **PVC = Request for storage**

---
## 4. How Pod Uses PVC

```yaml
volumeMounts:
  - name: storage
    mountPath: /data

volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: my-pvc
```

The application accesses:
```shell
/data
```

and the data is stored through:
```shell
Pod → PVC → PV → Actual Storage
```

---
## 5. Access Modes

### RWO — ReadWriteOnce
```shell
Read + Write
One node
```

Common with block storage such as AWS EBS.
### ROX — ReadOnlyMany
```shell
Read only
Multiple nodes
```

### RWX — ReadWriteMany
```shell
Read + Write
Multiple nodes
```

Requires a storage backend that supports shared access, such as NFS/EFS.

---
## 6. Reclaim Policy

### Retain
```shell
PVC deleted
   ↓
PV/data retained
```

Good for important data.
### Delete
```shell
PVC deleted
   ↓
Storage may be deleted
```

Common with dynamically provisioned cloud storage, depending on the StorageClass configuration.

---
## 7. StorageClass

**StorageClass = dynamic storage provisioning.**

Instead of manually creating PVs:
```shell
PVC
 ↓
StorageClass
 ↓
Storage automatically created
 ↓
PV
```

In AWS:
```shell
PVC
 ↓
StorageClass
 ↓
EBS Volume
 ↓
PV
```

---
## 8. Production Use

For a production application:
```shell
Deployment
    ↓
Pod
    ↓
PVC
    ↓
StorageClass
    ↓
PV
    ↓
Cloud Storage
```

Example:
```shell
PostgreSQL
    ↓
postgres-pvc
    ↓
EBS-backed PV
```

If the PostgreSQL Pod is deleted:
```shell
Old Pod ❌
   ↓
New Pod ✅
   ↓
Same PVC
   ↓
Same persistent data
```

---
## 9. Important Commands
```shell
# PV
kubectl get pv
kubectl describe pv my-pv

# PVC
kubectl get pvc
kubectl describe pvc my-pvc

# StorageClass
kubectl get storageclass

# Check Pod
kubectl get pods

# Check mounted volume
kubectl describe pod <pod-name>
```