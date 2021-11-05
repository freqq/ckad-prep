----------------------------
## Core Concepts (13%)
----------------------------

1. List all the namespaces in the cluster

kubectl get ns

2. List all the pods in all namespaces

kubectl get pods -A

3. List all the pods in the particular namespace

kubectl get pods -n <namespace>

4. List all the services in the particular namespace

kubectl get svc -n <namespace>

5. List all the pods showing name and namespace with a json path expression

kubectl get pods -o=jsonpath="{.items[*]['metadata.name', 'metadata.namespace']}"

6. Create an nginx pod in a default namespace and verify the pod running

kubectl run nginx --restart=Never --image=nginx
kubectl get pod nginx

7. Create the same nginx pod with a yaml file

apiVersion: v1
kind: Pod
metadata:
    name: nginx
spec:
    containers:
    - name: nginx
      image: nginx
      resources: {}
    dnsPolicy: ClusterFirst

OR

kubectl run nginx --restart=Never --image=nginx --dry-run=client -o yaml > pod.yaml
kubectl create -f pod.yaml

8. Output the yaml file of the pod you just created

kubectl get pod nginx -o yaml

9. Output the yaml file of the pod you just created without the cluster-specific information

kubectl get po nginx -o yaml --export

10. Get the complete details of the pod you just created

kubectl describe pod nginx

11. Delete the pod you just created

kubectl delete pod nginx

12. Delete the pod you just created without any delay (force delete)

kubectl delete pod --force --grace-period=0

13. Create the nginx pod with version 1.17.4 and expose it on port 80

kubectl run nginx --image=nginx:1.17.4 --port=80 --restart=Never

14. Change the Image version to 1.15-alpine for the pod you just created and verify the image version is updated

kubectl set image pod/nginx nginx=nginx:1.15-alpine
kubectl describe pod nginx

15. Change the Image version back to 1.17.1 for the pod you just updated and observe the changes

kubectl set image pod/nginx nginx=nginx:1.17.1
kubectl describe po nginx
kubectl get pod nginx -w

16. Check the Image version without the describe command

kubectl get pod nginx -o yaml | grep -i image:

17. Create the nginx pod and execute the simple shell on the pod

kubectl run nginx --image=nginx --restart=Never -- /bin/sh -c 'ls'

18. Get the IP Address of the pod you just created

kubectl get pod nginx -o wide

19. Create a busybox pod and run command ls while creating it and check the logs

kubectl run busybox --image=busybox --restart=Never -- /bin/sh -c 'ls'
kubectl logs busybox

20. If pod crashed check the previous logs of the pod

kubectl logs busybox -p

21. Create a busybox pod with command sleep 3600

kubectl run busybox --image=busybox --restart=Never -- /bin/sh -c 'sleep 3600'

22. Check the connection of the nginx pod from the busybox pod

kubectl get pod nginx -o wide
kubectl exec -it busybox -- wget -O- <nginx-ip>

23. Create a busybox pod and echo message ‘How are you’ and delete it manually

kubectl run busybox --image=busybox --restart=Never -- /bin/sh -c 'echo "How are you"'
kubectl delete pod busybox

24. Create a busybox pod and echo message ‘How are you’ and have it deleted immediately

kubectl run busybox --image=busybox --restart=Never --rm -- /bin/sh -c 'echo "How are you"'

25. Create an nginx pod and list the pod with different levels of verbosity

kubectl run nginx --image=nginx --restart=Never --port=80
kubectl get po nginx --v=7
kubectl get po nginx --v=8
kubectl get po nginx --v=9

26. List the nginx pod with custom columns POD_NAME and POD_STATUS

kubectl get pod nginx -o=custom-columns="POD_NAME:.metadata.name, POD_STATUS:.status.containerStatuses[].state"

27. List all the pods sorted by name

kubectl get pods --sort-by=.metadata.name

28. List all the pods sorted by created timestamp

kubectl get pods --sort-by=.metadata.creationTimestamp

