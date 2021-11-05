1. Create a new pod with the nginx image.

kubectl run nginx --image=nginx --restart=Never

2. Create a new pod with the name redis and with the image redis:1.99.

kubectl run redis --image=redis:1.99 --restart=Never

3. Identify the problem with the pod.

kubectl describe pod redis

4. Rectify the problem with the pod and wait until the pod is ready and healthy.

kubectl edit pod redis

...
    image: redis:latest
...

5. Create a new replicaset rs-1 using image nginx with labels tier:frontend and selectors as tier:frontend. Create 3 replicas for the same.

kubectl create rs replicaset-pod --image=nginx -l tier=frontend --replicas
kubectl annotate rs replicaset-pod tier=frontend

6. Scale the above replicaset to 5 replicas.

kubectl scale rs replicaset --replicas=5

7. Update the image of the above replicaset to use nginx:alpine image.

kubectl set image rs/replicaset-pod nginx=nginx:alpine

8. Scale down the replicaset to 2 replicas.

kubectl scale rs replicaset --replicas=2

9. Create a new deployment deploy-1 using image busybox with 3 replicas and command sleep 3600.

kubectl create deployment deploy-1 --image=busybox --replicas=3 -- /bin/sh -c 'sleep 3600'

10. Update the image of the above deployment with image ubuntu.

kubectl set image deployment/deploy-1 busybox=ubuntu

11. Create a namespace finance.

kubectl create ns finance

12. Create a pod redis01 using image redis in the finance namespace.

kubectl run redis01 --image=redis -n finance

13. Deploy a redis02 pod using the redis:alpine image with the labels set to tier=db.

kubectl run redis02 --image=redis:alpine -l tier=db

14. Create a service redis-service to expose the redis02 application within the cluster on port 6379.

kubectl expose pod redis-service --port=6379 --target-port=80

15. Create a new pod called custom-nginx using the nginx image and expose it on container port 8080.

kubectl run custom-nginx --image=nginx --port=8080

16. Create a pod called httpd using the image httpd:alpine in the default namespace. Next, create a service of type ClusterIP by the same name (httpd). The target port for the service should be 80.

kubectl run httpd --image=httpd:alpine --port=80
kubectl create service ClusterIP httpd --tcp=80

17. Create a pod with the ubuntu image to run a container to sleep for 5000 seconds.

kubectl run ubuntu --image=ubuntu -- /bin/sh -c 'sleep 5000'

18. Create a new ConfigMap for the webapp-color pod. Set key-value pairs as COLOR=blue.

kubectl create cm webapp-color --from-literal=COLOR=blue

19. Create a deployment nginx01 using nginx image, which has environment variable CREATED_BY with value octocat.

kubectl create deployment nginx01 --image=nginx --dry-run=client -o yaml > deploy.yaml

Add:

...
    env:
    - name: CREATED_BY
      value: octocat
...

kubectl create -f depoy.yaml

20. Verify the environment variable in the containers of the above deployment.

kubectl exec -it nginx01 -- env

21. Create a new config map db-config with key-value pairs as:
a] db-host = "localhost.example.com"
b] db-port = 3306
c] db-name = "kubernetes-objects"

kubectl create cm db-config --from-literal=db-host="localhost.example.com" --from-literal=db-port=3306 --from-literal=db-name="kubernetes-objects"

22. Mount the config map db-config in the deployment ubuntu01 with image ubuntu which has 2 replicas.

kubectl create deployment ubuntu01 --image=ubuntu --replicaas=2 --dry-run=client -o yaml > deployment.yaml

Add:

...
    envFrom:
     configMapRef:
        name: db-config
...

kubectl create -f deployment.yaml

23. Create a deployment named multi-con-01, which has two containers, one container uses nginx image and has port 80 exposed. Mount the config map db-config as environment variable. Create the second container in the same deployment with image ubuntu, keep the container alive using the command sleep 5000 and mount the config map in db-config with different keys where the new keys are mapped as:

db-host --> DB_URL
db-port --> DB_PORT
db-name --> DB_NAME

kubectl create deployment multi-con-01 --image=nginx --port=80 --dry-run=client -o yaml > deployment.yaml

Add:

...
    envFrom:
        configMapRef:
            name: db-config
- name: ubuntu
  image: ubuntu
  args:
  - /bin/sh
  - -c
  - 'sleep 5000'
  env:
  - name: DB_URL
    valueFrom:
        configMapKeyRef:
            name: db-config
            key: db-host
  - name: DB_PORT
    valueFrom:
        configMapKeyRef:
            name: db-config
            key: db-port
  - name: DB_NAME
    valueFrom:
        configMapKeyRef:
            name: db-config
            key: db-name
