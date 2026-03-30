# Projet-Docker-LGTM

📊 Stack LGTM - Observabilité Full Stack
Loki | Grafana | Tempo | Mimir

Ce projet déploie une infrastructure de monitoring complète et moderne via Docker. Elle permet de centraliser les Métriques, les Logs et les Traces (le fameux pilier de l'observabilité) pour un environnement Proxmox ou un Home-Lab Linux.

🚀 Composants de la Stack
Grafana : Visualisation et Dashboards.

Mimir : Stockage à long terme des métriques (Prometheus-compatible).
Loki : Agrégation et gestion des logs.
Tempo : Traçage distribué des requêtes.
Prometheus : Collecteur local (Agent) pour l'ingestion.
Promtail : Agent de collecte des logs système.

📡 Installation de la sonde sur Proxmox (Node Exporter)
Pour que Prometheus puisse récolter les métriques de votre serveur Proxmox (CPU, RAM, Disque, Réseau), vous devez installer une sonde (exporter) directement sur l'hôte Proxmox.

Connectez-vous en SSH à votre serveur Proxmox et lancez :

apt update && apt install prometheus-node-exporter -y

La sonde tournera automatiquement en tâche de fond sur le port 9100.
Dans le fichier prometheus.yml de votre stack Docker, assurez-vous d'avoir le même bloc que moi afin de lire la sonde. 

🛠️ Installation de la Stack Docker
1. Prérequis
Docker & Docker Compose installés sur la VM ou le CT qui héberge la stack.

Un utilisateur avec les droits sudo.

2. Clonage et Préparation

git clone <url-de-ton-repo>
cd lgtm-stack

3. Configuration des fichiers
Assurez-vous que les fichiers suivants sont présents à la racine :

docker-compose.yml
mimir-config.yml (Configuré en mode filesystem)
tempo.yaml (Configuré en mode target: all avec le grpc_listen_port: 9095)
prometheus.yml
promtail-config.yml

Note importante : Dans prometheus.yml, vérifiez que le point de sortie (remote_write) pointe bien vers Mimir sur son port interne : http://mimir:8080/api/v1/push.

4. Lancement

docker compose up -d

Service,Port Interne,Port Externe,Usage
Grafana,3000,3000,Interface Web
Mimir,8080,9009,Stockage Métriques
Loki,3100,3100,Stockage Logs
Tempo,3200,3200,API Traces
Tempo (OTLP),4317/4318,4317/4318,Réception Traces
Prometheus,9090,9090,Collecte (Scraping)

💡Troubleshooting (Le coin des galères)
Le cas "Mimir" (Erreur 404 / Connection Refused)
Si Grafana ne parvient pas à joindre Mimir sur le port 9009, utilisez le port interne 8080 dans la configuration de la Data Source Grafana : http://mimir:8080/prometheus.

Le cas "Tempo" (Kafka Error)
Si le conteneur Tempo redémarre en boucle avec une erreur concernant Kafka (topic not configured), vérifiez que votre tempo.yaml contient bien :
Le port gRPC explicite : grpc_listen_port: 9095.
Le bloc ingester configuré avec store: inmemory.
L'absence de configuration vide sur les protocoles (otlp).

📈 Dashboards Recommandés
Une fois connecté sur Grafana (admin/yourpassword), configurez vos Data Sources puis importez ces IDs pour un résultat immédiat :

10048 : Proxmox / LXC / VM (via Mimir)
1860 : Node Exporter Full (via Mimir)
15141 : Loki Logs Visualizer

📜 Licence
Projet libre d'utilisation. Amusez-vous bien avec vos données !

Fait avec ❤️ par Sasha