----------------------------
## Multi-Container Pods (10%)
----------------------------

29. Create a Pod with three busy box containers with commands “ls; sleep 3600;”, “echo Hello World; sleep 3600;” and “echo this is the third container; sleep 3600” respectively and check the status

kubectl run busybox --image=busybox --restart=Never -o yaml --dry-run=client -- /bin/sh -c 'ls; sleep 3600;' > pod.yaml

Modify:

...
  containers:
  - args:
    - /bin/sh
    - -c
    - ls; sleep 3600;
    image: busybox
    name: busybox
    resources: {}
  - args:
    - /bin/sh
    - -c
    - echo Hello World; sleep 3600;
    image: busybox
    name: busybox2
    resources: {}
  - args:
    - /bin/sh
    - -c
    - echo this is the third container; sleep 3600;
    image: busybox
    name: busybox3
    resources: {}
...

kubectl create -f pod.yaml
kubectl get pod busybox

30. Check the logs of each container that you just created

kubectl logs busybox -c busybox1
kubectl logs busybox -c busybox2
kubectl logs busybox -c busybox3

31. Check the previous logs of the second container busybox2 if any

kubectl logs busybox -c busybox2 -p

32. Run command ls in the third container busybox3 of the above pod

kubectl exec -it busybox -c busybox3 -- /bin/sh -c 'ls'

33. Show metrics of the above pod containers and puts them into the file.log and verify

kubectl top pod busybox --containers > file.log
cat file.log

34. Create a Pod with main container busybox and which executes this “while true; do echo ‘Hi I am from Main container’ >> /var/log/index.html; sleep 5; done” and with sidecar container with nginx image which exposes on port 80. Use emptyDir Volume and mount this volume on path /var/log for busybox and on path /usr/share/nginx/html for nginx container. Verify both containers are running.

kubectl run busybyx --image=busybox --restart=Never --dry-run=client -o yaml -- /bin/sh -c "while true; do echo ‘Hi I am from Main container’ >> /var/log/index.html; sleep 5; done" > pod.yaml

Add:

...
    volumeMounts:
    - name: empty-dir
      mountPath: /var/log
...
  - image: nginx
    name: busybox2
    ports:
      - containerPort: 80
    volumeMounts:
    - name: empty-dir
      mountPath: /usr/share/nginx/html
    resources: {}
...
  volumes:
  - name: empty-dir
    emptyDir: {}
...

kubectl create -f pod.yaml

35. Exec into both containers and verify that main.txt exist and query the main.txt from sidecar container with curl localhost

kubectl exec -it busybox -c busybox -- /bin/sh
cat /var/log/main.txt
exit

kubectl exec -it busybox -c busybox2 -- /bin/sh
cat /usr/share/nginx/html/index.html

kubectl exec -it busybox -c busybox2 -- /bin/sh
apt-get update
apt-get install -y curl
curl localhost

----------------------------
## Pod Design (20%)
----------------------------

36. Get the pods with label information

kubectl get pods --show-labels

37. Create 5 nginx pods in which two of them is labeled env=prod and three of them is labeled env=dev

kubectl run nginx1 --image=nginx --restart=Never -l=env=prod
kubectl run nginx2 --image=nginx --restart=Never -l=env=prod
kubectl run nginx3 --image=nginx --restart=Never -l=env=dev
kubectl run nginx4 --image=nginx --restart=Never -l=env=dev
kubectl run nginx5 --image=nginx --restart=Never -l=env=dev

38. Verify all the pods are created with correct labels

kubectl get pods --show-labels

39. Get the pods with label env=dev

kubectl get pods --selector=env=dev

40. Get the pods with label env=dev and also output the labels

kubectl get pods --selector=env=dev --show-labels

41. Get the pods with label env=prod

kubectl get pods --selector=env=prod

42. Get the pods with label env=prod and also output the labels

kubectl get pods --selector=env=dev --show-labels

43. Get the pods with label env

