# Example Voting App on Kubernetes

A multi-tier voting application deployed on Kubernetes. The project contains two sets of manifests for the same app: plain **Pods** and **Deployments**, each with the Services that connect the tiers.

The container images come from Docker's [example voting app](https://github.com/dockersamples/example-voting-app).

## Architecture

```
vote (web) --> redis --> worker --> postgres (db) <-- result (web)
```

| Component | Image | Service type | Port |
|-----------|-------|--------------|------|
| vote | `dockersamples/examplevotingapp_vote:before` | NodePort (pods) / LoadBalancer (deployment) | 31001 |
| result | `dockersamples/examplevotingapp_result:before` | NodePort (pods) / LoadBalancer (deployment) | 31002 |
| worker | `dockersamples/examplevotingapp_worker` | none (no incoming traffic) | n/a |
| redis | `redis:alpine` | ClusterIP | internal |
| db | `postgres:15-alpine` | ClusterIP | internal |

Redis and PostgreSQL are only reachable from inside the cluster. The vote and result apps are exposed on fixed node ports.

## Repository structure

```
.
├── pods/          # one Pod + Service manifest per component
└── deployment/    # one Deployment + Service manifest per component (1 replica each)
```

## How to run

Requirements: a Kubernetes cluster (for example [minikube](https://minikube.sigs.k8s.io/)) and `kubectl`.

Deploy **one** set only. Both sets use node ports 31001 and 31002, so they conflict if applied together.

```bash
# Option 1: Deployments
kubectl apply -f deployment/

# Option 2: Pods
kubectl apply -f pods/
```

Check that everything is running:

```bash
kubectl get pods
kubectl get services
```

Open the apps in a browser, using your cluster's node IP (`minikube ip` on minikube):

- Vote: `http://<node-ip>:31001`
- Result: `http://<node-ip>:31002`

To remove everything:

```bash
kubectl delete -f deployment/   # or: kubectl delete -f pods/
```

## What this project covers

- Running a multi-tier app on Kubernetes
- Pods compared with Deployments for the same workload
- Service types: ClusterIP for internal communication, NodePort and LoadBalancer for external access
- Connecting a web front end, a message queue (Redis), a worker, and a database (PostgreSQL)
