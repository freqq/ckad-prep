1. „Create a Pod YAML file with two containers that use the image alpine:3.12.0. Provide a command for both containers that keep them running forever.”

kubectl run alpine --image=alpine:3.12.0 --restart=Never -o yaml --dry-run=client -- /bin/sh -c "while true; do sleep 60; done;" > pod.yaml

Duplicate containers.

2. „Define a Volume of type emptyDir for the Pod. Container 1 should mount the Volume to path /etc/a, and container 2 should mount the Volume to path /etc/b.”

...
volumeMounts:
- name: empty-dir
    mountPath: /etc/b
...
volumes:
- name: empty-dir
emptyDir: {}
...

kubectl create -f pod.yaml

3. „Open an interactive shell for container 1 and create the directory data in the mount path. Navigate to the directory and create the file hello.txt with the contents “Hello World.” Exit out of the container.”

kubectl exec -it alpine -c container1 -- /bin/sh
cd /etc/a
mkdir data
cd data
echo "Hello World." > hello.txt
exit

4. „Open an interactive shell for container 2 and navigate to the directory /etc/b/data. Inspect the contents of file hello.txt. Exit out of the container.”

kubectl exec -it alpine -c container2 -- /bin/sh
cat /etc/b/data/hello.txt
exit

5. „Create a PersistentVolume named logs-pv that maps to the hostPath /var/logs. The access mode should be ReadWriteOnce and ReadOnlyMany. Provision a storage capacity of 5Gi. Ensure that the status of the PersistentVolume shows Available.”

apiVersion: v1
kind: PersistentVolume
metadata:
  name: logs-pv
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
    - ReadOnlyMany
  hostPath:
    path: /var/logs

kubectl create -f pv.yaml
kubectl get pv

6. „Create a PersistentVolumeClaim named logs-pvc. The access it uses is ReadWriteOnce. Request a capacity of 2Gi. Ensure that the status of the PersistentVolume shows Bound.”

apiVersion: v1
kind: PersistenVolumeClaim
metadata:
  name: logs-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi

kubectl create -f pvc.yaml
kubectl get pvc

7. „Mount the PersistentVolumeClaim in a Pod running the image nginx at the mount path /var/log/nginx.”

kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml

...
      volumeMounts:
        - name: pvc-volume
          mountPath: /var/log/nginx
...
  volumes:
    - name: pvc-volume
      persistentVolumeClaim:
        claimName: logs-pvc
...


8. „Open an interactive shell to the container and create a new file named my-nginx.log in /var/log/nginx. Exit out of the Pod.”

kubectl exec -it nginx -- /bin/sh
cd /var/log/nginx
touch my-nginx.log
exit

9. „Delete the Pod and re-create it with the same YAML manifest. Open an interactive shell to the Pod, navigate to the directory /var/log/nginx, and find the file you created before.”

kubectl delete pod nginx
kubectl create -f pod.yml
kubectl exec -it nginx -- /bin/sh
ls /var/log/nginx
exit