kubectl get pods --selector=env --show-labels

44. Get the pods with labels env=dev and env=prod

kubectl get pods -l 'env in(dev, prod)'

45. Get the pods with labels env=dev and env=prod and output the labels as well

kubectl get pods -l 'env in(dev, prod)' --show-labels

46. Change the label for one of the pod to env=uat and list all the pods to verify

kubectl label pod nginx1 env=uat --overwrite
kubectl get pods nginx1 --show-labels

47. Remove the labels for the pods that we created now and verify all the labels are removed

kubectl label pod nginx1 env-
kubectl label pod nginx2 env-
kubectl label pod nginx3 env-
kubectl label pod nginx4 env-
kubectl label pod nginx5 env-

kubectl get pods --show-labels

48. Let’s add the label app=nginx for all the pods and verify

kubectl label pod nginx1 app=nginx
kubectl label pod nginx2 app=nginx
kubectl label pod nginx3 app=nginx
kubectl label pod nginx4 app=nginx
kubectl label pod nginx5 app=nginx

kubectl get pods --show-labels

49. Get all the nodes with labels (if using minikube you would get only master node)

kubectl get nodes --show-labels

50. Label the node (minikube if you are using) nodeName=nginxnode

kubectl label node minikube nodeName=nginxnode

51. Create a Pod that will be deployed on this node with the label nodeName=nginxnode

kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml

Add:

...
  nodeSelector:
    nodeName: "nginxnode"
...

kubectl create -f pod.yaml

52. Verify the pod that it is scheduled with the node selector

kubectl describe pod nginx | grep -i node-selectors

53. Verify the pod nginx that we just created has this label

kubectl get pod nginx --show-labels

54. Annotate the pods with name=webapp

kubectl annotate pod nginx name=webapp

55. Verify the pods that have been annotated correctly

kubectl describe pod nginx | grep -i annotations

56. Remove the annotations on the pods and verify

kubectl annotate pod nginx name-
kubectl describe pod nginx | grep -i annotations

57. Remove all the pods that we created so far

kubectl delete po --all

58. Create a deployment called webapp with image nginx with 5 replicas

kubectl create deployment webapp --image=nginx --replicas=5

59. Get the deployment you just created with labels

kubectl get deployment --show-labels

60. Output the yaml file of the deployment you just created

kubectl get deployment webapp -o yaml

61. Get the pods of this deployment

kubectl get deploy --show-labels
kubectl get pods -l app=webapp

62. Scale the deployment from 5 replicas to 20 replicas and verify

kubectl scale deployment webapp --replicas=20
kubectl get pods -l app=webapp | wc -l

63. Get the deployment rollout status

kubectl rollout status deployment webapp

64. Get the replicaset that created with this deployment

kubectl get rs -l app=webapp

65. Get the yaml of the replicaset and pods of this deployment

kubectl get rs -l app=webapp -o yaml
kubectl get pods -l app=webapp -o yaml

66. Delete the deployment you just created and watch all the pods are also being deleted

kubectl delete deployment webapp
kubectl get pods -l app=webapp -w

67. Create a deployment of webapp with image nginx:1.17.1 with container port 80 and verify the image version

kubectl create deployment webapp --image=nginx:1.17.1 --port=80
kubectl describe deployment | grep -i image

68. Update the deployment with the image version 1.17.4 and verify

kubectl set image deployment webapp nginx=nginx:1.17.4
kubectl describe deployment | grep -i image

69. Check the rollout history and make sure everything is ok after the update

kubectl rollout history deployment webapp
kubectl get deploy webapp --show-labels
kubectl get rs -l app=webapp
kubectl get po -l app=webapp

70. Undo the deployment to the previous version 1.17.1 and verify Image has the previous version

kubectl rollout undo deployment webapp
kubectl describe deployment | grep -i image

71. Update the deployment with the image version 1.16.1 and verify the image and also check the rollout history

