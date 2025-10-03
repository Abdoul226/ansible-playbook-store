# ⚙️ Playbooks Système – Ansible Playbook Store
Cette section regroupe des playbooks utiles pour la gestion de base des serveurs Linux.
Ils couvrent les opérations essentielles : création d’utilisateurs, mise à jour des paquets, configuration système, etc.
---

## 1. Créer un utilisateur administrateur
*(voir [playbooks/system/create-user.yml](../playbooks/system/create-user.yml))*

** 📌 Objectif**
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
*(voir [playbooks/system/update-packages.yml](../playbooks/system/update-packages.yml))*

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
*(voir [playbooks/system/configure-ssh.yml](../playbooks/system/configure-ssh.yml))*

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
*(voir [playbooks/system/set-timezone.yml](../playbooks/system/set-timezone.yml))*

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
ansible-playbook -i inventory/hosts.ini playbooks/system/set-timezone.yml
```
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

## 5. Installer les utilitaires système (`install-tools.yml`)
*(voir [playbooks/system/install-tools.yml](../playbooks/system/install-tools.yml))*

**Objectif :**  
Installer rapidement un ensemble d’outils de base (git, curl, htop, vim, etc.) sur Debian/Ubuntu et distributions RHEL-like.

### Variables
```perl
| Variable              | Défaut                                                                 | Description |
|-----------------------|------------------------------------------------------------------------|-------------|
| `common_packages`     | `['git','curl','htop','vim','unzip','tar','wget','net-tools','lsof','tree']` | Liste standard d’outils installés sur toutes les machines |
| `extra_packages`      | `[]`                                                                   | Paquets additionnels à installer (passés en `-e`) |
| `apt_update_cache`    | `true`                                                                 | Mettre à jour le cache APT avant installation (Debian/Ubuntu) |
| `apt_cache_valid_time`| `3600`                                                                 | Validité du cache APT (secondes) |
```
### Exemples d’exécution

- Installation standard :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/system/install-tools.yml
```
- Ajouter des paquets supplémentaires :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/system/install-tools.yml \
  -e "extra_packages=['jq','ncdu']"

```
- Désactiver l’`apt update` préalable (si déjà fait par ailleurs) :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/system/install-tools.yml \
  -e "apt_update_cache=false"
```

**Bonnes pratiques**

- Centraliser des paquets spécifiques par **environnement** via `group_vars`/ (ex: `group_vars/webservers.yml`).
- Ajouter `jq`, `ncdu`, `tmux`, `rsync`, `ripgrep (rg)` selon tes habitudes.
- Coupler avec `update-packages.yml` dans tes pipelines de préparation d’hôtes.
---

## 6. Sauvegarder /etc (`backup-etc.yml`)
*(voir [playbooks/system/backup-etc.yml](../playbooks/system/backup-etc.yml))*

**Objectif :**  
Créer une **archive compressée** de `/etc` avec un **timestamp** (ex: `etc-20251003-141530.tar.gz`) et mettre en place une **rotation automatique** (on garde seulement les `keep_last` dernières sauvegardes).

### Variables
```perl
| Variable        | Défaut              | Description |
|-----------------|---------------------|-------------|
| `backup_dir`    | `/var/backups/etc`  | Répertoire de destination des archives |
| `backup_prefix` | `etc`               | Préfixe de nommage des fichiers |
| `keep_last`     | `7`                 | Nombre d’archives à conserver |
| `exclude_paths` | voir playbook       | Chemins exclus de l’archive (ex: clés privées) |
```
> **Format de fichier :** `{{ backup_prefix }}-YYYYMMDD-HHMMSS.tar.gz`

### Exemples d’exécution

- Sauvegarde standard + rotation (garde 7 archives) :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/system/backup-etc.yml
```
- Changer le dossier et garder 14 archives :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/system/backup-etc.yml \
  -e "backup_dir=/srv/backups/etc keep_last=14"
```
- Exclure d’autres chemins sensibles :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/system/backup-etc.yml \
  -e "exclude_paths=['/etc/ssl/private','/etc/shadow']"
