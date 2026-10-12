# Notes App on Kubernetes (minikube)

A small project that uses the main Kubernetes workload, networking, configuration and reliability topics together on a 3-node minikube cluster.

## What is in the project

| Topic | Where it is used |
|---|---|
| Deployment | `notes-web` (nginx web app, 3 replicas) |
| StatefulSet | `notes-db` (Postgres with its own 1Gi disk) |
| DaemonSet | `node-logger` (one Pod on every node) |
| Job | `notes-db-init` (creates the database table once) |
| CronJob | `notes-heartbeat` (adds a row every 2 minutes) |
| ClusterIP Service | `notes-db` (internal only) |
| NodePort Service | `notes-web` |
| LoadBalancer Service | `notes-web-lb` (using `minikube tunnel`) |
| ConfigMap | App settings and the web page |
| Secret | Database password |
| Startup, readiness, liveness probes | On the web Deployment |
| Requests and limits | On every container |
| Rolling update and rollback | Web image 1.25 to 1.26 and back |
| PodDisruptionBudget | `minAvailable: 2` for the web Pods |
| Bad probes demo | `demo/bad-probe-demo.yaml` |

## Folder structure

    notes-app/
    ├── namespace.yaml
    ├── config/      (configmap.yaml, secret.example.yaml)
    ├── database/    (service, statefulset, init job)
    ├── web/         (deployment, services, pdb)
    ├── jobs/        (daemonset, cronjob)
    └── demo/        (bad probe demo)

## How to run

    minikube start --nodes 3 --cpus 2 --memory 2048
    kubectl apply -f namespace.yaml
    kubectl config set-context --current --namespace=notes-app

    cp config/secret.example.yaml config/secret.yaml   # then set your own password
    kubectl apply -f config/
    kubectl apply -f database/
    kubectl apply -f web/
    kubectl apply -f jobs/

For the LoadBalancer IP, keep `minikube tunnel` running in another terminal.

## Demos

- Rolling update: `kubectl set image deployment/notes-web web=nginx:1.26`, then `kubectl rollout undo deployment/notes-web`
- Disruption budget: `kubectl drain <worker-node> --ignore-daemonsets --delete-emptydir-data`
- Bad probes: apply `demo/bad-probe-demo.yaml` and watch the RESTARTS column grow. The liveness probe calls `/healthz`, which does not exist, so healthy Pods are killed again and again. Change the path to `/` to fix it.

## Clean up

    kubectl delete namespace notes-app
