# 🚀 Infrastructure GitOps Production

Infrastructure complète d'entreprise de développement déployée via **GitOps pur** - **Aucun script nécessaire** !

## 🏗️ Architecture

```
🔄 ArgoCD (GitOps Controller)
├── 🔐 Keycloak (Authentication SSO)
├── 🔒 HashiCorp Vault (Secrets Management)
├── 📊 Prometheus + Grafana (Monitoring)
├── 📝 Loki (Log Aggregation)
├── 🔑 Bitwarden (Password Manager)
├── 🛡️ Coraza WAF (Web Application Firewall)
├── 🔥 OPNsense (Network Firewall)
├── 🦊 GitLab Enterprise (Git + CI/CD)
├── 💾 Backup System (Automated S3 Backups)
└── 🌐 Traefik (Load Balancer - K3s intégré)
```

## 🎯 Déploiement 100% GitOps

### Prérequis

- **Cluster K3s** avec Traefik intégré
- **kubectl** configuré
- **helm** installé
- **Domaine DNS** pointé vers votre cluster

### 🚀 Déploiement en 3 étapes

#### 1️⃣ Configuration des secrets

Éditez `infra/manifests/gitlab-secrets.yaml` et remplacez les valeurs :

```bash
# Exemple pour encoder vos secrets
echo -n "votre-mot-de-passe" | base64

# Clés SSH pour GitLab
ssh-keygen -t rsa -b 4096 -f gitlab_rsa -N ""
ssh-keygen -t ed25519 -f gitlab_ed25519 -N ""
```

#### 2️⃣ Installation initiale (une seule fois)

```bash
# Cert-Manager pour SSL automatique
helm repo add jetstack https://charts.jetstack.io
helm repo update
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.3/cert-manager.crds.yaml
helm install cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace --version v1.13.3

# Créer ClusterIssuer Let's Encrypt
kubectl apply -f - <<EOF
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: clement@clementpnn.com
    privateKeySecretRef:
      name: letsencrypt-prod-key
    solvers:
    - http01:
        ingress:
          class: traefik
EOF

# ArgoCD
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
kubectl create namespace argocd
helm install argocd argo/argo-cd --namespace argocd --set configs.params.server.insecure=true

# Attendre qu'ArgoCD soit prêt
kubectl wait --for=condition=available deployment/argocd-server -n argocd --timeout=300s
```

#### 3️⃣ Déploiement automatique

```bash
# Appliquer l'application root - ArgoCD fait TOUT le reste !
kubectl apply -f infra/bootstrap/root-app.yaml

# C'est tout ! 🎉
```

## 📊 Surveillance du déploiement

```bash
# Voir toutes les applications ArgoCD
kubectl get applications -n argocd

# Voir le statut en temps réel
kubectl get applications -n argocd --watch

# Voir les pods de chaque namespace
kubectl get pods --all-namespaces
```

## 🌐 Interfaces Web

Une fois déployé (15-20 minutes), accédez à :

| Service | URL | Description |
|---------|-----|-------------|
| 🔄 **ArgoCD** | https://argocd.clementpnn.com | GitOps Dashboard |
| 🔐 **Keycloak** | https://auth.clementpnn.com | SSO Authentication |
| 📊 **Grafana** | https://grafana.clementpnn.com | Monitoring Dashboard |
| 📈 **Prometheus** | https://prometheus.clementpnn.com | Metrics Collection |
| 📝 **Loki** | https://loki.clementpnn.com | Log Aggregation |
| 🔒 **Vault** | https://vault.clementpnn.com | Secrets Management |
| 🔑 **Bitwarden** | https://passwords.clementpnn.com | Password Manager |
| 🔥 **OPNsense** | https://firewall.clementpnn.com | Network Firewall |
| 🦊 **GitLab** | https://gitlab.clementpnn.com | Git + CI/CD Platform |
| 🐳 **Registry** | https://registry.clementpnn.com | Docker Registry |
| 📦 **MinIO** | https://minio.clementpnn.com | Object Storage |
| 📄 **Pages** | https://pages.clementpnn.com | Static Sites |

## 📁 Structure du projet

```
infra/
├── apps/                    # Applications ArgoCD
│   ├── backup.yaml         # Système de backup automatisé
│   ├── bitwarden.yaml      # Gestionnaire de mots de passe
│   ├── coraza.yaml         # Web Application Firewall
│   ├── gitlab.yaml         # GitLab Enterprise
│   ├── keycloak.yaml       # Authentification SSO
│   ├── loki.yaml           # Agrégation des logs
│   ├── opnsense.yaml       # Firewall réseau
│   ├── prometheus.yaml     # Monitoring + Grafana
│   └── vault.yaml          # Gestion des secrets
│
├── bootstrap/
│   └── root-app.yaml       # Application racine ArgoCD
│
├── charts/                 # Charts Helm personnalisés
│   ├── backup/            # Chart backup system
│   ├── bitwarden/         # Chart Bitwarden
│   ├── coraza/            # Chart Coraza WAF
│   ├── gitlab/            # Chart GitLab Enterprise
│   ├── keycloak/          # Chart Keycloak
│   ├── loki/              # Chart Loki
│   ├── opnsense/          # Chart OPNsense
│   ├── prometheus/        # Chart Prometheus stack
│   └── vault/             # Chart HashiCorp Vault
│
└── manifests/             # Manifests Kubernetes purs
    ├── gitlab-certificates.yaml    # Certificats SSL GitLab
    ├── gitlab-ingressroute.yaml   # Routes Traefik GitLab
    ├── gitlab-secrets.yaml        # Secrets GitLab
    ├── keycloak-ingressroute.yaml # Routes Traefik Keycloak
    ├── monitoring-ingressroute.yaml # Routes Traefik Monitoring
    ├── secrets-ingressroute.yaml  # Routes Traefik Vault
    └── security-ingressroute.yaml # Routes Traefik Sécurité
```

