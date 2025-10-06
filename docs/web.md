# 🌐 Playbooks Web – Ansible Playbook Store

Playbooks pour installer et configurer des serveurs web, déployer des applis et mettre en place HTTPS / reverse proxy.

---

## 1. Installer Nginx (`install-nginx.yml`)

**Objectif :**  
Installer **Nginx**, déployer une **page d’accueil**, ouvrir le **firewall** si demandé, et appliquer une **conf par défaut** simple qui écoute sur `http_port`.

### Variables
```perl
| Variable          | Défaut            | Description |
|-------------------|-------------------|-------------|
| `http_port`       | `80`              | Port HTTP d’écoute |
| `web_root`        | `/var/www/html`   | Répertoire racine du site |
| `index_file`      | `index.html`      | Nom du fichier index |
| `index_title`     | `Serveur Nginx déployé par Ansible` | Titre HTML |
| `index_message`   | `Bienvenue ! 🚀`  | Message dans la page d’accueil |
| `manage_firewall` | `true`            | Ouvre le port (UFW ou firewalld) si dispo |
```
> La conf minimale est écrite dans `/etc/nginx/conf.d/00-default-port.conf` (compatible Debian et RHEL-like).

### Exemples d’exécution

- Installation par défaut :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/web/install-nginx.yml
```
- Changer le port et le répertoire web :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/web/install-nginx.yml \
  -e "http_port=8080 web_root=/srv/www"
```
- Ne pas toucher le firewall :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/web/install-nginx.yml \
  -e "manage_firewall=false"
```
- Personnaliser la page d’accueil :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/web/install-nginx.yml \
  -e "index_title='Mon App' index_message='Déployé via CI/CD 🎯'"
```
**Vérification**
- Tester : `curl -I http://<ip>:<port>` → doit renvoyer `200 OK`.
- Afficher : `http://<ip>:<port>` → page d’accueil HTML avec le message.

**Bonnes pratiques**

- Mettre les vhosts applicatifs dans des fichiers séparés (ex: conf.d/app.conf).
- Coupler ensuite avec :
	- `reverse-proxy` (proxy_pass vers backend),
	- `Let’s Encrypt` (TLS),
	- `fail2ban` (si login HTTP basique),
	- monitoring (`nginx_exporter`).

---

## 2. Installer Apache2 / httpd (`install-apache.yml`)

**Objectif :**  
Installer **Apache** (Apache2 sur Debian/Ubuntu, httpd sur RHEL-like), créer un **VirtualHost minimal**, déployer une **page d’accueil**, **ouvrir le firewall** si demandé, et **valider la conf**.

### Variables
```perl
| Variable         | Défaut           | Description |
|------------------|------------------|-------------|
| `http_port`      | `80`             | Port HTTP d’écoute |
| `server_name`    | `_`              | Nom de serveur (mettre un FQDN si besoin) |
| `web_root`       | `/var/www/html`  | Racine du site |
| `index_file`     | `index.html`     | Nom du fichier d’accueil |
| `index_title`    | `Serveur Apache déployé par Ansible` | Titre de la page |
| `index_message`  | `Bienvenue ! 🚀` | Message dans la page |
| `manage_firewall`| `true`           | Ouvrir le port via UFW/firewalld |
| `apache_pkg`     | auto             | `apache2` (Debian) / `httpd` (RHEL) |
| `apache_svc`     | auto             | `apache2` / `httpd` |
| `apache_user`    | auto             | `www-data` / `apache` |
| `apache_group`   | auto             | `www-data` / `apache` |
| `apache_ctl`     | auto             | `apache2ctl` / `apachectl` |
| `vhost_path`     | auto             | Fichier de vhost par défaut selon OS |
```
### Exemples d’exécution

- Installation par défaut :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/web/install-apache.yml
```
- Changer port et docroot :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/web/install-apache.yml \
  -e "http_port=8080 web_root=/srv/www server_name=example.local"
```
- Ne pas toucher au firewall :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/web/install-apache.yml \
  -e "manage_firewall=false"
```
Vérification
- `curl -I http://<ip>:<port>` → 200 OK attendu.
- Naviguer sur `http://<ip>:<port>` → page d’accueil HTML.

**Bonnes pratiques**
- Placer chaque application dans un **vhost dédié** (un fichier `.conf` par app).
- Activer **mod_ssl** et utiliser **Let’s Encrypt** pour HTTPS (voir playbook dédié).
- Journaliser et superviser avec **Filebeat/ELK** ou **Loki/Promtail**.

---

## 3. Déployer une app PHP avec Apache (`deploy-php-apache.yml`)

**Objectif :**  
Installer Apache + PHP, créer un **VirtualHost**, déployer l’app soit depuis un **dépôt Git**, soit via une **app de démonstration**, ouvrir le firewall si besoin, et **valider la conf**.

### Variables
```perl
| Variable            | Défaut               | Description |
|---------------------|----------------------|-------------|
| `server_name`       | `_`                  | Nom du serveur (FQDN conseillé en prod) |
| `http_port`         | `80`                 | Port d’écoute |
| `web_root`          | `/var/www/app-php`   | Racine de l’app |
| `enable_htaccess`   | `true`               | `AllowOverride All` si `true` |
| `manage_firewall`   | `true`               | Ouvrir le port via UFW/firewalld |
| `repo_url`          | `""`                 | URL Git de l’app (laisser vide pour démo) |
| `repo_version`      | `main`               | Branche/tag à déployer |
| `app_owner`/`group` | auto (www-data/apache) | Propriétaire/groupe du répertoire |
| `app_mode`          | `0755`               | Permissions du répertoire |
```
> Paquets PHP communs installés : `mbstring`, `xml`, `curl`, `mysql`.  
> Sur Debian, `libapache2-mod-php` est activé (mod_php).

### Exemples d’exécution

- Déployer **une app de démo** (phpinfo) :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/web/deploy-php-apache.yml
```
- Déployer depuis un repo Git :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/web/deploy-php-apache.yml \
  -e "repo_url=https://github.com/tonuser/ton-app-php.git repo_version=main server_name=app.local web_root=/var/www/app"
```
- Changer le port + désactiver .htaccess :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/web/deploy-php-apache.yml \
  -e "http_port=8080 enable_htaccess=false"
```

**Vérification**
- `curl -I http://<ip>:<port>` → `200 OK`.
- Naviguer sur `http://<ip>:<port>` :
	- Démo → page phpinfo()
	- Git → ta vraie application

**Bonnes pratiques**
- En prod, préférer **FQDN + HTTPS** (voir playbook Let’s Encrypt).
- Isoler chaque **app** dans son vhost et son répertoire.
- Si ton app nécessite des permissions d’écriture (uploads, cache), appliquer des droits fins sur ces sous-dossiers uniquement.

---

## 4. 
