1. „Create a new Pod named frontend that uses the image nginx. Assign the labels tier=frontend and app=nginx. Expose the container port 80.”

kubectl run frontend --restart=Never --image=nginx --port=80 -l="tier=frontend,app=nginx"

2. „Create a new Pod named backend that uses the image nginx. Assign the labels tier=backend and app=nginx. Expose the container port 80.”

kubectl run backend --image=nginx --restart=Never --port=80 -l="tier=backend,app=nginx"

3. „Create a new Service named nginx-service of type ClusterIP. Assign the port 9000 and the target port 8081. The label selector should use the criteria tier=backend and deployment=app.”

kubectl create svc clusterip nginx-service --tcp=9000:8081 --dry-run=client -o yaml > svc.yaml

Add:

...
selector:
  tier: backend
  deployment: app
...

kubectl create -f svc.yaml

4. „Try to access the set of Pods through the Service from within the cluster. Which Pods does the Service select?”

kubectl run busybox --image=busybox --rm -it --restart=Never -- /bin/sh -c "wget --spider --timeout=1 <svc-ip>:9000"

It times out because of wrong labels and port

5. „Fix the Service assignment to properly select the backend Pod and assign the correct target port.”

kubectl edit svc nginx-service

...
ports:
  - name: 9000-80
    port: 9000
    protocol: TCP
    targetPort: 80
...
selector:
  tier: backend
  app: nginx
...


kubectl run busybox --image=busybox --rm -it --restart=Never -- /bin/sh -c "wget --spider --timeout=1 <svc-ip>:9000"

Now it works

6. „Expose the Service to be accessible from outside of the cluster. Make a call to the Service.”

kubectl patch service nginx-service -p '{ "spec": {"type": "NodePort"} }'
kubectl get svc
kubectl get nodes
kubectl describe node minikube | grep InternalIP
wget --spider --timeout=1 <minikube-ip>:32682

7. „Assume an application stack that defines three different layers: a frontend, a backend, and a database. Each of the layers runs in a Pod. You can find the definition in the YAML file app-stack.yaml:”

„kind: Pod
apiVersion: v1
metadata:
name: frontend
namespace: app-stack
labels:
app: todo
tier: frontend
spec:
containers:

- name: frontend
  image: nginx

---

kind: Pod
apiVersion: v1
metadata:
name: backend
namespace: app-stack
labels:
app: todo
tier: backend
spec:
containers:
„ - name: backend
image: nginx

---

kind: Pod
apiVersion: v1
metadata:
name: database
namespace: app-stack
labels:
app: todo
tier: database
spec:
containers:

- name: database
  image: mysql
  env:
  - name: MYSQL_ROOT_PASSWORD
    value: example

Create the namespace and the Pods using the file app-stack.yaml.

kubectl create ns app-stack
kubectl apply -f app-stack.yml
kubectl get pods -n app-stack

8. „Create a network policy in the file app-stack-network-policy.yaml. The network policy should allow incoming traffic from the backend to the database but disallow incoming traffic from the frontend.”

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: app-stack-network-policy
  namespace: app-stack
spec:
  podSelector:
    matchLabels:
      app: todo
      tier: database
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: todo
          tier: backend

kubectl create -f app-stack-network-policy.yaml
kubectl get networkpolicy -n app-stack 

9. „Reconfigure the network policy to only allow incoming traffic to the database on TCP port 3306 and no other port.”

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: app-stack-network-policy
  namespace: app-stack
spec:
  podSelector:
    matchLabels:
      app: todo
      tier: database
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: todo
          tier: backend
    ports:
    - protocol: TCP
      port: 3306

