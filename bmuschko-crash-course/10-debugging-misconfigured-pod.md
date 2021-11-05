## Fixing a Misconfigured Pod

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

kubectl logs failing-pod

# Error
/bin/sh: can't create /root/tmp/curr-date.txt: nonexistent directory

3. Follow the logs of the running container and identify an issue.

kubectl logs failing-pod -f

# Error
/bin/sh: can't create /root/tmp/curr-date.txt: nonexistent directory

4. Fix the issue by shelling into the container. After resolving the issue the current date should be written to a file. Render the output.

kubectl exec -it failing-pod -- /bin/sh
mkdir -p /root/tmp
cd /root/tmp
cat curr-date.txt
exit
kubectl logs failing-pod -f
