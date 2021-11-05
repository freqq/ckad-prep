---

## CONFIGMAPS

---

1. Create a configmap named config with values foo=lala,foo2=lolo

kubectl create configmap cm --from-literal=foo=lala --from-literal=foo2=lolo

2. Display its values

kubectl get cm cm -o yaml

3. Create and display a configmap from a file

kubectl create cm cm-from-file --from-file=config.txt

4. Create and display a configmap from a .env file

kubectl create cm-from-env --from-env-file=config.env

5. Create and display a configmap from a file, giving the key 'special'

kubectl create cm cm-special --from-file=special=config.txt

6. Create a configMap called 'options' with the value var5=val5. Create a new nginx pod that loads the value from variable 'var5' in an env variable called 'option'

kubectl create cm options --from-literal=var5=val5
kubectl run nginx --image=nginx --restart=Never --dry-run=client -o yaml > pod.yaml

Add:

...
    env:
    - name: option
      envFrom:
        configMapRef:
          name: options
          value: var5
...

7. Create a configMap 'anotherone' with values 'var6=val6', 'var7=val7'. Load this configMap as env variables into a new nginx pod

kubectl create cm anotherone --from-literal=var6=val6 --from-literal=var7=val7
kubectl run nginx --image=nginx --restart=Never --dry-run=client -o yaml > pod.yaml

Add:

...
    envFrom:
    - configMapRef: 
        name: anotherone
...

8. Create a configMap 'cmvolume' with values 'var8=val8', 'var9=val9'. Load this as a volume inside an nginx pod on path '/etc/lala'. Create the pod and 'ls' into the '/etc/lala' directory.

kubectl create cm cmvolume --from-literal=var8=val8 --from-literal=var9=val9
kubectl run nginx --image=nginx --restart=Never --dry-run=client -o yaml > pod.yaml

Add:

...
      volumeMounts:
        - name: cm-volume
          mountPath: /etc/lala
...
  volumes:
    - name: cm-volume
      configMap:
        name: cmvolume
...

kubectl create -f pod.yaml
kubectl exec -it nginx -- /bin/sh
ls /etc/lala

---

## SECURITY CONTEXT

---

9. Create the YAML for an nginx pod that runs with the user ID 101. No need to create the pod

kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml

Add:

...
 securityContext: # insert this line
    runAsUser: 101 # UID for the user
...

10. Create the YAML for an nginx pod that has the capabilities "NET_ADMIN", "SYS_TIME" added on its single container

kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml

Add:

...
 securityContext: # insert this line
    capabilities:
        add: ["NET_ADMIN", "SYS_TIME"]
...

---

## REQUESTS AND LIMITS

---

11. Create an nginx pod with requests cpu=100m,memory=256Mi and limits cpu=200m,memory=512Mi

kubectl run nginx --image=nginx --restart=Never --dry-run=client -o yaml | kubectl set resources -f - --requests=cpu=100m,memory=256Mi --limits=cpu=200m,memory=512Mi --local -o yaml > nginx-pod.yml
kubectl create -f nginx-pod.yaml

---

## SECRETS

---

12. Create a secret called mysecret with the values password=mypass

kubectl create secret generic mysecret --from-literal=password=mypass

13. Create a secret called mysecret2 that gets key/value from a file

kubectl create secret generic mysecret2 --from-file=username

14. Get the value of mysecret2

kubectl get secret mysecret2 -o yaml
echo -n YWRtaW4= | base64 -d

15. Create an nginx pod that mounts the secret mysecret2 in a volume on path /etc/foo

kubectl run nginx --image=nginx --restart=Never -o yaml --dry-run=client > pod.yaml

Add:

...
      volumeMounts:
        - name: secret-volume
          mountPath: /etc/foo
...
  volumes:
    - name: secret-volume
      secret:
        secretName: mysecret2
...

kubectl create -f pod.yaml

16. Delete the pod you just created and mount the variable 'username' from secret mysecret2 onto a new nginx pod in env variable called 'USERNAME'

kubectl delete pod nginx
kubectl run nginx --image=nginx --restart=Never -o yaml --dry-run=client > pod.yaml

Add:

...
      env:
        - name: USERNAME
          valueFrom:
            secretKeyRef:
              name: mysecret2
              key: username
...

---

## SERVICE ACCOUNTS

---

17. See all the service accounts of the cluster in all namespaces

kubectl get sa -A

18. Create a new serviceaccount called 'myuser'

kubectl create sa myuser

19. Create an nginx pod that uses 'myuser' as a service account

kubectl run nginx --image=nginx --restart=Never --serviceaccount=myuser --dry-run=client -o yaml > pod.yaml
kubectl create -f pod.yaml