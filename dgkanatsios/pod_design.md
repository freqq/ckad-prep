---
## LABELS AND ANNOTATIONS
---

1. Create 3 pods with names nginx1,nginx2,nginx3. All of them should have the label app=v1

kubectl run nginx1 --image=nginx --labels=app=v1 --restart=Never
kubectl run nginx2 --image=nginx --labels=app=v1 --restart=Never
kubectl run nginx3 --image=nginx --labels=app=v1 --restart=Never

2. Show all labels of the pods

kubectl get pods --show-labels

3. Change the labels of pod 'nginx2' to be app=v2

kubectl label pod nginx2 app=v2 --overwrite

4. Get the label 'app' for the pods (show a column with APP labels)

kubectl get pods -L app

5. Get only the 'app=v2' pods

kubectl get pods -l app=v2

6. Remove the 'app' label from the pods we created before

kubectl label pod nginx1 nginx2 nginx3 app-

7. Create a pod that will be deployed to a Node that has the label 'accelerator=nvidia-tesla-p100'

kubectl label nodes minikube accelerator=nvidia-tesla-p100
kubectl get nodes --show-labels
kubectl run nginx --image=nginx --dry-run=client --restart=Never -o yaml > pod.yaml

Add nodeSelector

nodeSelector: # add this
accelerator: nvidia-tesla-p100 # the selection label

8. Annotate pods nginx1, nginx2, nginx3 with "description='my description'" value

kubectl annotate pod nginx1 nginx2 nginx3 description='my description'

9. Check the annotations for pod nginx1

kubectl annotate pod nginx --list

or

kubectl describe pod nginx | grep -i annotations

10. Remove the annotations for these three pods

kubectl annotate pod nginx{1..3} description-

11. Remove these pods to have a clean state in your cluster

kubectl delete pod nginx{1..3}

---

## DEPLOYMENTS

---