kubectl set image deployment webapp nginx=nginx:1.16.1
kubectl describe deployment | grep -i image
kubectl rollout history deployment webapp

72. Update the deployment to the Image 1.17.1 and verify everything is ok

kubectl rollout undo deployment webapp
kubectl rollout history deployment webapp
kubectl get deploy webapp --show-labels
kubectl get rs -l app=webapp
kubectl get po -l app=webapp

73. Update the deployment with the wrong image version 1.100 and verify something is wrong with the deployment

kubectl set image deployment webapp nginx=nginx:1.100
kubectl rollout status deployment webapp
kubectl get pods 
kubectl describe deployment webapp

74. Undo the deployment with the previous version and verify everything is Ok

kubectl rollout undo deployment webapp
kubectl rollout status deployment webapp
kubectl get pods 
kubectl describe deployment webapp

75. Check the history of the specific revision of that deployment

kubectl rollout history deployment webapp --revision=2

76. Pause the rollout of the deployment

kubectl rollout pause deployment webapp

77. Update the deployment with the image version latest and check the history and verify nothing is going on

kubectl set image deployment webapp nginx=nginx:latest
kubectl rollout history deployment webapp

78. Resume the rollout of the deployment

kubectl rollout resume deployment webapp

79. Check the rollout history and verify it has the new version

kubectl rollout history deployment webapp

80. Apply the autoscaling to this deployment with minimum 10 and maximum 20 replicas and target CPU of 85% and verify hpa is created and replicas are increased to 10 from 1

kubectl autoscale deployment webapp --min=10 --max=20 --cpu-percent=85
kubectl get hpa
kubectl get deployment webapp

81. Clean the cluster by deleting deployment and hpa you just created

kubectl delete deployment webapp
kubectl delete hpa webapp

82. Create a Job with an image node which prints node version and also verifies there is a pod created for this job

kubectl create job node --image=node -- /bin/sh -c 'node --version'
kubectl get job -w
kubectl get pods

83. Get the logs of the job just created

kubectl logs <job-pod-name>

84. Output the yaml file for the Job with the image busybox which echos “Hello I am from job”

kubectl create job busybox --image=busybox -- /bin/sh -c 'echo "Hello I am from job"'
kubectl get job busybox -o yaml

85. Copy the above YAML file to hello-job.yaml file and create the job

---

86. Verify the job and the associated pod is created and check the logs as well

kubectl get job busybox
kubectl get pods 
kubectl logs busybox-x4tfz

87. Delete the job we just created

kubectl delete job busybox

88. Create the same job and make it run 10 times one after one

kubectl create job busybox --image=busybox -- /bin/sh -c 'echo "Hello I am from job"' > job.yaml

...
spec:
    completions: 10
...

kubeclt create -f job.yaml

89. Watch the job that runs 10 times one by one and verify 10 pods are created and delete those after it’s completed

kubectl get job -w
kubectl get pod
kubectl delete job busybox

90. Create the same job and make it run 10 times parallel

kubectl create job busybox --image=busybox -- /bin/sh -c 'echo "Hello I am from job"' > job.yaml

...
spec:
    parallelism: 10
...

kubeclt create -f job.yaml

91. Watch the job that runs 10 times parallelly and verify 10 pods are created and delete those after it’s completed

kubectl get job -w
kubectl get pod
kubectl delete job busybox

92. Create a Cronjob with busybox image that prints date and hello from kubernetes cluster message for every minute

kubectl create cj busybox --image=busybox --schedule="*/1 * * * *" -- /bin/sh -c 'date; echo "hello from kubernetes'

93. Output the YAML file of the above cronjob

kubectl get cj busybox -o yaml

94. Verify that CronJob creating a separate job and pods for every minute to run and verify the logs of the pod

kubectl get job -w 
kubeclt get pod -w
kubectl logs <job-pod-name>

95. Delete the CronJob and verify all the associated jobs and pods are also deleted.

kubectl delete cj busybox
kubectl get job
kubectl get pod 

