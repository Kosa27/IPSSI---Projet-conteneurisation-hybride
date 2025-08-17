# IPSSI---Projet-conteneurisation-hybride

# Projet – Conteneurisation Hybride

Ce projet a pour but de :
- Reproduire une infrastructure (Apache + MariaDB + Reverse Proxy) avec Docker sur Ubuntu 22.04.
- Préparer la migration vers LXD pour Apache et MariaDB, tout en conservant le reverse proxy sous Docker.
- Automatiser le déploiement avec des scripts.
- Sécuriser l’accès aux bases de données via iptables.

---

# Partie 1 – Infrastructure de base (Docker)

### Livrables fournis
- Dockerfile Apache (`docker/apache/Dockerfile` + `index.html`)
- Dockerfile MariaDB (`docker/mariadb/Dockerfile`)
- Dockerfile Reverse Proxy (`docker/reverse-proxy/Dockerfile` + `default.conf`)
- Script d’automatisation `deploy_docker.sh`

### Procédure de déploiement manuel (bash)
1. Créer un réseau :
   docker network create webnet
   
2 . Construire les images :
docker build -t myapache ./docker/apache
docker build -t mymariadb ./docker/mariadb

3. Lancer les conteneurs :
docker run -d --name monsite-web --network webnet -p 8080:80 myapache
docker run -d --name monsite-db --network webnet mymariadb

4. Tester :
curl http://localhost:8080

### Automatisation (lancer le script fourni) :
chmod +x deploy_docker.sh
./deploy_docker.sh monsite 8080


# Partie 2 – Infrastructure cible (LXD)

### Livrables fournis
- Procédure de mise en place d’Apache + MariaDB sous LXD
- Script d’automatisation deploy_lxd.sh
- Volume partagé hôte ↔ conteneur web
- Règles iptables pour limiter l’accès aux bases de données
  
### Déploiement manuel (bash)

1. Créer le réseau :
lxc network create lxnet ipv4.address=10.50.50.1/24 ipv4.nat=true ipv6.address=none

2. Lancer les conteneurs :
lxc launch images:ubuntu/22.04 monsite-web -n lxnet
lxc launch images:ubuntu/22.04 monsite-db -n lxnet

3. Installer Apache dans le web :
lxc exec monsite-web -- bash -c "apt update && apt install -y apache2 && echo '<h1>LXD Web Server</h1>' > /var/www/html/index.html"

4. Installer MariaDB dans la DB :
lxc exec monsite-db -- bash -c "apt update && apt install -y mariadb-server"


### Procédure de migration
- Dans l’infra Docker, Apache + MariaDB tournaient dans Docker.
- Dans l’infra cible :
Apache et MariaDB passent dans LXD
Le reverse proxy reste sous Docker pour compatibilité.


### Automatisation (lancer le script fourni):
chmod +x deploy_lxd.sh
./deploy_lxd.sh monsite 8081

### Volume partagé :
lxc config device add monsite-web shared disk source=$(pwd)/shared path=/var/www/html

### Sécurité :
iptables -A INPUT -p tcp --dport 3306 -s <IP_WEB> -j ACCEPT
iptables -A INPUT -p tcp --dport 3306 -j DROP





