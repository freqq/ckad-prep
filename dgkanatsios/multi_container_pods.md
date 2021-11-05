1. Create a Pod with two containers, both with image busybox and command "echo hello; sleep 3600". Connect to the second container and run 'ls'

kubectl run busybox --image=busybox --restart=Never --dry-run=client -o yaml -- /bin/sh -c "echo hello; sleep 3600" > pod.yaml
vim pod.yaml (duplicate pod containers)
kubectl exec busybox -it -c busybox2 -- ls

2. Create pod with nginx container exposed at port 80. Add a busybox init container which downloads a page using "wget -O /work-dir/index.html http://neverssl.com/online". Make a volume of type emptyDir and mount it in both containers. For the nginx container, mount it on "/usr/share/nginx/html" and for the initcontainer, mount it on "/work-dir". When done, get the IP of the created pod and create a busybox pod and run "wget -O- IP"

kubectl run nginx --image=nginx --restart=Never --port=80 --dry-run=client -o yaml > pod.yaml
vim pod.yaml

a. create emptyDir

volumes:

- name: vol
  emptyDir: {}

b. add initContainer

initContainers:

- args:
  - /bin/sh
  - -c
  - wget -O /work-dir/index.html http://neverssl.com/online
    image: busybox
    name: box
    volumeMounts:
  - name: vol
    mountPath: /work-dir

c. mount volume to nginx container

volumeMounts: - name: vol
mountPath: /usr/share/nginx/html

kubectl get pod -o wide
kubectl run busybox --image=busybox --restart=Never --rm -it -- /bin/sh -c "wget -O- 172.17.0.2"
