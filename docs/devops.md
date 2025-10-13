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

## 3. 


