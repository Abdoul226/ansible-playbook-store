# ⚙️ Playbooks Système – Ansible Playbook Store
Cette section regroupe des playbooks utiles pour la gestion de base des serveurs Linux.
Ils couvrent les opérations essentielles : création d’utilisateurs, mise à jour des paquets, configuration système, etc.
---

## 1. Créer un utilisateur administrateur

### 📌 Objectif
Automatiser la création d’un utilisateur avec droits administrateur (`sudo`), accès SSH et clé publique configurée.

### Actions principales :

* Création de l’utilisateur avec mot de passe haché (SHA-512).
* Ajout au groupe sudo.
* Création du répertoire .ssh avec permissions correctes.
* Déploiement d’une clé publique dans authorized_keys.
* Ajout à sudoers pour exécuter des commandes sans mot de passe.

### Variables principales 
```perl
| Variable                   | Valeur par défaut        | Description |
|----------------------------|--------------------------|-------------|
| `new_user`                 | `devops`                | Nom du nouvel utilisateur |
| `new_user_password`        | `"SuperSecret123"`       | Mot de passe en clair (sera haché automatiquement) |
| `new_user_password_hash`   | généré par Ansible       | Hash SHA-512 du mot de passe |
| `src` (clé publique)       | `files/id_rsa.pub`       | Chemin vers la clé publique à déployer |
```

### ▶️ Exemples d’utilisation
* Créer un utilisateur aziz :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/system/create-user.yml -e "new_user=aziz"
```
* Créer un utilisateur avec mot de passe personnalisé :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/system/create-user.yml \
  -e "new_user=aziz new_user_password=UltraSecret456"
```

**Bonnes pratiques :**

- Toujours définir un mot de passe **fort** ou utiliser une **clé SSH**.
- Stocker les mots de passe dans **Ansible Vault** plutôt que dans des variables en clair.
- Tester la connexion avec le nouvel utilisateur avant de désactiver l’accès `root`.
- Ne jamais supprimer tous les accès administrateurs d’un serveur en production.
---

## 2. Mettre à jour les paquets (`update-packages.yml`)

**Objectif :**  
Effectuer une mise à jour complète des paquets sur Debian/Ubuntu et distributions RHEL-like (CentOS/Rocky/Alma), avec options d’`autoremove`, nettoyage de cache et **redémarrage conditionnel** si nécessaire (ex. mise à jour noyau).

### Variables

```perl
| Variable               | Défaut  | Description |
|------------------------|---------|-------------|
| `do_dist_upgrade`      | `true`  | Debian/Ubuntu : `full-upgrade` (gère changements de dépendances/kernels). Si `false`, fait un simple `upgrade`. |
| `do_autoremove`        | `true`  | Supprimer les paquets devenus inutiles (Debian/Ubuntu). |
| `do_autoclean`         | `true`  | Nettoyer l’ancien cache APT (Debian/Ubuntu). |
| `reboot_if_required`   | `true`  | Redémarrer automatiquement si nécessaire. |
| `reboot_timeout`       | `900`   | Timeout du reboot (secondes). |
| `cache_valid_time`     | `3600`  | Durée de validité du cache APT (évite les `apt update` trop fréquents). |
| `package_state_latest` | `true`  | RHEL : applique `state=latest` pour mettre à jour tous les paquets. |
```
### Exemples d’exécution

