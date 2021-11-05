## Creating a Security Context for a Pod

1. Create a Pod named `secured` that uses the image `nginx` for a single container. Mount an `emptyDir` volume to the directory `/data/app`.

kubectl run secured --image=nginx -o yaml --dry-run=client --restart=Never > pod.yaml

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

kubectl create -f pod.yaml

2. Files created on the volume should use the filesystem group ID 3000.

Add:

...
spec:
    securityContext:
        fsGroup: 3000
...

3. Get a shell to the running container and create a new file named `logs.txt` in the directory `/data/app`. List the contents of the directory and write them down.

kubectl exec -it secured -- /bin/sh
cd /data/app
touch logs.txt
ls
exit