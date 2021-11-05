----------------------------
# Pod Design (20%)
----------------------------

1. Get the pods with label information

kubectl get pods --show-labels

2. Create 5 nginx pods in which two of them is labeled env=prod and three of them is labeled env=dev

kubectl run nginx1 --image=nginx --restart=Never -l env=prod
kubectl run nginx2 --image=nginx --restart=Never -l env=prod
kubectl run nginx3 --image=nginx --restart=Never -l env=dev
kubectl run nginx4 --image=nginx --restart=Never -l env=dev
kubectl run nginx5 --image=nginx --restart=Never -l env=dev

3. Verify all the pods are created with correct labels

kubectl get pods --show-labels

4. Get the pods with label env=dev

kubectl get pods -l env=dev

5. Get the pods with label env=dev and also output the labels

kubectl get pods -l env=dev --show-labels

6. Get the pods with label env=prod

kubectl get pods -l env=prod

7. Get the pods with label env=prod and also output the labels

kubectl get pods -l env=prod --show-labels

8. Get the pods with label env

kubectl get pods -l env

9. Get the pods with labels env=dev and env=prod

kubectl get pods -l 'env in (dev,prod)'

10. Get the pods with labels env=dev and env=prod and output the labels as well

kubectl get pods -l 'env in (dev,prod)' --show-labels

11. Change the label for one of the pod to env=uat and list all the pods to verify

kubectl label pod nginx1 env=uat --overwrite
kubectl get pods --show-labels

12. Remove the labels for the pods that we created now and verify all the labels are removed

kubectl label pod nginx{1..5} env-

13. Let’s add the label app=nginx for all the pods and verify

kubectl label pod nginx{1..5} app=nginx
kubectl get pods --show-labels

14. Get all the nodes with labels (if using minikube you would get only master node)

kubectl get nodes --show-labels

15. Label the node (minikube if you are using) nodeName=nginxnode

kubectl label node minikube nodeName=nginxnode

16. Create a Pod that will be deployed on this node with the label nodeName=nginxnode

kubectl run nginx --image=nginx --restart=Never --dry-run=client -o yaml > pod.yaml

Add:

...
spec:
    nodeSelector:
        nodeName: nginxnode
...

kubectl create -f pod.yaml

17. Verify the pod that it is scheduled with the node selector

kubectl describe pod nginx | grep -i node-selectors:

18. Verify the pod nginx that we just created has this label

kubectl get pods nginx --show-labels

19. Annotate the pods with name=webapp

kubectl annotate pod nginx name=webapp

20. Verify the pods that have been annotated correctly

kubectl describe pod nginx | grep -i annotations:

21. Remove the annotations on the pods and verify

kubectl annotate pod nginx name-

22. Remove all the pods that we created so far

kubectl delete pods --all

23. Create a deployment called webapp with image nginx with 5 replicas

kubectl create deployment webapp --replicas=5 --image=nginx

24. Get the deployment you just created with labels

kubectl get deployment webapp --show-labels

25. Output the yaml file of the deployment you just created

kubectl get deployment webapp -o yaml

26. Get the pods of this deployment

kubectl get pods -l app=webapp

27. Scale the deployment from 5 replicas to 20 replicas and verify

kubectl scale deployment webapp --replicas=20
kubectl get pods -l app=webapp

28. Get the deployment rollout status

kubectl rollout status deployment webapp

29. Get the replicaset that created with this deployment

kubectl get rs -l app=webapp

30. Get the yaml of the replicaset and pods of this deployment

kubectl get rs -l app=webapp -o yaml
kubectl get pods -l app=webapp -o yaml

31. Delete the deployment you just created and watch all the pods are also being deleted

kubectl delete deployment webapp
kubectl get pods -l app=webapp -w

32. Create a deployment of webapp with image nginx:1.17.1 with container port 80 and verify the image version

kubectl create deployment webapp --image=nginx:1.17.1 --port=80
kubectl describe deployment webapp | grep -i image:

33. Update the deployment with the image version 1.17.4 and verify

kubectl set image deployment webapp nginx=nginx:1.17.4
kubectl describe deployment webapp | grep -i image:

34. Check the rollout history and make sure everything is ok after the update

kubectl rollout history deployment webapp
kubectl rollout status deployment webapp
kubectl get pods -l app=webapp

35. Undo the deployment to the previous version 1.17.1 and verify Image has the previous version

kubectl rollout undo deployment webapp
kubectl describe deployment webapp | grep -i image:

36. Update the deployment with the image version 1.16.1 and verify the image and also check the rollout history

kubectl set image deployment webapp nginx=nginx:1.16.1
kubectl describe deployment webapp | grep -i image:
kubectl rollout history deployment webapp

37. Update the deployment to the Image 1.17.1 and verify everything is ok

kubectl rollout undo deployment web-app --to-revision=3
kubectl describe deployment webapp | grep -i image:
kubectl rollout history deployment webapp

38. Update the deployment with the wrong image version 1.100 and verify something is wrong with the deployment

kubectl set image deployment webapp nginx=nginx:1.100
kubectl rollout status deployment webapp
kubectl get pods -l app=webapp

39. Undo the deployment with the previous version and verify everything is Ok

kubectl rollout undo deployment webapp
kubectl describe deployment webapp | grep -i image:
kubectl rollout status deployment webapp
kubectl get pods -l app=webapp

40. Check the history of the specific revision of that deployment

kubectl rollout history deployment webapp --revision=3

41. Pause the rollout of the deployment

kubectl rollout pause deployment webapp

42. Update the deployment with the image version latest and check the history and verify nothing is going on

kubectl set image deployment webapp nginx=nginx:latest
kubectl rollout history deployment webapp

43. Resume the rollout of the deployment

kubectl rollout resume deployment webapp

44. Check the rollout history and verify it has the new version

kubectl rollout history deployment webapp
kubectl rollout history deployment webapp --revision=9

45. Apply the autoscaling to this deployment with minimum 10 and maximum 20 replicas and target CPU of 85% and verify hpa is created and replicas are increased to 10 from 1

kubectl autoscale deployment webapp --min=10 --max=20 --cpu-percent=85
kubectl get hpa
kubectl describe deployment webapp | grep -i replicas:

46. Clean the cluster by deleting deployment and hpa you just created

kubectl delete deployment webapp
kubectl delete hpa webapp

47. Create a Job with an image node which prints node version and also verify there is a pod created for this job

kubectl create job job-name --image=node -- 'node -v'
kubect get jobs
kubectl get pods -w

48. Get the logs of the job just created

kubectl get pods 
kubectl logs <job-pod-name>

49. Output the yaml file for the Job with the image busybox which echos “Hello I am from job”

kubectl create job job-name --image=busybox --dry-run=client -o yaml -- /bin/sh -c "echo 'Hello I am from job'"

50. Copy the above YAML file to hello-job.yaml file and create the job

kubectl create job job-name --image=busybox --dry-run=client -o yaml -- /bin/sh -c "echo 'Hello I am from job'" > job.yaml
kubectl create -f job.yaml

51. Verify the job and the associated pod is created and check the logs as well

kubectl get job job-name
kubectl get pods
kubectl logs <job-pod-name>

52. Delete the job we just created

kubectl delete job job-name

53. Create the same job and make it run 10 times one after one

kubectl create job job-name --image=busybox --dry-run=client -o yaml -- /bin/sh -c "echo 'Hello I am from job'" > job.yaml

Add:

...
spec:
    completions: 10
...

kubectl create -f job.yaml

54. Watch the job that runs 10 times one by one and verify 10 pods are created and delete those after it’s completed

kubectl get jobs job-name -w
kubectl get pods
kubectl delete job job-name

55. Create the same job and make it run 10 times parallel

kubectl create job job-name --image=busybox --dry-run=client -o yaml -- /bin/sh -c "echo 'Hello I am from job'" > job.yaml

Add:

...
spec:
    parallelism: 10
...

kubectl create -f job.yaml

56. Watch the job that runs 10 times parallelly and verify 10 pods are created and delete those after it’s completed

kubectl get jobs job-name -w
kubectl get pods
kubectl delete job job-name

57. Create a Cronjob with busybox image that prints date and hello from kubernetes cluster message for every minute

kubectl create cj busybox-cj --image=busybox --schedule="*/1 * * * *" -- /bin/sh -c 'date; echo "Hello from kubernetes"'

58. Output the YAML file of the above cronjob

kubectl get cj busybox-cj -o yaml

59. Verify that CronJob creating a separate job and pods for every minute to run and verify the logs of the pod

kubectl get job
kubectl get pods
kubectl logs <cj-pod-name>

60. Delete the CronJob and verify all the associated jobs and pods are also deleted

kubectl delete cj busybox-cj
kubectl get jobs
kubectl get cj
kubeclt get pods