...

kubectl create -f deployment.yaml

24. Create an opaque secret named db-creds with following key value pairs:

db_user=root
db_password=pass404
db_role=admin

kubectl create secret generic db-creds --from-literal=db_user=root --from-literal=db_password=pass404 --from-literal=db_role=admin

25. Mount the secret db-creds in the deployment multi-con-01 [created in (23)] as environment variables in the ubuntu container with the same keys as mentioned in the secret.

kubectl create deployment multi-con-01 --image=nginx --port=80 --dry-run=client -o yaml > deployment.yaml

Add:

...
    envFrom:
     secretMapRef:
        name: db-creds
...

kubectl create -f deployment.yaml

26. Create a pod ubuntu02 with image ubuntu and run the command sleep 3400 as user 1001.

kubectl run ubuntu02 --image=ubuntu --dry-run=client -o yaml -- 'sleep 3400' > pod.yaml

Add:

...
spec:
    containers:
        securityContext:
            runAsUser: 1001
...

kubectl create -f pod.yaml

27. Create a pod ubuntu03 with image ubuntu and add capabilities for SYS_TIME to each container. Run the command date -s '01JAN 2020 10:00:00 to verify if it is successful.

kubectl run ubuntu03 --image=ubuntu --dry-run=client -o yaml > pod.yaml

Add:

...
spec:
    containers:
        securityContext:
            capabilities: 
                add: ["SYS_TIME"]
...

kubectl create -f pod.yaml
kubectl exec -it ubuntu03 -- "date -s '01JAN 2020 10:00:00"

28. Taint one of the worker nodes in your cluster with the following property, mode=maintainance:NoSchedule. Create a pod nginx02 with image nginx:alpine and ensure the pod runs on the tainted node as above.
Hint: You might want to use tolerations and nodeselectors both in case you have multiple worker nodes..

kubectl get nodes
kubectl annotate node minikube mode=maintainance:NoSchedule
kubectl run nginx02 --image=nginx:alpine --dry-run=client --restart=Never -o yaml > pod.yaml

Add:

...
spec:
    nodeSelector:
        mode: maintainance:NoSchedule 
...

kubectl create -f pod.yaml

29. Create a pod box0 with image busybox and command sleep 30; touch /tmp/debug; sleep 30; rm /tmp/debug. The pod should not be ready until the file /tmp/debug is available in the pod. If the file doesn't exist, the pod must restart.

kubectl run box0 --image=busybox -- /bin/sh -c "sleep 30; touch /tmp/debug; sleep 30; rm /tmp/debug"
kubectl logs box0 -f

30. Create a pod nginx03 with image nginx, make sure the server is up and running every 10 seconds. The nginx server ideally takes 30 seconds to warm up so ensure that there are no false negatives when reporting the health of the pod.
Note: Nginx runs on port 80..

kubectl run nginx03 --image=nginx --dry-run=client -o yaml > pod.yaml

Add:

...
    livenessProbe:
        httpGet:
            path: /
            port: 80
        initialDelaySeconds: 30
        periodSeconds: 10
...

kubectl create -f pod.yaml

31. This is a multistage problem:
a] Create a deployment deploy01 with image nginx with port 80 exposed.
b] Update the deployment with image nginx:dev and make sure you record the execution of this action.
c] If the deployment in step b is unsuccessful, rollback the deployment to the previous state by setting any fields explicitly.
d] If the nginx deployment is in a healthy state, scale the deployment to run 3 replicas, also record this execution for audit-keeping.
e] Display the history of deployment deploy-01 and store it in file /opt/answers/31.txt.

a) kubectl create deployment deploy01 --image=nginx --port=80
b) kubectl set image deployment deploy01 nginx=nginx:dev
c) kubectl rollout undo deployment deploy01
d) kubectl scale deployment deploy01 --replicas=3
e) kubectl rollout history deployment deploy01 > /opt/answers/31.txt

32. Create a job that will roll a dice using image kodekloud/throw-dice. The image generates a random number between 1 and 6 and exits. It is a success only if it is a 6 else it is a failure. Check how many times it takes the job to trigger to get a successful outcome.

kubectl create job dice-job --image=kodekloud/throw-dice --restart=Always

33. Create a job with the above image to generate 8 successful outcomes. Also, ensure that the job will attempt no more than 15 attempts and see if the job is successful. You can add an element of 3 parallel executions if required.

kubectl create job dice-job --image=kodekloud/throw-dice --restart=Always --dry-run=client -o yaml > job.yaml

Add:

...
spec:
    completions: 8
    backoffLimit: 15
...

kubectl create -f job.yaml

