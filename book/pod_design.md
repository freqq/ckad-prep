1. „Create three Pods that use the image nginx. The names of the Pods should be pod-1, pod-2, and pod-3. Assign the label tier=frontend to pod-1 and the label tier=backend to pod-2 and pod-3. All pods should also assign the label team=artemidis.”

kubectl run pod-1 --image=nginx --restart=Never -l="tier=frontend,team=artemidis"
kubectl run pod-2 --image=nginx --restart=Never -l="tier=backend,team=artemidis"
kubectl run pod-3 --image=nginx --restart=Never -l="tier=backend,team=artemidis"
kubectl get pods --show-labels

2. „Assign the annotation with the key deployer to pod-1 and pod-3. Use your own name as the value.”

kubectl annotate pod pod-1 pod-3 deployer=przemek
kubectl describe pod pod-1 pod-3 | grep -i annotations

3. „From the command line, use label selection to find all Pods with the team artemidis or aircontrol and that are considered a backend service.”

kubectl get pods -l tier=backend,'team in (artemidis,aircontrol)' --show-labels

4. „Create a new Deployment named server-deployment. The Deployment should control two replicas using the image grand-server:1.4.6.”

kubectl create deployment server-deployment--image=grand-server:1.4.6
kubectl scale deployment server-deployment --replicas=2

5. „Inspect the Deployment and find out the root cause for its failure.”

kubectl get deployments
kubectl get pods
kubectl describe pod <pod-name>

6. „Fix the issue by assigning the image nginx instead. Inspect the rollout history. How many revisions would you expect to see?”

kubectl set image deployment server-deployment grand-server=nginx
kubectl rollout history deployments server-deployment

7. „Create a new CronJob named google-ping. When executed, the Job should run a curl command for google.com. Pick an appropriate image. The excution should occur every two minutes.”

kubectl create cj google-ping --image=busybox--schedule="*/2 * * * *" -- /bin/sh -c "curl google.com"

8. „Tail the logs of the CronJob at runtime. Check the command-line options of the relevant command or consult the Kubernetes documentation.”

kubectl get cronjob -w

9. „Reconfigure the CronJob to retain a history of seven executions.”

kubectl edit cj google-ping

...
spec:
  successfulJobsHistoryLimit: 7
...

10. „Reconfigure the CronJob to disallow a new execution if the current execution is still running. Consult the Kubernetes documentation for more information.”

kubectl edit cj google-ping

...
spec:
  concurrencyPolicy: Forbid
...