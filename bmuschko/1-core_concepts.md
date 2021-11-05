----------------------------
# Creating a Pod and Inspecting it
----------------------------

1. Create the namespace `ckad-prep`.

kubectl create ns ckad-prep

2. In the namespace `ckad-prep` create a new Pod named `mypod` with the image `nginx:2.3.5`. Expose the port 80.

kubectl run mypod --image=nginx:2.3.5 --restart=Never -n ckad-prep --port=80

3. Identify the issue with creating the container. Write down the root cause of issue in a file named `pod-error.txt`.

kubectl describe pod -n ckad-prep mypod
echo "ImagePullBackOff" > pod-error.txt

4. Change the image of the Pod to `nginx:1.15.12`.

kubectl set image pod mypod nginx=nginx:1.15.12 -n ckad-prep

5. List the Pod and ensure that the container is running.

kubectl get pods -n ckad-prep mypod -w

6. Log into the container and run the `ls` command. Write down the output. Log out of the container.

kubectl exec -it -n ckad-prep mypod -- /bin/sh
ls > output.txt
exit

7. Retrieve the IP address of the Pod `mypod`.

kubectl get pods -n ckad-prep mypod -o wide

8. Run a temporary Pod using the image `busybox`, shell into it and run a `wget` command against the `nginx` Pod using port 80.

kubectl run busybox -n ckad-prep --image=busybox --rm --restart=Never -- /bin/sh -c 'wget -O- <pod-ip>:80'

9. Render the logs of Pod `mypod`.

kubectl logs -n ckad-prep mypod

10. Delete the Pod and the namespace.

kubectl delete pod -n ckad-prep mypod
kubectl delete ns ckad-prep