## 🔐 Ordre de déploiement (Sync Waves)

ArgoCD déploie automatiquement dans cet ordre :

```
Wave 0: 🔐 Keycloak (Authentication first)
Wave 1: 🔒 Vault, 📊 Prometheus, 📝 Loki, 🔑 Bitwarden, 🛡️ Coraza, 🔥 OPNsense
Wave 2: 🦊 GitLab, 💾 Backup System
```

## 🛠️ Configuration post-déploiement

### Récupérer les mots de passe

```bash
# ArgoCD admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

# GitLab root password
kubectl get secret gitlab-initial-root-password -n gitlab -o jsonpath='{.data.password}' | base64 -d

# Grafana admin password
kubectl get secret prometheus-grafana -n monitoring -o jsonpath='{.data.admin-password}' | base64 -d
```

### Configuration Keycloak

1. Accédez à https://auth.clementpnn.com
2. Créez le realm `infrastructure`
3. Configurez les clients OAuth pour chaque service
4. Ajoutez vos utilisateurs et groupes

### Premier commit GitLab

```bash
# Cloner votre nouveau GitLab
git clone https://gitlab.clementpnn.com/root/mon-projet.git
cd mon-projet

# Ajouter du contenu
echo "# Mon Premier Projet" > README.md
git add README.md
git commit -m "Initial commit"
git push origin main
```

## 🔧 Maintenance

### Mettre à jour une application

```bash
# Modifier le chart ou les valeurs
vim infra/charts/gitlab/values.yaml

# Commit et push - ArgoCD sync automatiquement
git add infra/
git commit -m "feat: update GitLab configuration"
git push origin main
```

### Voir les logs

```bash
# Logs ArgoCD
kubectl logs -n argocd deployment/argocd-server

# Logs GitLab
kubectl logs -n gitlab deployment/gitlab-webservice-default

# Logs Grafana
kubectl logs -n monitoring deployment/prometheus-grafana
```

### Redémarrer une application

```bash
# Via ArgoCD UI ou CLI
argocd app sync gitlab

# Ou via kubectl
kubectl rollout restart deployment/gitlab-webservice-default -n gitlab
```

## 🚨 Troubleshooting

### Application en erreur

```bash
# Voir le détail de l'erreur
kubectl describe application gitlab -n argocd

# Forcer la synchronisation
kubectl patch application gitlab -n argocd -p '{"spec":{"syncPolicy":{"automated":null}}}' --type merge
kubectl patch application gitlab -n argocd -p '{"operation":{"initiatedBy":{"username":"admin"},"sync":{}}}' --type merge
```

### Certificat SSL en erreur

```bash
# Vérifier cert-manager
kubectl get certificates --all-namespaces
kubectl describe certificate gitlab-tls -n gitlab

# Forcer le renouvellement
kubectl delete certificate gitlab-tls -n gitlab
kubectl apply -f infra/manifests/gitlab-certificates.yaml
```

### Service inaccessible

```bash
# Vérifier Traefik
kubectl get ingressroutes --all-namespaces
kubectl logs -n kube-system deployment/traefik

# Vérifier DNS
nslookup gitlab.clementpnn.com
```

## 🎯 Sécurité

- **SSL/TLS** : Certificats Let's Encrypt automatiques
- **WAF** : Coraza protection applicative
- **Firewall** : OPNsense protection réseau
- **SSO** : Authentification centralisée Keycloak
- **Secrets** : Chiffrement HashiCorp Vault
- **Backup** : Sauvegarde chiffrée S3
- **Network Policies** : Isolation réseau Kubernetes
- **RBAC** : Permissions granulaires

## 📈 Monitoring

- **Métriques** : Prometheus + Grafana
- **Logs** : Loki + Promtail
- **Alertes** : AlertManager + notifications
- **Dashboards** : Grafana préconfigurés
- **Backup monitoring** : Alertes échecs sauvegarde

## 🔄 CI/CD

- **GitLab Runners** : Exécution Kubernetes
- **Container Registry** : Images Docker privées
- **Packages Registry** : NPM, Maven, etc.
- **Security Scanning** : Analyse automatique
- **GitLab Pages** : Déploiement sites statiques

## 📞 Support

- **Documentation** : https://docs.gitlab.com
- **Monitoring** : https://grafana.clementpnn.com
- **Issues** : https://gitlab.clementpnn.com/infrastructure/issues

---

## 🎉 Infrastructure as Code

**Tout est dans Git = Tout est reproductible !**

- ✅ Zero script bash
- ✅ 100% déclaratif
- ✅ GitOps pur
- ✅ Self-healing
- ✅ Rollback automatique
- ✅ Production-ready

**Prêt à déployer ? Let's Go ! 🚀**
