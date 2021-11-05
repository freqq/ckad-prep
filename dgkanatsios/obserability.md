---

## Liveness and readiness probes

---

1. Create an nginx pod with a liveness probe that just runs the command 'ls'. Save its YAML in pod.yaml. Run it, check its probe status, delete it.

kubectl run nginx --image=nginx --dry-run=client -o yaml --restart=Never > pod.yaml

Add:

...
      livenessProbe:
        exec:
          command: ["ls"]
...

kubectl create -f pod.yaml
kubectl describe pod nginx
kubectl delete pod nginx

2. Modify the pod.yaml file so that liveness probe starts kicking in after 5 seconds whereas the interval between probes would be 5 seconds. Run it, check the probe, delete it.

...
      livenessProbe:
        exec:
          command: ["ls"]
        initialDelaySeconds: 5
...

3. Create an nginx pod (that includes port 80) with an HTTP readinessProbe on path '/' on port 80. Again, run it, check the readinessProbe, delete it.

kubectl run nginx --image=nginx --port=80 --dry-run=client -o yaml --restart=Never > pod.yaml

Add:

...
      readinessProbe:
        httpGet:
          path: "/"
          port: 80
...

kubectl create -f pod.yaml
kubectl describe pod nginx
kubectl delete pod nginx

4. Lots of pods are running in qa,alan,test,production namespaces. All of these pods are configured with liveness probe. Please list all pods whose liveness probe are failed in the format of <namespace>/<pod name> per line.

kubectl get events -n qa | grep -i "Liveness probe failed"
kubectl get events -n alan | grep -i "Liveness probe failed"
kubectl get events -n test | grep -i "Liveness probe failed"
kubectl get events -n production | grep -i "Liveness probe failed"

---

## Logging

---

5. Create a busybox pod that runs 'i=0; while true; do echo "$i: $(date)"; i=$((i+1)); sleep 1; done'. Check its logs

kubectl run busybox --image=busybox --restart=Never -- /bin/sh -c 'i=0; while true; do echo "$i: $(date)"; i=$((i+1)); sleep 1; done'
kubectl logs busybox

---

## Debugging

---

6. Create a busybox pod that runs 'ls /notexist'. Determine if there's an error (of course there is), see it. In the end, delete the pod

kubectl run busybox --image=busybox --restart=Never -- /bin/sh -c "ls /notexist"
kubectl logs busybox
kubectl delete pod busybox

7. Create a busybox pod that runs 'notexist'. Determine if there's an error (of course there is), see it. In the end, delete the pod forcefully with a 0 grace period

kubectl run busybox --image=busybox --restart=Never -- /bin/sh -c "notexist"
kubectl logs busybox
kubectl describe pod buysbox
kubectl delete pod busybox --grace-period=0 --force

8. Get CPU/memory utilization for nodes (metrics-server must be running)

kubectl top nodes