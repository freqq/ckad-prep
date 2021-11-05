1. „Create a directory with the name config. Within the directory, create two files. The first file should be named db.txt and contain the key-value pair password=mypwd. The second file is named ext-service.txt and should define the key-value pair api_key=LmLHbYhsgWZwNifiqaRorH8T.”

mkdir config
cd config
echo "password=mypwd" > db.txt
echo "api_key=LmLHbYhsgWZwNifiqaRorH8T" > ext-service.txt

2. „Create a Secret named ext-service-secret that uses the directory as data source and inspect the YAML representation of the object.”

kubectl create secret generic ext-service-secret --from-file=config
kubectl get secret ext-service-secret -o yaml

3. „Create a Pod named consumer with the image nginx and mount the Secret as a Volume with the mount path /var/app. Open an interactive shell and inspect the values of the Secret.”

kubectl run consumer --image=nginx --restart=Never --dry-run=client -o yaml > pod.yaml

Add:

...
volumeMounts:
- name: secret-volume
    mountPath: /var/app
...
volumes:
- name: secret-volume
secret:
    secretName: ext-service-secret
...

kubectl apply -f pod.yaml
kubectl exec -it consumer -- /bin/sh -c "ls /var/app"

4. „Use the declarative approach to create a ConfigMap named ext-service-configmap. Feed in the key-value pairs api_endpoint=https://myapp.com/api and username=bot as literals.”

kubectl create cm ext-service-configmap -o yaml --dry-run=client > cm.yaml

Add:

...
data:
  api_endpoint: https://myapp.com/api
  username: bot
...

5. „Inject the ConfigMap values into the existing Pod as environment variables. Ensure that the keys conform to typical naming conventions of environment variables.”

...
env:
    - name: API_ENDPOINT
        valueFrom:
        configMapKeyRef:
            name: ext-service-configmap
            key: api_endpoint
    - name: USERNAME
        valueFrom:
            configMapKeyRef:
            name: ext-service-configmap
            key: username
...

6. „Open an interactive shell and inspect the values of the ConfigMap.”

kubectl exec -it consumer -- /bin/sh
env

7. „Define a security context on the container level of a new Pod named security-context-demo that uses the image alpine. The security context adds the Linux capability CAP_SYS_TIME to the container. Explain if the value of this security context can be redefined in a Pod-level security context.”

kubectl run security-context-deom --restart=Never --dry-run=client --image=alpine -o yaml -> pod.yaml

...
containers:
- args:
- '-'
image: alpine
name: security-context-deom
resources: {}
securityContext: # Add this
    capabilities:
    add: ["SYS_TIME"]
dnsPolicy: ClusterFirst
...

8. „Define a ResourceQuota for the namespace project-firebird. The rules should constrain the count of Secret objects within the namespace to 1.”

kubectl create namespace project-firebird
kubectl create quota firebird-quota --namespace=project-firebird --dry-run=client -o yaml > rq.yaml

Add:

...
spec:
  hard:
    secrets: 1

9. „Create as many Secret objects within the namespace until the maximum number enforced by the ResourceQuota has been reached.”

kubectl describe quota -n project-firebird firebird-quota
kubectl create secret generic my-secret --from-literal=test=hello --namespace=project-firebird

10. „Create a new Service Account named monitoring and assign it to a new Pod with an image of your choosing. Open an interactive shell and locate the authentication token of the assigned Service Account.”

kubectl create sa monitoring
kubectl get sa monitoring
kubectl run nginx --image=nginx --restart=Never --serviceaccount=monitoring
kubeclt exec -it nginx -- /bin/sh
cat /var/run/secrets/kubernetes.io/serviceaccount/token