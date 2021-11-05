1. „Create a YAML manifest for a Pod named complex-pod. The main application container named app should use the image nginx and expose the container port 80. Modify the YAML manifest so that the Pod defines an init container named setup that uses the image busybox. The init container runs the command wget -O- google.com.”

kubectl run complex-pod --restart=Never --image=nginx --port=80 --dry-run=client -o yaml > pod.yaml

Add:

...
initContainers:

- name: setup
  image: busybox
  args:
- /bin/sh
- -c
- "wget -O- google.com"
  ...

2. „Create the Pod from the YAML manifest.”

kubectl apply -f pod.yaml
kubectl get pods complex-pod -w

3. „Download the logs of the init container. You should see the output of the wget command.”

kubectl logs complex-pod -c setup

4. „Open an interactive shell to the main application container and run the ls command. Exit out of the container.”

kubectl exec -it complex-pod -c app -- /bin/sh
ls
exit

5. „Force-delete the Pod.”

kubectl delete pod complex-pod --grace-period=0 --force

6. „Create a YAML manifest for a Pod named data-exchange. The main application container named main-app should use the image busybox. The container runs a command that writes a new file every 30 seconds in an infinite loop in the directory /var/app/data. The filename follows the pattern {counter++}-data.txt. The variable counter is incremented every interval and starts with the value 1.”

kubectl run data-exchange --image=busybox --restart=Never --dry-run=client -o yaml > pod.yaml

Add: 

...
args:
- /bin/sh
- -c
- 'mkdir -p "/var/app/data/" && counter=1; while true; do touch "/var/app/data/$counter-data.txt"; counter=$((counter+1)); sleep 30; done'
...

7. „Modify the YAML manifest by adding a sidecar container named sidecar. The sidecar container uses the image busybox and runs a command that counts the number of files produced by the main-app container every 60 seconds in an infinite loop. The command writes the number of files to standard output.”

...
- name: sidebar
image: busybox
args:
- /bin/sh
- -c
- 'while true; do ls -dq "/var/app/data/*-data.txt" | wc -l; sleep 60; done'
...

8. „Define a Volume of type emptyDir. Mount the path /var/app/data for both containers.”

...
volumeMounts:
- name: empty-dir
   mountPath: /var/app/data
...
restartPolicy: Never
volumes:
- name: empty-dir
   emptyDir: {}
...

9. „Create the Pod. Tail the logs of the sidecar container.”

kubectl create -f pod.yml

10. Delete the Pod.

kubectl delete pod data-exchange