12. Create a deployment with image nginx:1.18.0, called nginx, having 2 replicas, defining port 80 as the port that this container exposes (don't create a service for this deployment)

kubectl create deployment nginx --image=nginx:1.18.0 --port=80 --replicas=2

13. View the YAML of this deployment

kubectl get deployment nginx -o yaml

14. View the YAML of the replica set that was created by this deployment

kubecltl describe deployment nginx
kubectl get rs <ReplicaSetName> -o yaml

15. Get the YAML for one of the pods

kubectl get pods
kubectl get pod <pod-name> -o yaml

16. Check how the deployment rollout is going

kubectl rollout status deployment nginx

17. Update the nginx image to nginx:1.19.8

kubectl set image deployment nginx nginx=nginx:1.19.8

18. Check the rollout history and confirm that the replicas are OK

kubectl rollout history deployment nginx
kubectl get deployment nginx
kubectl get rs
kubectl get pods

19. Undo the latest rollout and verify that new pods have the old image (nginx:1.18.0)

kubectl rollout undo deployment nginx
kubeclt get pods
kubectl describe pod <pod-name> | grep -i image

20. Do an on purpose update of the deployment with a wrong image nginx:1.91

kubectl set image deployment nginx nginx=nginx:1.91

21. Verify that something's wrong with the rollout

kubectl rollout status deployment nginx
kubectl get pods

22. Return the deployment to the second revision (number 2) and verify the image is nginx:1.19.8

kubectl rollout undo deployment nginx --to-revision=2
kubectl rollout status deployment nginx
kubectl describe deployment nginx | grep -i image

23. Check the details of the fourth revision (number 4)

kubectl rollout history deployment nginx --revision=4

24. Scale the deployment to 5 replicas

kubectl scale deployment nginx --replicas=5

25. Autoscale the deployment, pods between 5 and 10, targetting CPU utilization at 80%

kubectl autoscale deployment --min=5 --max=10 --cpu-percent=80
kubectl get hpa nginx

26. Pause the rollout of the deployment

kubecltl rollout pause deployment nginx

27. Update the image to nginx:1.19.9 and check that there's nothing going on, since we paused the rollout

kubectl set image deployment nginx nginx=nginx:1.19.9
kubectl rollout history deployment nginx

28. Resume the rollout and check that the nginx:1.19.9 image has been applied

kubectl rollout resume deployment nginx
kubectl describe deployment nginx | grep -i image

29. Delete the deployment and the horizontal pod autoscaler you created

kubectl delete deployment nginx
kubectl delete hpa nginx

---

## JOBS

---

30. Create a job named pi with image perl that runs the command with arguments "perl -Mbignum=bpi -wle 'print bpi(2000)'"

kubectl create job pi --image=perl -- perl -Mbignum=bpi -wle 'print bpi(2000)'

31. Wait till it's done, get the output

kubectl get jobs -w
kubectl get pod
kubectl logs <pod-name>

32. Create a job with the image busybox that executes the command 'echo hello;sleep 30;echo world'

kubectl create job busybox --image=busybox -- /bin/sh -c 'echo hello;sleep 30;echo world'

33. Follow the logs for the pod (you'll wait for 30 seconds)

kubectl get pods
kubectl logs <pod-name> -f

34. See the status of the job, describe it and see the logs

kubectl get jobs
kubectl describe job busybox
kubectl logs job/busybox

35. Delete the job

kubectl delete job busybox

36. Create a job but ensure that it will be automatically terminated by kubernetes if it takes more than 30 seconds to execute

kubectl create job example --image=busybox --dry-run=client -o yaml -- /bin/sh -c "while true; do echo hello; sleep 100; done" > job.yaml

Add this:

spec:
    activeDeadlineSeconds: 30 # add this line

37. Create the same job, make it run 5 times, one after the other. Verify its status and delete it

kubectl create job example --image=busybox --dry-run=client -o yaml -- /bin/sh -c "while true; do echo hello; sleep 100; done" > job.yaml

Add this:

spec:
    completions: 5 # Add this line

kubectl get job busybox -w
kubectl delete jobs busybox

38. Create the same job, but make it run 5 parallel times

kubectl create job example --image=busybox --dry-run=client -o yaml -- /bin/sh -c "while true; do echo hello; sleep 100; done" > job.yaml

Add this:

spec:
    parallelism: 5 # Add this line

kubectl get job busybox -w
kubectl delete jobs busybox

---

## CRON JOBS

---

39. Create a cron job with image busybox that runs on a schedule of "*/1 * * * *" and writes 'date; echo Hello from the Kubernetes cluster' to standard output

kubectl create cj busybox --image=busybox --schedule="*/1 * * * *" -- /bin/sh -c 'date; echo Hello from the Kubernetes cluster'

40. See its logs and delete it

kubectl get cj
kubectl get jobs --watch
kubectl get po --show-labels # Observe that the pods have a label that mentions their 'parent' job
kubectl logs busybox-1529745840-m867r
kubectl delete cj busybox

41. Create a cron job with image busybox that runs every minute and writes 'date; echo Hello from the Kubernetes cluster' to standard output. The cron job should be terminated if it takes more than 17 seconds to start execution after its scheduled time (i.e. the job missed its scheduled time).

kubectl create cj busybox --image=busybox --schedule="* * * * *" --dry-run=client -o yaml -- /bin/sh -c 'date; echo Hello from the Kubernetes cluster' > cj.yaml

Add this:

spec:
    startingDeadlineSeconds: 17 # add this line

42. Create a cron job with image busybox that runs every minute and writes 'date; echo Hello from the Kubernetes cluster' to standard output. The cron job should be terminated if it successfully starts but takes more than 12 seconds to complete execution.

kubectl create cj busybox --image=busybox --schedule="* * * * *" --dry-run=client -o yaml -- /bin/sh -c 'date; echo Hello from the Kubernetes cluster' > cj.yaml

Add this:

spec:
    activeDeadlineSeconds: 12 # add this line