----------------------------
# Multi-Container Pods (10%)
----------------------------

1. Create a Pod with three busy box containers with commands “ls; sleep 3600;”, “echo Hello World; sleep 3600;” and “echo this is the third container; sleep 3600” respectively and check the status

kubectl run busybox --image=busybox --restart=Never --dry-run=client -o yaml -- /bin/sh -c “ls; sleep 3600;” > pod.yaml

Add:

...
spec:
    containers:
    - name: busybox1
      image: busybox
      args:
      - /bin/sh
      - -c
      - “ls; sleep 3600;”
    - name: busybox2
      image: busybox
      args:
      - /bin/sh
      - -c
      - “echo Hello World; sleep 3600;”
    - name: busybox3
      image: busybox
      args:
      - /bin/sh
      - -c
      - “echo this is the third container; sleep 3600”
...

kubectl create -f pod.yaml
kubectl get pods busybox

2. Check the logs of each container that you just created

kubectl logs busybox -c busybox1
kubectl logs busybox -c busybox2
kubectl logs busybox -c busybox3

3. Check the previous logs of the second container busybox2 if any

kubectl logs busybox -c busybox2 -p

4. Run command ls in the third container busybox3 of the above pod

kubectl exec -it busybox -c busybox3 -- ls

5. Show metrics of the above pod containers and puts them into the file.log and verify

kubectl top pod busybox > file.log

6. Create a Pod with main container busybox and which executes this “while true; do echo ‘Hi I am from Main container’ >> /var/log/index.html; sleep 5; done” and with sidecar container with nginx image which exposes on port 80. Use emptyDir Volume and mount this volume on path /var/log for busybox and on path /usr/share/nginx/html for nginx container. Verify both containers are running.

kubectl run busybox --image=busybox --restart=Never --dry-run=client -o yaml -- /bin/sh -c “while true; do echo ‘Hi I am from Main container’ >> /var/log/index.html; sleep 5; done” > pod.yaml

Add:

...
spec:
    containers:
    - name: nginx
      image: nginx
      ports:
        containerPort: 80
      volumeMounts:
      - name: empty-dir
        mountPath: /usr/share/nginx/html
    containers:
    - name: busybox
      ...
      volumeMounts:
      - name: empty-dir
        mountPath: /var/log
...
    volumes:
    - name: empty-dir
      emptyDir: {}
...

kubectl create -f pod.yaml
kubectl get pods

7. Exec into both containers and verify that main.txt exist and query the main.txt from sidecar container with curl localhost

kubectl exec -it busybox -c busybox -- /bin/sh
cd /var/log
cat main.txt
exit
kubectl exec -it busybox -c nginx -- /bin/sh
curl localhost
exit