----------------------------
## State Persistence (8%)
----------------------------

96. List Persistent Volumes in the cluster

kubectl get pv

97. Create a hostPath PersistentVolume named task-pv-volume with storage 10Gi, access modes ReadWriteOnce, storageClassName manual, and volume at /mnt/data and verify

apiVersion: v1
kind: PersistentVolume
metadata:
    name: task-pv-volume
spec:
    storageClassName: manual
    hostPath:
        path: /mnt/data
    accessModes:
    - ReadWriteOnce
    capacity:
        storagea: 10Gi

98. Create a PersistentVolumeClaim of at least 3Gi storage and access mode ReadWriteOnce and verify status is Bound

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
    name: task-pvc-volume
spec:
    storageClassName: manual
    accessModes:
    - ReadWriteOnce
    resources:
        requests:
            storage: 3Gi

99. Delete persistent volume and PersistentVolumeClaim we just created

kubectl delete pv task-pv-volume
kubectl delete pvc task-pvc-volume

100. Create a Pod with an image Redis and configure a volume that lasts for the lifetime of the Pod

kubectl run redis --image=redis -o yaml --dry-run=client > pod.yaml

Add:

...
    volumeMounts:
    - name: empty-dir
      mountPath: /data-redis
...
    volumes:
    - name: empty-dir
      emptyDir: {}
...

kubectl create -f pod.yaml

101. Exec into the above pod and create a file named file.txt with the text ‘This is called the file’ in the path /data/redis and open another tab and exec again with the same pod and verifies file exist in the same path.

1st tab:

kubectl exec -it redis -- /bin/sh
echo "This i called the first file" > /data/redis/file.txt
exit

2nd tab:
kubectl exec -it redis -- /bin/sh
cat /data/redis/file.txt
exit

102. Delete the above pod and create again from the same yaml file and verifies there is no file.txt in the path /data/redis

kubectl delete pod redis
kubectl create -f pod.yaml
kubectl exec -it redis -- /bin/sh
cat /data/redis/file.txt
exit

103. Create PersistentVolume named task-pv-volume with storage 10Gi, access modes ReadWriteOnce, storageClassName manual, and volume at /mnt/data and Create a PersistentVolumeClaim of at least 3Gi storage and access mode ReadWriteOnce and verify status is Bound

apiVersion: v1
kind: PersistentVolume
metadata:
    name: task-pv-volume
spec:
    accessModes:
    - ReadWriteOnce
    storageClassName: manual
    capacity:
        storage: 10Gi
    hostPath:
        path: /mnt/data

---

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
    name: task-pvc-volume
spec:
    storageClassName: manual
    accessModes:
    - ReadeWriteOnce
    resorces:
        requests:
            storage: 3Gi

kubectl create -f pv.yaml
kubectl create -f pvc.yaml

kubectl get pv
kubectl get pvc

104. Create an nginx pod with containerPort 80 and with a PersistentVolumeClaim task-pv-claim and has a mouth path "/usr/share/nginx/html"

kubectl run nginx --image=nginx --port=80 --dry-run=client -o yaml > pod.yaml

Add:

...
    volumeMounts:
    - name: task-pvc-volume
      mountPath: /usr/share/nginx/html
...
    volumes:
    - name: task-pvc-volume
      persistentVolumeClaim:
        claimName: task-pvc-volume
...

----------------------------
## Configuration (18%)
----------------------------

105. List all the configmaps in the cluster

kubectl get cm

106. Create a configmap called myconfigmap with literal value appname=myapp

kubectl create cm configmap --from-literal=app-name=myapp

107. Verify the configmap we just created has this data

kubectl get cm configmap -o yaml

108. Delete the configmap myconfigmap we just created

kubectl delete cm configmap

109. Create a file called config.txt with two values key1=value1 and key2=value2 and verify the file

touch config.txt
vi config.txt

Add:

key1=value1
key2=value2

