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

## 4. Déployer une app Python Flask avec Nginx + Gunicorn (`deploy-flask-nginx-gunicorn.yml`)

**Objectif :**  
Déployer une appli **Flask** servie par **Gunicorn** (service `systemd`) derrière **Nginx** (reverse-proxy).  
Supporte un déploiement **depuis Git** ou une **app de démo** prête à tester.

### Variables principales
```perl
| Variable | Défaut | Description |
|---|---|---|
| `app_name` | `flaskapp` | Nom logique de l’app |
| `app_dir` | `/opt/flaskapp` | Dossier de l’app |
| `repo_url` | `""` | Repo Git (laisser vide = app de démo) |
| `repo_version` | `main` | Branche/tag |
| `wsgi_module` | `app:app` | Module WSGI (`module:objet`) |
| `venv_path` | `/opt/flaskapp/venv` | Virtualenv |
| `gunicorn_bind` | `unix:/opt/flaskapp/gunicorn.sock` | Socket Gunicorn |
| `gunicorn_workers` | `2` | Workers Gunicorn |
| `server_name` | `_` | FQDN/Nom vhost (mettre ton domaine) |
| `http_port` | `80` | Port Nginx |
| `manage_firewall` | `true` | Ouvrir le port via UFW/firewalld |
```
### Exemples

- **Démo immédiate** (hello world) :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/web/deploy-flask-nginx-gunicorn.yml
```
 - Depuis un repo Git + domaine :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/web/deploy-flask-nginx-gunicorn.yml \
  -e "repo_url=https://github.com/you/your-flask-app.git repo_version=main server_name=app.example.com app_name=myapp"
```
- Ajuster workers et timeout :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/web/deploy-flask-nginx-gunicorn.yml \
  -e "gunicorn_workers=4 gunicorn_timeout=120"
```
**Vérification**
- `systemctl status gunicorn-<app_name>` → service **actif**
- `curl -I http://<ip>:<port>` → `200 OK`
- Naviguer sur `http://<ip>` → “Hello from Flask…”

**Bonnes pratiques**
- Ajouter ensuite **HTTPS** (Let’s Encrypt) → voir playbook dédié.
- Pour haut trafic, passer à socket **TCP** (ex: `0.0.0.0:8000`) et tuning Nginx/Gunicorn.
- Séparer **logs** et monitoring (ex: `nginx_exporter`, Loki/Promtail).
- Sur RHEL/SELinux enforcing, autoriser Nginx à accéder au socket si besoin (ex: `setsebool -P httpd_can_network_connect 1` ou politique socket).

---

## 5. Configurer HTTPS avec Let’s Encrypt (`lets-encrypt.yml`) >> **A vérifier**

**Objectif :**  
Obtenir/renouveler automatiquement un **certificat TLS** via **Let’s Encrypt** (plugin **webroot**) et activer un **vhost HTTPS** sous **Nginx** ou **Apache**.  
Le challenge est servi depuis `{{ webroot }}` (route `/.well-known/acme-challenge/`).

### Variables principales
```perl
| Variable | Défaut | Description |
|---|---|---|
| `server` | `nginx` | Choix du serveur (`nginx` ou `apache`) |
| `domains` | `['example.com','www.example.com']` | Domaines à certifier (CN = 1er) |
| `email` | `admin@example.com` | Contact Let’s Encrypt |
| `agree_tos` | `true` | Acceptation des CGU LE |
| `redirect_http_to_https` | `true` | Redirection 80 → 443 |
| `webroot` | `/var/www/letsencrypt` | Dossier pour le challenge ACME |
| `ssl_conf_name` | `00-ssl-app.conf` | Nom du fichier de conf SSL |
| `manage_firewall` | `true` | Ouvre le port 443 |
```
> Les certificats sont stockés dans `/etc/letsencrypt/live/<domain>/`.  
> **Renouvellement auto** : Certbot installe un **systemd timer**/cron par défaut (`certbot.timer`).

### Exemples

- HTTPS sur Nginx pour `example.com` :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/web/lets-encrypt.yml \
  -e "server=nginx domains=['example.com','www.example.com'] email=admin@example.com"
``` 
- HTTPS sur Apache + pas de redirection :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/web/lets-encrypt.yml \
  -e "server=apache redirect_http_to_https=false domains=['site.tld'] email=me@site.tld"
```
- Générer des dhparam :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/web/lets-encrypt.yml \
  -e "generate_dhparam=true"
``` 

**Vérification**
- `sudo certbot certificates` → liste des certs.
- `curl -I https://example.com` → `200` et certificat valide.
- Naviguer : cadenas **HTTPS** dans le navigateur.

**Bonnes pratiques**
- Avoir un **DNS A/AAAA** correct vers le serveur avant l’exécution.
- Laisser le vhost **HTTP** accessible au moins pour le challenge.
- En prod, forcer `TLSv1.2+`, envisager `HSTS` et **ciphers** durcis.
- Sur Nginx, proxifier vers ton app (ex: Flask/Gunicorn) en adaptant le `root/location`.

---

## 6. Configurer un Reverse Proxy Nginx (`nginx-reverse-proxy.yml`) >> ** A vérifier et tester**

**Objectif :**  
Faire de **Nginx** un **reverse proxy** vers une application amont (Flask, NodeJS, Go, PHP-FPM derrière Apache, etc.), avec support **WebSocket**, **gzip**, **timeouts**, taille d’upload et **ouverture du firewall**.

