## Defining and Querying Labels and Annotations

1. Create three different Pods with the names `frontend`, `backend` and `database` that use the image `nginx`.

kubectl run frontend --image=nginx --restart=Never
kubectl run backend --image=nginx --restart=Never
kubectl run database --image=nginx --restart=Never

2. Declare labels for those Pods as follows:

- `frontend`: `env=prod`, `team=shiny`
- `backend`: `env=prod`, `team=legacy`, `app=v1.2.4`
- `database`: `env=prod`, `team=storage`

kubectl label pod frontend env=prod team=shiny
kubectl label pod backend env=prod team=legacy app=v.1.2.4
kubectl label pod database env=prod team=storage

3. Declare annotations for those Pods as follows:

- `frontend`: `contact=John Doe`, `commit=2d3mg3`
- `backend`: `contact=Mary Harris`

kubectl annotate pod frontend contact="John Doe" commit=2d3mg3
kubectl annotate pod backend contact="Mary Harris"

4. Render the list of all Pods and their labels.

kubectl get pods -A --show-labels

5. Use label selectors on the command line to query for all production Pods that belong to the teams `shiny` and `legacy`.

kubectl get pods -l 'team in (shiny, legacy)'

6. Remove the label `env` from the `backend` Pod and rerun the selection.

kubectl label pod backend env-
kubectl get pods -A --show-labels

7. Render the surrounding 3 lines of YAML of all Pods that have annotations.

kubectl get pods -o yaml | grep -C 3 -i annotations: 