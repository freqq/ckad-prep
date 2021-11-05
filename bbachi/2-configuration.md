----------------------------
# Configuration (18%)
----------------------------

1. Create a configmap called myconfigmap with literal value appname=myapp

kubectl create cm myconfigmap --from-literal=appname=myapp

2. Verify the configmap we just created has this data

kubectl get cm myconfigmap -o yaml

3. Delete the configmap myconfigmap we just created

kubectl delete cm myconfigmap

4. Create a file called config.txt with two values key1=value1 and key2=value2 and verify the file

Create the file 'config.txt'

5. Create a configmap named keyvalcfgmap and read data from the file config.txt and verify that configmap is created correctly

kubectl create cm keyvalcfgmap --from-file=config.txt
kubectl get cm keyvalcfgmap -o yaml

6. Create an nginx pod and load environment values from the above configmap keyvalcfgmap and exec into the pod and verify the environment variables and delete the pod

kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml

Add:

...
    envFrom:
        configMapRef:
            name: keyvalcfgmap
...

kubectl create -f pod.yaml
kubectl exec -it nginx -- env

7. Create an env file file.env with var1=val1 and create a configmap envcfgmap from this env file and verify the configmap

Create the file 'file.env'
kubectl create cm envcfgmap --from-env-file=file.env
kubectl get cm envcfgmap -o yaml

8. Create an nginx pod and load environment values from the above configmap envcfgmap and exec into the pod and verify the environment variables and delete the pod

kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml

Add:

...
    envFrom:
        configMapRef:
            name: envcfgmap
...

kubectl create -f pod.yaml
kubectl exec -it nginx -- env

9. Create a configmap called cfgvolume with values var1=val1, var2=val2 and create an nginx pod with volume nginx-volume which reads data from this configmap cfgvolume and put it on the path /etc/cfg

kubectl create cm cfgvolume --from-literal=var1=val1 --from-literal=var2=val2
kubectl run nginx --image=nginx --restart=Never -o yaml --dry-run=client > pod.yaml

Add:

...
    volumeMounts:
    - name: cm-volume
      mountPath: /etc/cfg
...
    volumes:
    - name: cm-volume
      configMap:
        name: cfgvolume
...

kubectl create -f pod.yaml

10. Create a pod called secbusybox with the image busybox which executes command sleep 3600 and makes sure any Containers in the Pod, all processes run with user ID 1000 and with group id 2000 and verify.

kubectl run secbusybox --image=busybox --restart=Never --dry-run=client -o yaml -- 'sleep 3600' > pod.yaml

Add:

...
spec:
    securityContext:
        runAsUser: 1000
        runAsGroup: 2000
...

kubectl create -f pod.yaml
kubectl exec -it secbusybox -- /bin/sh
id
exit

11. Create the same pod as above this time set the securityContext for the container as well and verify that the securityContext of container overrides the Pod level securityContext.

kubectl run secbusybox --image=busybox --restart=Never --dry-run=client -o yaml -- 'sleep 3600' > pod.yaml

Add:

...
spec:
    securityContext:
        runAsUser: 1000
        runAsGroup: 2000
    containers:
    - name: secbusybox
      securityContext:
        runAsUser: 1001
        runAsGroup: 2001
...

kubectl create -f pod.yaml
kubectl exec -it secbusybox -- /bin/sh
id # It overrides
exit

12. Create pod with an nginx image and configure the pod with capabilities NET_ADMIN and SYS_TIME verify the capabilities

kubectl run nginx --image=nginx --restart=Never --dry-run=client -o yaml > pod.yaml

Add:

...
spec:
    securityContext:
        capabilities:
            add: ["NET_ADMIN", "SYS_TIME"]
...

kubectl create -f pod.yaml
kubectl exec -it secbusybox -- /bin/sh
cat /proc/1/status

# Expected values

CapPrm: 00000000aa0435fb
CapEff: 00000000aa0435fb

exit

13. Create a Pod nginx and specify a memory request and a memory limit of 100Mi and 200Mi respectively.

kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml

Add:

...
spec:
    containers:
        resources:
            requests:
                memory: 100Mi
            limits:
                memory: 200Mi
...

kubectl create -f pod.yaml

14. Create a Pod nginx and specify a CPU request and a CPU limit of 0.5 and 1 respectively.

kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml

Add:

...
spec:
    containers:
        resources:
            requests:
                cpu: 0.5
            limits:
                cpu: 1
...

kubectl create -f pod.yaml

15. Create a Pod nginx and specify both CPU, memory requests and limits together and verify.

kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml

Add:

...
spec:
    containers:
        resources:
            requests:
                cpu: 0.5
                memory: 100Mi
            limits:
                cpu: 1
                memory: 200Mi
...

kubectl create -f pod.yaml
kubectl describe pod nginx

16. Create a Pod nginx and specify a memory request and a memory limit of 100Gi and 200Gi respectively which is too big for the nodes and verify pod fails to start because of insufficient memory

Add:

...
spec:
    containers:
        resources:
            requests:
                memory: 100Gi
            limits:
                memory: 200Gi
...

kubectl create -f pod.yaml
kubectl describe pod nginx
kubectl get pods nginx

17. Create a secret mysecret with values user=myuser and password=mypassword

kubectl create secret generic mysecret --from-literal=user=myuser --frol-literal=password=mypassword

18. List the secrets in all namespaces

kubectl get secrets -A

19. Output the yaml of the secret created above

kubectl get secret mysecret -o yaml

20. Create an nginx pod which reads username as the environment variable

kubectl run nginx --image=nginx --dry-run=client --restart=Never -o yaml > pod.yaml

Add:

...
    env:
    - name: username
      valueFrom:
        secretKeyRef:
            name: mysecret
            value: user
...

kubectl create -f pod.yaml

21. Create an nginx pod which loads the secret as environment variables

kubectl run nginx --image=nginx --dry-run=client --restart=Never -o yaml > pod.yaml

Add:

...
    envFrom:
        secretRef:
            name: mysecret
...

kubectl create -f pod.yaml

22. List all the service accounts in the default namespace

kubectl get sa

23. List all the service accounts in all namespaces

kubectl get sa -A

24. Create a service account called admin

kubectl create sa admin

25. Output the YAML file for the service account we just created

kubectl get sa admin -o yaml

26. Create a busybox pod which executes this command sleep 3600 with the service account admin and verify

kubectl run busybox --image=busybox --dry-run=client --restart=Never -o yaml -- /bin/sh -c 'sleep 3600' > pod.yaml

Add:

...
spec:
    serviceAccount: admin
...

kubectl create -f pod.yaml

27. List all the configmaps in the cluster

kubectl get cm