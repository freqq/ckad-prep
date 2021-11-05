----------------------------
# Routing Traffic to Pods from Inside and Outside of a Cluster
----------------------------

1. Create a deployment named `myapp` that creates 2 replicas for Pods with the image `nginx`. Expose the container port 80.

kubectl create deployment myapp --replicas=2 --image=nginx --port=80

2. Expose the Pods so that requests can be made against the service from inside of the cluster.

kubectl expose deployment myapp --type=ClusterIP --port=80 --target-port=80

3. Create a temporary Pods using the image `busybox` and run a `wget` command against the IP of the service.

kubectl get service myapp -o wide
kubectl run temp --image=busybox  --rm -it --restart=Never -- /bin/sh -c "wget -O- <service-ip>:<service-port>"

4. Change the service type so that the Pods can be reached from outside of the cluster.

kubectl edit svc myapp

...
spec:
  type: NodePort
...

kubectl get svc

5. Run a `wget` command against the service from outside of the cluster.

minikube ip
wget -O- <minikube-ip>:<svc-node-port>

5. (Optional) Can you expose the Pods as a service without a deployment?

----------------------------
# Restricting Access to and from a Pod
----------------------------

Let's assume we are working on an application stack that defines three different layers: a frontend, a backend and a database. Each of the layers runs in a Pod. You can find the definition in the YAML file `app-stack.yaml`. The application needs to run in the namespace `app-stack`.

```yaml
kind: Pod
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
    - name: backend
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
```

1. Create the required namespace.

kubectl create ns app-stack

2. Copy the Pod definition to the file `app-stack.yaml` and create all three Pods. Notice that the namespace has already been defined in the YAML definition.

kubectl create -f app-stack.yaml

3. Create a network policy in the YAML file `app-stack-network-policy.yaml`.

```yaml
apiVersion: v1
kind: NetworkPolicy
metadata:
  name: my-np
  namespace: app-stack
spec:
  podSelector:
    matchLabels:
      app: todo
      tier: database
  polictTypes:
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
```

4. The network policy should allow incoming traffic from the backend to the database but disallow incoming traffic from the frontend.

Above 

5. Incoming traffic to the database should only be allowed on TCP port 3306 and no other port.

Above