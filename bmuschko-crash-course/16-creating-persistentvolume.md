## Defining and Mounting a PersistentVolume

1. Create a Persistent Volume named `pv`, access mode `ReadWriteMany`, storage class name `shared`, 512MB of storage capacity and the host path `/data/config`.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv
spec:
  accessModes:
  - ReadWriteMany
  storageClassName: shared
  capacity:
    storage: 512MB
  hostPath:
    path: /data/config
```

kubectl create pv.yaml

2. Create a Persistent Volume Claim named `pvc` that requests the Persistent Volume in step 1. The claim should request 256MB. Ensure that the Persistent Volume Claim is properly bound after its creation.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc
spec:
  accessModes:
  - ReadWriteMany
  storageClassName: shared
  requests:
    resources: 256MB
```

kubectl create pvc.yaml
kubectl get pvc

3. Mount the Persistent Volume Claim from a new Pod named `app` with the path `/var/app/config`. The Pod uses the image `nginx`.

kubectl create app --image=nginx --dry-run=client -o yaml > pod.yaml

Add:

...
    volumeMounts:
    - name: app-pv
      mountPath: /var/app/config
...
    volumes:
    - name: app-pv
      persistentVolumeClaim:
        claimName: pvc
...

kubectl create -f pod.yaml

4. Check the events of the Pod after starting it to ensure that the Persistent Volume was mounted properly.

kubectl describe pod app