110. Create a configmap named keyvalcfgmap and read data from the file config.txt and verify that configmap is created correctly

kubectl create cm keyvalcfgmap --from-file=config.txt
kubectl get cm keyvalcfgmap -o yaml

111. Create an nginx pod and load environment values from the above configmap keyvalcfgmap and exec into the pod and verify the environment variables and delete the pod

kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml

Add:

...
    envFrom:
    - configMapRef:
        name: keyvalcfgmap
...

kubectl create -f pod.yaml
kubectl exec -it nginx -- env
kubectl delete pod nginx

112. Create an env file file.env with var1=val1 and create a configmap envcfgmap from this env file and verify the configmap

touch file.env
vi file.env

Add:

var1=val1

kubectl create envcfgmap --from-env-file=file.env
kubectl get cm envcfgmap -o yaml

113. Create an nginx pod and load environment values from the above configmap envcfgmap and exec into the pod and verify the environment variables and delete the pod

kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml

Add:

...
    env:
    - name: ENVIRONMENT
      valueFrom:
        configMapKeyRef:
          name: envcfgmap
          key: var1
...

kubectl create -f pod.yaml
kubectl exec -it nginx -- env
kubectl delete pod nginx

114. Create a configmap called cfgvolume with values var1=val1, var2=val2 and create an nginx pod with volume nginx-volume which reads data from this configmap cfgvolume and put it on the path /etc/cfg

kubectl create cm cfgvolume --from-literal=var1=val1 --from-literal=var2=val2
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml

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

115. Create a pod called secbusybox with the image busybox which executes command sleep 3600 and makes sure any Containers in the Pod, all processes run with user ID 1000 and with group id 2000 and verify.

kubectl run secbusybox --image=busybox --dry-run=client -o yaml -- /bin/sh -c 'sleep 3600' > pod.yaml

Add:

...
    securityContext:
        runAsUser: 1000
        runAsGroup: 2000
...

kubectl create -f pod.yaml
kubectl exec -it secbusybox -- sh
id
exit

116. Create the same pod as above this time set the securityContext for the container as well and verify that the securityContext of container overrides the Pod level securityContext.

kubectl run secbusybox --image=busybox --dry-run=client -o yaml -- /bin/sh -c 'sleep 3600' > pod.yaml

Add:

...
    securityContext:
        runAsUser: 1000
    containers:
    ...
        securityContext:
            runAsUser: 1001
    ...
...

kubectl create -f pod.yaml
kubectl exec -it secbusybox -- sh
id    ## Container securityContext overrides
exit

117. Create pod with an nginx image and configure the pod with capabilities NET_ADMIN and SYS_TIME verify the capabilities

kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml

Add:

...
spec:
    containers:
    ...
        securityContext:
            capabilities:
                add: ['NET_ADMIN', 'SYS_TIME']
    ...
...

kubectl create -f pod.yaml
kubectl exec -it nginx -- sh
cd /proc/1
cat status

// Expected values
CapPrm: 00000000aa0435fb
CapEff: 00000000aa0435fb

118. Create a Pod nginx and specify a memory request and a memory limit of 100Mi and 200Mi respectively.

kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml

Add:

...
spec:
    containers:
    ...
        resources:
            requests:
                memory: 100Mi
            limits:
                memory: 200Mi
    ...
...


kubectl create -f pod.yaml
kubectl top pod

119. Create a Pod nginx and specify a CPU request and a CPU limit of 0.5 and 1 respectively.

kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml

Add:

...
spec:
    containers:
    ...
        resources:
            requests:
                cpu: 0.5
            limits:
                cpu: 1
    ...
...


kubectl create -f pod.yaml
kubectl top pod

120. Create a Pod nginx and specify both CPU, memory requests and limits together and verify.

kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml

Add:

...
spec:
    containers:
    ...
        resources:
            requests:
                memory: 100Mi
                cpu: 0.5
            limits:
                memory: 200Mi
                cpu: 1
    ...
