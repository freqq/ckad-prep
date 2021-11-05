## Configuring a Pod to Use a Secret

1. Create a new Secret named `db-credentials` with the key/value pair `db-password=passwd`.

kubectl create secret generic db-credentials --from-literal=db-password=passwd

2. Create a Pod named `backend` that uses the Secret as environment variable named `DB_PASSWORD` and runs the container with the image `nginx`.

kubectl run backend --image=nginx --dry-run=client -o yaml --restart=Never > pod.yaml

Add:

...
    envFrom:
        secretRef:
            name: db-credentials
...

kubectl create -f pod.yaml

3. Shell into the Pod and print out the created environment variables. You should find `DB_PASSWORD` variable.

kubectl exec -it backend -- env

4. (Optional) Discuss: What is one of the benefit of using a Secret over a ConfigMap?

Its encrypted in Base64.