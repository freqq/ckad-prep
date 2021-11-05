----------------------------
# State Persistence (8%)
----------------------------

1. List Persistent Volumes in the cluster

kubectl get pv

2. Create a hostPath PersistentVolume named task-pv-volume with storage 10Gi, access modes ReadWriteOnce, storageClassName manual, and volume at /mnt/data and verify

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
    name: task-pv-volume
spec:
    storageClassName: manual
    hostPath:
        path: /mnt/data
    capacity:
        storage: 10Gi
    accessModes:
    - ReadWriteOnce
```

kubeclt create pv.yaml
kubectl get pv

3. Create a PersistentVolumeClaim of at least 3Gi storage and access mode ReadWriteOnce and verify status is Bound

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
    name: task-pv-claim
spec:
    storageClassName: manual
    resources:
        requests:
            storage: 3Gi
    accessModes:
    - ReadWriteOnce
```

kubeclt create pvc.yaml
kubectl get pvc

4. Delete persistent volume and PersistentVolumeClaim we just created

kubectl delete pv task-pv-volume
kubectl delete pvc task-pv-claim

5. Create a Pod with an image Redis and configure a volume that lasts for the lifetime of the Pod

kubectl run redis --image=redis --restart=Never --dry-run=client -o yaml > pod.yaml

Add:

...
    volumeMounts:
    - name: empty-dir
      mountPath: /data/redis
...
    volumes:
    - name: empty-dir
      emptyDir: {}
...

kubectl create -f pod.yaml

6. Exec into the above pod and create a file named file.txt with the text ‘This is called the file’ in the path /data/redis and open another tab and exec again with the same pod and verifies file exist in the same path.

kubectl exec redis -it -- /bin/sh
cat "This is called the file" > /data/redis/file.txt

Another tab:
kubectl exec redis -it -- /bin/sh
cat /data/redis/file.txt

8. Delete the above pod and create again from the same yaml file and verifies there is no file.txt in the path /data/redis.

kubectl delete pod redis
kubectl create -f pod.yaml
kubectl exec redis -it -- /bin/sh
cat /data/redis/file.txt # File doesn't exists

9. Create PersistentVolume named task-pv-volume with storage 10Gi, access modes ReadWriteOnce, storageClassName manual, and volume at /mnt/data and Create a PersistentVolumeClaim of at least 3Gi storage and access mode ReadWriteOnce and verify status is Bound

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
    name: task-pv-volume
spec:
    storageClassName: manual
    hostPath:
        path: /mnt/data
    capacity:
        storage: 10Gi
    accessModes:
    - ReadWriteOnce
```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
    name: task-pv-claim
spec:
    storageClassName: manual
    resources:
        requests:
            storage: 3Gi
    accessModes:
    - ReadWriteOnce
```

kubectl create -f pv.yaml
kubectl create -f pvc.yaml
kubectl get pvc

10. Create an nginx pod with containerPort 80 and with a PersistentVolumeClaim task-pv-claim and has a mouth path "/usr/share/nginx/html"

kubectl run nginx --image=nginx --dry-run=client -o yaml --restart=Never > pod.yaml

Add:

...
    volumeMounts:
    - name: pvc-volume
      mountPath: /usr/share/nginx/html
...
    volumes:
    - name: pvc-volume
      persistentVolumeClaim:
        claimName: task-pv-claim
...

kubectl create -f pod.yaml