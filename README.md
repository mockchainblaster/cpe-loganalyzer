# FileNet CPE Log Analyzer

A **Kubernetes-aware log analysis tool** for IBM FileNet Content Platform Engine (CPE).

This utility helps you analyze CPE log files — either from **local directories** or directly from **OpenShift / Kubernetes** pods.  
It automatically **collects**, **filters**, and **summarizes** log errors and warnings, then generates human-readable reports (Markdown, CSV, or PDF with charts).

It is designed for **containerized FileNet deployments** (e.g., Cloud Pak for Business Automation) and is especially useful for:
- Migration & health-check scenarios
- Offline analysis on developer notebooks
- Environments with autoscaling or ephemeral pods

---

## 📦 Pre-Installation

### 1. Python & dependencies
Install Python (3.8+) and the required libraries:

```bash
pip install -r requirements.txt
```

### 2. Optional: Kubernetes CLI

If you want to collect logs directly from a running CPE pod, install and configure `kubectl`:

```bash
# Verify your cluster connection
kubectl get pods -n filenet
```

---

## 🚀 Quick Start

### Clone and run
```bash
git clone https://github.com/mod242/cpe-loganalyzer.git
cd cpe-loganalyzer
chmod +x filenet_cpe_log_analyzer.py
```

---

## 🧩 Local Analysis Example

Analyze existing local FileNet CPE logs (e.g. from `/opt/ibm/wlp/usr/servers/defaultServer/FileNet`):

```bash
python filenet_cpe_log_analyzer.py /home/user/logs \
  --pattern "ce_system*.log" \
  --since "24h" \
  --outdir ./results \
  --pdf-report ./results/filenet_cpe_report.pdf \
  --pdf-title "FileNet CPE Error Summary (Last 24h)" \
  --pdf-landscape
```

This will:
1. Scan logs recursively
2. Extract all WARN and ERROR entries from the last 24 hours
3. Consolidate frequent errors
4. Generate:
   - `summary_markdown.md`
   - CSV exports (summary, examples, timeseries)
   - `filenet_cpe_report.pdf` with charts and formatted tables

---

## ☸️ Kubernetes / OpenShift Integration

If your FileNet CPE runs on **Kubernetes/OpenShift**, the script can directly copy matching log files from a running pod.

### Example (robust TAR mode)

```bash
python filenet_cpe_log_analyzer.py /home/user/analyze \
  --kube-sync \
  --kube-single \
  --kube-flatten \
  --kube-namespace filenet \
  --kube-pod-prefix demo-cpe-deploy \
  --kube-container demo-cpe-deploy \
  --kube-remote-path /opt/ibm/wlp/usr/servers/defaultServer/FileNet \
  --kube-local-path /home/user/analyze \
  --kube-file-pattern "ce_system*.log" \
  --pattern "ce_system*.log" \
  --since "24h" \
  --outdir ./results \
  --pdf-report ./results/filenet_cpe_report.pdf \
  --pdf-title "FileNet CPE Kubernetes Logs – Last 24h" \
  --pdf-landscape
```

### What happens:
1. Finds the running pod matching prefix `demo-cpe-deploy-*` in namespace `filenet`
2. Executes:
   ```bash
   find . -type f -name "ce_system*.log" -print0 | \
   tar --null -T - -czf - | tar -xzf - -C <localdir>
   ```
   (Only matching logs are streamed locally)
3. Analyzes the downloaded files
4. Produces Markdown, CSV, and PDF reports

---

## 📊 Sample PDF Output

The generated PDF includes:

- Title page with timestamp (and optional logo)
- **Top error families** (wrapped labels)
- **Error count per day/hour**
- **Consolidated error table**
- **Example log messages per family**

### Example snippet:

| Count | Error Family |
|-------|---------------|
| 850 | WSIAuthenticatorImpl login exception (authentication failures) |
| 548 | checkNameCollision / FNRCE0043E (E_NOT_UNIQUE) – Name already exists |
| 10  | getContent failures (content retrieval) |

**Charts included:**
- Top error families (horizontal bar chart)
- Errors per day
- Errors per hour

---

## ⚙️ Command Reference

### Core options

| Option | Description |
|--------|--------------|
| `--pattern` | File glob pattern (default: `*.log`) |
| `--since` | Relative start (e.g., `24h`, `2d6h`) |
| `--until` | Relative end (default: `0h` = now) |
| `--outdir` | Directory for reports and CSVs |
| `--examples` | Example rows per family (default: 3) |

