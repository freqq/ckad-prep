## Creating a Pod and Inspecting it

1. Create the namespace `ckad-prep`.

kubectl create ns ckad-prep

2. In the namespace `ckad-prep`, create a new Pod named `mypod` with the image `nginx:2.3.5`. Expose the port 80.

kubectl run mypod -n ckad-prep --image=nginx:2.3.5 --port=80

3. Identify the issue with creating the container. Write down the root cause of issue in a file named `pod-error.txt`.

kubectl describe pod -n ckad-prep
echo "Warning  Failed     1s    kubelet            Error: ImagePullBackOff" > pod-error.txt

4. Change the image of the Pod to `nginx:1.15.12`.

kubectl set image pod mypod -n ckad-prep mypod=nginx:1.15.12

5. List the Pod and ensure that the container is running.

kubectl get pods -n ckad-prep mypod

6. Log into the container and run the `ls` command. Write down the output. Log out of the container.

kubectl exec -it -n ckad-prep mypod -- ls > output.txt 

7. Retrieve the IP address of the Pod `mypod`.

kubectl get pods -n ckad-prep mypod -o wide

8. Run a temporary Pod in the namespace `ckad-prep` using the image `busybox`, shell into it and run a `wget` command against the `nginx` Pod using port 80.

kubectl run busybox -n ckad-prep --image=busybox --restart=Never -- /bin/sh -c 'wget -O- <ngin-ip>:80'

9. Render the logs of Pod `mypod`.

kubectl logs -n ckad-prep mypod

10. Delete the Pod and the namespace.

kubectl delete ns ckad-prep