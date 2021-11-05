## Using a ServiceAccount

1. Create a new service account named `backend-team`.

kubectl create sa backend-team

2. Print out the token for the service account in YAML format.

kubectl get sa backend-team -o yaml | grep -i token

3. Create a Pod named `backend` that uses the image `nginx` and the identity `backend-team` for running processes.

kubectl run backend --image=nginx --restart=Never --dry-run=client -o yaml > pod.yaml

Add:

...
spec:
    serviceAccount: backend-team
...

kubectl create -f pod.yalm

4. Get a shell to the running container and print out the token of the service account.

kubectl exec -it backend -- /bin/sh
cat /var/run/secrets/kubernetes.io/serviceaccount/token
exit
