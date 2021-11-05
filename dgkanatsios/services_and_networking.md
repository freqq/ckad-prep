1. Create a pod with image nginx called nginx and expose its port 80

kubectl run nginx --image=nginx --restart=Never --port=80 --expose

2. Confirm that ClusterIP has been created. Also check endpoints

kubectl get svc

3. Get service's ClusterIP, create a temp busybox pod and 'hit' that IP with wget 

kubectl run busybox --rm --image=busybox -it --restart=Never -- /bin/sh -c 'wget -O- <pod-cluster-ip>'

4. Convert the ClusterIP to NodePort for the same service and find the NodePort port. Hit service using Node's IP. Delete the service and the pod at the end.

kubectl edit svc nginx
kubectl get svc
kubectl get nodes -o wide OR minikube ip
wget -O- <node-ip>:<node-port-svc-port>
kubectl delete svc nginx
kubectl delete pod nginx

5. Create a deployment called foo using image 'dgkanatsios/simpleapp' (a simple server that returns hostname) and 3 replicas. Label it as 'app=foo'. Declare that containers in this pod will accept traffic on port 8080 (do NOT create a service yet)

kubectl create deployment simpleapp --image=dgkanatsios/simpleapp --port=8080 --replicas=3
kubectl label deployment simpleapp app=foo --overwrite

6. Get the pod IPs. Create a temp busybox pod and try hitting them on port 8080

kubectl get pod -o wide | grep simpleapp
kubectl run busybox --rm --image=busybox --restart=Never -- /bin/sh -c 'wget -O- <ip>:8080'

7. Create a service that exposes the deployment on port 6262. Verify its existence, check the endpoints

kubectl expose deployment simpleapp --target-port=8080 --port=6262
kubectl get svc simpleapp
kubectl get endpoints simpleapp

8. Create a temp busybox pod and connect via wget to foo service. Verify that each time there's a different hostname returned. Delete deployment and services to cleanup the cluster

Run multiple times to see that different pods answers:
kubectl run busybox --image=busybox --rm -it --restart=Never -- /bin/sh -c 'wget -O- simpleapp:6262'
kubectl delete deployment simpleapp
kubectl delete svc simpleapp

9. Create an nginx deployment of 2 replicas, expose it via a ClusterIP service on port 80. Create a NetworkPolicy so that only pods with labels 'access: granted' can access the deployment and apply it

kubectl create deployment nginx --image=nginx --replicas=2 --port=80
kubectl expose deployment --port=80 --target-port=80

Create NetworkPolicy:

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: access-nginx # pick a name
spec:
  podSelector:
    matchLabels:
      app: nginx # selector for the pods
  ingress: # allow ingress traffic
  - from:
    - podSelector: # from pods
        matchLabels: # with this label
          access: granted

kubectl create -f policy.yaml

# This should not work. --timeout is optional here. But it helps to get answer more quickly (in seconds vs minutes)
kubectl run busybox --image=busybox --rm -it --restart=Never -- wget -O- http://nginx:80 --timeout 2     

# This should be fine
kubectl run busybox --image=busybox --rm -it --restart=Never --labels=access=granted -- wget -O- http://nginx:80 --timeout 2  
