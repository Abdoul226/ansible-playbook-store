📘 Ansible Playbook Store

Un dépôt de 50 playbooks Ansible prêts à l’emploi, organisés par thématique (système, web, bases de données, DevOps, monitoring, sécurité, Kubernetes).
Chaque playbook est testé, documenté et conçu pour être directement utilisable en production ou en lab.

🚀 Objectifs

Fournir des playbooks simples et réutilisables.

Aider les débutants à gagner du temps et comprendre les bonnes pratiques.

Servir de boîte à outils pour les ingénieurs DevOps & SysAdmins.

📂 Structure du projet
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

🛠️ Prérequis

Ansible 2.13+

Python 3.8+

Accès SSH aux hôtes cibles

Inventaire correctement configuré (inventory/hosts.ini)

📥 Installation

Clone le dépôt :

git clone https://github.com/ton-compte/ansible-playbook-store.git
cd ansible-playbook-store

⚙️ Utilisation
Exemple 1 : Ping des serveurs
ansible -i inventory/hosts.ini all -m ping

Exemple 2 : Lancer un playbook

Installer Nginx :

ansible-playbook -i inventory/hosts.ini playbooks/web/install-nginx.yml


Créer un utilisateur administrateur :

ansible-playbook -i inventory/hosts.ini playbooks/system/create-user.yml -e "new_user=devops new_user_password=SuperSecret123"

📖 Documentation

Chaque catégorie a une documentation dédiée dans le dossier docs/.
Exemple :

docs/system.md → Playbooks système

docs/web.md → Serveurs web

docs/db.md → Bases de données

🔥 Liste des playbooks

System : gestion utilisateurs, mise à jour, SSH, timezone…

Web : Nginx, Apache, reverse proxy, load balancer…

DevOps : Docker, Jenkins, GitLab Runner, ArgoCD…

DB : MySQL, PostgreSQL, Redis, MongoDB…

Monitoring : Prometheus, Grafana, Node Exporter, ELK…

Security : firewall, Fail2ban, hardening SSH, VPN…

Kubernetes : K3s, ingress-nginx, MetalLB, Longhorn…

🧑‍💻 Contribution

Les contributions sont les bienvenues 🚀

Fork le projet

Crée une branche (git checkout -b feature/mon-playbook)

Commit tes changements (git commit -m "Ajout playbook install-redis")

Push (git push origin feature/mon-playbook)

Crée une Pull Request

📜 Licence

Ce projet est publié sous licence MIT → libre d’utilisation et modification.
