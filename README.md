# T-DOP-603 MAR 6

Ce depot contient le deploiement Kubernetes d'une stack Vote App avec :
- `poll` : interface de vote
- `result` : interface de resultat
- `worker` : traitement asynchrone des votes
- `redis` : file/cache temporaire
- `postgres` : stockage persistant
- `traefik` : ingress controller
- `cAdvisor` : supervision des conteneurs

L'objectif de ce README est de permettre de refaire le projet proprement sur Minikube.

## Architecture

Flux logique :

1. l'utilisateur envoie un vote vers `poll`
2. `poll` envoie les donnees vers `redis`
3. `worker` lit les votes depuis `redis`
4. `worker` ecrit les resultats dans `postgres`
5. `result` lit les donnees depuis `postgres`
6. `traefik` expose `poll` et `result`
7. `cAdvisor` expose les metriques de conteneurs

## Arborescence

- [k8s/PostgreSql/postgres.volume.yaml](/home/mevar/Epitech/T-DOP-603-MAR_6/k8s/PostgreSql/postgres.volume.yaml:1)
- [k8s/PostgreSql/postgres.configmap.yaml](/home/mevar/Epitech/T-DOP-603-MAR_6/k8s/PostgreSql/postgres.configmap.yaml:1)
- [k8s/PostgreSql/postgres.service.yaml](/home/mevar/Epitech/T-DOP-603-MAR_6/k8s/PostgreSql/postgres.service.yaml:1)
- [k8s/PostgreSql/postgres.statefulset.yaml](/home/mevar/Epitech/T-DOP-603-MAR_6/k8s/PostgreSql/postgres.statefulset.yaml:1)
- [k8s/Redis/redis.configmap.yaml](/home/mevar/Epitech/T-DOP-603-MAR_6/k8s/Redis/redis.configmap.yaml:1)
- [k8s/Redis/redis.service.yaml](/home/mevar/Epitech/T-DOP-603-MAR_6/k8s/Redis/redis.service.yaml:1)
- [k8s/Redis/redis.statefulset.yaml](/home/mevar/Epitech/T-DOP-603-MAR_6/k8s/Redis/redis.statefulset.yaml:1)
- [k8s/poll/poll.deployment.yaml](/home/mevar/Epitech/T-DOP-603-MAR_6/k8s/poll/poll.deployment.yaml:1)
- [k8s/poll/poll.service.yaml](/home/mevar/Epitech/T-DOP-603-MAR_6/k8s/poll/poll.service.yaml:1)
- [k8s/poll/poll.ingress.yaml](/home/mevar/Epitech/T-DOP-603-MAR_6/k8s/poll/poll.ingress.yaml:1)
- [k8s/result/result.deployment.yaml](/home/mevar/Epitech/T-DOP-603-MAR_6/k8s/result/result.deployment.yaml:1)
- [k8s/result/result.service.yaml](/home/mevar/Epitech/T-DOP-603-MAR_6/k8s/result/result.service.yaml:1)
- [k8s/result/result.ingress.yaml](/home/mevar/Epitech/T-DOP-603-MAR_6/k8s/result/result.ingress.yaml:1)
- [k8s/worker/worker.deployment.yaml](/home/mevar/Epitech/T-DOP-603-MAR_6/k8s/worker/worker.deployment.yaml:1)
- [k8s/cadvisor/cadvisor.daemonset.yaml](/home/mevar/Epitech/T-DOP-603-MAR_6/k8s/cadvisor/cadvisor.daemonset.yaml:1)
- [k8s/cadvisor/cadvisor.service.yaml](/home/mevar/Epitech/T-DOP-603-MAR_6/k8s/cadvisor/cadvisor.service.yaml:1)
- [k8s/Traefik/traefik.values.yaml](/home/mevar/Epitech/T-DOP-603-MAR_6/k8s/Traefik/traefik.values.yaml:1)
- [k8s/COMMANDS.md](/home/mevar/Epitech/T-DOP-603-MAR_6/k8s/COMMANDS.md:1)

