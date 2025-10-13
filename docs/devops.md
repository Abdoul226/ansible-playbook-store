# Section Devops

## 1. Installer Docker (`install-docker.yml`)

**Objectif :**  
Installer **Docker CE** depuis le **référentiel officiel** (Debian/Ubuntu & RHEL-like), activer **docker compose v2** (plugin), démarrer au boot, ajouter des **utilisateurs** au groupe `docker`, et configurer `daemon.json`.

### Variables
```perl
| Variable | Défaut | Description |
|---|---|---|
| `docker_users` | `[]` | Liste d’utilisateurs à ajouter au groupe `docker` |
| `docker_daemon_options` | voir playbook | Options `/etc/docker/daemon.json` (logs, cgroupdriver, etc.) |
| `docker_packages` | liste standard | Paquets Docker officiels (`docker-ce`, `docker-compose-plugin`, …) |
```
> Par défaut, on définit :  
> - `log-driver: json-file` + rotation `10m` / `3` fichiers  
> - `exec-opts: native.cgroupdriver=systemd` (recommandé avec k8s)

### Exemples

- Installation standard :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/devops/install-docker.yml
```
- Ajouter l’utilisateur `aziz` au groupe `docker` :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/devops/install-docker.yml \
  -e "docker_users=['aziz']"
```
- Personnaliser `daemon.json` (journaux + data-root) :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/devops/install-docker.yml \
  -e 'docker_daemon_options={"log-driver":"json-file","log-opts":{"max-size":"50m","max-file":"5"},"data-root":"/data/docker"}'
```

**Vérification**
- `docker --version` et `docker compose version`
- `getent group docker` → vérifier l’appartenance au groupe (logout/login requis)
- `systemctl status docker` → **active (running)**

**Bonnes pratiques**
- Après ajout au groupe `docker`, **reconnecte-toi** (nouvelle session) pour appliquer les droits.
- En prod, externaliser `data-root` (disque dédié) et définir une **politique de logs** stricte.
- **Vault-iser** toute info sensible si tu ajoutes des credentials de registry plus tard.
- Sur RHEL/SELinux, si tu montes des répertoires hôtes, ajouter le contexte `:z`/`:Z` dans Compose ou ajuster les contextes SELinux.

---

## 2. Installer Docker Compose (binaire v1.x standalone) — `install-docker-compose.yml`

**Objectif :**  
Installer la version **classique** (v1.x) de **Docker Compose** sous `/usr/local/bin`, pour compatibilité avec d’anciens projets ou serveurs CI/CD ne disposant pas de Docker Compose v2 intégré.

### Variables principales
```perl
| Variable | Valeur par défaut | Description |
|-----------|------------------|--------------|
| `compose_version` | `"1.29.2"` | Version à installer |
| `compose_install_path` | `/usr/local/bin/docker-compose` | Chemin du binaire |
| `compose_symlink_path` | `/usr/bin/docker-compose` | Lien symbolique pour compatibilité |
| `compose_owner` | `root` | Propriétaire du fichier |
| `compose_mode` | `0755` | Permissions d’exécution |
```

### Exemples

- **Installation standard :**
```bash
ansible-playbook -i inventory/hosts.ini playbooks/devops/install-docker-compose.yml
```
- **Installation d’une autre version :**
```bash
ansible-playbook -i inventory/hosts.ini playbooks/devops/install-docker-compose.yml \
  -e "compose_version=1.28.6"
