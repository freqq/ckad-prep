----------------------------
# Defining and Querying Labels and Annotations
----------------------------

1. Create three different Pods with the names `frontend`, `backend` and `database` that use the image `nginx`.

kubectl run frontend --image=nginx --restart=Never
kubectl run backend --image=nginx --restart=Never
kubectl run database --image=nginx --restart=Never

2. Declare labels for those Pods as follows:

- `frontend`: `env=prod`, `team=shiny`
- `backend`: `env=prod`, `team=legacy`, `app=v1.2.4`
- `database`: `env=prod`, `team=storage`

kubectl label pod frontend env=prod team=shiny
kubectl label pod backend env=prod team=legacy app=v1.2.4
kubectl label pod database env=prod team=storage

3. Declare annotations for those Pods as follows:

- `frontend`: `contact=John Doe`, `commit=2d3mg3`
- `backend`: `contact=Mary Harris`

kubectl annotate pod frontend contact="John Doe" commit=2d3mg3
kubectl annotate pod backend contact="Mary Harris"

4. Render the list of all Pods and their labels.

kubectl get pods --show-labels

5. Use label selectors on the command line to query for all production Pods that belong to the teams `shiny` and `legacy`.

kubectl get pods -l 'team in (shiny,legacy)'

6. Remove the label `env` from the `backend` Pod and rerun the selection.

kubectl annotate pod backend env-
kubectl get pods -l 'team in (shiny,legacy)'

7. Render the surrounding 3 lines of YAML of all Pods that have annotations.

kubectl get pods -o yaml | grep -C 3 -i annotations

----------------------------
# Performing Rolling Updates for a Deployment
----------------------------

1. Create a Deployment named `deploy` with 3 replicas. The Pods should use the `nginx` image and the name `nginx`. The Deployment uses the label `tier=backend`. The Pods should use the label `app=v1`.

kubectl create deployment deploy --replicas=3 --image=nginx --dry-run=client -o yaml > deployment.yaml

Add:

...
metadata:
    labels:
        tier: backend
...
spec:
    selector:
        matchLabels:
        app: v1
    template:
        metadata:
            labels:
                app: v1
...

kubectl create -f deployment.yaml

2. List the Deployment and ensure that the correct number of replicas is running.

kubectl get deployment deploy

3. Update the image to `nginx:latest`.

kubectl set image deployment deploy nginx=nginx:latest

4. Verify that the change has been rolled out to all replicas.

kubectl rollout status deployment deploy

5. Scale the Deployment to 5 replicas.

kubectl scale deployment deploy --replicas=5

6. Have a look at the Deployment rollout history.

kubectl rollout history deployment deploy

7. Revert the Deployment to revision 1.

kubectl rollout undo deployment deploy --to-revision=1

8. Ensure that the Pods use the image `nginx`.

kubectl get pods -l app=v1
kubectl describe pod deploy-<id>-<id> | grep -i image:

----------------------------
# Creating a Scheduled Container Operation
----------------------------

1. Create a CronJob named `current-date` that runs every minute and executes the shell command `echo "Current date: $(date)"`.

kubectl create cj current-date --image=busybox --schedule="*/1 * * * *" -- /bin/sh -c "echo `echo "Current date: $(date)"`

2. Watch the jobs as they are being scheduled.

kubectl get jobs -w

3. Identify one of the Pods that ran the CronJob and render the logs.

kubectl get pods 
kubectl logs <pod-name>

4. Determine the number of successful executions the CronJob will keep in its history.

kubectl describe cj current-date | grep -i successfulJobsHistoryLimit

5. Delete the Job.

kubectl delete cj current-date