## Prerequis

Outils attendus :
- `kubectl`
- `minikube`
- `helm`
- `docker`

Cluster :
- un cluster Minikube actif
- contexte `kubectl` pointe sur Minikube

Verification :

```bash
kubectl config current-context
kubectl get node -o wide
minikube ip
```

## Demarrage de Minikube

Si Minikube n'est pas lance :

```bash
minikube start
```

Verification :

```bash
minikube status
kubectl get nodes
```

## Ressources Kubernetes utilisees

### PostgreSQL

Le dossier [k8s/PostgreSql](/home/mevar/Epitech/T-DOP-603-MAR_6/k8s/PostgreSql:1) contient :
- un `PersistentVolume`
- un `PersistentVolumeClaim`
- un `ConfigMap`
- un `Service`
- un `StatefulSet`

Role de chaque fichier :
- `postgres.volume.yaml` : cree le volume persistant et la claim `postgres-pvc`
- `postgres.configmap.yaml` : expose `database`, `database_host`, `database_port`
- `postgres.service.yaml` : cree le service interne `postgres-service:5432`
- `postgres.statefulset.yaml` : lance le conteneur `postgres:9.6`

Dependances :
- le `StatefulSet` depend du `ConfigMap`
- le `StatefulSet` depend des `Secrets`
- le `StatefulSet` depend du `PVC`

### Redis

Le dossier [k8s/Redis](/home/mevar/Epitech/T-DOP-603-MAR_6/k8s/Redis:1) contient :
- un `ConfigMap`
- un `Service`
- un `StatefulSet`

Role de chaque fichier :
- `redis.configmap.yaml` : expose `REDIS_HOST` et `REDIS_PORT`
- `redis.service.yaml` : cree le service `redis:6379`
- `redis.statefulset.yaml` : lance `redis:5.0`

### Poll

Le dossier [k8s/poll](/home/mevar/Epitech/T-DOP-603-MAR_6/k8s/poll:1) contient :
- un `Deployment`
- un `Service`
- un `Ingress`

Role :
- `poll.deployment.yaml` : lance 2 replicas de `epitechcontent/t-dop-600-poll:k8s`
- `poll.service.yaml` : expose `poll` en interne sur le port `80`
- `poll.ingress.yaml` : route `poll.dop.io` vers le service `poll`

Dependances :
- `poll` depend du `ConfigMap` Redis
- `poll` depend de Traefik pour l'acces externe

### Result

Le dossier [k8s/result](/home/mevar/Epitech/T-DOP-603-MAR_6/k8s/result:1) contient :
- un `Deployment`
- un `Service`
- un `Ingress`

Role :
- `result.deployment.yaml` : lance 2 replicas de `epitechcontent/t-dop-600-result:k8s`
- `result.service.yaml` : expose `result` en interne sur le port `80`
- `result.ingress.yaml` : route `result.dop.io` vers le service `result`

Dependances :
- `result` depend du `ConfigMap` PostgreSQL
- `result` depend des `Secrets` PostgreSQL
- `result` depend de Traefik pour l'acces externe

### Worker

Le dossier [k8s/worker](/home/mevar/Epitech/T-DOP-603-MAR_6/k8s/worker:1) contient :
- un `Deployment`

Role :
- `worker.deployment.yaml` : lance le worker `epitechcontent/t-dop-600-worker:k8s`

Dependances :
- `worker` depend du `ConfigMap` Redis
- `worker` depend du `ConfigMap` PostgreSQL
- `worker` depend des `Secrets` PostgreSQL

### Traefik

Le dossier [k8s/Traefik](/home/mevar/Epitech/T-DOP-603-MAR_6/k8s/Traefik:1) contient :
- le fichier Helm `traefik.values.yaml`

Role :
- installer Traefik proprement avec Helm
- surveiller les ingresses du namespace `default`
- exposer `poll` et `result`

### cAdvisor

