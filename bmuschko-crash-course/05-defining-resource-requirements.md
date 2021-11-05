## Defining a Pod’s Resource Requirements

Create a resource quota named `app` under the namespace `rq-demo` using the following YAML definition in the file `rq.yaml`.

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

1. Create a new Pod that exceeds the limits of the resource quota requirements e.g. by defining 1G of memory. Write down the error message.

kubectl create ns rq-demo
kubectl create -f -n rq-demo rq.yaml

kubectl run busybox --image=busybox --dry-run=client -o yaml --restart=Never > pod.yaml

Add:

...
  resources:
    requests:
      memory: 1Gi
...

kubectl create -f -n rq-demo pod.yaml

Error from server (Forbidden): error when creating "pod.yaml": pods "mypod" is forbidden: exceeded quota: app, requested: requests.memory=1G, used: requests.memory=0, limited: requests.memory=500m


2. Change the request limits to fulfill the requirements to ensure that the Pod could be created successfully. Write down the output of the command that renders the used amount of resources for the namespace.

kubectll edit pod busybox

...
  resources:
    requests:
      memory: 200m
...

kubectl describe quota --namespace=rq-demo