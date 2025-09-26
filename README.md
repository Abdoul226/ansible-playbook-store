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

```perl
ansible-playbook-store/
│
├── inventory/
│   ├── hosts.ini         # Inventaire des machines
│   └── group_vars/       # Variables globales par groupe
│
├── playbooks/
│   ├── system/           # Playbooks liés au système
│   ├── web/              # Serveurs web
│   ├── devops/           # CI/CD & outils DevOps
│   ├── db/               # Bases de données
│   ├── monitoring/       # Monitoring & logging
│   ├── security/         # Sécurité & durcissement
│   └── k8s/              # Kubernetes & containers
│
├── roles/                # Rôles Ansible réutilisables
├── docs/                 # Documentation détaillée
└── Makefile              # Commandes rapides
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
git clone https://github.com/TonPseudo/ansible-playbook-store.git
cd ansible-playbook-store
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

- System : gestion utilisateurs, mise à jour, SSH, timezone…
- Web : Nginx, Apache, reverse proxy, load balancer…
- DevOps : Docker, Jenkins, GitLab Runner, ArgoCD...
- DB : MySQL, PostgreSQL, Redis, MongoDB…
- Monitoring : Prometheus, Grafana, Node Exporter, ELK…
- Security : firewall, Fail2ban, hardening SSH, VPN…
- Kubernetes : K3s, ingress-nginx, MetalLB, Longhorn…

---

## 🧑‍💻 Contribution
Les contributions sont les bienvenues 🚀

1. Fork le projet

2. Crée une branche (```bash git checkout -b feature/mon-playbook```)

3. Commit tes changements :

```bash
git commit -m "Ajout playbook install-redis"
```
4. Push la branche :

```bash
git push origin feature/mon-playbook
```
5. Crée une Pull Request

## 📜 Licence
Ce projet est sous licence MIT → libre d’utilisation et modification.
