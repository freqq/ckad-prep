## Creating a Scheduled Container Operation

1. Create a CronJob named `current-date` that runs every minute and executes the shell command `echo "Current date: $(date)"`.

kubectl create cj current-date --schedule="*/1 * * * *" --image=busybox -- /bin/sh -c 'echo "Current date: $(date)"'

2. Watch the jobs as they are being scheduled.

kubectl get jobs -w

3. Identify one of the Pods that ran the CronJob and render the logs.

kubectl get pods
kubectl logs <pod-name>

4. Determine the number of successful executions the CronJob will keep in its history.

kubectl describe cj current-date | grep "Successful Job History Limit:"

5. Delete the Job.

kubectl delete cj current-date