Le dossier [k8s/cadvisor](/home/mevar/Epitech/T-DOP-603-MAR_6/k8s/cadvisor:1) contient :
- un `DaemonSet`
- un `Service`

Role :
- lancer un pod `cAdvisor` par noeud
- exposer l'interface web sur le port `8080`

## Secrets attendus

Les manifests utilisent deja ces secrets :
- `pg-secrets`
- `pg-secrets-username`

Sans eux, `postgres`, `result` et `worker` ne fonctionneront pas.

Creation minimale :

```bash
kubectl create secret generic pg-secrets --from-literal=password='<postgres_password>'
kubectl create secret generic pg-secrets-username --from-literal=username='postgres'
```

Verification :

```bash
kubectl get secrets
```

## Ordre de deploiement recommande

Il faut respecter les dependances.

1. creer les secrets PostgreSQL
2. deployer le stockage PostgreSQL
3. deployer PostgreSQL
4. deployer Redis
5. deployer Traefik
6. deployer `poll`
7. deployer `result`
8. deployer `worker`
9. deployer `cAdvisor`

## Deploiement detaille

### 1. Creer les secrets

```bash
kubectl create secret generic pg-secrets --from-literal=password='<postgres_password>'
kubectl create secret generic pg-secrets-username --from-literal=username='postgres'
```

### 2. Deployer PostgreSQL

```bash
kubectl apply -f k8s/PostgreSql/postgres.volume.yaml
kubectl apply -f k8s/PostgreSql/postgres.configmap.yaml
kubectl apply -f k8s/PostgreSql/postgres.service.yaml
kubectl apply -f k8s/PostgreSql/postgres.statefulset.yaml
```

Verification :

```bash
kubectl get pv,pvc
kubectl get pods,svc | grep postgres
```

### 3. Deployer Redis

```bash
kubectl apply -f k8s/Redis/redis.configmap.yaml
kubectl apply -f k8s/Redis/redis.service.yaml
kubectl apply -f k8s/Redis/redis.statefulset.yaml
```

Verification :

```bash
kubectl get pods,svc | grep redis
```

### 4. Installer Traefik

```bash
helm repo add traefik https://traefik.github.io/charts
helm repo update
cd k8s/Traefik
helm upgrade --install traefik traefik/traefik -n traefik --create-namespace -f traefik.values.yaml
cd ../..
```

Verification :

```bash
kubectl get pods -n traefik
kubectl get svc -n traefik
kubectl get ingressclass
```

Note :
- si un ancien Traefik manuel existe deja dans le cluster, il faut eviter de le garder en meme temps que celui installe avec Helm

### 5. Deployer Poll

```bash
kubectl apply -f k8s/poll/poll.deployment.yaml
kubectl apply -f k8s/poll/poll.service.yaml
kubectl apply -f k8s/poll/poll.ingress.yaml
```

Verification :

```bash
kubectl get pods,svc,ingress | grep poll
```

### 6. Deployer Result

```bash
kubectl apply -f k8s/result/result.deployment.yaml
kubectl apply -f k8s/result/result.service.yaml
kubectl apply -f k8s/result/result.ingress.yaml
```

Verification :

```bash
kubectl get pods,svc,ingress | grep result
```

### 7. Deployer Worker

```bash
kubectl apply -f k8s/worker/worker.deployment.yaml
```

Verification :

```bash
kubectl get pods | grep worker
```

### 8. Deployer cAdvisor

```bash
kubectl apply -f k8s/cadvisor/cadvisor.daemonset.yaml
kubectl apply -f k8s/cadvisor/cadvisor.service.yaml
```

Verification :

```bash
kubectl get pods -n kube-system | grep cadvisor
kubectl get svc -n kube-system cadvisor
kubectl get endpoints -n kube-system cadvisor
```

## Commandes de deploiement complet

