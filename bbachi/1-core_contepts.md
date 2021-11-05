----------------------------
# Core Concepts (13%)
----------------------------

1. List all the namespaces in the cluster

kubectl gett ns

2. List all the pods in all namespaces

kubectl get pods -A

3. List all the pods in the particular namespace

kubectl get pods -n <namespace>

4. List all the services in the particular namespace

kubectl get svc -n <namespace>

5. List all the pods showing name and namespace with a json path expression

kubectl get pods -o=jsonpath="{.items[*]['metadata.name', 'metadata.namespace']}"

6. Create an nginx pod in a default namespace and verify the pod running

kubectl run nginx --image=nginx --restart=Never
kubectl get pods nginx

7. Create the same nginx pod with a yaml file

```yaml
apiVersion: v1
kind: Pod
metadata:
    name: nginx
spec:
    dnsPolicy: ClusterFirst
    restart: Never
    containers: 
    - name: nginx
      image: nginx
      resources: {}
```

8. Output the yaml file of the pod you just created

kubectl get pods nginx  -o yaml

9. Output the yaml file of the pod you just created without the cluster-specific information

--export deprecated (!)

10. Get the complete details of the pod you just created

kubectl describe pod nginx

11. Delete the pod you just created

kubectl delete pod nginx

12. Delete the pod you just created without any delay (force delete)

kubectl delete pod --grace-period=0 --force

13. Create the nginx pod with version 1.17.4 and expose it on port 80

kubectl run nginx --image=nginx:1.17.4 --port=80

14. Change the Image version to 1.15-alpine for the pod you just created and verify the image version is updated

kubectl set image pod nginx nginx=nginx:1.15-alpine
kubectl describe pod nginx | grep -i image:

15. Change the Image version back to 1.17.1 for the pod you just updated and observe the changes

kubectl set image pod nginx nginx=nginx:1.17.1
kubectl describe pod nginx | grep -i image:
kubectl get pod nginx -w

16. Check the Image version without the describe command

kubectl get pod nginx -o yaml | grep -i image:

17. Create the nginx pod and execute the simple shell on the pod

kubectl run nginx --image=nginx -- ls

18. Get the IP Address of the pod you just created

kubectl get pod nginx -o wide

19. Create a busybox pod and run command ls while creating it and check the logs

kubectl run busybox --image=busybox --restart=Never -- /bin/sh -c "ls"
kubectl logs busybox

20. If pod crashed check the previous logs of the pod

kubectl logs busybox -p

21. Create a busybox pod with command sleep 3600

kubectl run busybox --image=busybox --restart=Never -- 'sleep 3600'

22. Check the connection of the nginx pod from the busybox pod

kubectl get pods nginx -o wide
kubectl exec -it busybox -- /bin/sh
wget -O- <nginx-ip>:<nginx-port>
exit

23. Create a busybox pod and echo message ‘How are you’ and delete it manually

kubectl run busybox --image=busybox --restart=Never -- /bin/sh -c "echo 'How are you'"
kubectl delete pod busybox

24. Create a busybox pod and echo message ‘How are you’ and have it deleted immediately

kubectl run busybox --image=busybox --rm --restart=Never -- /bin/sh -c "echo 'How are you'"

25. Create an nginx pod and list the pod with different levels of verbosity

kubectl run nginx --image=nginx --restart=Never
kubectl get pod nginx --v=0

26. List the nginx pod with custom columns POD_NAME and POD_STATUS

kubectl get pods nginx -o=custom-columns="POD_NAME:.metadata.name, POD_STATUS:.status.containerStatuses[].state"

27. List all the pods sorted by name

kubectl get pods --sort-by=.metadata.name

28. List all the pods sorted by created timestamp

kubectl get pods --sort-by=.metadata.creationTimestamp
