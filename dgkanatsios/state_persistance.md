1. Create busybox pod with two containers, each one will have the image busybox and will run the 'sleep 3600' command. Make both containers mount an emptyDir at '/etc/foo'. Connect to the second busybox, write the first column of '/etc/passwd' file to '/etc/foo/passwd'. Connect to the first busybox and write '/etc/foo/passwd' file to standard output. Delete pod.

kubectl run busybox --image=busybox --restart=Never --dry-run=client -o yaml -- /bin/sh -c 'sleep 3600' > pod.yaml

Add:

...
    volumeMounts:
    - name: empty-dir
      mountPath: '/etc/foo'
...
  volumes:
  - name: empty-dir
    emptyDir: {}
...

Duplicate containers.

kubectl create -f pod.yaml
kubectl exec -it busybox -c busybox2 -- /bin/sh
cat /etc/passwd | head -1 > /etc/foo/passwd
exit
kubectl exec -it busybox -c busybox1 -- /bin/sh
cat /etc/foo/passwd
exit
kubectl delete pod busybox

2. Create a PersistentVolume of 10Gi, called 'myvolume'. Make it have accessMode of 'ReadWriteOnce' and 'ReadWriteMany', storageClassName 'normal', mounted on hostPath '/etc/foo'. Save it on pv.yaml, add it to the cluster. Show the PersistentVolumes that exist on the cluster

Create PV:

apiVersion: v1
kind: PersistentVolume
metadata:
    name: myvolume
spec:
    capacity: 
        storage: '10Gi'
    accessModes: 
    - ReadWriteOnce
    - ReadWriteMany
    storageClassName: 'normal'
    hostPath:
        path: '/etc/foo'

kubectl create -f pv.yaml
kubectl get pv

3. Create a PersistentVolumeClaim for this storage class, called 'mypvc', a request of 4Gi and an accessMode of ReadWriteOnce, with the storageClassName of normal, and save it on pvc.yaml. Create it on the cluster. Show the PersistentVolumeClaims of the cluster. Show the PersistentVolumes of the cluster

Create PVC:

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
    name: mypvc
spec:
    resources: 
        requests:
            storage: '4Gi'
    accessModes: 
    - ReadWriteOnce
    storageClassName: 'normal'

kubectl create -f pvc.yaml
kubectl get pvc

4. Create a busybox pod with command 'sleep 3600', save it on pod.yaml. Mount the PersistentVolumeClaim to '/etc/foo'. Connect to the 'busybox' pod, and copy the '/etc/passwd' file to '/etc/foo/passwd'

kubectl run busybox --image=busybox --restart=Never --dry-run=client -o yaml -- /bin/sh -c 'sleep 3600' > pod.yaml

Add:

...
    volumeMounts:
      - mountPath: /etc/foo
        name: pvc-volume
...
  volumes:
  - name: pvc-volume
    persistentVolumeClaim:
      claimName: mypvc
...

kubectl exec -it busybox -- /bin/sh
cp /etc/passwd /etc/foo/passwd
exit

5. Create a second pod which is identical with the one you just created (you can easily do it by changing the 'name' property on pod.yaml). Connect to it and verify that '/etc/foo' contains the 'passwd' file. Delete pods to cleanup. Note: If you can't see the file from the second pod, can you figure out why? What would you do to fix that?

Change metadata.name to 'busybox2'
kubectl create -f pod.yaml
kubectl exec -it busybox2 -- /bin/sh
ls /etc/foo
exit
kubectl delete pod busybox
kubectl delete pod busybox2

6. Create a busybox pod with 'sleep 3600' as arguments. Copy '/etc/passwd' from the pod to your local folder

kubectl run busybox --image=busybox --restart=Never --dry-run=client -o yaml -- /bin/sh -c 'sleep 3600' > pod.yaml
kubectl create -f pod.yaml
kubectl cp busybox:/etc/passwd passwd