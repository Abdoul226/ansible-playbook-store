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

ansible-playbook-store/
├── inventory/
│ ├── hosts.ini # Inventaire des machines
│ └── group_vars/ # Variables globales par groupe
│
├── playbooks/
│ ├── system/ # Playbooks liés au système
│ ├── web/ # Serveurs web
│ ├── devops/ # CI/CD & outils DevOps
│ ├── db/ # Bases de données
│ ├── monitoring/ # Monitoring & logging
│ ├── security/ # Sécurité & durcissement
│ └── k8s/ # Kubernetes & containers
│
├── roles/ # Rôles Ansible réutilisables
├── docs/ # Documentation détaillée
└── Makefile # Commandes rapides

yaml
Copier le code

---

## 🛠️ Prérequis

- Ansible **2.13+**  
- Python **3.8+**

---

## 📥 Installation

Cloner le dépôt :

```bash
git clone https://github.com/TonPseudo/ansible-playbook-store.git
cd ansible-playbook-store
⚙️ Utilisation
Exemple 1 : Ping des serveurs
bash
Copier le code
ansible -i inventory/hosts.ini all -m ping
Exemple 2 : Installer Nginx
bash
Copier le code
ansible-playbook -i inventory/hosts.ini playbooks/web/install-nginx.yml
Exemple 3 : Créer un utilisateur administrateur
bash
Copier le code
ansible-playbook -i inventory/hosts.ini playbooks/system/create-user.yml -e "new_user=devops new_user_password=SuperSecret123"
📖 Documentation
Chaque catégorie possède sa doc dédiée :

docs/system.md → Playbooks système

docs/security.md → Playbooks sécurité

docs/web.md → Serveurs web

docs/devops.md → CI/CD & DevOps

docs/db.md → Bases de données

docs/monitoring.md → Monitoring

docs/k8s.md → Kubernetes

🧑‍💻 Contribution
Les contributions sont les bienvenues 🚀

Fork le projet

Crée une branche :

bash
Copier le code
git checkout -b feature/mon-playbook
Commit tes changements :

bash
Copier le code
git commit -m "Ajout playbook install-redis"
Push la branche :

bash
Copier le code
git push origin feature/mon-playbook
Crée une Pull Request

📜 Licence
Ce projet est sous licence MIT.
