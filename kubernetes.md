# Déploiement sur Kubernetes

Cette section décrit comment déployer l'application MERN sur un cluster Kubernetes (Docker Desktop, Minikube ou autre).

## Architecture Kubernetes

L’application est découpée en plusieurs objets Kubernetes :

| Composant | Type K8s | Rôle |
|---|---|---|
| MongoDB | Deployment + Service (ClusterIP) | Base de données interne |
| Serveur Node.js | Deployment + Service (ClusterIP) | API backend |
| Client React | Deployment + Service (NodePort) | Frontend accessible depuis le navigateur |
| ConfigMap | ConfigMap | Variables d'environnement |

Schéma simplifié :

React (NodePort 30002) --> server-service (5000) --> server pods
                                     |
                                     --> mongodb-service (27017) --> mongo pod

![Kubernetes deployment](k8s.PNG)

## Fichiers Kubernetes

Les principaux fichiers fournis sont :

- `app-configmap.yaml`
- `mongodb-deployment.yaml`
- `mongodb-service.yaml`
- `server-deployment.yaml`
- `server-service.yaml`
- `client-deployment.yaml`
- `client-service.yaml`

## Déploiement

1. Assurez-vous que votre cluster Kubernetes est prêt :

```bash
kubectl get nodes
```

2. Appliquez les fichiers dans l'ordre suivant :

- ConfigMap
  ```bash
  kubectl apply -f app-configmap.yaml
  ```
- MongoDB
  ```bash
  kubectl apply -f mongodb-deployment.yaml
  kubectl apply -f mongodb-service.yaml
  ```
- Serveur
  ```bash
  kubectl apply -f server-deployment.yaml
  kubectl apply -f server-service.yaml
  ```
- Client
  ```bash
  kubectl apply -f client-deployment.yaml
  kubectl apply -f client-service.yaml
  ```

3. Vérifiez que tout est en cours d'exécution :

```bash
kubectl get pods
kubectl get svc
```

4. Exemple de services attendus :

```
client-service      NodePort   3000:30002/TCP
server-service      ClusterIP  5000/TCP
mongodb-service     ClusterIP  27017/TCP
```

## Accès à l'application

- Frontend (React) : si `client-service` est exposé en `NodePort` (ex. `30002`) :

  `http://localhost:30002`

- Backend (Node.js) via port-forward si nécessaire :

  ```bash
  kubectl port-forward svc/server-service 9000:5000
  ```

  Le tunnel redirige `localhost:9000` vers `server-service:5000` dans le cluster.

## Nettoyage

Pour supprimer toutes les ressources créées :

```bash
kubectl delete -f app-configmap.yaml
kubectl delete -f mongodb-deployment.yaml
kubectl delete -f mongodb-service.yaml
kubectl delete -f server-deployment.yaml
kubectl delete -f server-service.yaml
kubectl delete -f client-deployment.yaml
kubectl delete -f client-service.yaml
```

Ou, si tous les manifests sont dans le même dossier :

```bash
kubectl delete -f .
```
