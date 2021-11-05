## Routing Traffic to Pods from Inside and Outside of a Cluster

1. Create a Service named `myapp` of type `ClusterIP` that exposes port 80 and maps to the target port 80.

kubectl create service clusterip myapp --tcp=80:80

2. Create a Deployment named `myapp` that creates 1 replica running the image `nginx`. Expose the container port 80.

kubectl create deployment myapp --replicas=1 --image=nginx --port=80

3. Scale the Deployment to 2 replicas.

kubectl scale deployment myapp --replicas=2

4. Create a temporary Pods using the image `busybox` and run a `wget` command against the IP of the service.

kubectl get svc myapp -o wide
kubectl run busybox --image=busybox -it --rm -- /bin/sh -c 'wget -O- 10.104.125.105:80'

5. Change the service type so that the Pods can be reached from outside of the cluster.

kubectl edit svc myapp

Edit:

...
spec:
    type: NodePort
...

6. Run a `wget` command against the service from outside of the cluster.

kubectl get svc myapp -o wide
wget -O- 10.104.125.105:31808

7. (Optional) Discuss: Can you expose the Pods as a service without a deployment?
8. (Optional) Discuss: Under what condition would you use the service type `LoadBalancer`?