## Configuring a Pod to Use a ConfigMap

1. Create a new file named `config.txt` with the following environment variables as key/value pairs on each line.

- `DB_URL` equates to `localhost:3306`
- `DB_USERNAME` equates to `postgres`

touch config.txt
vim config.txt

2. Create a new ConfigMap named `db-config` from that file.

kubectll create cm db-config --from-file=config.txt

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

5. (Optional) Discuss: How would you approach hot reloading of values defined by a ConfigMap consumed by an application running in Pod?

Just do it man ;_).