🔐 Playbooks Sécurité – Ansible Playbook Store

Cette section regroupe des playbooks permettant de durcir la sécurité des serveurs Linux de manière automatisée avec Ansible.
Chaque playbook est conçu pour être simple, réutilisable et personnalisable.

📂 Liste des playbooks
1. configure-ssh.yml

But : Sécuriser la configuration SSH.

Actions principales :

Désactiver la connexion root (PermitRootLogin no).

Forcer l’authentification par clé SSH (PasswordAuthentication no).

Limiter les utilisateurs autorisés (AllowUsers).

Définir un port SSH personnalisé.

Restreindre certaines options dangereuses (X11Forwarding, TCPForwarding…).

Appliquer la configuration via un fichier override (/etc/ssh/sshd_config.d/99-hardening.conf).

⚠️ Recommandations :

Toujours tester avec --check avant application.

Garder une session SSH ouverte au cas où la nouvelle configuration bloque l’accès.

Si vous changez le port SSH (ssh_port), ouvrez-le dans le firewall (UFW ou firewalld).

2. configure-firewall.yml (à venir)

But : Configurer un firewall basique avec UFW (Ubuntu/Debian) ou firewalld (RedHat/CentOS).

Règles par défaut :

Autoriser SSH (port défini dans ssh_port).

Autoriser HTTP/HTTPS (80/443).

Refuser tout le reste par défaut.

3. fail2ban.yml (à venir)

But : Protéger le serveur contre les tentatives de brute force SSH.

Actions principales :

Installation de fail2ban.

Activation de la jail SSH (/etc/fail2ban/jail.local).

Configuration personnalisable (bantime, findtime, maxretry).

4. ssh-hardening.yml (alias complet)

But : Variante avancée pour appliquer un ensemble de règles de sécurité SSH encore plus strictes.

Exemple de durcissements :

Forcer les algorithmes modernes (ex: Ciphers, MACs).

Désactiver protocoles et options obsolètes.

⚙️ Variables personnalisables (exemple configure-ssh.yml)
Variable	Valeur par défaut	Description
ssh_port	22	Port SSH à utiliser
permit_root_login	no	Désactive le login root (yes / no / prohibit-password)
password_auth	no	Désactive l’authentification par mot de passe
pubkey_auth	yes	Active les clés SSH
allow_users	[]	Liste des utilisateurs autorisés à se connecter
client_alive_interval	300	Timeout keepalive en secondes
client_alive_count_max	2	Nombre max de réponses keepalive
▶️ Exemples d’utilisation
Exécution standard
ansible-playbook -i inventory/hosts.ini playbooks/security/configure-ssh.yml

Changer le port SSH en 2222
ansible-playbook -i inventory/hosts.ini playbooks/security/configure-ssh.yml \
  -e "ssh_port=2222 allow_users=['aziz']"

Exécution en dry-run (sans appliquer)
ansible-playbook -i inventory/hosts.ini playbooks/security/configure-ssh.yml --check --diff

🧑‍💻 Bonnes pratiques générales

Toujours créer au moins un utilisateur sudo avec clé SSH avant de désactiver root.

Ne jamais déployer une configuration SSH sans avoir un accès console alternatif (VM console, IPMI, Proxmox, etc.).

Utiliser un firewall en complément (configure-firewall.yml).

Surveiller les logs SSH dans /var/log/auth.log ou via un outil de monitoring (ex: ELK, Grafana Loki).