### Variables
```perl
| Variable | Défaut | Description |
|---|---|---|
| `server_name` | `_` | FQDN (ex: `app.example.com`) |
| `listen_port` | `80` | Port d’écoute du vhost |
| `manage_firewall` | `true` | Ouvre le port via UFW/firewalld |
| `gzip_enabled` | `true` | Active la compression gzip |
| `client_max_body_size` | `50m` | Limite d’upload côté Nginx |
| `upstream_scheme` | `http` | `http` ou `https` |
| `upstream_host` | `127.0.0.1` | Hôte de l’app |
| `upstream_port` | `8000` | Port de l’app |
| `upstream_path` | `/` | Chemin amont (ex: `/api/`) |
| `enable_websocket` | `true` | Upgrade des connexions pour WS |
| `proxy_*_timeout` | `60s` / `5s` | Timeouts proxy |
| `extra_locations` | `[]` | Emplacements supplémentaires (ex: `/static/`) |
```
**Exemple `extra_locations` :**
```yaml
extra_locations:
  - path: /static/
    root: /opt/app/static
    try_files: "$uri =404"
```
**Exemples d’exécution**
- Reverse proxy par défaut (→ `127.0.0.1:8000/`) :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/web/nginx-reverse-proxy.yml
```
- Vers un backend sur `10.0.0.5:5000` + domaine :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/web/nginx-reverse-proxy.yml \
  -e "server_name=app.example.com upstream_host=10.0.0.5 upstream_port=5000"
```
- Chemin d’API dédié + taille d’upload à 10 Mo :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/web/nginx-reverse-proxy.yml \
  -e "upstream_path=/api/ client_max_body_size=10m"
```

**Bonnes pratiques**
Combine avec `lets-encrypt.yml` pour activer le **HTTPS** : garder le reverse proxy tel quel, et faire écouter le vhost TLS sur `443` par le playbook Let’s Encrypt.
- Pour des **uploads volumineux**, ajuste `client_max_body_size` et les timeouts.
- Si l’amont est en **HTTPS**, mets `upstream_scheme: https` (et gère la validation TLS si nécessaire, via `proxy_ssl_*` directives dans une variante avancée).

---

## 7.
### Load balancer HAProxy (`haproxy-loadbalancer.yml`)

**Objectif :**  
Fournir un **équilibreur de charge** HAProxy pour HTTP/TCP avec **health checks**, **algorithmes de répartition**, **sticky sessions** (optionnel), **page de stats** (optionnelle), et **termination TLS** (optionnelle).

**Fichier :** `playbooks/web/haproxy-loadbalancer.yml`

#### Variables clés
```perl
| Variable 		  | Défaut  | Description |
|-------------------------|---------|-------------|
| `http_enabled`	  | `true`  | Frontend HTTP sur `http_bind_port` |
| `http_bind_port`	  | `80`    | Port frontend HTTP |
| `ssl_termination`	  | `false` | Active un frontend HTTPS qui **termine TLS** (cert PEM requis) |
| `https_bind_port` 	  | `443`   | Port HTTPS si termination |
| `ssl_pem_path` 	  | `/etc/ssl/private/haproxy.pem` | Chemin du **bundle PEM** (cert+key) |
| `tcp_enabled`		  | `false` | Frontend TCP (ex: **TLS passthrough**) |
| `tcp_bind_port`	  | `443`   | Port frontend TCP |
| `backend_mode`	  | `http`  | Mode par défaut des backends (`http`/`tcp`) |
| `balance_algo`	  | `roundrobin` | Algorithme (ex: `leastconn`, `source`) |
| `sticky_cookie_enabled` | `false` | Active cookie sticky côté backend HTTP |
| `backends_http`	  | liste   | Serveurs HTTP (nom/host/port/weight/check) |
| `backends_tcp`	  | liste   | Serveurs TCP |
```
> **Stats** : si `stats_enabled=true`, interface sur `http://<ip>:<stats_bind_port>{{stats_uri}}` (auth basique).  
> Health-check HTTP par défaut : `GET /health` puis `/`.

#### Exemples

- **HTTP LB** classique (80 → 2 backends 8080) :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/web/haproxy-loadbalancer.yml \
  -e "backends_http=[{name:'app1',host:'10.0.0.11',port:8080},{name:'app2',host:'10.0.0.12',port:8080}]"
```
- Sticky sessions + leastconn :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/web/haproxy-loadbalancer.yml \
  -e "sticky_cookie_enabled=true balance_algo=leastconn"
```
- TLS termination (cert PEM déjà en place) :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/web/haproxy-loadbalancer.yml \
  -e "ssl_termination=true ssl_pem_path=/etc/ssl/private/haproxy.pem"
```
- TLS passthrough (mode TCP) :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/web/haproxy-loadbalancer.yml \
  -e "tcp_enabled=true backend_mode=tcp backends_tcp=[{name:'s1',host:'10.0.0.21',port:443},{name:'s2',host:'10.0.0.22',port:443}]"
```

**Vérification**
- `haproxy -c -f /etc/haproxy/haproxy.cfg` → OK
- `systemctl status haproxy` → actif
- `curl -I http://<ip>` → `200` (si backends HTTP OK)
- Stats (si activées) : `http://<ip>:8404/haproxy?stats`

**Bonnes pratiques**
- En **prod**, **vault-iser** `stats_user/stats_pass` et les **certificats**.
- Si termination TLS : générer un **PEM** `fullchain + privkey` concaténés.
- Préférer `leastconn` pour des charges **non homogènes**, source pour **sticky** sans cookie (hash IP client).
- Coupler avec un **monitoring** (Exporter Prometheus HAProxy) et des **alertes**.
