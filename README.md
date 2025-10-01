# 📘 Ansible Playbook Store

Un dépôt de **50 playbooks Ansible prêts à l’emploi**, organisés par thématique :  
- Système  
- Web  
- Bases de données  
- DevOps  
- Monitoring  
- Sécurité  
- Kubernetes  

Chaque playbook est **testé, documenté** et conçu pour être directement utilisable en production ou en lab.

---

## 🚀 Objectifs

- Fournir des playbooks simples et réutilisables.  
- Aider les débutants à gagner du temps et comprendre les bonnes pratiques.  
- Servir de boîte à outils pour les ingénieurs **DevOps & SysAdmins**.  

---

## 📂 Structure du projet

```graphql
ansible-playbook-store/
│
├── inventory/
│   ├── hosts.ini         # Inventaire des machines
│   └── group_vars/       # Variables globales par groupe
│
├── playbooks/
│   ├── system/           # Playbooks liés au système
│   │   ├── create-user.yml
│   │   ├── update-packages.yml
│   │   └── configure-ssh.yml
│   │
│   ├── web/              # Playbooks pour serveurs web
│   │   ├── install-nginx.yml
│   │   ├── install-apache.yml
│   │   └── deploy-php-app.yml
│   │
│   ├── devops/           # Playbooks CI/CD & outils DevOps
│   │   ├── install-docker.yml
│   │   ├── install-jenkins.yml
│   │   ├── install-gitlab-runner.yml
│   │   └── install-argo-cd.yml
│   │
│   ├── db/               # Bases de données
│   │   ├── install-mysql.yml
│   │   ├── install-postgresql.yml
│   │   └── backup-mysql.yml
│   │
│   ├── monitoring/       # Monitoring & observabilité
│   │   ├── install-prometheus.yml
│   │   ├── install-grafana.yml
│   │   └── install-node-exporter.yml
│   │
│   ├── security/         # Sécurité & durcissement
│   │   ├── configure-firewall.yml
│   │   ├── fail2ban.yml
│   │   └── ssh-hardening.yml
│   │
│   └── k8s/              # Kubernetes & containers
│       ├── install-k3s.yml
│       ├── deploy-nginx-ingress.yml
│       ├── setup-metallb.yml
│       └── deploy-app-on-k8s.yml
│
├── roles/                # Rôles Ansible réutilisables
│   ├── nginx/
│   ├── mysql/
│   ├── docker/
│   └── prometheus/
│
├── docs/
│   ├── README.md         # Documentation principale
│   ├── system.md         # Explication des playbooks système
│   ├── web.md            # Explication des playbooks web
│   ├── devops.md         # Explication des playbooks DevOps
│   ├── db.md             # Explication des playbooks BDD
│   ├── monitoring.md     # Explication monitoring
│   ├── security.md       # Explication sécurité
│   └── k8s.md            # Explication Kubernetes
│
└── Makefile              # Commandes rapides (ex: make ping, make deploy)
```

---

## 🛠️ Prérequis

- Ansible **2.13+**  
- Python **3.8+**
- Accès SSH aux hôtes cibles
- Inventaire correctement configuré (inventory/hosts.ini)
---

## 📥 Installation

Cloner le dépôt :

```bash
git clone  https://github.com/Abdoul226/ansible-playbook-store.git
```
---

## &#9881; Utilisation

### Exemple 1 : Ping des serveurs
```bash
ansible -i inventory/hosts.ini all -m ping
```

### Exemple 2 : Installer Nginx
```bash
ansible-playbook -i inventory/hosts.ini playbooks/web/install-nginx.yml
```

### Exemple 3 : Créer un utilisateur administrateur
```bash
ansible-playbook -i inventory/hosts.ini playbooks/system/create-user.yml -e "new_user=devops new_user_password=SuperSecret123"
```
---

## 📖 Documentation
Chaque catégorie possède sa doc dédiée :

- docs/system.md → Playbooks système
- docs/security.md → Playbooks sécurité
- docs/web.md → Serveurs web
- docs/devops.md → CI/CD & DevOps
- docs/db.md → Bases de données
- docs/monitoring.md → Monitoring
- docs/k8s.md → Kubernetes

---

## 🔥 Liste des playbooks
# 📖 Table des matières – 50 Playbooks Ansible

