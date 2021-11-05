----------------------------
# Observability (18%)
----------------------------

1. Create an nginx pod with containerPort 80 and it should only receive traffic only it checks the endpoint / on port 80 and verify and delete the pod

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

2. Create an nginx pod with containerPort 80 and it should check the pod running at endpoint / healthz on port 80 and verify and delete the pod

kubectl run nginx --image=nginx --port=80 --dry-run=client -o yaml > pod.yaml

Add:

...
    livenessProbe:
        httpGet:
            path: /healtz
            port: 80
...

kubectl create -f pod.yaml
kubectl describe pod nginx | grep -i liveness
kubectl delete pod nginx

3. Create an nginx pod with containerPort 80 and it should check the pod running at endpoint /healthz on port 80 and it should only receive traffic only it checks the endpoint / on port 80. verify the pod

kubectl run nginx --image=nginx --port=80 --dry-run=client -o yaml > pod.yaml

Add:

...
    readinessProbe:
        httpGet:
            path: /
            port: 80
    livenessProbe:
        httpGet:
            path: /healtz
            port: 80
...

kubectl create -f pod.yaml
kubectl describe pod nginx | grep -i readiness
kubectl describe pod nginx | grep -i liveness
kubectl delete pod nginx

4. Check what all are the options that we can configure with readiness and liveness probes

kubectl explain pod.spec.containers.readinessProbe
kubectl explain pod.spec.containers.livenessProbe

5. Create the pod nginx with the above liveness and readiness probes so that it should wait for 20 seconds before it checks liveness and readiness probes and it should check every 25 seconds.

kubectl run nginx --image=nginx --port=80 --dry-run=client -o yaml > pod.yaml

Add:

...
    readinessProbe:
        httpGet:
            path: /
            port: 80
        initialDelaySeconds: 20
        periodSeconds: 20
    livenessProbe:
        httpGet:
            path: /healtz
            port: 80
        initialDelaySeconds: 20
        periodSeconds: 20
...

kubectl create -f pod.yaml
kubectl describe pod nginx | grep -i readiness
kubectl delete pod nginx

6. Create a busybox pod with this command “echo I am from busybox pod; sleep 3600;” and verify the logs

kubectl run busybox --image=busybox --restart=Never -- /bin/sh -c “echo I am from busybox pod; sleep 3600;”
kubectl logs busybox

7. copy the logs of the above pod to the busybox-logs.txt and verify

kubectl logs busybox > busybox-logs.txt

8. List all the events sorted by timestamp and put them into file.log and verify

kubectl get events --sort-by=.metadata.creationStamp > file.log
cat file.log

9. Create a pod with an image alpine which executes this command ”while true; do echo ‘Hi I am from alpine’; sleep 5; done” and verify and follow the logs of the pod

kubectl run apline --image=alpine --restart=Never -- /bin/sh -c ”while true; do echo ‘Hi I am from alpine’; sleep 5; done”
kubectl logs alpine -f

10. Create the pod with this kubectl create -f https://gist.githubusercontent.com/bbachi/212168375b39e36e2e2984c097167b00/raw/1fd63509c3ae3a3d3da844640fb4cca744543c1c/not-running.yml. The pod is not in the running state. Debug it.

Wrong image.

11. This following yaml creates 4 namespaces and 4 pods. One of the pod in one of the namespaces are not in the running state. Debug and fix it. `https://gist.githubusercontent.com/bbachi/1f001f10337234d46806929d12245397/raw/84b7295fb077f15de979fec5b3f7a13fc69c6d83/problem-pod.yaml`.

Wrong image.

12. Get the memory and CPU usage of all the pods and find out top 3 pods which have the highest usage and put them into the cpu-usage.txt file

kubectl top pods -A | sort --reverse --key 3 --numeric | head -3 > cpu-usage.txt
cat cpu-usage.txt