...


kubectl create -f pod.yaml
kubectl top pod

121. Create a Pod nginx and specify a memory request and a memory limit of 100Gi and 200Gi respectively which is too big for the nodes and verify pod fails to start because of insufficient memory

kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml

Add:

...
spec:
    containers:
    ...
        resources:
            requests:
                memory: 100Gi
            limits:
                memory: 200Gi
    ...
...


kubectl create -f pod.yaml
kubectl describe pod nginx

122. Create a secret mysecret with values user=myuser and password=mypassword

kubectl create secret generic mysecret --from-literal=user=myuser --from-literal=password=mypassword

123. List the secrets in all namespaces

kubectl get secrets

124. Output the yaml of the secret created above

kubectl get secret mysecret -o yaml

125. Create an nginx pod which reads username as the environment variable

kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml

Add:

...
    env:
    - name: USERNAME
      valueFrom:
        secretKeyRef:
          name: mysecret
          key: username
...

kubectl create -f pod.yaml
kubectl exec -it nginx -- env

126. Create an nginx pod which loads the secret as environment variables

kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml

Add:

...
    envFrom:
        secretRef:
            name: mysecret
...

kubectl create -f pod.yaml
kubectl exec -it nginx -- env

127. List all the service accounts in the default namespace

kubectl get sa

128. List all the service accounts in all namespaces

kubectl get sa -A

129. Create a service account called admin

kubectl create sa admin

130. Output the YAML file for the service account we just created

kubectl get sa admin -o yaml

131. Create a busybox pod which executes this command sleep 3600 with the service account admin and verify

kubectl run busybox --image=busybox --dry-run=client -o yaml -- /bin/sh -c 'sleep 3600' > pod.yaml

Add:

...
    serviceAccountName: "admin"
...

kubectl create -f pod.yaml
kubectl describe pod busybox

----------------------------
## Observability (18%)
----------------------------

132. Create an nginx pod with containerPort 80 and it should only receive traffic only it checks the endpoint / on port 80 and verify and delete the pod.

kubectl run nginx --image=nginx --port=80 --dry-run=client -o yaml > pod.yaml

Add:

...
    readinessProbe:
        httpGet:
            path: /
            port: 80
...


kubectl create -f pod.yaml
kubectl describe pod nginx | grep -i readiness
kubectl delete pod nginx

133. Create an nginx pod with containerPort 80 and it should check the pod running at endpoint / healthz on port 80 and verify and delete the pod.

kubectl run nginx --image=nginx --port=80 --dry-run=client -o yaml > pod.yaml

Add:

...
    livenessProbe:
        httpGet:
            path: /healthz
            port: 80
...


kubectl create -f pod.yaml
kubectl describe pod nginx | grep -i liveness
kubectl delete pod nginx

134. Create an nginx pod with containerPort 80 and it should check the pod running at endpoint /healthz on port 80 and it should only receive traffic only it checks the endpoint / on port 80. verify the pod.

kubectl run nginx --image=nginx --port=80 --dry-run=client -o yaml > pod.yaml

Add:

...
    readinessProbe:
        httpGet:
            path: /
            port: 80
    livenessProbe:
        httpGet:
            path: /healthz
            port: 80
...


kubectl create -f pod.yaml
kubectl describe pod nginx | grep -i readiness
kubectl describe pod nginx | grep -i liveness
kubectl delete pod nginx

135. Check what all are the options that we can configure with readiness and liveness probes

kubectl explain pod.spec.containers.readinessProbe
kubectl explain pod.spec.containers.livenessProbe

136. Create the pod nginx with the above liveness and readiness probes so that it should wait for 20 seconds before it checks liveness and readiness probes and it should check every 25 seconds.

kubectl run nginx --image=nginx --port=80 --dry-run=client -o yaml > pod.yaml

Add:

