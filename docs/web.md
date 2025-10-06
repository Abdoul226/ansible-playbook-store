# 🌐 Playbooks Web – Ansible Playbook Store

Playbooks pour installer et configurer des serveurs web, déployer des applis et mettre en place HTTPS / reverse proxy.

---

## Installer Nginx (`install-nginx.yml`)

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
