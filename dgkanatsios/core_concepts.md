1. Create a namespace called 'mynamespace' and a pod with image nginx called nginx on this namespace

kubectl create ns mynamespace
kubectl run nginx -n mynamespace --image=nginx

2. Create the pod that was just described using YAML

kubectl run nginx -n mynamespace --image=nginx --dry-run=client -o yaml > pod.yaml

3. Create a busybox pod (using kubectl command) that runs the command "env". Run it and see the output

kubectl run busybox --image=busybox --command --restart=Never -it -- env
kubectl logs busybox

4. Create a busybox pod (using YAML) that runs the command "env". Run it and see the output

kubectl run busybox --image=busybox --restart=Never --dry-run=client -o yaml --command -- env > pod.yaml
cat pod.yaml
kubectl apply -f pod.yaml

5. Get the YAML for a new namespace called 'myns' without creating it

kubectl create ns myns --dry-run=client -o yaml > ns.yaml

6. Get the YAML for a new ResourceQuota called 'myrq' with hard limits of 1 CPU, 1G memory and 2 pods without creating it

kubectl create quota myrq --hard=cpu=1,memory=16,pods=2 --dry-run=client -o yaml

7. Get pods on all namespaces

kubectl get pods -A

8. Create a pod with image nginx called nginx and expose traffic on port 80

kubectl run nginx --image=nginx --port=80 --restart=Never

9. Change pod's image to nginx:1.7.1. Observe that the container will be restarted as soon as the image gets pulled

kubectl set image pod/nginx nginx=nginx:1.7.1
kubectl describe pod nginx
kubectl get pod nginx -w

10. Get nginx pod's ip created in previous step, use a temp busybox image to wget its '/'

kubectl get pods -o wide | grep nginx
kubectl run busybox --image=busybox --rm -it --restart=Never -- wget -O- 172.17.0.2

11. Get pod's YAML

kubectl get pod nginx -o yaml

12. Get information about the pod, including details about potential issues (e.g. pod hasn't started)

kubectl describe pod nginx

13. Get pod logs

kubectl logs nginx

14. If pod crashed and restarted, get logs about the previous instance

kubectl logs nginx -p

15. Execute a simple shell on the nginx pod

kubectl exec -it nginx -- ls

16. Create a busybox pod that echoes 'hello world' and then exits

kubectl run busybox --image=busybox --restart=Never -it -- echo "hello world"

17. Do the same, but have the pod deleted automatically when it's completed

kubectl run busybox --image=busybox --restart=Never --rm -it -- echo "hello world"

18. Create an nginx pod and set an env value as 'var1=val1'. Check the env value existence within the pod

kubectl run nginx --image=nginx --env=var1=val1 --rm --restart=Never -it -- env
