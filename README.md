# Kubernetes-partie-2-

# Guide d'installation et d'utilisation d'ArgoCD

Ce guide explique comment installer **ArgoCD**, se connecter à l’interface web, déployer une application via la CLI, tester le **HPA (Horizontal Pod Autoscaler)** et générer du trafic avec Istio.

---

## Prérequis

- Cluster Kubernetes fonctionnel (minikube, kind, AWS, etc.)  
- `kubectl` installé et configuré  
- `argocd` CLI installé sur votre machine  

---

## Installation d’ArgoCD

Créer le namespace dédié :

```bash
kubectl create namespace argocd

kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
kubectl get svc argocd-server -n argocd -o=jsonpath='{.status.loadBalancer.ingress[0].ip}'
kubectl port-forward svc/argocd-server -n argocd 8080:443
````
## Accès à l’interface web
```bash
https://localhost:8080
````

- Utilisateur : admin

- Mot de passe : <mot de passe extrait> (voir étape suivante)
  
```bash
kubectl -n argo get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo             
````

## Déployer une application via CLI

```bash
argocd login localhost:8080 --username admin --password <mot de passe> --insecure
kubectl config set-context --current --namespace=argocd

argocd app create hello-kubernetes \
  --repo https://github.com/paulbouwer/hello-kubernetes \
  --path . \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default  
````
<img width="945" height="467" alt="image" src="https://github.com/user-attachments/assets/fe86f948-ab04-4389-b27d-b6f292cb490d" />


## Tester le HPA (Horizontal Pod Autoscaler)
```bash
kubectl autoscale deployment php-apache --min=1 --max=10 --cpu-percent=50
````
<img width="945" height="70" alt="image" src="https://github.com/user-attachments/assets/d1cde212-7f76-46e8-913a-bcfc41a425bd" />


## Générer du load pour tester le HPA
```bash
kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"
````

<img width="945" height="167" alt="image" src="https://github.com/user-attachments/assets/700cf2d1-eff9-4148-8040-a31a9a1e443f" />

## Mettre fin au load 
Une fois le load terminé, les ressources du cluster redeviennent stables.

<img width="945" height="195" alt="image" src="https://github.com/user-attachments/assets/822ec3fc-646d-41cb-a361-c05a3f686ffe" />


## Service Mesh avec Istio

Connecter l’application via Istio.

Observer le trafic généré par l’application

<img width="945" height="467" alt="image" src="https://github.com/user-attachments/assets/6d2f453d-9fed-4977-8b3f-491fce5fc419" />
<img width="945" height="451" alt="image" src="https://github.com/user-attachments/assets/418ee801-7f84-4b40-aa0e-91ac0d3c8157" />

<img width="945" height="466" alt="image" src="https://github.com/user-attachments/assets/fd4342ed-030c-4ed3-bd11-461052e8eb8c" />


## Application onlineboutique
l'application a bien été déployé

<img width="945" height="650" alt="image" src="https://github.com/user-attachments/assets/168eb61c-3078-4b5d-b01c-c31a1ffed742" />



