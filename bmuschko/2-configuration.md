----------------------------
# Configuring a Pod to Use a ConfigMap
----------------------------

1. Create a new file named `config.txt` with the following environment variables as key/value pairs on each line.

- `DB_URL` equates to `localhost:3306`
- `DB_USERNAME` equates to `postgres`

2. Create a new ConfigMap named `db-config` from that file.

kubectl create cm db-config --from-file=config.txt

3. Create a Pod named `backend` that uses the environment variables from the ConfigMap and runs the container with the image `nginx`.

kubectl run backend --image=nginx --restart=Never --dry-run=client -o yaml > pod.yaml

Add:

...
    envFrom:
        configMapRef:
            name: db-config
...

kubectl create -f pod.yaml

4. Shell into the Pod and print out the created environment variables. You should find `DB_URL` and `DB_USERNAME` with their appropriate values.

kubectl exec -it backend -- env

----------------------------
# Configuring a Pod to Use a Secret
----------------------------

1. Create a new Secret named `db-credentials` with the key/value pair `db-password=passwd`.

kubectl create secret generic db-credentials --from-literal=db-password=passwd

2. Create a Pod named `backend` that defines uses the Secret as environment variable named `DB_PASSWORD` and runs the container with the image `nginx`.

kubectl run backend --image=nginx --restart=Never --dry-run=client -o yaml > pod.yaml

Add:

...
    env:
    - name: DB_PASSWORD:
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: db-password
...

kubectl create -f pod.yaml

3. Shell into the Pod and print out the created environment variables. You should find `DB_PASSWORD` variable.

kubectl exec -it backend -- env

----------------------------
# Creating a Security Context for a Pod
----------------------------

1. Create a Pod named `secured` that uses the image `nginx` for a single container. Mount an `emptyDir` volume to the directory `/data/app`.

kubectl run secured --image=nginx --restart=Never --dry-run=client -o yaml > pod.yaml

Add:

...
    volumeMounts:
    - name: empty-dir
      mountPath: /data/app
...
    volumes:
    - name: empty-dir
      emptyDir: {}
...


2. Files created on the volume should use the filesystem group ID 3000.

Add:

...
spec:
    securityContext:
        fsGroup: 3000
...

kubectl create -f pod.yaml


3. Get a shell to the running container and create a new file named `logs.txt` in the directory `/data/app`. List the contents of the directory and write them down.

kubectl exec -it secured -- /bin/sh
cd /data/app
touch logs.txt
ls
exit

----------------------------
# Defining a Pod’s Resource Requirements
----------------------------

Create a resource quota named `apps` under the namespace `rq-demo` using the following YAML definition in the file `rq.yaml`.

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: app
spec:
  hard:
    pods: "2"
    requests.cpu: "2"
    requests.memory: 500m
```

1. Create a new Pod that exceeds the limits of the resource quota requirements. Write down the error message.

kubectl create ns rq-demo
kubectl create -f rq.yaml -n rq-demo
kubectl describe quota --namespace=rq-demo

kubectl run nginx --image=nginx -o yaml --dry-run=client --restart=Never > pod.yaml

Add:

...
spec:
    containers:
        resources:
            requests:
                cpu: 2.1
                memory: 600Mi
...

kubectl create -f -n rs-demo pod.yaml

Error from server (Forbidden): error when creating "pod.yaml": pods "nginx" is forbidden: exceeded quota: app, requested: requests.memory=600Mi, used: requests.memory=0, limited: requests.memory=500m

2. Change the request limits to fulfill the requirements to ensure that the Pod could be created successfully. Write down the output of the command that renders the used amount of resources for the namespace.

kubectl edit pod nginx -n rq-demo

...
spec:
    containers:
        resources:
            requests:
                cpu: 0.2
                memory: 300Mi
...

kubectl get pods -n rq-demo nginx -w
kubectl describe quota -n rq-demo


----------------------------
# Using a Service Account
----------------------------

1. Create a new service account named `backend-team`.

kubectl create sa backend-team

2. Print out the token for the service account in YAML format.

kubectl describe sa backend-team | grep -i tokens

3. Create a Pod named `backend` that uses the image `nginx` and the identity `backend-team` for running processes.

kubectl run backend --image=nginx --restart=Never --dry-run=client -o yaml > pod.yaml

Add:

...
spec:
    serviceAccount: backend-team
...

kubectl create -f pod.yaml

4. Get a shell to the running container and print out the token of the service account.

kubectl exec -it backend -- /bin/sh
cat /var/run/secrets/kubernetes.io/serviceaccount/token
exit