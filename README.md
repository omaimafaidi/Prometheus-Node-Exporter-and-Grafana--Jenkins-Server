# Prometheus-Node-Exporter-and-Grafana--Jenkins-Server
# Mise à jour des listes de paquets
sudo apt update -y
## Installation de Prometheus et création de son service ##
# Création d'un utilisateur système pour Prometheus 
sudo useradd --system --no-create-home --shell /bin/false prometheus 
# Téléchargement de Prometheus (version 2.47.1) 
wget https://github.com/prometheus/prometheus/releases/download/v2.47.1/prometheus-2.47.1.linux-amd64.tar.gz 
# Décompression de l'archive 
tar -xvf prometheus-2.47.1.linux-amd64.tar.gz 
# Navigation dans le répertoire décompressé 
cd prometheus-2.47.1.linux-amd64/ 
# Création des répertoires de données et de configuration 
sudo mkdir -p /data /etc/prometheus 
# Déplacement des exécutables Prometheus et promtool 
sudo mv prometheus promtool /usr/local/bin/ 
# Déplacement des fichiers de console et de bibliothèques 
sudo mv consoles/ console_libraries/ /etc/prometheus/ 
# Déplacement du fichier de configuration 
sudo mv prometheus.yml /etc/prometheus/prometheus.yml 
# Attribution des droits de propriétaire aux répertoires de données et de configuration 
sudo chown -R prometheus:prometheus /etc/prometheus/ /data/ 
# Création du service systemd pour Prometheus
sudo cat > /etc/systemd/system/node_exporter.service << EOF
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

StartLimitIntervalSec=500
StartLimitBurst=5

[Service]
User=node_exporter
Group=node_exporter
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/node_exporter --collector.logind

[Install]
WantedBy=multi-user.target
EOF
# Activation et démarrage du service Prometheus 
sudo systemctl enable prometheus 
sudo systemctl start prometheus 
## Installation de Node Exporter et création de son service ## 
# Création d'un utilisateur système pour Node Exporter 
sudo useradd --system --no-create-home --shell /bin/false node_exporter # Téléchargement de Node Exporter (version 1.6.1) 
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz 
# Décompression de l'archive 
tar -xvf node_exporter-1.6.1.linux-amd64.tar.gz 
# Déplacement de l'exécutable Node Exporter 
sudo mv node_exporter-1.6.1.linux-amd64/node_exporter /usr/local/bin/ 
# Suppression du répertoire décompressé 
rm -rf node_exporter* 
# Création du service systemd pour Node Exporter
sudo cat > /etc/systemd/system/node_exporter.service << EOF
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

StartLimitIntervalSec=500
StartLimitBurst=5

[Service]
User=node_exporter
Group=node_exporter
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/node_exporter --collector.logind

[Install]
WantedBy=multi-user.target
EOF
# Activation et démarrage du node_exporter
sudo systemctl enable node_exporter
sudo systemctl start node_exporter
## Installation de Grafana ##
# Mise à jour de la liste des paquetages 
sudo apt-get update 
# Installation des prérequis 
sudo apt-get install -y apt-transport-https software-properties-common # Ajout de la clé GPG de Grafana 
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add - # Ajout du dépôt Grafana 
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list 
# Mise à jour de la liste des paquetages 
sudo apt-get update 
# Installation de Grafana 
sudo apt-get install -y grafana 
# Activation et démarrage du service Grafana 
sudo systemctl enable grafana-server 
sudo systemctl start grafana-server 
# Vérification de l'état du service Prometheus 
sudo systemctl status prometheus 
# Vérification de l'état du service Node Exporter 
sudo systemctl status node_exporter
# Vérification de l'état du service Grafana 
sudo systemctl status grafana-server 
# Accès au répertoire de configuration de Prometheus 
cd /etc/prometheus/ 
# Modification du fichier de configuration de Prometheus 
sudo nano prometheus.yml
# Ajout de la configuration pour Node Exporter (remplacer 'IP-Adresse' par l'adresse IP réelle) 
- job_name: 'node_exporter' 
  static_configs: 
    - targets: ['IP-Adresse:9100']
# Vérification de la syntaxe de la configuration de Prometheus 
promtool check config /etc/prometheus/prometheus.yml 
# Rechargement de la configuration de Prometheus 
curl -X POST http://localhost:9090/-/reload 
# Accès au répertoire de configuration de Prometheus
cd /etc/prometheus/ 
# Modification du fichier de configuration de Prometheus 
sudo nano prometheus.yml
# Ajout de la configuration pour Jenkins (remplacer 'IP-Adresse' par l'adresse IP réelle du serveur Jenkins) 
- job_name: 'jenkins' 
    metrics_path: '/prometheus' 
    static_configs: 
      - targets: ['IP-Adresse:8080']
