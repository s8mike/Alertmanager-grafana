# 📊 Alertmanager Integration with Grafana Dashboard

This project sets up a local observability stack to integrate **Prometheus**, **Alertmanager**, and **Grafana** for visualizing alert statuses, silences, and notification flows.

---

## ✅ Objectives

* Set up and run Prometheus, Alertmanager, and Grafana in Docker containers.
* Configure alert rules to trigger test alerts.
* Integrate Alertmanager alerts into Grafana dashboards.
* Provide a clear visualization of firing/silenced alerts.
* Document step-by-step instructions for reproduction and future enhancements.

---

## 🛠️ System Requirements

> Ensure your system meets these minimum requirements:

* **Docker** installed and running
* **Docker Compose** installed
* 2 GB+ RAM (4 GB recommended)
* Internet connection (to pull Docker images)
* Ports `9090`, `9093`, `3000`, `9100`, and `8080` open (locally)

---

## 📁 Folder Structure

```bash
alertmanager-grafana/
├── docker-compose.yml
├── prometheus/
│   ├── prometheus.yml
│   └── alert.rules.yml
├── alertmanager/
│   └── alertmanager.yml
├── dashboards/
│   ├── alertmanager_alerts.json
│   ├── node-exporter-dashboard.json      # (Optional)
│   └── docker-monitoring-dashboard.json  # (Optional)
├── .gitignore
└── README.md
```

---

## 🚀 Quick Start

### 1. Clone this repository

```bash
git clone https://your-repo-url/alertmanager-grafana.git
cd alertmanager-grafana
```

### 2. Start the stack

```bash
docker-compose up -d --build
```

Wait for all containers to be up. Check with:

```bash
docker ps
```

---

## 📍 Access the Services

