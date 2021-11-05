## Restricting Access to and from a Pod

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
  name: network-policy
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
  - ports:
    - port: 3006
      protocol: TCP
```

4. The network policy should allow incoming traffic from the backend to the database but disallow incoming traffic from the frontend.

Above

5. Incoming traffic to the database should only be allowed on TCP port 3306 and no other port.

Above