```bash
kubectl create secret generic pg-secrets --from-literal=password='<postgres_password>'
kubectl create secret generic pg-secrets-username --from-literal=username='postgres'

kubectl apply -f k8s/PostgreSql/postgres.volume.yaml
kubectl apply -f k8s/PostgreSql/postgres.configmap.yaml
kubectl apply -f k8s/PostgreSql/postgres.service.yaml
kubectl apply -f k8s/PostgreSql/postgres.statefulset.yaml

kubectl apply -f k8s/Redis/redis.configmap.yaml
kubectl apply -f k8s/Redis/redis.service.yaml
kubectl apply -f k8s/Redis/redis.statefulset.yaml

helm repo add traefik https://traefik.github.io/charts
helm repo update
helm upgrade --install traefik traefik/traefik -n traefik --create-namespace -f k8s/Traefik/traefik.values.yaml

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

## Acces aux services

### IP Minikube

Pour connaitre l'IP du noeud :

```bash
minikube ip
kubectl get node -o wide
```

### Poll et Result via Traefik

Les ingresses definissent :
- `poll.dop.io`
- `result.dop.io`

Il faut donc ajouter une entree hosts sur la machine :

```text
<MINIKUBE_IP> poll.dop.io result.dop.io
```

Ensuite :
- `http://poll.dop.io`
- `http://result.dop.io`

Si Traefik n'est pas expose sur le port 80 de ta machine, teste avec `curl` sur l'IP ou sur l'URL donnee par Minikube selon ta config.

Tests utiles :

```bash
curl -H 'Host: poll.dop.io' http://<MINIKUBE_IP>
curl -H 'Host: result.dop.io' http://<MINIKUBE_IP>
```

### cAdvisor

Dans ce setup Minikube avec driver Docker, l'acces le plus fiable est :

```bash
minikube service -n kube-system cadvisor --url
```

Cette commande retourne une URL locale de type :

```text
http://127.0.0.1:<port>
```

Il faut laisser la commande active tant que tu veux utiliser cette URL.

## Ports utiles

Ports internes :
- `postgres-service` : `5432`
- `redis` : `6379`
- `poll` : `80`
- `result` : `80`
- `cadvisor` : `8080`

Ports/hosts exposes selon la config :
- `poll.dop.io`
- `result.dop.io`
- `cadvisor` `NodePort` : `30080`

Traefik selon [k8s/Traefik/traefik.values.yaml](/home/mevar/Epitech/T-DOP-603-MAR_6/k8s/Traefik/traefik.values.yaml:1) :
- entrypoint `traefik` : `9000`
- entrypoint `web` : `80`
- entrypoint `websecure` : `443`

## Verification globale

Commandes utiles pour verifier l'etat du projet :

```bash
kubectl get pods -A
kubectl get svc -A
kubectl get ingress -A
kubectl get ingressclass
kubectl get pv,pvc
```

Pour voir les erreurs :

```bash
kubectl describe pod <pod_name>
kubectl logs <pod_name>
kubectl get events -A --sort-by=.metadata.creationTimestamp
```

## Problemes frequents

### Secrets manquants

Symptomes :
- pods `result`, `worker` ou `postgres` en erreur

Verification :

```bash
kubectl get secrets
```

### Traefik deja present dans le cluster

Symptomes :
- conflit d'`IngressClass`
- install Helm qui echoue
- comportement incoherent du routage

Cause :
- un Traefik deploye a la main coexiste avec un Traefik Helm

### cAdvisor inaccessible depuis l'IP Minikube

Symptome :
- le pod tourne
- le service existe
- l'IP `NodePort` ne repond pas

Cause probable :
- setup Minikube Docker

Solution :

```bash
minikube service -n kube-system cadvisor --url
```

### Hosts non configures

Symptome :
- `poll` et `result` ne repondent pas par nom de domaine

Solution :
- ajouter `poll.dop.io` et `result.dop.io` dans le fichier hosts

## Notes finales

- le fichier de memo rapide est [k8s/COMMANDS.md](/home/mevar/Epitech/T-DOP-603-MAR_6/k8s/COMMANDS.md:1)
- ce README est le guide principal pour reconstruire le projet
