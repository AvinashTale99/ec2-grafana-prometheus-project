
# 📊 EC2 Monitoring Setup with Prometheus, Grafana & Node Exporter

This guide walks you through creating a GitHub repository and setting up a full monitoring stack (Prometheus, Grafana, and Node Exporter) on an EC2 Linux instance using a custom script.

---

## 🛠️ Step 1: Create a GitHub Repository

1. Go to [https://github.com](https://github.com)
2. Click on **New** to create a repository.
3. Name it something like `ec2-monitoring-setup`
4. Initialize with a README (optional)

---

## 📁 Step 2: Clone Your Repository into EC2

```bash
sudo yum install git -y
git config --global user.name "Your Name"
git config --global user.email "your@email.com"

git clone https://github.com/YOUR_USERNAME/ec2-monitoring-setup.git
cd ec2-monitoring-setup
```

---

## 📂 Step 3: Create Folder Structure

```bash
mkdir -p ec2-grafana-prometheus-project
cd ec2-grafana-prometheus-project
```

---

## 📜 Step 4: Create `script.sh` File

Create a file called `script.sh` in the `ec2-grafana-prometheus-project` folder and paste the following content:

<details>
<summary>Click to expand the script content</summary>

```bash
#!/bin/bash

# Exit on error
set -e

# Ensure root
if [ "$EUID" -ne 0 ]; then
  echo "❌ Please run as root or use sudo"
  exit 1
fi

# Set versions
PROM_VERSION="2.52.0"
NODE_EXPORTER_VERSION="1.8.0"
GRAFANA_RPM="https://dl.grafana.com/enterprise/release/grafana-enterprise-12.0.2-1.x86_64.rpm"

echo "🔄 Updating system packages..."
dnf update -y

############################################
# 1. Grafana Installation
############################################
echo "📦 Installing Grafana Enterprise..."
dnf install -y $GRAFANA_RPM || echo "⚠️ Grafana may already be installed"
systemctl enable --now grafana-server
grafana_version=$(grafana-server -v | head -n 1)
echo "✅ Grafana installed successfully. Version: $grafana_version"

############################################
# 2. Prometheus Installation
############################################

# Add Prometheus user if not exists
if id "prometheus" &>/dev/null; then
    echo "ℹ️ User 'prometheus' already exists."
else
    useradd --no-create-home --shell /sbin/nologin prometheus
fi

# Create necessary directories
mkdir -p /etc/prometheus /var/lib/prometheus
chown -R prometheus:prometheus /etc/prometheus /var/lib/prometheus

# Download and extract Prometheus
cd /tmp
wget -q https://github.com/prometheus/prometheus/releases/download/v$PROM_VERSION/prometheus-$PROM_VERSION.linux-amd64.tar.gz
tar -xzf prometheus-$PROM_VERSION.linux-amd64.tar.gz
cd prometheus-$PROM_VERSION.linux-amd64

# Install binaries and configs
install -m 0755 prometheus promtool /usr/local/bin/
cp -r consoles console_libraries /etc/prometheus/
cp prometheus.yml /etc/prometheus/
touch /etc/prometheus/rules.yml
chown -R prometheus:prometheus /etc/prometheus /usr/local/bin/prometheus /usr/local/bin/promtool

# Create systemd service
cat <<EOF > /etc/systemd/system/prometheus.service
[Unit]
Description=Prometheus Monitoring
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \\
  --config.file=/etc/prometheus/prometheus.yml \\
  --storage.tsdb.path=/var/lib/prometheus/ \\
  --web.console.templates=/etc/prometheus/consoles \\
  --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reexec
systemctl daemon-reload
systemctl enable --now prometheus
echo "✅ Prometheus installed and running."

############################################
# 3. Node Exporter Installation
############################################

# Add Node Exporter user if not exists
if id "node_exporter" &>/dev/null; then
    echo "ℹ️ User 'node_exporter' already exists."
else
    useradd --no-create-home --shell /sbin/nologin node_exporter
fi

cd /tmp
wget -q https://github.com/prometheus/node_exporter/releases/download/v$NODE_EXPORTER_VERSION/node_exporter-$NODE_EXPORTER_VERSION.linux-amd64.tar.gz
tar -xzf node_exporter-$NODE_EXPORTER_VERSION.linux-amd64.tar.gz

# Stop node_exporter if already running
if systemctl is-active --quiet node_exporter; then
    echo "⏹️ Stopping node_exporter before update..."
    systemctl stop node_exporter
fi

# Replace binary
cp -f node_exporter-$NODE_EXPORTER_VERSION.linux-amd64/node_exporter /usr/local/bin/
chown node_exporter:node_exporter /usr/local/bin/node_exporter

# Create Node Exporter systemd service
cat <<EOF > /etc/systemd/system/node_exporter.service
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable --now node_exporter
echo "✅ Node Exporter installed and running."

############################################
# Summary
############################################
echo -e "\n🎉 Monitoring stack setup complete!"
echo "➡️ Prometheus: http://<your-ec2-ip>:9090"
echo "➡️ Grafana:    http://<your-ec2-ip>:3000 (login: admin / admin)"
echo "➡️ Node Exporter: http://<your-ec2-ip>:9100"
```

</details>

---

## 🚀 Step 5: Make It Executable and Run

```bash
chmod +x script.sh
sudo ./script.sh
```

---

## 🌐 Access Your Monitoring Tools

Replace `<your-ec2-ip>` with your instance’s public IP:

- Prometheus: [http://your-ec2-ip:9090](http://your-ec2-ip:9090)
- Grafana: [http://your-ec2-ip:3000](http://your-ec2-ip:3000)
- Node Exporter: [http://your-ec2-ip:9100](http://your-ec2-ip:9100)

## 👨‍💻 Author

**Avinash Tale**

### 🔗 Connect with Me

- 💼 [LinkedIn](https://www.linkedin.com/in/avinash-tale-3348b7217/)
- 🐙 [GitHub](https://github.com/AvinashTale99)
- 🐳 [Docker Hub](https://hub.docker.com/u/avinashtale99)
- 📷 [Instagram](https://www.instagram.com/avinash_tale_patil)
- 🌐 [Website](https://avinashtale99.github.io/AvinashRepo/)