...
    readinessProbe:
        httpGet:
            path: /
            port: 80
        initialDelaySeconds: 20
        periodSeconds: 25
    livenessProbe:
        httpGet:
            path: /healthz
            port: 80
        initialDelaySeconds: 20
        periodSeconds: 25
...


kubectl create -f pod.yaml
kubectl describe pod nginx | grep -i readiness
kubectl describe pod nginx | grep -i liveness
kubectl delete pod nginx

137. Create a busybox pod with this command “echo I am from busybox pod; sleep 3600;” and verify the logs.

kubectl run busybox --image=busybox -- /bin/sh -c 'echo "I am from busybox pod"; sleep 3600'
kubectl logs busybox

138. Copy the logs of the above pod to the busybox-logs.txt and verify

kubectl logs busybox > busybox-logs.txt
cat busybox-logs.txt

139. List all the events sorted by timestamp and put them into file.log and verify

kubectl get events --sort-by=.metadata.creationTimestamp > file.log
cat file.log

140. Create a pod with an image alpine which executes this command ”while true; do echo ‘Hi I am from alpine’; sleep 5; done” and verify and follow the logs of the pod.

kubectl run alpine --image=alpine -- /bin/sh -c ”while true; do echo ‘Hi I am from alpine’; sleep 5; done” 
kubectl logs alpine -f

141. Create the pod with this kubectl create -f https://gist.githubusercontent.com/bbachi/212168375b39e36e2e2984c097167b00/raw/1fd63509c3ae3a3d3da844640fb4cca744543c1c/not-running.yml. The pod is not in the running state. Debug it.

kubectl create -f https://gist.githubusercontent.com/bbachi/212168375b39e36e2e2984c097167b00/raw/1fd63509c3ae3a3d3da844640fb4cca744543c1c/not-running.yml
kubectl describe pod not-running # ImagePullBackOff

142. This following yaml creates 4 namespaces and 4 pods. One of the pod in one of the namespaces are not in the running state. Debug and fix it. https://gist.githubusercontent.com/bbachi/1f001f10337234d46806929d12245397/raw/84b7295fb077f15de979fec5b3f7a13fc69c6d83/problem-pod.yaml.

kubectl create -f https://gist.githubusercontent.com/bbachi/1f001f10337234d46806929d12245397/raw/84b7295fb077f15de979fec5b3f7a13fc69c6d83/problem-pod.yaml
kubectl get pods -A ## ImagePullBackOff

143. Get the memory and CPU usage of all the pods and find out top 3 pods which have the highest usage and put them into the cpu-usage.txt file

kubectl top pod -A | sort --reverse --key 3 --numeric | head -3 > cpu-usage.txt

----------------------------
## Observability (18%)
----------------------------

144. Create an nginx pod with a yaml file with label my-nginx and expose the port 80

kubectl run nginx --image=nginx -l app=my-nginx --port=80

145. Create the service for this nginx pod with the pod selector app: my-nginx

kubectl create service clusterip my-nginx --tcp=80

146. Find out the label of the pod and verify the service has the same label

kubectl get pods nginx --show-labels
kubectl get svc my-nginx -o wide

147. Delete the service and create the service with kubectl expose command and verify the label

kubectl delete svc my-nginx
kubectl expose pod nginx --port=80 --target-port=80
kubectl get svc -l app=my-nginx

148. Delete the service and create the service again with type NodePort

kubectl delete svc my-nginx
kubectl expose pod nginx --port=80 --type=NodePort

149. Create the temporary busybox pod and hit the service. Verify the service that it should return the nginx page index.html.

kubectl get svc -l app=mynginx -o wide # <svc-port>
minikube ip # <cluster-ip>
kubectl run busybox --image=busybox --restart=Never --rm -- /bin/sh -c 'wget -o- <cluster-ip>:<svc-port>'

150. Create a NetworkPolicy which denies all ingress traffic

apiVersion: v1
kind: NetworkPolicy
metadata:
    name: np
spec:
    podSelector: {}
    policyTypes:
    - Ingress

kubectl create -f np.yaml

