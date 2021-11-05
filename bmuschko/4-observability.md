----------------------------
# Defining a Pod’s Readiness and Liveness Probe
----------------------------

1. Create a new Pod named `hello` with the image `bonomat/nodejs-hello-world` that exposes the port 3000. Provide the name `nodejs-port` for the container port.

kubectl run hello --image=bonomat/nodejs-hello-world --port=3000 --dry-run=client -o yaml > pod.yaml

Add:

...
spec:
  containers:
  - ports:
    - containerPort: 3000
      name: nodejs-port
...

2. Add a Readiness Probe that checks the URL path / on the port referenced with the name `nodejs-port` after a 2 seconds delay. You do not have to define the period interval.

Add:

...
spec:
  containers:
    readinessProbe:
      httpGet:
        path: /
        port: nodejs-port
      initialDelaySeconds: 2
...

3. Add a Liveness Probe that verifies that the app is up and running every 8 seconds by checking the URL path / on the port referenced with the name `nodejs-port`. The probe should start with a 5 seconds delay.

Add:

...
spec:
  containers:
    livenessProbe:
      httpGet:
        path: /
        port: nodejs-port
      initialDelaySeconds: 5
      periodSeconds: 8
...

kubectl create -f pod.yaml

4. Shell into container and curl `localhost:3000`. Write down the output. Exit the container.

kubectl exec -it hello -- /bin/sh
curl localhost:3000
exit

5. Retrieve the logs from the container. Write down the output.

kubectl logs hello

----------------------------
## Fixing a Misconfigured Pod
----------------------------

1. Create a new Pod with the following YAML.

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: failing-pod
  name: failing-pod
spec:
  containers:
  - args:
    - /bin/sh
    - -c
    - while true; do echo $(date) >> ~/tmp/curr-date.txt; sleep
      5; done;
    image: busybox
    name: failing-pod
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

kubectl create -f pod.yaml

2. Check the Pod's status. Do you see any issue?

kubectl describe pod failing-pod # No issues

3. Follow the logs of the running container and identify an issue.

kubectl logs failing-pod -f

Issue:
/bin/sh: can't create /root/tmp/curr-date.txt: nonexistent directory

4. Fix the issue by shelling into the container. After resolving the issue the current date should be written to a file. Render the output.

kubectl exec -it failing-pod -- /bin/sh
mkdir -p /root/tmp
cat /root/tmp/curr-date.txt
exit