# Prometheus + Grafana Monitoring Stack on Rocky Linux

> Full production-grade monitoring setup using Prometheus 3.11.3, Node Exporter 1.11.1, and Grafana on Rocky Linux — with live CPU, memory, disk, and network dashboards accessible from a browser.

**Environment:** Rocky Linux (Server) + Ubuntu (Client) inside Oracle VirtualBox
**Server IP:** 192.168.200.5

📝 **Full write-up with screenshots:** [Medium Article](https://medium.com/@gurungatwork98/how-to-set-up-prometheus-and-grafana-monitoring-on-rocky-linux-64f8bfb41db4?postPublishedType=repub)

---

## Architecture

```
Rocky Linux Server                         Ubuntu Browser
┌──────────────────────────┐               ┌─────────────────┐
│  Node Exporter :9100     │               │                 │
│  Collects system metrics │               │    Grafana      │
│           ↓              │──────────────►│    Dashboard    │
│  Prometheus :9090        │               │                 │
│  Scrapes & stores data   │               └─────────────────┘
│           ↓              │
│  Grafana :3000           │
│  Visualizes everything   │
└──────────────────────────┘
```

---

## Versions Used

| Component | Version |
|---|---|
| Node Exporter | 1.11.1 |
| Prometheus | 3.11.3 |
| Grafana | Latest stable |
| Rocky Linux | 10 |

---

## Ports

| Service | Port | Purpose |
|---|---|---|
| Node Exporter | 9100 | Raw metrics endpoint |
| Prometheus | 9090 | Web UI and API |
| Grafana | 3000 | Dashboard UI |

---

## Files in This Folder

| File | Description |
|---|---|
| `prometheus.yml` | Prometheus scrape configuration |
| `node_exporter.service` | Systemd unit file for Node Exporter |
| `prometheus.service` | Systemd unit file for Prometheus |
| `grafana.repo` | Grafana DNF repository file |

---

## Quick Setup

### 1. Node Exporter

```bash
# Create user
sudo useradd --no-create-home --shell /bin/false node_exporter

# Download and install
cd /tmp
wget https://github.com/prometheus/node_exporter/releases/download/v1.11.1/node_exporter-1.11.1.linux-amd64.tar.gz
tar -xzf node_exporter-1.11.1.linux-amd64.tar.gz
sudo mv node_exporter-1.11.1.linux-amd64/node_exporter /usr/local/bin/
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
rm -rf node_exporter-1.11.1.linux-amd64*

# Install and start service
sudo cp node_exporter.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable node_exporter --now

# Open firewall
sudo firewall-cmd --permanent --add-port=9100/tcp
sudo firewall-cmd --reload
```

### 2. Prometheus

```bash
# Create user and directories
sudo useradd --no-create-home --shell /bin/false prometheus
sudo mkdir -p /etc/prometheus /var/lib/prometheus
sudo chown prometheus:prometheus /etc/prometheus /var/lib/prometheus

# Download and install
cd /tmp
wget https://github.com/prometheus/prometheus/releases/download/v3.11.3/prometheus-3.11.3.linux-amd64.tar.gz
tar -xzf prometheus-3.11.3.linux-amd64.tar.gz
cd prometheus-3.11.3.linux-amd64
sudo cp prometheus promtool /usr/local/bin/
sudo chown prometheus:prometheus /usr/local/bin/prometheus /usr/local/bin/promtool

# Install config and service
sudo cp ../prometheus.yml /etc/prometheus/
sudo chown prometheus:prometheus /etc/prometheus/prometheus.yml
sudo cp ../prometheus.service /etc/systemd/system/

# Validate config
/usr/local/bin/promtool check config /etc/prometheus/prometheus.yml

# Start service
sudo systemctl daemon-reload
sudo systemctl enable prometheus --now

# Open firewall
sudo firewall-cmd --permanent --add-port=9090/tcp
sudo firewall-cmd --reload
```

### 3. Grafana

```bash
# Add repo and install
sudo cp grafana.repo /etc/yum.repos.d/
sudo dnf install grafana -y

# Start service
sudo systemctl enable grafana-server --now

# Open firewall
sudo firewall-cmd --permanent --add-port=3000/tcp
sudo firewall-cmd --reload
```

---

## Accessing the Stack

| Interface | URL | Credentials |
|---|---|---|
| Prometheus UI | http://192.168.200.5:9090 | No login required |
| Grafana Dashboard | http://192.168.200.5:3000 | admin / admin |

---

## Grafana Setup Steps

1. Login at `http://192.168.200.5:3000`
2. Go to **Connections → Data Sources → Add data source**
3. Select **Prometheus** and set URL to `http://localhost:9090`
4. Click **Save and Test** — confirm green success message
5. Go to **Dashboards → Import**
6. Enter dashboard ID `1860` and click **Load**
7. Select your Prometheus data source and click **Import**

---

## Verification

```bash
# Check all three services are running
sudo systemctl status node_exporter
sudo systemctl status prometheus
sudo systemctl status grafana-server

# Verify Node Exporter is collecting metrics
curl http://localhost:9100/metrics | head -10

# Validate Prometheus config
/usr/local/bin/promtool check config /etc/prometheus/prometheus.yml
```

---

## What I Learned

- Prometheus uses a pull model — it scrapes targets rather than targets pushing data to it
- Node Exporter exposes over 1000 system metrics through a simple HTTP endpoint on port 9100
- Always validate `prometheus.yml` with `promtool` before restarting the service
- YAML indentation errors are the most common cause of Prometheus startup failures
- Grafana dashboard ID 1860 is the community standard Node Exporter dashboard
- `--storage.tsdb.retention.time=30d` keeps 30 days of metrics history

---

## Connect

- 📝 **Medium:** [medium.com/@gurungatwork98](https://medium.com/@gurungatwork98)
- 💻 **GitHub:** [github.com/Rags-cyber/rocky-linux-sysadmin-labs](https://github.com/Rags-cyber/rocky-linux-sysadmin-labs)