| Service      | URL                                            |
| ------------ | ---------------------------------------------- |
| Prometheus   | [http://localhost:9090](http://localhost:9090) |
| Alertmanager | [http://localhost:9093](http://localhost:9093) |
| Grafana      | [http://localhost:3000](http://localhost:3000) |

* **Grafana Login**: `admin / admin`

---

## 📦 Prometheus Configuration

### `prometheus/prometheus.yml`

* Scrapes metrics from:

  * Prometheus itself
  * Alertmanager
  * A **fake target** to simulate alerts (broken-target:1234)
  * (Optional) `node-exporter`, `cadvisor`

### `prometheus/alert.rules.yml`

Includes an alert rule for when a target is **down** for more than 30 seconds:

```yaml
groups:
- name: example-alerts
  rules:
  - alert: TargetDown
    expr: up == 0
    for: 30s
    labels:
      severity: critical
    annotations:
      summary: "Target {{ $labels.instance }} is down"
      description: "The job {{ $labels.job }} on {{ $labels.instance }} has been unreachable for more than 30 seconds."
```

---

## 🐙 Run Sample Queries in Prometheus

To verify your metrics are being collected correctly, perform the following in the Prometheus UI:

1. Open the **Graph** page at [http://localhost:9090](http://localhost:9090) → **Graph** → **Expression**.
2. Enter each of these expressions one at a time and click **Execute**:

   * `node_cpu_seconds_total`
   * `node_load1`
   * `container_memory_usage_bytes`
3. You should see live metric data or a graph indicating current values.

---

## 🧪 Testing Alerts

To simulate an alert:

1. Stop Prometheus manually:

```bash
docker stop prometheus
```

2. Visit:

   * [Alertmanager Alerts](http://localhost:9093)
   * [Prometheus Alerts](http://localhost:9090/alerts) (once restarted)

3. The alert **TargetDown** will fire based on the fake target.

4. You can also **Silence the alert** in Alertmanager UI for testing suppression.

---

## 📊 Grafana Dashboards

### Alertmanager Dashboard

Imported using:
* Data source: `prometheus`
* Query: `ALERTS{alertstate="firing"}`
* Visualization: `Table`

> Panels show: alert name, state, job, severity, etc.

### Importing Dashboards into Grafana

There are two options:

#### Option A — Upload from JSON File

* Go to **Grafana > Dashboards > Import**
* Upload the JSON file from `dashboards/`

#### Option B — Import by Grafana ID

| Exporter      | Grafana ID | Dashboard Name         | File Name                        |
| ------------- | ---------- | ---------------------- | -------------------------------- |
| Node Exporter | `1860`     | Node Exporter Full     | node-exporter-dashboard.json     |
| cAdvisor      | `193`      | Docker Container Stats | docker-monitoring-dashboard.json |
| Alertmanager  | *(manual)* | Custom (no public ID)  | alertmanager\_alerts.json        |

> ℹ️ **Alertmanager Dashboard Note**:
> This dashboard was manually created and exported. Grafana does **not** provide an official dashboard ID for Alertmanager.
> If needed, you can [submit your dashboard](https://grafana.com/grafana/dashboards) to obtain a public ID.

---

## 🛠️ Testing Silence

1. Visit [Alertmanager → Silences](http://localhost:9093/#/silences)
2. Create a new silence for the alert:

   * Matchers: `alertname=TargetDown`
   * Duration: e.g., `2h`
   * Comment: e.g., "Testing silence for #206"
3. Confirm alert is no longer firing in Grafana & Alertmanager.

📝 *Silence ends automatically after the duration.*

---

## ✨ Optional Enhancements

These are available in `docker-compose.yml` (enabled by default):

### ✔ Node Exporter (host system metrics)

```yaml
# ─── Optional: Node Exporter (for system metrics) ───
node-exporter:
  image: prom/node-exporter:latest
  container_name: node-exporter
  ports:
    - "9100:9100"
  networks:
    - monitoring
  restart: unless-stopped
```

### ✔ cAdvisor (container-level metrics)

```yaml
# ─── Optional: cAdvisor (for container-level metrics) ───
cadvisor:
  image: gcr.io/cadvisor/cadvisor:latest
  container_name: cadvisor
  ports:
    - "8080:8080"
  volumes:
    - /:/rootfs:ro
    - /var/run:/var/run:ro
    - /sys:/sys:ro
    - /var/lib/docker/:/var/lib/docker:ro
  networks:
    - monitoring
  restart: unless-stopped
```

### To **disable optional services**, comment them out in `docker-compose.yml`:

```yaml
# node-exporter:
#   ...
# cadvisor:
#   ...
```

> You may also delete their dashboards from Grafana under **Manage Dashboards**.

---

## 📌 Notes

* `docker-compose up -d` starts existing containers.
* `docker-compose up -d --build` rebuilds containers before starting.
* If you encounter zombie containers, consider adding `init: true` under affected services in `docker-compose.yml` to enable proper signal handling.
* All configs are tested on Docker Compose v2.
* Ideal for **local testing**, learning, or demo use.

---

## 😗 Clean Up

```bash
docker-compose down
```

---

## 👨‍💼 Maintained By

---

## 🔒 .gitignore

```gitignore
# Logs
*.log

# OS-generated
.DS_Store
Thumbs.db

# Docker
**/.docker
*.pid

# VSCode
.vscode/

# Python cache (if used)
__pycache__/
*.py[cod]

# Grafana Session Data
grafana/data/
```

---





# BACK UP README.md. WILL BE DELETED WHEN ABOVE IS CONFIRMED OKAY.


<An updated README.md snippet with a **Prerequisites / Tools Requirements** section added near the top, before the setup instructions.

---

```markdown
# Alertmanager & Grafana Integration

## Prerequisites / Tools Requirements

Before you begin, ensure you have the following installed on your local machine:

- **Docker** (version 20.x or later)  
  To run Prometheus, Alertmanager, Grafana, and exporters in containers.  
  [Install Docker](https://docs.docker.com/get-docker/)

- **Docker Compose** (version 1.29.x or later)  
  For managing multi-container Docker applications.  
  [Install Docker Compose](https://docs.docker.com/compose/install/)

- A modern **web browser** (Chrome, Firefox, Edge, etc.)  
  To access the Prometheus, Alertmanager, and Grafana web UIs.

- Optional (for advanced use):  
  - A code editor (VS Code, Sublime, etc.)  
  - Git (to clone and manage this repo)

---

## Overview

This project integrates **Alertmanager** into the **Grafana** dashboard to visualize alert statuses, silences, and notification flows. It enhances monitoring by providing visibility into active and historical alerts directly within Grafana.

---

## Folder Structure

```

alertmanager-grafana/
├── alertmanager/
│   └── alertmanager.yml
├── prometheus/
│   ├── prometheus.yml
│   └── alert.rules.yml
├── grafana/
│   ├── dashboards/
│   │   ├── node-exporter-dashboard.json
│   │   └── docker-monitoring-dashboard.json
│   └── provisioning/  (optional for auto provisioning)
├── docker-compose.yml
└── README.md

````

---

## Setup and Execution Steps

### 1. Start Services

- Run `docker-compose up -d --build` to launch Prometheus, Alertmanager, Grafana, and optional exporters.

### 2. Verify Prometheus & Alertmanager

- Access Prometheus UI: http://localhost:9090  
- Access Alertmanager UI: http://localhost:9093  

You should see your configured alerts (e.g., `TargetDown`) firing when simulating service failures.

### 3. Verify Grafana Dashboards

- Access Grafana UI: http://localhost:3000  
- Import dashboards JSON files manually via **Create > Import**:

  - `node-exporter-dashboard.json` (Optional system metrics)  
  - `docker-monitoring-dashboard.json` (Optional container metrics)  
  - `alertmanager-dashboard.json` (Main Alertmanager dashboard)

---

## Testing Alert Silence

Alert silences temporarily mute alerts to prevent notifications during known maintenance or incidents.

**Steps:**

1. In Alertmanager UI, navigate to **Silences** tab.

2. Click **New Silence**.

3. Set **Matchers** to target specific alert labels (e.g., `alertname=TargetDown`, `instance=broken-target:1234`).

4. Set **Duration** (e.g., 2 hours).

5. Add **Creator** name and optional **Comment** (e.g., "Testing silence for #206").

6. Save the silence.

**Effect:**  
During silence, matching alerts will be suppressed in Alertmanager and Grafana dashboards. After the duration expires, alerts resume firing if unresolved.

---

## Managing Dashboards JSON Files

- To export dashboards from Grafana UI:

  1. Open desired dashboard.

  2. Click **Share** → **Export** → **Save to file**.

  3. Save JSON into `grafana/dashboards/`.

- To import dashboards into Grafana:

  1. Log into Grafana.

  2. Click **"+"** (Create) → **Import**.

  3. Upload JSON file from `grafana/dashboards/`.

- Alternatively, use Grafana's provisioning feature (advanced) to auto-load dashboards from filesystem at startup.

---

## Optional Enhancements (Commented Out)

Optional exporters like **Node Exporter** and **cAdvisor** are included in the `docker-compose.yml` as commented blocks for future use.

```yaml
# ─────────── Optional: Node Exporter (for system metrics) ───────────
# node-exporter:
#   image: prom/node-exporter:latest
#   container_name: node-exporter
#   ports:
#     - "9100:9100"
#   networks:
#     - monitoring
#   restart: unless-stopped

# ─────────── Optional: cAdvisor (for container-level metrics) ───────────
# cadvisor:
#   image: gcr.io/cadvisor/cadvisor:latest
#   container_name: cadvisor
#   ports:
#     - "8080:8080"
#   volumes:
#     - /:/rootfs:ro
#     - /var/run:/var/run:ro
#     - /sys:/sys:ro
#     - /var/lib/docker/:/var/lib/docker:ro
#   networks:
#     - monitoring
#   restart: unless-stopped
````

Uncomment these sections in `docker-compose.yml` to enable them later.

---

## Summary

* Prometheus, Alertmanager, and Grafana are running locally via Docker Compose.

* Alert rules configured and firing properly.

* Alert silences can be created and expire automatically.

* Grafana dashboards imported manually from JSON files.

* Optional exporters included for future monitoring enhancements.

---

## Next Steps

* Continue to build and customize Grafana Alertmanager dashboards.

* Extend alert rules to real-world services (e.g., Node Exporter).

* Implement auto dashboard provisioning (optional).

---

# End of README


#README.md 2

Great! Here's the **complete and ready-to-save `README.md`** file for your project, based on everything we've implemented for **Ticket #206 – Integrate Alertmanager into Grafana Dashboard**, including the optional enhancements.

---

### 📁 `README.md`

````markdown
# 📊 Alertmanager Integration with Grafana Dashboard

This project sets up a local observability stack to integrate **Prometheus**, **Alertmanager**, and **Grafana** for visualizing alert statuses, silences, and notification flows.

---

## ✅ Objectives

- Set up and run Prometheus, Alertmanager, and Grafana in Docker containers.
- Configure alert rules to trigger test alerts.
- Integrate Alertmanager alerts into Grafana dashboards.
- Provide a clear visualization of firing/silenced alerts.
- Document step-by-step instructions for reproduction and future enhancements.

---

## 🛠️ System Requirements

> Ensure your system meets these minimum requirements:

- **Docker** installed and running
- **Docker Compose** installed
- 2 GB+ RAM (4 GB recommended)
- Internet connection (to pull Docker images)
- Ports `9090`, `9093`, `3000`, `9100`, and `8080` open (locally)

---

## 📁 Folder Structure

```bash
alertmanager-grafana/
├── docker-compose.yml
├── prometheus/
│   ├── prometheus.yml
│   └── alert.rules.yml
├── alertmanager/
│   └── alertmanager.yml
├── dashboards/
│   ├── alertmanager_alerts.json
│   ├── node-exporter-dashboard.json      # (Optional)
│   └── docker-monitoring-dashboard.json  # (Optional)
└── README.md
````

---

## 🚀 Quick Start

### 1. Clone this repository

```bash
git clone https://your-repo-url/alertmanager-grafana.git
cd alertmanager-grafana
```

### 2. Start the stack

```bash
docker-compose up -d --build
```

Wait for all containers to be up. Check with:

```bash
docker ps
```

---

## 📍 Access the Services

| Service      | URL                                            |
| ------------ | ---------------------------------------------- |
| Prometheus   | [http://localhost:9090](http://localhost:9090) |
| Alertmanager | [http://localhost:9093](http://localhost:9093) |
| Grafana      | [http://localhost:3000](http://localhost:3000) |

* **Grafana Login**: `admin / admin`

---

## 📦 Prometheus Configuration

### `prometheus/prometheus.yml`

* Scrapes metrics from:

  * Prometheus itself
  * Alertmanager
  * A **fake target** to simulate alerts (broken-target:1234)
  * (Optional) `node-exporter`, `cadvisor`

### `prometheus/alert.rules.yml`

Includes an alert rule for when a target is **down** for more than 30 seconds:

```yaml
groups:
- name: example-alerts
  rules:
  - alert: TargetDown
    expr: up == 0
    for: 30s
    labels:
      severity: critical
    annotations:
      summary: "Target {{ $labels.instance }} is down"
      description: "The job {{ $labels.job }} on {{ $labels.instance }} has been unreachable for more than 30 seconds."
```

---

## 🧪 Testing Alerts

To simulate an alert:

1. Stop Prometheus manually:

```bash
docker stop prometheus
```

2. Visit:

   * [Alertmanager Alerts](http://localhost:9093)
   * [Prometheus Alerts](http://localhost:9090/alerts) (once restarted)

3. The alert **TargetDown** will fire based on the fake target.

4. You can also **Silence the alert** in Alertmanager UI for testing suppression.

---

## 📊 Grafana Dashboards

### Alertmanager Dashboard

Imported using:

* Query: `ALERTS{alertstate="firing"}`
* Visualization: `Table` or `Stat` panel

> Panels show: alert name, state, job, severity, etc.

### Importing Dashboards from JSON

* Go to **Grafana > Dashboards > Import**
* Upload the JSON file from `dashboards/alertmanager_alerts.json`
* (Optionally import `node-exporter-dashboard.json`, `docker-monitoring-dashboard.json`)

---

## 🧪 Testing Silence

1. Visit [Alertmanager → Silences](http://localhost:9093/#/silences)
2. Create a new silence for the alert:

   * Matchers: `alertname=TargetDown`
   * Duration: e.g., `2h`
   * Comment: e.g., "Testing silence for #206"
3. Confirm alert is no longer firing in Grafana & Alertmanager.

📝 *Silence ends automatically after the duration.*

---

## ✨ Optional Enhancements

These are available in `docker-compose.yml` (enabled by default):

### ✔ Node Exporter (host system metrics)

```yaml
# ─────────── Optional: Node Exporter (for system metrics) ───────────
node-exporter:
  image: prom/node-exporter:latest
  ...
```

### ✔ cAdvisor (container-level metrics)

```yaml
# ─────────── Optional: cAdvisor (for container-level metrics) ───────────
cadvisor:
  image: gcr.io/cadvisor/cadvisor:latest
  ...
```

### To **disable optional services**, comment them out in `docker-compose.yml`:

```yaml
# node-exporter:
#   ...
# cadvisor:
#   ...
```

> You may also delete their dashboards from Grafana under **Manage Dashboards**.

---

## 📌 Notes

* All configs are tested on Docker Compose v2.
* Prometheus UID-based dashboards used for Alertmanager/Node Exporter.
* This setup is ideal for **local testing**, learning, or demo use.

---

## 🏁 Final Result (Screenshots)

Include screenshots of:

* Alert firing in Alertmanager
* Alert visible in Grafana
* Silence state
* Dashboards from Node Exporter and Docker (if enabled)

---

## 📚 Documentation Summary

| Component    | Config Path                     | Dashboard JSON                    |
| ------------ | ------------------------------- | --------------------------------- |
| Prometheus   | `prometheus/prometheus.yml`     | —                                 |
| Alert Rules  | `prometheus/alert.rules.yml`    | —                                 |
| Alertmanager | `alertmanager/alertmanager.yml` | `alertmanager_alerts.json`        |
| Grafana      | (volumes, password in compose)  | Node/Docker (optional) dashboards |
| Optional     | Node Exporter, cAdvisor         | `node-exporter-dashboard.json`    |

---

## 📦 Clean Up

To stop and remove everything:

```bash
docker-compose down
```

---

```

---

# README.md 3 (option 3)

# 📊 Alertmanager Integration with Grafana Dashboard

This project sets up a local observability stack to integrate **Prometheus**, **Alertmanager**, and **Grafana** for visualizing alert statuses, silences, and notification flows.

---

## ✅ Objectives

* Set up and run Prometheus, Alertmanager, and Grafana in Docker containers.
* Configure alert rules to trigger test alerts.
* Integrate Alertmanager alerts into Grafana dashboards.
* Provide a clear visualization of firing/silenced alerts.
* Document step-by-step instructions for reproduction and future enhancements.

---

## 🛠️ System Requirements

> Ensure your system meets these minimum requirements:

* **Docker** installed and running
* **Docker Compose** installed
* 2 GB+ RAM (4 GB recommended)
* Internet connection (to pull Docker images)
* Ports `9090`, `9093`, `3000`, `9100`, and `8080` open (locally)

---

## 📁 Folder Structure

```bash
alertmanager-grafana/
├── docker-compose.yml
├── prometheus/
│   ├── prometheus.yml
│   └── alert.rules.yml
├── alertmanager/
│   └── alertmanager.yml
├── dashboards/
│   ├── alertmanager_alerts.json
│   ├── node-exporter-dashboard.json      # (Optional)
│   └── docker-monitoring-dashboard.json  # (Optional)
├── .gitignore
└── README.md
```

---

## 🚀 Quick Start

### 1. Clone this repository

```bash
git clone https://your-repo-url/alertmanager-grafana.git
cd alertmanager-grafana
```

### 2. Start the stack

```bash
docker-compose up -d --build
```

Wait for all containers to be up. Check with:

```bash
docker ps
```

---

## 📍 Access the Services

| Service      | URL                                            |
| ------------ | ---------------------------------------------- |
| Prometheus   | [http://localhost:9090](http://localhost:9090) |
| Alertmanager | [http://localhost:9093](http://localhost:9093) |
| Grafana      | [http://localhost:3000](http://localhost:3000) |

* **Grafana Login**: `admin / admin`

---

## 📦 Prometheus Configuration

### `prometheus/prometheus.yml`

* Scrapes metrics from:

  * Prometheus itself
  * Alertmanager
  * A **fake target** to simulate alerts (broken-target:1234)
  * (Optional) `node-exporter`, `cadvisor`

### `prometheus/alert.rules.yml`

Includes an alert rule for when a target is **down** for more than 30 seconds:

```yaml
groups:
- name: example-alerts
  rules:
  - alert: TargetDown
    expr: up == 0
    for: 30s
    labels:
      severity: critical
    annotations:
      summary: "Target {{ $labels.instance }} is down"
      description: "The job {{ $labels.job }} on {{ $labels.instance }} has been unreachable for more than 30 seconds."
```

---

## 🧪 Testing Alerts

To simulate an alert:

1. Stop Prometheus manually:

```bash
docker stop prometheus
```

2. Visit:

   * [Alertmanager Alerts](http://localhost:9093)
   * [Prometheus Alerts](http://localhost:9090/alerts) (once restarted)

3. The alert **TargetDown** will fire based on the fake target.

4. You can also **Silence the alert** in Alertmanager UI for testing suppression.

---

## 📊 Grafana Dashboards

### Alertmanager Dashboard

Imported using:

* Query: `ALERTS{alertstate="firing"}`
* Visualization: `Table`

> Panels show: alert name, state, job, severity, etc.

### Importing Dashboards from JSON

* Go to **Grafana > Dashboards > Import**
* Upload the JSON file from `dashboards/`

#### UID References

| Dashboard           | File                             | UID Example    |
| ------------------- | -------------------------------- | -------------- |
| Alertmanager Alerts | alertmanager\_alerts.json        | `alertmanager` |
| Node Exporter Full  | node-exporter-dashboard.json     | `node`         |
| Docker Monitoring   | docker-monitoring-dashboard.json | `docker`       |

---

## 🧰 Testing Silence

1. Visit [Alertmanager → Silences](http://localhost:9093/#/silences)
2. Create a new silence for the alert:

   * Matchers: `alertname=TargetDown`
   * Duration: e.g., `2h`
   * Comment: e.g., "Testing silence for #206"
3. Confirm alert is no longer firing in Grafana & Alertmanager.

📝 *Silence ends automatically after the duration.*

---

## ✨ Optional Enhancements

These are available in `docker-compose.yml` (enabled by default):

### ✔ Node Exporter (host system metrics)

```yaml
# ─── Optional: Node Exporter (for system metrics) ───
node-exporter:
  image: prom/node-exporter:latest
  container_name: node-exporter
  ports:
    - "9100:9100"
  networks:
    - monitoring
  restart: unless-stopped
```

### ✔ cAdvisor (container-level metrics)

```yaml
# ─── Optional: cAdvisor (for container-level metrics) ───
cadvisor:
  image: gcr.io/cadvisor/cadvisor:latest
  container_name: cadvisor
  ports:
    - "8080:8080"
  volumes:
    - /:/rootfs:ro
    - /var/run:/var/run:ro
    - /sys:/sys:ro
    - /var/lib/docker/:/var/lib/docker:ro
  networks:
    - monitoring
  restart: unless-stopped
```

### To **disable optional services**, comment them out in `docker-compose.yml`:

```yaml
# node-exporter:
#   ...
# cadvisor:
#   ...
```

