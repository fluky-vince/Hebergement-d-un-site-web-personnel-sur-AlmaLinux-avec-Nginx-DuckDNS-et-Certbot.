# Hebergement-d-un-site-web-personnel-sur-AlmaLinux-avec-Nginx-DuckDNS-et-Certbot.
# Apprendre à déployer un serveur web complet accessible depuis internet.
# Projet : Déploiement d’un site web statique avec Nginx, DuckDNS, FirewallD, Port Forwarding et HTTPS

##  Description du projet
Ce projet consiste à déployer un site web statique sur une machine virtuelle Linux en utilisant Nginx comme serveur web. 
Le site est accessible via un domaine DuckDNS et sécurisé avec un certificat SSL Let’s Encrypt obtenu via Certbot. 
L’objectif est de démontrer la mise en place complète d’un service web accessible publiquement, incluant réseau, DNS dynamique, 
firewall, port forwarding et HTTPS.

---

##  Architecture du projet

- Machine virtuelle Linux en mode **bridged**
- Serveur web **Nginx**
- Domaine dynamique **DuckDNS**
- Redirection de ports 80/443 sur le routeur
- Certificat SSL **Let’s Encrypt** (Certbot)
- Firewall configuré pour HTTP/HTTPS
- Page HTML statique (ex. CV ou portfolio)

---

##  Technologies utilisées

- Linux (AlmaLinux / Rocky / CentOS)
- Nginx
- DuckDNS
- Certbot + plugin Nginx
- FirewallD
- HTML / CSS
- VMware Workstation (mode bridged)

---

##  Étapes du projet

### 1. Configuration de la machine virtuelle
La VM est configurée en mode **bridged**, ce qui lui permet d’obtenir une adresse IP du réseau local et d’être accessible 
directement par le routeur. Cela est nécessaire pour que le port forwarding fonctionne correctement.

---

### 2. Configuration du firewall
Le firewall doit autoriser les services HTTP et HTTPS :

sudo firewall-cmd --zone=public --add-service=http --permanent
sudo firewall-cmd --zone=public --add-service=https --permanent
sudo firewall-cmd --reload

---

### 3. Création du domaine DuckDNS
1. Se connecter sur : https://www.duckdns.org  
2. Créer un sous-domaine gratuit  
3. DuckDNS détecte automatiquement l’adresse IP publique  

Méthodes pour vérifier l’adresse IP publique :

curl ifconfig.me

Ou via navigateur :

- https://ifconfig.me  
- https://icanhazip.com  

Vérification DNS :

dig mon-domaine.duckdns.org

---

### 4. Configuration DNS A/AAAA
DuckDNS gère automatiquement l’enregistrement **A (IPv4)**.  
Aucune configuration manuelle n’est nécessaire.

Pour ce projet :

- Seul l’enregistrement A est utilisé  
- Aucun fichier de zone n’est requis  
- DuckDNS met à jour l’IP automatiquement  

---

### 5. Installation de Nginx

Installation du dépôt EPEL :

sudo dnf install epel-release -y

Installation de Nginx :

sudo dnf install nginx -y

Activation du service :

sudo systemctl enable nginx --now
sudo systemctl status nginx

Tests locaux :

curl http://localhost

Ou via navigateur avec l’IP locale de la VM.

À ce stade, **Nginx fonctionne localement**, mais n’est pas encore accessible depuis Internet tant que le port forwarding n’est pas configuré.

---

### 6. Port Forwarding sur le routeur

Le port forwarding permet de rediriger le trafic provenant d’Internet vers une machine interne du réseau local.

Dans ce projet :

- Port **80 → VM** (HTTP)
- Port **443 → VM** (HTTPS)

Étapes générales :

1. Accéder à l’interface du routeur (ex. 192.168.0.1 ou 192.168.1.1)
2. Se connecter avec les identifiants administrateur
3. Aller dans **Port Forwarding** ou **NAT**
4. Ajouter deux règles :
   - TCP 80 → IP locale de la VM
   - TCP 443 → IP locale de la VM

Schéma :

[ DuckDNS ] ---> [ Internet ] ---> [ Routeur ]
|
| Port Forwarding
| - 80 → VM (HTTP)
| - 443 → VM (HTTPS)
v
[ VM Linux (192.168.x.x) ]
|
v
[ Nginx ]

---

### 7. Déploiement du site HTML

Le répertoire web par défaut de Nginx est :

/usr/share/nginx/html/

Sauvegarde du fichier par défaut :

sudo cp /usr/share/nginx/html/index.html /usr/share/nginx/html/index.html.bak

Remplacement du fichier :

sudo nano /usr/share/nginx/html/index.html

Coller le code HTML du site (ex. CV ou portfolio).

Test :

http://mon-domaine.duckdns.org (mon-domaine.duckdns.org in Bing)

---

### 8. Configuration du VirtualHost Nginx

Créer un fichier :

sudo nano /etc/nginx/conf.d/site.conf

Contenu :

server {
listen 80;
server_name mon-domaine.duckdns.org;

root /usr/share/nginx/html;
index index.html;

location / {
try_files $uri $uri/ =404;
}
}

Vérification :

sudo nginx -t

Redémarrage :

sudo systemctl restart nginx

---

### 9. Activation du HTTPS avec Certbot

Installation :

sudo dnf install certbot python3-certbot-nginx -y

Obtention du certificat :

sudo certbot --nginx

Certbot :

- détecte automatiquement le domaine
- valide via HTTP
- installe le certificat SSL
- configure la redirection HTTP → HTTPS

Redémarrage final :

sudo systemctl restart nginx

Test :

https://mon-domaine.duckdns.org (mon-domaine.duckdns.org in Bing)