- Mise à jour standard + reboot si nécessaire :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/system/update-packages.yml
```
- Sans dist-upgrade ni reboot :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/system/update-packages.yml \
  -e "do_dist_upgrade=false reboot_if_required=false"
```
- Exécution dry-run (prévisualiser sans appliquer) :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/system/update-packages.yml --check --diff
```
**Notes & bonnes pratiques**

- **Debian/Ubuntu** : un fichier ```bash /var/run/reboot-required``` indique qu’un reboot est nécessaire.
- **RHEL-like** : ```bash needs-restarting -r``` (paquet ```bash yum-utils```/```bash dnf-plugins-core```) renvoie ```bash rc=1``` si un reboot est recommandé.
- Planifie ce playbook régulièrement via **cron/CI** (ex. hebdomadaire) et **surveille** les reboots automatiques en prod.
- Combine avec un **maintenance window** si tu actives ```bash reboot_if_required``` en environnement critique.


---

## 3. Configurer SSH (configure-ssh.yml)

**Objectif :**  
Appliquer une configuration **basique et sûre** de SSH : changer le **port**, désactiver le **login root** par mot de passe et **forcer la clé**.

### Variables
```perl
| Variable             | Défaut                | Description |
|---------------------|-----------------------|-------------|
| `ssh_port`          | `22`                  | Port SSH. Si modifié, ouvrir le firewall avant/après. |
| `permit_root_login` | `prohibit-password`   | `yes` / `no` / `prohibit-password` (reco) |
| `password_auth`     | `no`                  | `no` = clés SSH uniquement (recommandé) |
| `pubkey_auth`       | `yes`                 | Active l’auth par clé publique |
| `override_path`     | `/etc/ssh/sshd_config.d/20-basic.conf` | Fichier d’override géré par Ansible |
```

### Exemples d’exécution
- Configuration standard (port 22, root interdit en mot de passe, clés uniquement) :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/system/configure-ssh.yml
```
- Changer le port en 2222 :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/system/configure-ssh.yml \
  -e "ssh_port=2222"
```
- Autoriser (temporairement) l’authentification par mot de passe :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/system/configure-ssh.yml \
  -e "password_auth=yes"
```
**Bonnes pratiques**
- **Toujours** garder une session SSH ouverte pendant l’application.
- Vérifier que la **clé publique** de ton utilisateur admin est déjà en place.
- Si tu modifies ssh_port, **ouvre le firewall** correspondant (UFW/firewalld) et **mets à jour tes règles de sécurité**.
- Utiliser d’abord --check --diff en environnements sensibles.

---

## 4. Configurer timezone & locale (`set-timezone.yml`)

**Objectif :**  
Définir un **fuseau horaire cohérent** sur l’ensemble des serveurs pour garantir la bonne cohérence des logs, des backups et des tâches automatisées.

### Variables
```perl
| Variable   | Défaut          | Description |
|------------|-----------------|-------------|
| `timezone` | `Europe/Paris`  | Fuseau horaire à appliquer. Ex : `UTC`, `Africa/Ouagadougou`, `America/New_York` |
```
### Exemples d’exécution

- Appliquer le fuseau horaire **Europe/Paris** (par défaut) :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/system/set-timezone.yml```
- Appliquer le fuseau horaire **UTC** :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/system/set-timezone.yml -e "timezone=UTC"
```
- Appliquer le fuseau horaire **Africa/Ouagadougou** :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/system/set-timezone.yml -e "timezone=Africa/Ouagadougou"
```
**Vérification**

Après exécution, le playbook affiche la sortie de timedatectl.
Exemple attendu :
```yaml
Local time: sam. 2025-10-04 09:15:42 CEST
Universal time: sam. 2025-10-04 07:15:42 UTC
Time zone: Europe/Paris (CEST, +0200)
```
Bonnes pratiques

- Utiliser **UTC** dans les environnements distribués (multi-serveurs, cloud).
- Utiliser un timezone local (ex: `Europe/Paris`, `Africa/Ouagadougou`) pour les environnements orientés utilisateurs.

- Toujours vérifier avec `timedatectl` ou `date` après application.

---

## 5. install-tools.yml (à venir)

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

## 🧑‍💻 Bonnes pratiques générales

Toujours créer un utilisateur non-root pour l’administration quotidienne.

Conserver un accès console (VM/Hyperviseur) au cas où la config SSH est cassée.

Mettre régulièrement à jour les paquets pour appliquer les correctifs de sécurité.

Automatiser l’installation des outils indispensables sur chaque nouvelle machine.