```
**Bonnes pratiques**

- **Sécurité** : l’archive est en `0640` et stockée sous `root:root`. Évite d’y inclure des secrets comme `/etc/shadow` si tu exportes hors de l’hôte.

- **Stockage externe** : pour plus de résilience, réplique ensuite `{{ backup_dir }}` vers un autre disque/serveur (rsync, NFS, sauvegarde cloud).

- **Planification** : exécute ce playbook via **cron/CI** (quotidien, hebdo) selon tes besoins.

- **Restauration** : extraire avec `tar -xzf etc-YYYYMMDD-HHMMSS.tar.gz -C /` (⚠️ à faire prudemment, idéalement fichier par fichier).

## 7. Gérer les partitions & montage (`partition-mount.yml`)
*(voir [playbooks/system/partition-mount.yml](../playbooks/system/partition-mount.yml))*

**Objectif :**  
Créer une **partition** sur un disque, y appliquer un **système de fichiers** (ext4/xfs), **monter** sur un répertoire et **persister** dans `fstab`.  
Prend aussi en charge le **démontage** et la **suppression** de la partition (`state: absent`).

### Variables
```perl
| Variable        | Défaut        | Description |
|----------------|---------------|-------------|
| `device`       | `/dev/sdb`    | Disque ciblé (ex: `/dev/sdb`, `/dev/nvme0n1`) |
| `part_number`  | `1`           | Numéro de partition |
| `part_start`   | `1MiB`        | Début de la partition |
| `part_end`     | `100%`        | Fin (100% = tout le disque restant) |
| `part_label`   | `gpt`         | Table de partition (`msdos` ou `gpt`) |
| `fs_type`      | `ext4`        | Système de fichiers (`ext4`, `xfs`, etc.) |
| `fs_label`     | `data`        | Label du FS (optionnel) |
| `mount_point`  | `/data`       | Répertoire de montage |
| `mount_options`| `defaults,noatime` | Options fstab |
| `owner`/`group`| `root`        | Propriétaire/Group du point de montage |
| `mode`         | `0755`        | Permissions du répertoire |
| `state`        | `present`     | `present` → créer/monter ; `absent` → démonter/supprimer |
```
> 💡 Support **nvme/mmcblk** : le playbook détecte et utilise automatiquement le séparateur `p` (ex: `/dev/nvme0n1p1`).

### Exemples d’exécution

- Créer une partition **/dev/sdb1** pleine taille, ext4, montée sur `/data` :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/system/partition-mount.yml \
  -e "device=/dev/sdb part_number=1 fs_type=ext4 mount_point=/data"
```
- Utiliser XFS et des options de montage custom :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/system/partition-mount.yml \
  -e "device=/dev/sdb fs_type=xfs mount_point=/srv mount_options=noatime,nodiratime"
```
- Supprimer la partition et retirer le montage :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/system/partition-mount.yml \
  -e "device=/dev/sdb state=absent"
```

**Bonnes pratiques**

- **Attention destructif** : la création d’un FS **écrase** les données de la partition ciblée. Exécuter d’abord en `--check` ne suffit pas pour simuler `filesystem/parted` → tester en lab.
- En prod, définir une **fenêtre de maintenance** et vérifier les dépendances (services, conteneurs, NFS…).
- Choisir `gpt` pour les disques modernes, `msdos` pour compatibilité legacy.
- Sur **cloud/VM**, vérifier le nom exact du disque (`lsblk`, `fdisk -l`).
---

## 🧑‍💻 Bonnes pratiques générales

Toujours créer un utilisateur non-root pour l’administration quotidienne.

Conserver un accès console (VM/Hyperviseur) au cas où la config SSH est cassée.

Mettre régulièrement à jour les paquets pour appliquer les correctifs de sécurité.

Automatiser l’installation des outils indispensables sur chaque nouvelle machine.
