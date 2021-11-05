## Performing Rolling Updates for a Deployment

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

kubectl get deployments

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

kubectl get pods -l app=v1 -o yaml | grep -i image:

9. (Optional) Discuss: Can you foresee potential issues with a rolling deployment? How do you configure a update process that first kills all existing containers with the current version before it starts containers with the new version?