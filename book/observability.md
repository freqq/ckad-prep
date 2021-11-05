1. „Define a new Pod named web-server with the image nginx in a YAML manifest. Expose the container port 80. Do not create the Pod yet.”

kubectl run web-server --image=nginx --dry-run=client -o yaml --restart=Never --port=80 > pod.yml

2. „For the container, declare a startup probe of type httpGet. Verify that the root context endpoint can be called. Use the default configuration for the probe.”

...
startupProbe:
    httpGet:
    path: /
    port: nginx-port
ports:
- containerPort: 80
    name: nginx-port
...

3. „For the container, declare a readiness probe of type httpGet. Verify that the root context endpoint can be called. Wait five seconds before checking for the first time.”

...
readinessProbe:
    httpGet:
        path: /
        port: nginx-port
    initialDelaySeconds: 5
...

4. „For the container, declare a liveness probe of type httpGet. Verify that the root context endpoint can be called. Wait 10 seconds before checking for the first time. The probe should run the check every 30 seconds.”

...
livenessProbe:
httpGet:
    path: /
    port: nginx-port
initialDelaySeconds: 10
periodSeconds: 30
...

5. „Create the Pod and follow the lifecycle phases of the Pod during the process."

kubectl create -f pod.yml
kubectl get pod web-server -w

6. „Inspect the runtime details of the probes of the Pod.”

kubectl describe pod web-server

7. „Retrieve the metrics of the Pod (e.g., CPU and memory) from the metrics server.”

kubectl top pod web-server

8. „Create a Pod named custom-cmd with the image busybox. The container should run the command top-analyzer with the command-line flag --all.”

kubectl run custom-cmd --image=busybox --restart=Never -- /bin/sh -c "top-analyzer --all"

9. „Inspect the status. How would you further troubleshoot the Pod to identify the root cause of the failure?”

kubectl logs custom-cmd