```
**Vérification**
`docker-compose --version` --> `docker-compose version 1.29.2, build 5becea4c`

**Bonnes pratiques**
- Pour les environnements récents, **préférer Docker Compose v2** (`docker compose` sans tiret) intégré dans Docker.
- Conserver ce playbook pour les **CI/CD anciens** (Jenkins, GitLab Runner) où `docker-compose` est requis.
- Peut être intégré dans une **pipeline Ansible multi-playbooks** (`install-docker.yml` → `install-docker-compose.yml` → `deploy-stack.yml`).

---

## 3. Déployer une stack Docker Compose (WordPress + MariaDB) — `deploy-compose-wordpress.yml`

**Objectif :**  
Déployer **WordPress** + **MariaDB** en Docker Compose, avec persistance des données, healthcheck, et port HTTP exposé.

### Variables
```perl
| Variable | Défaut | Description |
|---|---|---|
| `project_name` | `wpstack` | Nom logique de la stack (préfixe containers) |
| `data_dir` | `/srv/compose` | Répertoire racine où vivent les stacks |
| `wp_port` | `8080` | Port HTTP exposé côté hôte |
| `db_image` | `mariadb:10.11` | Image base de données |
| `wp_image` | `wordpress:6.6.2-apache` | Image WordPress |
| `db_name` | `wordpress` | Nom de la base |
| `db_user` | `wpuser` | Utilisateur DB |
| `db_password` | `changeme-wpuser` | Mot de passe DB (vault conseillé) |
| `db_root_password` | `changeme-root` | Mot de passe root DB (vault conseillé) |
```

> Les données persistantes sont stockées dans `{{data_dir}}/{{project_name}}/db` (MariaDB) et `{{data_dir}}/{{project_name}}/wordpress` (fichiers WordPress).

### Exemples

- Déploiement standard :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/devops/deploy-compose-wordpress.yml
```
- Port 80 + mots de passe Vault :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/devops/deploy-compose-wordpress.yml \
  -e "wp_port=80 db_password=@vault.txt db_root_password=@vault.txt"
```
- Changer les images :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/devops/deploy-compose-wordpress.yml \
  -e "db_image=mariadb:11.4 wp_image=wordpress:latest"
```

**Vérification**
- `docker ps` → 2 conteneurs `wp` & `db` **Up**
- Naviguer : `http://<IP_SERVEUR>:<wp_port>/` → assistant d’installation WordPress
- Logs si besoin : `docker logs -f wpstack-wp`

**Bonnes pratiques**
- **Vault-iser** `db_password` / `db_root_password` (Ansible Vault).
- Sauvegardes :
```bash
# Dump DB
docker exec -i wpstack-db mariadb-dump -u root -p"$MYSQL_ROOT_PASSWORD" wordpress > backup.sql
# Fichiers
tar -czf wpfiles.tar.gz -C /srv/compose/wpstack/wordpress .
```
- Pour **HTTPS**, place un reverse-proxy (Nginx / Caddy) devant, ou utilise ton playbook **Let’s Encrypt** sur le serveur.
- Pour mises à jour : `docker compose pull` && `docker compose up -d` dans le dossier de la stack.

---

## 4. Installer Jenkins (LTS) — `install-jenkins.yml`

**Objectif :**  
Installer **Jenkins LTS** via le **repo officiel**, avec **Java 17**, configuration du **port**, ouverture du **firewall**, démarrage du **service** et affichage du **mot de passe initial**.  
Optionnel : **créer un admin** et **installer des plugins** automatiquement via `init.groovy.d`.

### Variables
```perl
| Variable | Défaut | Description |
|---|---|---|
| `jenkins_http_port` | `8080` | Port HTTP |
| `manage_firewall` | `true` | Ouvrir le port via UFW/firewalld |
| `java_pkg_debian` | `openjdk-17-jdk` | Paquet Java Debian/Ubuntu |
| `java_pkg_rhel` | `java-17-openjdk` | Paquet Java RHEL-like |
| `jenkins_auto_config` | `false` | **Si true** : crée un admin et installe des plugins au 1er démarrage |
| `jenkins_admin_user` | `admin` | Nom du compte admin auto |
| `jenkins_admin_pass` | `ChangeMe123!` | Mot de passe admin auto (vaultisez) |
| `jenkins_plugins` | `['git','workflow-aggregator','credentials-binding','blueocean']` | Plugins à installer (si auto_config) |
```

### Utilisation
- **Installation standard** (assistant de déverrouillage) :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/devops/install-jenkins.yml
```
→ Ouvrez `http://<IP>:8080` et utilisez le secret affiché (fichier …/initialAdminPassword).
- Auto-config (admin + plugins) :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/devops/install-jenkins.yml \
  -e "jenkins_auto_config=true jenkins_admin_user=admin jenkins_admin_pass='SuperSecret!2025' jenkins_plugins=['git','workflow-aggregator','blueocean']"
```
→ Ouvrez `http://<IP>:8080` et connectez-vous directement avec l’admin.

**Bonnes pratiques**
- **Vaultisez** le mot de passe admin si vous utilisez `jenkins_auto_config`.
- Placez Jenkins derrière un **reverse proxy** (Nginx/HAProxy) + **HTTPS** (Let’s Encrypt).
- Sauvegardez `JENKINS_HOME` (`/var/lib/jenkins`) et vos jobs/pipelines.
- Mettez à jour régulièrement Jenkins et ses plugins.

---

## 5.


