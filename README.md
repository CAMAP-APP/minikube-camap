# Minikube camap

## Install minikube

Documentation:
- https://minikube.sigs.k8s.io/docs/start/?arch=%2Fwindows%2Fx86-64%2Fstable%2F.exe+download

### Exemple d'une installation sous win11 avec docker desktop et winget

## Installer minikube

`winget install Kubernetes.minikube`

## Démarrer minikube

Avec docker par exemple (sinon cf https://minikube.sigs.k8s.io/docs/drivers/)

~~minikube start --driver=docker --ports=80:80 --ports=443:443~~
~~minikube tunnel~~

- `minikube start --driver=docker`
- `minikube addons enable ingress`

## Launch dashboard

- `minikube addons enable metrics-server`

- `minikube dashboard`

## 🛠 Installer Argo CD

```bash
kubectl create namespace argocd

kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml --server-side --force-conflicts
```

### 🔐 Récupérer le mot de passe admin

```bash
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d
```

---

## 🌐 Accéder à l’interface Argo CD

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Puis aller sur [https://localhost:8080](https://localhost:8080)

Login : `admin`  
Password : (cf. commande précédente)

## 🔐 Ajouter le dépôt Git à Argo CD

### Depuis l’interface Argo CD :

- `Settings` > `Repositories` > `Connect Repo using HTTPS`
- `Repository URL` : `https://github.com/CAMAP-APP/minikube-camap.git`

---

## 📂 Structure du dépôt Git

```
.
├── root-app.yaml              # Déclaration de l'application "App of Apps"
├── apps/                      # Contient les applications ArgoCD et objets K8s
└── deployments/              # Contient les charts Helm personnalisés
```

---

## 📦 root-app.yaml — App of Apps

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: apps-root
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/CAMAP-APP/minikube-camap.git
    targetRevision: HEAD
    path: apps
    directory:
      recurse: true
  destination:
    server: https://kubernetes.default.svc
    namespace: argocd
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

Appliquer avec :

```bash
kubectl apply -f root-app.yaml -n argocd
```

---

## Installer Camap

Prérequis Minikube à mettre en place

> Entrées /etc/hosts (IP à récupérer via minikube ip ou minikube tunnel)
> Sous Windows : C:\Windows\System32\drivers\etc\hosts
> `127.0.0.1   camap.local api.camap.local mailpit.local`

### Secrets à créer manuellement:

`kubectl create ns camap`

#### MySQL

```sh
kubectl create secret generic camap-mysql -n camap \
  --from-literal=mysql-root-password=rootpass \
  --from-literal=mysql-database=camap \
  --from-literal=mysql-username=camap \
  --from-literal=mysql-password=camappass
```

#### Runtime

```sh
kubectl create secret generic camap-runtime -n camap \
  --from-literal=CAMAP_KEY=une_cle_aleatoire_32chars \
  --from-literal=JWT_ACCESS_TOKEN_SECRET=secret_access \
  --from-literal=JWT_REFRESH_TOKEN_SECRET=secret_refresh \
  --from-literal=SMTP_AUTH_USER="" \
  --from-literal=SMTP_AUTH_PASS=""
```