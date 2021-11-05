1. „Create a new Pod named nginx running the image nginx:1.17.10. Expose the container port 80. The Pod should live in the namespace named ckad.”

kubectl run nginx --image=nginx:1.17.10 --port=80 -n ckad

2. „Get the details of the Pod including its IP address.”

kubectl get pods -o wide

3. „Create a temporary Pod that uses the busybox image to execute a wget command inside of the container. The wget command should access the endpoint exposed by the nginx container. You should see the HTML response body rendered in the terminal.”

kubectl run busybox -n ckad --restart=Never --image=busybox -it --rm -- /bin/sh -c "wget -O- <nginx-pod-ip>"

4. „Get the logs of the nginx container.”

kubectl logs -n ckad nginx-<id>

5. „Add the environment variables DB_URL=postgresql://mydb:5432 and DB_USERNAME=admin to the container of the nginx Pod.”

kubectl delete pod nginx -n ckad
kubectl run nginx --image=nginx:1.17.10 --port=80 -n ckad --env="DB_URL=postgresql://mydb:5432,DB_USERNAME=admin"

6. „Open a shell for the nginx container and inspect the contents of the current directory ls -l.”

kubectl exec nginx -it -n ckad -- /bin/sh -c "ls -l"

7. „Create a YAML manifest for a Pod named loop that runs the busybox image in a container. The container should run the following command: for i in {1..10}; do echo "Welcome $i times"; done. Create the Pod from the YAML manifest. What’s the status of the Pod?”

kubectl run loop --dry-run=client -o yaml --restart=Never --image=busybox -- /bin/sh -c "for i in {1..10}; do echo "Welcome $i times"; done" > pod.yaml
kubectl apply -f pod.yaml
kubectl get pod loop -n ckad

8. „Edit the Pod named loop. Change the command to run in an endless loop. Each iteration should echo the current date.”

kubectl delete pod -n ckad loop
kubectl run loop --dry-run=client -o yaml --restart=Never --image=busybox -- /bin/sh -c "while true; do date; sleep 10;" > pod.yaml
kubectl apply -f pod.yaml

9. „Inspect the events and the status of the Pod loop.”

### Events
kubectl describe pod loop -n ckad | grep -Ci 10 events

### Status
kubectl get pod -n ckad

10. „Delete the namespace ckad and its Pods.”

kubectl delete ns ckad