34. Create a scheduled job such that it runs every minute. Use the image kodekloud/throw-dice to check the outcome.

kubectl crete cj cj-example --schedule="*/1 * * * *" --image=kodekloud/throw-dice

35. Create a Persistent Volume deploy-history which makes the storage available on each worker nodes at /tmp/deployment. Make sure it has the default provisioner of your cluster assigned. Label the Persistent Volume with audit: true, tier: middleware. The persistent volume must have a capacity of 2 GB.

apiVersion: v1
kind: PersistentVolume
metadata:
    name: deploy-history
    labels:
        audit: true
        tier: middleware
spec:
    capacity:
        storage: 2Gi
    storageClassName: normal
    hostPath:
        path: /tmp/deployment
    
36. Create a Persistent Volume persistent-data which uses a storage class name persistent and is of type hostPath which stores the data on /tmp/persistent on worker nodes. The persistent volume must have a capacity of 2 GB.

apiVersion: v1
kind: PersistentVolume
metadata:
    name: persistent-data
spec:
    capacity:
        storage: 2Gi
    storageClassName: persistent
    hostPath:
        path: /tmp/persistent

37. Create a Persistent Volume my-personal-data which uses a storage class name persistent and is of type hostPath which stores the data on /tmp/pii on worker nodes. The persistent volume must have a capacity of 2 GB. Since this will store personal data, make sure you can identify the persistent volume using the labels security: high, type: pii, audit: true.

apiVersion: v1
kind: PersistentVolume
metadata:
    name: my-personal-data
    labels:
        security: high
        type: pii
        audit: true
spec:
    capacity:
        storage: 2Gi
    storageClassName: persistent
    hostPath:
        path: /tmp/pii

38. Create a Persistent Volume Claim to bind to persistent volume which uses storage class persistent and requests 2 GB capacity.

apiVersion: v1
kind: PersistentVolume
metadata:
    name: my-pvc
spec:
    storageClassName: persistent
    resources:
        requests:
            storage: 2Gi

39. Create a Persistent Volume Claim to bind to a PV which ensures that the personal and private data is secure. Ensure that this PVC will bind to a PV of type pii.

apiVersion: v1
kind: PersistentVolume
metadata:
    name: my-secure-pvc
spec:
    storageClassName: persistent
    selector: 
        matchLabels:
            type: pii
    resources:
        requests:
            storage: 2Gi

40. Create a PVC such that it will bind to a PV that is qualified for middleware applications.

apiVersion: v1
kind: PersistentVolume
metadata:
    name: my-middleware-pvc
spec:
    storageClassName: persistent
    selector: 
        matchLabels:
            tier: middleware
    resources:
        requests:
            storage: 2Gi

41. Create a deployment deploy02 with image busybox, mount one of the PVC from above such that it will store logs in the /tmp/deployment directory. The containers of the deployment must:
a] Create a file in /tmp/deployment with the same name as the pod name with .txt extension.
b] Run the command i=0; while true; do; echo "$i $(date)" >> /tmp/deployment/<podname>.txt && sleep 60 ; done to the log file.
Hint: You can use init-containers, environment variables to achieve this.
c] Ensure there are 5 replicas of the deployment.
d] Ensure you can see the log files created and populated with data in /tmp/deployment directory on the scheduled worker nodes.

???


42. Create two deployments i.e. nginx04 and redis04 using images nginx and redis respectively with 3 replicas. Your goal is to ensure that:
a] each worker node must not have 2 or more nginx04 pods.
b] each worker node that has a nginx04 pod must have a redis04 pod scheduled on the same node.
Hint: Use pod affinity and anti-affinity to ensure the above scneario.

kubectl create deployment nginx04 --image=nginx --replicas=3
kubectl create deployment redis04 --image-redis --replicas=3

??

43. Create 3 pods nginx05, redis05, and httpd05 with images nginx, redis, and httpd respectively. Attach a sidecar debug container of image busybox to each of them. The security team has enforced the application team to limit the communication between unnecessary pods. Ensure that only the following communication is allowed between pods:

a] Allow:
    1. nginx05 <--> redis05
    2. nginx05 <--> httpd05
b] Deny:
    1. httpd05 <-/-> redis05
    2. redis05 <-/-> rest of the world
    3. nginx05 <-/-> the rest of the world

Identify a possible information flow in this system. Write the information flow in the format End-User::Pod1::Pod2::Pod3.

kubectl run nginx05 --image-nginx --restart=Never --dry-run=client -o yaml > nginx.yaml
kubectl run redis05 --image=redis --restart=Never --dry-run=client -o yaml > redis.yaml
kubectl run httpd05 --image=httpd --restart=Never --dry-run=client -o yaml > httpd.yaml