## 🔹 Catégorie 1 : Système (7)
1. [Créer un utilisateur administrateur](docs/system.md#créer-un-utilisateur-administrateur)  
2. [Mettre à jour les paquets](docs/system.md#mettre-à-jour-les-paquets)  
3. [Configurer SSH (désactiver root, changer port)](docs/system.md#configurer-ssh-désactiver-root-changer-port)  
4. [Configurer timezone & locale](docs/system.md#configurer-timezone--locale)  
5. [Gérer les partitions & montage](docs/system.md#gérer-les-partitions--montage)  
6. [Installer les utilitaires système (htop, curl…)](docs/system.md#installer-les-utilitaires-système-htop-curl)  
7. [Sauvegarder /etc](docs/system.md#sauvegarder-etc)  

---

## 🔹 Catégorie 2 : Web (7)
8. [Installer Nginx](docs/web.md#installer-nginx)  
9. [Installer Apache2](docs/web.md#installer-apache2)  
10. [Déployer une app PHP avec Apache](docs/web.md#déployer-une-app-php-avec-apache)  
11. [Déployer une app Python Flask avec Nginx + Gunicorn](docs/web.md#déployer-une-app-python-flask-avec-nginx--gunicorn)  
12. [Configurer HTTPS avec Let’s Encrypt](docs/web.md#configurer-https-avec-lets-encrypt)  
13. [Configurer un reverse proxy Nginx](docs/web.md#configurer-un-reverse-proxy-nginx)  
14. [Load balancer avec HAProxy](docs/web.md#load-balancer-avec-haproxy)  

---

## 🔹 Catégorie 3 : DevOps (10)
15. [Installer Docker](docs/devops.md#installer-docker)  
16. [Installer Docker Compose](docs/devops.md#installer-docker-compose)  
17. [Déployer une stack avec Compose (ex: WordPress + MySQL)](docs/devops.md#déployer-une-stack-avec-compose-ex-wordpress--mysql)  
18. [Installer Jenkins](docs/devops.md#installer-jenkins)  
19. [Installer GitLab Runner](docs/devops.md#installer-gitlab-runner)  
20. [Déployer une pipeline CI/CD basique](docs/devops.md#déployer-une-pipeline-cicd-basique)  
21. [Installer Ansible sur un hôte](docs/devops.md#installer-ansible-sur-un-hôte)  
22. [Installer ArgoCD sur Kubernetes](docs/devops.md#installer-argocd-sur-kubernetes)  
23. [Installer Helm](docs/devops.md#installer-helm)  
24. [Configurer un registre privé Docker](docs/devops.md#configurer-un-registre-privé-docker)  

---

## 🔹 Catégorie 4 : Bases de données (6)
25. [Installer MySQL](docs/db.md#installer-mysql)  
26. [Installer PostgreSQL](docs/db.md#installer-postgresql)  
27. [Sauvegarder une base MySQL (dump)](docs/db.md#sauvegarder-une-base-mysql-dump)  
28. [Restaurer une base MySQL](docs/db.md#restaurer-une-base-mysql)  
29. [Installer Redis](docs/db.md#installer-redis)  
30. [Installer MongoDB](docs/db.md#installer-mongodb)  

---

## 🔹 Catégorie 5 : Monitoring & Logging (7)
31. [Installer Prometheus](docs/monitoring.md#installer-prometheus)  
32. [Installer Grafana](docs/monitoring.md#installer-grafana)  
33. [Déployer Node Exporter](docs/monitoring.md#déployer-node-exporter)  
34. [Déployer cAdvisor](docs/monitoring.md#déployer-cadvisor)  
35. [Installer ELK (Elasticsearch + Logstash + Kibana)](docs/monitoring.md#installer-elk-elasticsearch--logstash--kibana)  
36. [Installer Filebeat](docs/monitoring.md#installer-filebeat)  
37. [Installer Loki + Promtail](docs/monitoring.md#installer-loki--promtail)  

---

## 🔹 Catégorie 6 : Sécurité (7)
38. [Configurer Firewall (UFW/iptables)](docs/security.md#configurer-firewall-ufwiptables)  
39. [Installer Fail2ban](docs/security.md#installer-fail2ban)  
40. [Activer auditd (surveillance système)](docs/security.md#activer-auditd-surveillance-système)  
41. [Mettre en place un IDS (Snort/Suricata)](docs/security.md#mettre-en-place-un-ids-snortsuricata)  
42. [Durcissement SSH (root, clés uniquement)](docs/security.md#durcissement-ssh-root-clés-uniquement)  
43. [Scanner vulnérabilités avec OpenVAS](docs/security.md#scanner-vulnérabilités-avec-openvas)  
44. [Mettre en place un VPN WireGuard](docs/security.md#mettre-en-place-un-vpn-wireguard)  

---

## 🔹 Catégorie 7 : Kubernetes & Containers (6)
45. [Installer K3s (cluster léger Kubernetes)](docs/k8s.md#installer-k3s-cluster-léger-kubernetes)  
46. [Déployer Nginx Ingress Controller](docs/k8s.md#déployer-nginx-ingress-controller)  
47. [Configurer MetalLB pour LoadBalancer on-premises](docs/k8s.md#configurer-metallb-pour-loadbalancer-on-premises)  
48. [Déployer une app (Flask ou NodeJS) sur K8s](docs/k8s.md#déployer-une-app-flask-ou-nodejs-sur-k8s)  
49. [Installer metrics-server](docs/k8s.md#installer-metrics-server)  
50. [Déployer Longhorn pour stockage persistant](docs/k8s.md#déployer-longhorn-pour-stockage-persistant)  

---

## 🧑‍💻 Contribution
Les contributions sont les bienvenues 🚀

1. Fork le projet
2. Crée une branche (```bash git checkout -b feature/mon-playbook```)
3. Commit tes changements ( ```bash git commit -m "Ajout playbook install-redis" ``` )
4. Push la branche ( ```bash git push origin feature/mon-playbook ```)
5. Crée une Pull Request

## 📜 Licence
Ce projet est sous licence MIT → libre d’utilisation et modification.
