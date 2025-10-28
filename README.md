# 🧭 Chapter 8: Monitoring Kubernetes Cluster

In this chapter, we’ll learn how to monitor a **Kubernetes cluster** using **Prometheus** and **Grafana**.  
Monitoring provides visibility into cluster health, application performance, and potential issues before they become critical.

---

## 🚀 1. Why Monitoring Matters

Think of monitoring as a **doctor’s check-up** for your cluster:

| Concept | Analogy | Description |
|----------|----------|-------------|
| **Metrics** | Heartbeat / Blood pressure | Key numerical indicators of system health (CPU, memory, etc.) |
| **Dashboards** | Health reports | Visual panels to easily identify performance trends or issues |

Monitoring helps you:
- Detect abnormal resource usage.
- Track pod/node availability.
- Measure application latency and throughput.
- Ensure system reliability and performance.

---

## ⚙️ 2. Install Prometheus & Grafana

The simplest way to deploy Prometheus and Grafana in Kubernetes is using the **Helm chart** called **kube-prometheus-stack**.

### 🧩 Commands:

```bash
# Add Prometheus Helm repository
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

# Create a namespace for monitoring components
kubectl create namespace monitoring

# Install kube-prometheus-stack (includes Prometheus, Grafana, Alertmanager, etc.)
helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack -n monitoring
```

> 📝 **Note:**  
> The `kube-prometheus-stack` Helm chart bundles:
> - **Prometheus** → Metrics collection engine  
> - **Grafana** → Visualization dashboard  
> - **Prometheus Operator** → Manages Prometheus instances  
> - **Alertmanager** → Handles alert notifications  
> - **Node Exporter** → Node-level system metrics  
> - **Kube-State-Metrics** → Kubernetes object metrics

---

### ✅ Verify Installation

```bash
kubectl get pods -n monitoring
kubectl get svc -n monitoring
```

You should see pods for:
- Prometheus  
- Grafana  
- Node Exporter  
- Alertmanager  
- Kube-State-Metrics  

Example output:
```
pod/kube-prometheus-stack-grafana-xxxxxxx     Running
pod/prometheus-kube-prometheus-stack-xxxxxxx  Running
pod/kube-prometheus-stack-prometheus-node-exporter-xxxxxxx  Running
```

---

## 🌐 3. Access Prometheus Dashboard

To view Prometheus metrics and targets:

```bash
kubectl port-forward svc/kube-prometheus-stack-prometheus -n monitoring 9090:9090 --address=0.0.0.0 &
```

Then open in browser:  
👉 **http://localhost:9090**

You can now:
- View **targets**: [Status → Targets] to confirm all exporters are `UP`
- Run PromQL queries such as:
  ```promql
  sum(rate(node_cpu_seconds_total{mode!="idle"}[5m])) by (instance)
  ```
  or
  ```promql
  node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes
  ```

**Prometheus UI Example:**

![prometheus-ui](output_images/image-15.png)

---

## 📊 4. Access Grafana Dashboard

Port-forward Grafana service:

```bash
kubectl port-forward svc/kube-prometheus-stack-grafana -n monitoring 3000:80 --address=0.0.0.0 &
```

Open browser → **http://localhost:3000**

### 🔑 Default Login:
```
Username: admin
Password: prom-operator
```

*(If the password doesn’t work, decode it from the Kubernetes secret:)*  
**Linux/macOS**
```bash
kubectl get secret kube-prometheus-stack-grafana -n monitoring -o jsonpath="{.data.admin-password}" | base64 --decode; echo
```
**Windows PowerShell**
```powershell
kubectl get secret kube-prometheus-stack-grafana -n monitoring -o jsonpath="{.data.admin-password}" | % { [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($_)) }
```

---

### 🎨 Grafana UI Walkthrough

**Login Page:**
![grafana-login](output_images/image-9.png)

**Grafana Home:**
![grafana-home](output_images/image-10.png)

**Pre-Configured Data Source:**
Grafana automatically detects Prometheus as a data source.
<img width="1902" height="867" alt="image" src="https://github.com/user-attachments/assets/64744750-426d-411e-aa3d-3b4ea914bdf7" />


---

## 🧠 5. Import Grafana Dashboards

Grafana lets you import ready-made dashboards to visualize Kubernetes and node metrics easily.

### Steps:

1. Go to → [https://grafana.com/grafana/dashboards/?search=kubernetes](https://grafana.com/grafana/dashboards/?search=kubernetes)

<img width="1432" height="792" alt="image" src="https://github.com/user-attachments/assets/b902bdaa-1ebb-4ae8-9311-06745b38b933" />

2. Find a dashboard (e.g., **Kubernetes Cluster Monitoring**, **Node Exporter Full**, etc.)
3. Copy its dashboard ID
4. In Grafana:
   - Click **+ (Create)** → **Import**
   - Paste the dashboard ID → Select data source = **Prometheus**
   - Click **Import**

### Popular Dashboard IDs:
| Dashboard | ID |
|------------|----|
| Node Exporter Full | `1860` |
| Kubernetes Cluster Monitoring (via Prometheus) | `315` |
| Kubernetes / Compute Resources / Namespace (Pods) | `6417` |

You’ll now see detailed panels for:
- Node CPU & memory usage  
- Pod resource utilization  
- Cluster-level resource overview  
- Disk and network I/O metrics  

---

## 🩺 6. Validation & Troubleshooting

| Issue | Check Command | Solution |
|--------|----------------|-----------|
| Prometheus targets DOWN | `kubectl get servicemonitor -n monitoring` | Ensure node-exporter / kube-state-metrics services exist |
| Grafana login fails | Get password from secret | Use decoded value |
| Missing dashboards | Import dashboard IDs manually | Use Grafana UI → Import |
| No node metrics | Verify node-exporter DaemonSet | `kubectl get ds -n monitoring` |

---

## 📘 Summary

| Component | Purpose |
|------------|----------|
| **Prometheus** | Scrapes and stores metrics |
| **Grafana** | Visualizes metrics as dashboards |
| **Alertmanager** | Sends alerts when thresholds exceed |
| **Node Exporter** | Reports node-level system metrics |
| **Kube-State-Metrics** | Exposes metrics about Kubernetes resources |

With these steps, you can now **monitor your entire Kubernetes cluster**, visualize performance data, and build insightful dashboards to ensure your system stays healthy and reliable.

---

✅ **You have successfully:**
1. Installed `kube-prometheus-stack`  
2. Accessed **Prometheus** (metrics)  
3. Accessed **Grafana** (visual dashboards)  
4. Imported custom dashboards for node & cluster monitoring  

---

💡 *Optional Next Step:*  
Add **alerts** using `PrometheusRule` and **Alertmanager** to automatically notify when resource usage exceeds thresholds.