### PDF options

| Option | Description |
|--------|--------------|
| `--pdf-report` | Output path for PDF file |
| `--pdf-title` | Custom report title |
| `--pdf-logo` | Optional logo image for title page |
| `--pdf-landscape` | Generate PDF in landscape orientation (recommended) |

### Kubernetes options

| Option | Description |
|--------|--------------|
| `--kube-sync` | Copy logs from Kubernetes before analysis |
| `--kube-namespace` | Namespace (e.g. `filenet`) |
| `--kube-pod-prefix` | Pod name prefix |
| `--kube-container` | Container name |
| `--kube-remote-path` | Path inside container |
| `--kube-local-path` | Local destination directory |
| `--kube-file-pattern` | Remote file pattern (e.g. `ce_system*.log`) |
| `--kube-single` | Use only one pod (shared log volume) |
| `--kube-flatten` | Do not create per-pod subfolders |
| `--kube-copy-mode` | One of: `tar` (default, robust), `snapshot`, `cp` |

---

## 🧾 Output Overview

| File | Description |
|------|--------------|
| `summary_markdown.md` | Markdown summary report |
| `summary.csv` | Summary of error families and counts |
| `examples.csv` | Example log entries per family |
| `raw_errors.csv` | All WARN/ERROR lines |
| `timeseries_overall_daily.csv` | Error count per day |
| `timeseries_overall_hourly.csv` | Error count per hour |
| `timeseries_family_daily.csv` | Error count per day per family |
| `timeseries_family_hourly.csv` | Error count per hour per family |
| `filenet_cpe_report.pdf` | Visual PDF report with charts |

---

## 🧪 Tested Environments

- IBM FileNet CPE 5.5.8 / 5.7 (Liberty)
- IBM Cloud Pak for Business Automation 24.0.x
- OpenShift 4.x / Kubernetes 1.27+
- Ubuntu 22.04, macOS 13+, Windows 11 (Python 3.10+)

---

## 🔧 Example: Full Workflow (Kubernetes + PDF)

```bash
# Step 1: Sync CPE logs from the running pod
python filenet_cpe_log_analyzer.py /tmp/analyze \
  --kube-sync \
  --kube-single \
  --kube-flatten \
  --kube-namespace filenet \
  --kube-pod-prefix demo-cpe-deploy \
  --kube-container demo-cpe-deploy \
  --kube-remote-path /opt/ibm/wlp/usr/servers/defaultServer/FileNet \
  --kube-local-path /tmp/analyze \
  --kube-file-pattern "ce_system*.log" \
  --pattern "ce_system*.log" \
  --since "24h" \
  --outdir ./results \
  --pdf-report ./results/cpe_analysis.pdf \
  --pdf-title "FileNet CPE Log Analysis – Last 24 Hours" \
  --pdf-landscape

# Step 2: View the report
xdg-open ./results/cpe_analysis.pdf
```

---

## 📘 Example: Markdown & CSV Results

### `summary_markdown.md`
```markdown
# Error overview (consolidated)

| Count | Error family |
|-------|---------------|
| 850 | WSIAuthenticatorImpl login exception (authentication failures) |
| 548 | checkNameCollision / FNRCE0043E (E_NOT_UNIQUE) – Name already exists |
| 10  | getContent failures (content retrieval) |

---

## Example entries per error family

### WSIAuthenticatorImpl login exception (authentication failures)
| Timestamp | Logfile | Example message |
|------------|----------|----------------|
| 2025-10-14T04:50:18.885 | ce_system0-cpe08.log | [WSIAuthenticatorImpl] login exception: Unable ... |
```

---

## 📄 License

Licensed under the **Apache License 2.0**.  
See [LICENSE](./LICENSE) for full terms.


## 💬 Motivation

In large-scale, distributed FileNet environments, logs are often scattered across multiple pods and rotated frequently.  
While cloud-native observability (e.g., Loki, OpenShift logging) handles streaming well, sometimes you just need an **offline, reproducible snapshot** — this script provides exactly that.

It’s ideal for:
- Migration and performance testing
- Offline troubleshooting (without cluster access)
- Quickly summarizing the most common error families

Inspired by real-world FileNet CPE scaling and migration projects.

---

