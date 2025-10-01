# ⚙️ Playbooks Système – Ansible Playbook Store
Cette section regroupe des playbooks utiles pour la gestion de base des serveurs Linux.
Ils couvrent les opérations essentielles : création d’utilisateurs, mise à jour des paquets, configuration système, etc.


## ### Créer un utilisateur administrateur

### 📌 Objectif
Automatiser la création d’un utilisateur avec droits administrateur (`sudo`), accès SSH et clé publique configurée.

---

#### Actions principales :

* Création de l’utilisateur avec mot de passe haché (SHA-512).
* Ajout au groupe sudo.
* Création du répertoire .ssh avec permissions correctes.
* Déploiement d’une clé publique dans authorized_keys.
* Ajout à sudoers pour exécuter des commandes sans mot de passe.

#### Variables principales 
```perl
| Variable                   | Valeur par défaut        | Description |
|----------------------------|--------------------------|-------------|
| `new_user`                 | `devops`                | Nom du nouvel utilisateur |
| `new_user_password`        | `"SuperSecret123"`       | Mot de passe en clair (sera haché automatiquement) |
| `new_user_password_hash`   | généré par Ansible       | Hash SHA-512 du mot de passe |
| `src` (clé publique)       | `files/id_rsa.pub`       | Chemin vers la clé publique à déployer |
```

## ▶️ Exemples d’utilisation
* Créer un utilisateur aziz :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/system/create-user.yml -e "new_user=aziz"
```
* Créer un utilisateur avec mot de passe personnalisé :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/system/create-user.yml \
  -e "new_user=aziz new_user_password=UltraSecret456"
```

##⚠️  Bonnes pratiques :

- Toujours définir un mot de passe fort ou utiliser une clé SSH.
- Stocker les mots de passe dans Ansible Vault plutôt que dans des variables en clair.
- Tester la connexion avec le nouvel utilisateur avant de désactiver l’accès root.
- Ne jamais supprimer tous les accès administrateurs d’un serveur en production.

2. update-packages.yml (à venir)

But : Mettre à jour les paquets système.

Actions principales :

apt upgrade pour Ubuntu/Debian.

yum update pour RedHat/CentOS.

Optionnel : nettoyage automatique (autoremove).

3. configure-ssh.yml (lié à la sécurité mais basique ici)

But : Configurer les paramètres SSH principaux.

Actions principales :

Désactiver l’accès root.

Forcer l’authentification par clé SSH.

Définir un port personnalisé.

4. set-timezone.yml (à venir)

But : Configurer le fuseau horaire du serveur.

Exemple :

- name: Définir le fuseau horaire
  hosts: all
  become: yes
  tasks:
    - name: Configurer timezone
      timezone:
        name: "Europe/Paris"

5. install-tools.yml (à venir)

But : Installer des utilitaires essentiels (curl, htop, git, vim…).

Exemple :

- name: Installer utilitaires
  hosts: all
  become: yes
  tasks:
    - name: Installer paquets
      package:
        name: [git, curl, htop, vim]
        state: present

⚙️ Variables personnalisables (exemple create-user.yml)
Variable	Valeur par défaut	Description
new_user	devops	Nom du nouvel utilisateur
new_user_password	"SuperSecret123"	Mot de passe en clair (sera haché automatiquement)
new_user_password_hash	généré par Ansible	Hash SHA-512 du mot de passe
src (clé publique)	files/id_rsa.pub	Fichier contenant la clé publique
▶️ Exemples d’utilisation
Créer un utilisateur aziz
ansible-playbook -i inventory/hosts.ini playbooks/system/create-user.yml -e "new_user=aziz"

Créer un utilisateur avec mot de passe personnalisé
ansible-playbook -i inventory/hosts.ini playbooks/system/create-user.yml \
  -e "new_user=aziz new_user_password=UltraSecret456"

Mettre à jour les paquets système
ansible-playbook -i inventory/hosts.ini playbooks/system/update-packages.yml

🧑‍💻 Bonnes pratiques générales

Toujours créer un utilisateur non-root pour l’administration quotidienne.

Conserver un accès console (VM/Hyperviseur) au cas où la config SSH est cassée.

Mettre régulièrement à jour les paquets pour appliquer les correctifs de sécurité.

Automatiser l’installation des outils indispensables sur chaque nouvelle machine.
