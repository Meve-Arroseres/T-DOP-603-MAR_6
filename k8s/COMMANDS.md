# Kubernetes Commands

## Minikube

```bash
minikube ip
kubectl get node -o wide
kubectl get pods,svc,ingress -A
```

## PostgreSQL

Fichiers :
- `k8s/PostgreSql/postgres.volume.yaml`
- `k8s/PostgreSql/postgres.configmap.yaml`
- `k8s/PostgreSql/postgres.service.yaml`
- `k8s/PostgreSql/postgres.statefulset.yaml`

Commandes :

```bash
kubectl apply -f k8s/PostgreSql/postgres.volume.yaml
kubectl apply -f k8s/PostgreSql/postgres.configmap.yaml
kubectl apply -f k8s/PostgreSql/postgres.service.yaml
kubectl apply -f k8s/PostgreSql/postgres.statefulset.yaml
```

Ports :
- `postgres-service`: `5432`

## Redis

Fichiers :
- `k8s/Redis/redis.configmap.yaml`
- `k8s/Redis/redis.service.yaml`
- `k8s/Redis/redis.statefulset.yaml`

Commandes :

```bash
kubectl apply -f k8s/Redis/redis.configmap.yaml
kubectl apply -f k8s/Redis/redis.service.yaml
kubectl apply -f k8s/Redis/redis.statefulset.yaml
```

Ports :
- `redis`: `6379`

## Poll

Fichiers :
- `k8s/poll/poll.deployment.yaml`
- `k8s/poll/poll.service.yaml`
- `k8s/poll/poll.ingress.yaml`

Commandes :

```bash
kubectl apply -f k8s/poll/poll.deployment.yaml
kubectl apply -f k8s/poll/poll.service.yaml
kubectl apply -f k8s/poll/poll.ingress.yaml
```

Ports :
- conteneur `poll`: `80`
- service `poll`: `80`
- host ingress: `poll.dop.io`

## Result

Fichiers :
- `k8s/result/result.deployment.yaml`
- `k8s/result/result.service.yaml`
- `k8s/result/result.ingress.yaml`

Commandes :

```bash
kubectl apply -f k8s/result/result.deployment.yaml
kubectl apply -f k8s/result/result.service.yaml
kubectl apply -f k8s/result/result.ingress.yaml
```

Ports :
- conteneur `result`: `80`
- service `result`: `80`
- host ingress: `result.dop.io`

## Worker

Fichiers :
- `k8s/worker/worker.deployment.yaml`

Commandes :

```bash
kubectl apply -f k8s/worker/worker.deployment.yaml
```

Ports :
- aucun port exposé

## cAdvisor

Fichiers :
- `k8s/cadvisor/cadvisor.daemonset.yaml`
- `k8s/cadvisor/cadvisor.service.yaml`

Commandes :

```bash
kubectl apply -f k8s/cadvisor/cadvisor.daemonset.yaml
kubectl apply -f k8s/cadvisor/cadvisor.service.yaml
kubectl get pods -n kube-system | grep cadvisor
kubectl get svc -n kube-system cadvisor
```

Ports :
- conteneur `cadvisor`: `8080`
- service `cadvisor`: `8080`
- nodePort `cadvisor`: `30080`

Acces :

```bash
minikube service -n kube-system cadvisor --url
```

Note :
- avec Minikube en driver Docker, l'URL retournee par `minikube service --url` est la methode la plus fiable

## Traefik avec Helm

Fichier :
- `k8s/Traefik/traefik.values.yaml`

Commandes :

```bash
helm repo add traefik https://traefik.github.io/charts
helm repo update
cd k8s/Traefik
helm upgrade --install traefik traefik/traefik -n traefik --create-namespace -f traefik.values.yaml
```

Ports declares dans `traefik.values.yaml` :
- entrypoint `traefik`: `9000`
- entrypoint `web`: `80`
- entrypoint `websecure`: `443`

Checks utiles :

```bash
kubectl get pods -n traefik
kubectl get svc -n traefik
kubectl get ingress -A
kubectl get ingressclass
```

## Deploiement complet

```bash
kubectl apply -f k8s/PostgreSql/postgres.volume.yaml
kubectl apply -f k8s/PostgreSql/postgres.configmap.yaml
kubectl apply -f k8s/PostgreSql/postgres.service.yaml
kubectl apply -f k8s/PostgreSql/postgres.statefulset.yaml
kubectl apply -f k8s/Redis/redis.configmap.yaml
kubectl apply -f k8s/Redis/redis.service.yaml
kubectl apply -f k8s/Redis/redis.statefulset.yaml
kubectl apply -f k8s/poll/poll.deployment.yaml
kubectl apply -f k8s/poll/poll.service.yaml
kubectl apply -f k8s/poll/poll.ingress.yaml
kubectl apply -f k8s/result/result.deployment.yaml
kubectl apply -f k8s/result/result.service.yaml
kubectl apply -f k8s/result/result.ingress.yaml
kubectl apply -f k8s/worker/worker.deployment.yaml
kubectl apply -f k8s/cadvisor/cadvisor.daemonset.yaml
kubectl apply -f k8s/cadvisor/cadvisor.service.yaml
```
