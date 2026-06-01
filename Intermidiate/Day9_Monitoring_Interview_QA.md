# Monitoring

---

## Table of Contents
1. [Monitoring Fundamentals (Q1–Q18)](#1-monitoring-fundamentals)
2. [Prometheus (Q19–Q42)](#2-prometheus)
3. [Grafana (Q43–Q55)](#3-grafana)
4. [ELK / EFK Stack (Q56–Q72)](#4-elk--efk-stack)
5. [Alerting & On-Call (Q73–Q84)](#5-alerting--on-call)
6. [Advanced & Real-World (Q85–Q110)](#6-advanced--real-world)

---

## 1. Monitoring Fundamentals

**Q1. What is observability and why is it important in DevOps?**
> Observability is the ability to understand the internal state of a system by looking at its outputs. The three pillars are:
> - **Metrics** — numerical measurements over time (CPU %, request rate, latency)
> - **Logs** — time-stamped records of events (error messages, audit trails)
> - **Traces** — end-to-end path of a request through distributed services
> Without observability, you're flying blind — you can't fix what you can't see.

---

**Q2. What is the difference between Monitoring and Observability?**
> - **Monitoring:** Watching known things. Dashboards, alerts, metrics you set up in advance.
>   *"Is CPU > 80%?"*
> - **Observability:** Ability to investigate unknown issues. Can answer questions you didn't think of beforehand.
>   *"Why is this specific user's request slow?"*
> Monitoring tells you WHEN something is broken. Observability helps you understand WHY.

---

**Q3. What are the four golden signals of monitoring?**
> Coined by Google SRE team — monitor these four for any service:
> 1. **Latency** — how long requests take (distinguish successful vs failed)
> 2. **Traffic** — how many requests per second
> 3. **Errors** — rate of failed requests (HTTP 5xx, exceptions)
> 4. **Saturation** — how "full" the system is (CPU %, memory %, queue depth)

---

**Q4. What is SLI, SLO, and SLA?**
> - **SLI (Service Level Indicator):** A metric that measures service performance.
>   Example: *"99.2% of requests returned in under 200ms"*
>
> - **SLO (Service Level Objective):** The target for that metric.
>   Example: *"99.9% of requests should complete in under 300ms"*
>
> - **SLA (Service Level Agreement):** Legal/business contract with consequences for missing SLOs.
>   Example: *"If uptime < 99.9%, customer gets 10% bill credit"*
>
> SLI measures → SLO targets → SLA enforces.

---

**Q5. What is an Error Budget?**
> Error Budget = 100% − SLO target
> - If SLO = 99.9% uptime → Error Budget = 0.1% = 43.8 minutes/month of allowed downtime
> - If you haven't used your error budget: deploy faster, take more risks
> - If you've used it: slow down, focus on reliability
> Error budgets make reliability a shared responsibility between Dev and Ops.

---

**Q6. What is the USE Method?**
> For each resource (CPU, memory, disk, network), check:
> - **U**tilization — how busy the resource is (%)
> - **S**aturation — how much work is queued/waiting
> - **E**rrors — error count
> Developed by Brendan Gregg. Great starting point for performance troubleshooting.

---

**Q7. What is the RED Method?**
> For each service/microservice, monitor:
> - **R**ate — requests per second
> - **E**rrors — failed requests per second
> - **D**uration — latency distribution (p50, p95, p99)
> Simpler version of the four golden signals. Best for request-driven services.

---

**Q8. What is a time-series database?**
> A database optimized for storing and querying timestamped data points (metrics).
> ```
> cpu_usage{host="web01"} 72.5 @1700000000
> cpu_usage{host="web01"} 74.1 @1700000060
> ```
> Examples: Prometheus, InfluxDB, TimescaleDB, VictoriaMetrics, Graphite.
> Regular databases (MySQL, PostgreSQL) are not optimized for this pattern.

---

**Q9. What is the difference between push and pull monitoring?**
> - **Pull (Prometheus model):** Monitoring system scrapes metrics from targets at intervals. Prometheus controls the schedule.
>   Pros: Central control, easy to detect dead targets. Cons: Firewall issues if targets are behind NAT.
>
> - **Push (StatsD, Graphite model):** Applications push metrics to a central server.
>   Pros: Works across firewalls. Cons: No clear indicator if app stops reporting.

---

**Q10. What are the types of metrics?**
> - **Counter:** Only goes up. Resets to 0 on restart. Example: total HTTP requests, total errors.
> - **Gauge:** Can go up and down. Example: current CPU %, memory in use, queue depth.
> - **Histogram:** Samples and counts observations in buckets. Example: request duration in buckets (0-100ms, 100-200ms, etc.)
> - **Summary:** Like histogram but calculates quantiles client-side. Example: p99 latency.

---

**Q11. What is Prometheus?**
> Prometheus is an open-source monitoring system and time-series database. It:
> - Scrapes metrics from targets (HTTP endpoints)
> - Stores data in its own time-series database
> - Provides a query language (PromQL)
> - Has a built-in alert manager integration
> Created at SoundCloud in 2012, now a CNCF graduated project.

---

**Q12. What is Grafana?**
> Grafana is an open-source visualization and dashboarding tool. It:
> - Connects to data sources (Prometheus, Elasticsearch, CloudWatch, InfluxDB, etc.)
> - Creates dashboards with charts, graphs, tables
> - Supports alerting
> - Provides a UI for exploring metrics and logs
> Grafana is the "visualization layer" — it doesn't store data itself.

---

**Q13. What is the ELK Stack?**
> Three open-source tools for log management:
> - **E**lasticsearch — distributed search and analytics engine (stores logs)
> - **L**ogstash — data processing pipeline (collects, transforms logs)
> - **K**ibana — visualization UI for Elasticsearch
> Often called the **Elastic Stack** now, since Beats was added.

---

**Q14. What is the EFK Stack?**
> Alternative to ELK — replaces Logstash with Fluentd:
> - **E**lasticsearch
> - **F**luentd (or Fluent Bit) — lighter weight log collector
> - **K**ibana
> EFK is popular in Kubernetes environments. Fluentd has better Kubernetes integration and is more resource-efficient than Logstash.

---

**Q15. What is the difference between logs, metrics, and traces?**
> | | Metrics | Logs | Traces |
> |---|---|---|---|
> | Format | Numbers over time | Text events | Spans with timing |
> | Volume | Low | High | Medium |
> | Cost | Low | High | Medium |
> | Best for | Dashboards, alerts | Debugging details | Request flow |
> | Tools | Prometheus | ELK, Loki | Jaeger, Zipkin, Tempo |

---

**Q16. What is distributed tracing?**
> Tracing tracks a request as it flows through multiple microservices. Each service adds a "span" to the trace.
> ```
> Request → API Gateway (50ms) → Auth Service (20ms) → DB (30ms)
>           └─────────────────────────────────────────────────────
>                              Total: 100ms
> ```
> Tools: **Jaeger**, **Zipkin**, **AWS X-Ray**, **Grafana Tempo**
> Standard: OpenTelemetry (OTel)

---

**Q17. What is OpenTelemetry?**
> OpenTelemetry (OTel) is an open standard for collecting and exporting telemetry data (metrics, logs, traces). It provides:
> - SDKs for all major languages
> - A collector agent (OTel Collector)
> - Vendor-neutral — send data to Prometheus, Jaeger, Datadog, etc.
> Goal: Instrument once, send anywhere.

---

**Q18. What is the difference between blackbox and whitebox monitoring?**
> - **Blackbox monitoring:** Test from the outside without knowing internals. Example: HTTP probe checking if website responds with 200.
>   Tools: Prometheus Blackbox Exporter, synthetic monitoring, Pingdom.
>
> - **Whitebox monitoring:** Instrument the application internally. Application reports its own metrics (requests handled, DB queries, errors).
>   Tools: Prometheus client libraries, OpenTelemetry SDKs.

---

## 2. Prometheus

**Q19. How does Prometheus work?**
> ```
> Prometheus Server
>   ├── Scrape targets (pull metrics every 15s by default)
>   │   ├── Node Exporter (system metrics)
>   │   ├── Application /metrics endpoint
>   │   └── kube-state-metrics (K8s objects)
>   ├── TSDB (stores scraped data)
>   ├── PromQL (query engine)
>   └── Alertmanager (send alerts)
>
> Grafana → queries Prometheus via PromQL → displays dashboards
> ```

---

**Q20. What is a Prometheus Exporter?**
> An exporter is a program that collects metrics from a system and exposes them in Prometheus format at `/metrics` endpoint.
> Common exporters:
> - **Node Exporter** — Linux system metrics (CPU, memory, disk, network)
> - **kube-state-metrics** — Kubernetes object state (pod status, deployment replicas)
> - **cAdvisor** — container metrics (built into kubelet)
> - **mysqld_exporter** — MySQL metrics
> - **redis_exporter** — Redis metrics
> - **blackbox_exporter** — HTTP/TCP/DNS probes
> - **jmx_exporter** — Java/JVM metrics

---

**Q21. What is the Prometheus data model?**
> Metrics are stored as time-series identified by metric name + labels:
> ```
> metric_name{label1="value1", label2="value2"} value timestamp
>
> # Examples:
> http_requests_total{method="GET", status="200", path="/api"} 1234
> node_cpu_seconds_total{cpu="0", mode="user"} 3456.78
> container_memory_usage_bytes{pod="myapp-abc123", namespace="prod"} 134217728
> ```

---

**Q22. What is PromQL?**
> PromQL (Prometheus Query Language) is used to query metrics from Prometheus.
```promql
# Basic query - get metric
http_requests_total

# Filter by label
http_requests_total{status="500"}

# Rate (requests per second over 5 minutes)
rate(http_requests_total[5m])

# Sum by label
sum(rate(http_requests_total[5m])) by (service)

# CPU usage percentage
100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Memory usage
(node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100

# Error rate
sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m])) * 100

# 99th percentile latency
histogram_quantile(0.99, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))
```

---

**Q23. What is the difference between `rate()` and `irate()`?**
> - `rate(counter[5m])` — average rate over 5 minute window. Smoother, better for alerting.
> - `irate(counter[5m])` — instantaneous rate (last two data points). More responsive, better for graphs.
> Use `rate()` for alerts, `irate()` for real-time graphs.

---

**Q24. What is `increase()` in PromQL?**
```promql
# Total increase over the last hour
increase(http_requests_total[1h])

# equivalent to:
rate(http_requests_total[1h]) * 3600
```
> Shows the total increase in a counter over a time range. Good for "how many requests in the last hour?"

---

**Q25. Write a Prometheus configuration file.**
```yaml
# prometheus.yml
global:
  scrape_interval: 15s      # scrape every 15 seconds
  evaluation_interval: 15s  # evaluate rules every 15 seconds
  external_labels:
    cluster: production
    region: ap-south-1

alerting:
  alertmanagers:
    - static_configs:
        - targets: ['alertmanager:9093']

rule_files:
  - "rules/*.yml"

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node-exporter'
    static_configs:
      - targets:
          - 'web01:9100'
          - 'web02:9100'
          - 'db01:9100'

  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        target_label: __metrics_path__
```

---

**Q26. What is service discovery in Prometheus?**
> Instead of listing static targets, Prometheus can discover targets automatically:
> - **Kubernetes SD** — discover Pods, Services, Nodes, Endpoints
> - **EC2 SD** — discover AWS EC2 instances by tags
> - **Consul SD** — discover services registered in Consul
> - **DNS SD** — discover via DNS SRV records
```yaml
scrape_configs:
  - job_name: 'kubernetes-nodes'
    kubernetes_sd_configs:
      - role: node
    relabel_configs:
      - action: labelmap
        regex: __meta_kubernetes_node_label_(.+)
```

---

**Q27. What is a Prometheus recording rule?**
> Pre-computes expensive PromQL expressions and stores results as new metrics. Speeds up dashboards.
```yaml
# rules/recording_rules.yml
groups:
  - name: http_rules
    interval: 1m
    rules:
      - record: job:http_requests_total:rate5m
        expr: sum(rate(http_requests_total[5m])) by (job)

      - record: job:http_errors_total:rate5m
        expr: sum(rate(http_requests_total{status=~"5.."}[5m])) by (job)
```
> Use recording rules for: Dashboard queries that run constantly, complex expressions, reducing Grafana load.

---

**Q28. What is a Prometheus alerting rule?**
```yaml
# rules/alerts.yml
groups:
  - name: application_alerts
    rules:
      - alert: HighErrorRate
        expr: |
          sum(rate(http_requests_total{status=~"5.."}[5m])) by (job)
          /
          sum(rate(http_requests_total[5m])) by (job)
          > 0.05
        for: 5m           # must be true for 5 minutes before firing
        labels:
          severity: critical
          team: backend
        annotations:
          summary: "High error rate for {{ $labels.job }}"
          description: "Error rate is {{ $value | humanizePercentage }} for job {{ $labels.job }}"

      - alert: NodeHighCPU
        expr: 100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) by (instance) * 100) > 85
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "High CPU on {{ $labels.instance }}"
          description: "CPU usage is {{ $value | humanize }}%"
```

---

**Q29. What is Alertmanager?**
> Alertmanager receives alerts from Prometheus and handles:
> - **Deduplication** — don't send same alert 100 times
> - **Grouping** — bundle related alerts into one notification
> - **Routing** — send different alerts to different teams
> - **Silencing** — suppress alerts during maintenance
> - **Inhibition** — suppress low-priority alerts when high-priority fires
```yaml
# alertmanager.yml
global:
  slack_api_url: 'https://hooks.slack.com/services/...'

route:
  group_by: ['alertname', 'job']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 4h
  receiver: 'slack-notifications'
  routes:
    - match:
        severity: critical
      receiver: 'pagerduty-critical'
    - match:
        team: database
      receiver: 'db-team-slack'

receivers:
  - name: 'slack-notifications'
    slack_configs:
      - channel: '#alerts'
        text: "{{ .CommonAnnotations.description }}"

  - name: 'pagerduty-critical'
    pagerduty_configs:
      - routing_key: 'YOUR_PAGERDUTY_KEY'
```

---

**Q30. What is a Pushgateway?**
> Prometheus is pull-based — but short-lived jobs (batch jobs, CI pipelines) finish before Prometheus can scrape. Pushgateway lets them push metrics which Prometheus then scrapes.
```bash
# Push metric to Pushgateway
echo "backup_duration_seconds 142" | curl --data-binary @- http://pushgateway:9091/metrics/job/backup
```
> ⚠️ Use sparingly — Pushgateway is not meant for services. Only for batch jobs/scripts.

---

**Q31. What are Prometheus labels and why are they important?**
> Labels are key-value pairs that identify dimensions of a metric. They enable powerful queries:
```promql
# Error rate per service
sum(rate(http_requests_total{status="500"}[5m])) by (service)

# Memory usage per namespace in K8s
sum(container_memory_usage_bytes) by (namespace)

# Latency per endpoint
histogram_quantile(0.99, sum(rate(http_request_duration_seconds_bucket[5m])) by (le, path))
```
> ⚠️ High cardinality labels (user IDs, UUIDs, IP addresses) cause performance issues — avoid them!

---

**Q32. What is cardinality in Prometheus and why is it a problem?**
> Cardinality = number of unique time series. Each unique label combination = one time series.
> ```
> http_requests_total{user_id="12345"} → 1 time series per user
> If 1 million users → 1 million time series!
> ```
> High cardinality:
> - Increases memory usage dramatically
> - Slows down queries
> - Can crash Prometheus
> Solution: Never use high-cardinality values as labels (use aggregations instead).

---

**Q33. What is Thanos?**
> Thanos extends Prometheus with:
> - **Long-term storage** — archive metrics to S3/GCS (Prometheus only keeps days/weeks)
> - **Global view** — query across multiple Prometheus instances
> - **High availability** — deduplicate data from HA Prometheus pairs
> Install alongside Prometheus as sidecars and store objects.

---

**Q34. What is VictoriaMetrics?**
> VictoriaMetrics is a high-performance, cost-efficient monitoring solution. Compared to Prometheus:
> - Faster ingestion and queries
> - Less memory usage
> - Better compression (uses less disk)
> - Built-in long-term storage
> - Compatible with Prometheus scrape configs and PromQL
> Used as a drop-in Prometheus replacement for large-scale setups.

---

**Q35. How do you instrument a Node.js application for Prometheus?**
```javascript
const client = require('prom-client');

// Auto-collect default metrics (CPU, memory, event loop lag)
client.collectDefaultMetrics();

// Custom metrics
const httpRequestDuration = new client.Histogram({
  name: 'http_request_duration_seconds',
  help: 'HTTP request duration in seconds',
  labelNames: ['method', 'route', 'status_code'],
  buckets: [0.001, 0.005, 0.01, 0.05, 0.1, 0.5, 1, 5]
});

const httpRequestTotal = new client.Counter({
  name: 'http_requests_total',
  help: 'Total HTTP requests',
  labelNames: ['method', 'route', 'status_code']
});

// Middleware to record metrics
app.use((req, res, next) => {
  const end = httpRequestDuration.startTimer();
  res.on('finish', () => {
    const labels = { method: req.method, route: req.path, status_code: res.statusCode };
    end(labels);
    httpRequestTotal.inc(labels);
  });
  next();
});

// Expose metrics endpoint
app.get('/metrics', async (req, res) => {
  res.set('Content-Type', client.register.contentType);
  res.send(await client.register.metrics());
});
```

---

**Q36. How do you instrument a Python application for Prometheus?**
```python
from prometheus_client import Counter, Histogram, Gauge, start_http_server
import time

# Define metrics
REQUEST_COUNT = Counter('http_requests_total', 'Total requests',
                        ['method', 'endpoint', 'status'])
REQUEST_LATENCY = Histogram('http_request_duration_seconds', 'Request latency',
                            ['endpoint'],
                            buckets=[0.01, 0.05, 0.1, 0.5, 1.0, 5.0])
ACTIVE_CONNECTIONS = Gauge('active_connections', 'Active connections')

# Use in your code
def handle_request(method, endpoint):
    ACTIVE_CONNECTIONS.inc()
    start = time.time()
    try:
        # ... handle request ...
        REQUEST_COUNT.labels(method=method, endpoint=endpoint, status='200').inc()
    except Exception:
        REQUEST_COUNT.labels(method=method, endpoint=endpoint, status='500').inc()
        raise
    finally:
        REQUEST_LATENCY.labels(endpoint=endpoint).observe(time.time() - start)
        ACTIVE_CONNECTIONS.dec()

# Start metrics server
start_http_server(8000)  # metrics at http://localhost:8000/metrics
```

---

**Q37. What is the Prometheus Operator?**
> A Kubernetes operator that manages Prometheus deployment using Custom Resources:
```yaml
# Define what to monitor using ServiceMonitor
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: myapp-monitor
spec:
  selector:
    matchLabels:
      app: myapp
  endpoints:
    - port: metrics
      path: /metrics
      interval: 15s
```
> Instead of editing prometheus.yml, you create Kubernetes CRDs. Prometheus Operator watches them and auto-updates Prometheus config.

---

**Q38. What are Prometheus metric naming conventions?**
> - Use snake_case
> - Format: `<namespace>_<subsystem>_<name>_<unit>`
> - Include units in name: `_seconds`, `_bytes`, `_total`
> - Counters end in `_total`
> - Don't put labels in metric name
```
✅ http_requests_total
✅ node_memory_bytes_total
✅ request_duration_seconds
✅ database_connections_active

❌ httpRequestsTotal (camelCase)
❌ http_requests_per_second (rate - use counter instead)
❌ request_latency_ms_p99 (don't hardcode quantile in name)
```

---

**Q39. What is `absent()` function in PromQL?**
```promql
# Alert when a metric stops reporting (target is down)
absent(up{job="myapp"} == 1)
# If myapp stops sending metrics, this becomes 1 (truthy) → fires alert
```
> Useful for detecting when an exporter or service goes down completely.

---

**Q40. What is `kube-prometheus-stack`?**
> A Helm chart that installs the complete Kubernetes monitoring stack in one command:
> - Prometheus Operator
> - Prometheus
> - Alertmanager
> - Grafana
> - kube-state-metrics
> - node-exporter
> - Pre-built dashboards for Kubernetes
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install monitoring prometheus-community/kube-prometheus-stack \
  --namespace monitoring --create-namespace
```

---

**Q41. What is `topk()` and `bottomk()` in PromQL?**
```promql
# Top 5 pods by memory usage
topk(5, sum(container_memory_usage_bytes) by (pod))

# Bottom 5 (lowest CPU usage pods)
bottomk(5, sum(rate(container_cpu_usage_seconds_total[5m])) by (pod))

# Use case: find your most resource-hungry services
topk(10, sum(rate(http_requests_total[5m])) by (service))
```

---

**Q42. What is federation in Prometheus?**
> Federation allows one Prometheus to scrape metrics from another Prometheus:
```yaml
# Central Prometheus scraping from regional Prometheus instances
scrape_configs:
  - job_name: 'federate'
    honor_labels: true
    metrics_path: '/federate'
    params:
      match[]:
        - '{job="node-exporter"}'
        - '{job="myapp"}'
    static_configs:
      - targets:
          - 'prometheus-us-east:9090'
          - 'prometheus-eu-west:9090'
```

---

## 3. Grafana

**Q43. What is Grafana and what data sources does it support?**
> Grafana is the leading open-source analytics and visualization platform. Supports:
> - Prometheus, Thanos, Cortex
> - Elasticsearch, Opensearch
> - Loki (logs)
> - Tempo (traces)
> - InfluxDB
> - AWS CloudWatch
> - Azure Monitor
> - Google Cloud Monitoring
> - MySQL, PostgreSQL
> - Jaeger, Zipkin

---

**Q44. What are Grafana panels?**
> Panels are individual visualizations on a dashboard:
> - **Time series** — line/area/bar charts over time (most common)
> - **Stat** — single number (current value)
> - **Gauge** — shows value as needle gauge
> - **Table** — tabular data
> - **Heatmap** — intensity over time (for histograms)
> - **Logs** — display log streams (from Loki)
> - **Node Graph** — service dependency maps
> - **Geomap** — geographic data

---

**Q45. What are Grafana Variables / Template Variables?**
> Variables make dashboards dynamic — change a dropdown to filter all panels:
```
Variable: $namespace → allows selecting Kubernetes namespace
Variable: $pod → allows selecting specific pod
All panels use: container_cpu_usage_seconds_total{namespace="$namespace", pod="$pod"}
```
```yaml
# Variable configuration
Type: Query
Query: label_values(kube_pod_info{namespace="$namespace"}, pod)
Refresh: On Dashboard Load
```
> Best practice: Add variables for environment, namespace, service — makes one dashboard work for everything.

---

**Q46. What is Grafana alerting?**
> Grafana can alert on any data source (not just Prometheus):
> - Define alert conditions in panel queries
> - Set thresholds (above/below/outside range)
> - Route to contact points (Slack, email, PagerDuty, OpsGenie)
> - Uses notification policies for routing (similar to Alertmanager)
```yaml
Alert rule:
  Query: avg(rate(http_requests_total{status="500"}[5m])) * 100
  Condition: IS ABOVE 5
  For: 5m
  Labels: severity=critical, team=backend
```

---

**Q47. What is Grafana Loki?**
> Loki is Grafana's log aggregation system. Like Prometheus but for logs:
> - Only indexes metadata (labels), not log content → much cheaper than Elasticsearch
> - Uses LogQL query language (similar to PromQL)
> - Integrates natively with Grafana
```logql
# Show all logs from production nginx
{app="nginx", env="production"}

# Filter for errors
{app="myapp"} |= "ERROR"

# Parse JSON logs and filter
{app="myapp"} | json | status_code >= 500

# Count error rate
sum(rate({app="myapp"} |= "error" [5m])) by (pod)
```

---

**Q48. What is Grafana Tempo?**
> Tempo is Grafana's distributed tracing backend. Like Loki for traces:
> - Integrates with Jaeger, Zipkin, OpenTelemetry
> - Stores traces in object storage (S3)
> - Very cost-effective compared to Jaeger with Elasticsearch
> - Native integration with Grafana — drill from metrics → traces → logs

---

**Q49. What is the Grafana LGTM Stack?**
> Grafana's full observability stack:
> - **L**oki — logs
> - **G**rafana — visualization
> - **T**empo — traces
> - **M**imir (or Prometheus) — metrics
> All work together: click on a spike in Grafana → jump to related logs in Loki → find trace in Tempo.

---

**Q50. How do you create a Grafana dashboard as code?**
```json
// dashboard.json (can be provisioned from file)
{
  "title": "Application Dashboard",
  "panels": [
    {
      "title": "Request Rate",
      "type": "timeseries",
      "targets": [
        {
          "expr": "sum(rate(http_requests_total[5m])) by (service)",
          "legendFormat": "{{service}}"
        }
      ]
    }
  ]
}
```
```yaml
# Provisioning config
# /etc/grafana/provisioning/dashboards/dashboards.yaml
apiVersion: 1
providers:
  - name: 'default'
    type: file
    options:
      path: /var/lib/grafana/dashboards
```
> Use **Grafonnet** (Jsonnet library) or **grafanalib** (Python) for programmatic dashboard creation.

---

**Q51. What is Grafana Provisioning?**
> Pre-configure Grafana resources via config files (no manual UI clicking):
```yaml
# Provision data sources
# /etc/grafana/provisioning/datasources/prometheus.yaml
apiVersion: 1
datasources:
  - name: Prometheus
    type: prometheus
    url: http://prometheus:9090
    isDefault: true
    editable: false
```
> Useful for: Kubernetes deployments, GitOps, reproducible setups.

---

**Q52. What are some useful pre-built Grafana dashboards?**
> Import from grafana.com by ID:
> - **1860** — Node Exporter Full (system metrics)
> - **315** — Kubernetes cluster monitoring
> - **6417** — Kubernetes pods overview
> - **7249** — NGINX Ingress Controller
> - **763** — Redis dashboard
> - **7362** — MySQL overview
> - **11159** — Docker container metrics
```bash
# Import via API
curl -X POST http://admin:admin@grafana:3000/api/dashboards/import \
  -H "Content-Type: application/json" \
  -d '{"dashboard": {"id": null}, "folderId": 0, "overwrite": true}'
```

---

**Q53. What is a Grafana annotation?**
> Annotations mark events on Grafana graphs — deployments, incidents, config changes:
```bash
# Add annotation via API (e.g., from CI/CD pipeline)
curl -X POST http://grafana:3000/api/annotations \
  -H "Authorization: Bearer $GRAFANA_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "time": '$(date +%s000)',
    "tags": ["deployment", "myapp"],
    "text": "Deployed v1.2.3 to production"
  }'
```
> Now your dashboards show exactly when deploys happened — correlate with metric changes.

---

**Q54. What is Grafana's Explore mode?**
> Explore is an ad-hoc query interface — no need to create a dashboard. Just query and explore:
> - Write PromQL queries and see results immediately
> - Query Loki logs
> - View traces from Tempo
> - Split screen: metrics on left, logs on right
> Best for: Incident investigation, ad-hoc debugging.

---

**Q55. How do you set up Grafana in Kubernetes?**
```yaml
# values.yaml for kube-prometheus-stack
grafana:
  enabled: true
  adminPassword: "secure-password"
  ingress:
    enabled: true
    hosts:
      - grafana.example.com
  persistence:
    enabled: true
    size: 10Gi
  sidecar:
    dashboards:
      enabled: true    # auto-load dashboards from ConfigMaps
    datasources:
      enabled: true
```

---

## 4. ELK / EFK Stack

**Q56. What is Elasticsearch?**
> Elasticsearch is a distributed, RESTful search and analytics engine. For logs/monitoring:
> - Stores logs as JSON documents
> - Full-text search on log content
> - Aggregations and analytics
> - Distributed — scales horizontally
> - REST API for all operations

---

**Q57. What is Logstash?**
> Logstash is a server-side data processing pipeline:
> - **Input:** Collect data (file beats, kafka, syslog, JDBC)
> - **Filter:** Parse, transform, enrich (grok, mutate, geoip)
> - **Output:** Send to destination (Elasticsearch, S3, Kafka)
```ruby
# logstash.conf
input {
  beats {
    port => 5044
  }
}

filter {
  if [type] == "nginx" {
    grok {
      match => { "message" => "%{COMBINEDAPACHELOG}" }
    }
    date {
      match => [ "timestamp", "dd/MMM/yyyy:HH:mm:ss Z" ]
    }
    geoip {
      source => "clientip"
    }
  }
}

output {
  elasticsearch {
    hosts => ["elasticsearch:9200"]
    index => "logs-%{+YYYY.MM.dd}"
  }
}
```

---

**Q58. What is Kibana?**
> Kibana is the visualization UI for Elasticsearch:
> - **Discover** — explore and search logs
> - **Visualize** — create charts, graphs
> - **Dashboard** — combine visualizations
> - **Alerting** — alert on log patterns
> - **APM** — application performance monitoring
> - **Lens** — drag-and-drop visualization builder

---

**Q59. What is Filebeat?**
> Filebeat is a lightweight log shipper. Runs on servers and ships log files to Logstash or Elasticsearch:
```yaml
# filebeat.yml
filebeat.inputs:
  - type: log
    enabled: true
    paths:
      - /var/log/nginx/*.log
      - /var/log/myapp/*.log
    json.keys_under_root: true
    fields:
      environment: production
      service: myapp

output.logstash:
  hosts: ["logstash:5044"]
```
> Much lighter than Logstash — use Filebeat on every server, Logstash on central server.

---

**Q60. What is the difference between Logstash and Beats?**
> | | Beats (Filebeat etc.) | Logstash |
> |---|---|---|
> | Purpose | Collect and ship | Process and route |
> | Resource | Lightweight | Heavy (JVM) |
> | Processing | Basic | Complex (grok, mutate, etc.) |
> | Deploy | Every server | Central server |
> Common pattern: Beats on servers → Logstash for processing → Elasticsearch for storage.

---

**Q61. What is Fluentd and Fluent Bit?**
> - **Fluentd** — open-source data collector, more features than Filebeat, native Kubernetes support.
> - **Fluent Bit** — lightweight version of Fluentd. Very low memory/CPU. Designed for Kubernetes/containers.
```yaml
# Fluent Bit config for Kubernetes
[INPUT]
    Name              tail
    Path              /var/log/containers/*.log
    Parser            docker
    Tag               kube.*

[FILTER]
    Name              kubernetes
    Match             kube.*
    Kube_URL          https://kubernetes.default.svc:443

[OUTPUT]
    Name              es
    Match             *
    Host              elasticsearch
    Port              9200
    Index             kubernetes-logs
```

---

**Q62. What is an Elasticsearch index?**
> An index is like a database table — a collection of documents with similar structure.
> ```
> Index: logs-2024.01.15
>   Document: {timestamp, level, message, service, pod}
>   Document: {timestamp, level, message, service, pod}
>   ...
> ```
> ILM (Index Lifecycle Management) automatically manages index creation, rollover, and deletion.

---

**Q63. What is ILM (Index Lifecycle Management)?**
> ILM automatically manages Elasticsearch indices through lifecycle phases:
> ```
> Hot Phase   → Active writes, kept on fast SSDs
>   ↓ (after 7 days or index > 50GB)
> Warm Phase  → Read-only, moved to slower nodes, merged
>   ↓ (after 30 days)
> Cold Phase  → Searchable but rare access, even slower storage
>   ↓ (after 90 days)
> Delete Phase → Index deleted
> ```

---

**Q64. What is Grok in Logstash/Elasticsearch?**
> Grok is a pattern matching syntax that parses unstructured text into structured fields:
```ruby
# Parse nginx log line:
# 192.168.1.1 - - [15/Jan/2024:10:30:00 +0000] "GET /api/users HTTP/1.1" 200 1234

grok {
  match => {
    "message" => '%{IPORHOST:client_ip} - - \[%{HTTPDATE:timestamp}\] "%{WORD:method} %{URIPATHPARAM:request} HTTP/%{NUMBER:http_version}" %{NUMBER:status_code} %{NUMBER:response_size}'
  }
}

# Output fields:
# client_ip: "192.168.1.1"
# method: "GET"
# request: "/api/users"
# status_code: "200"
# response_size: "1234"
```

---

**Q65. What is Kibana Query Language (KQL)?**
```
# Basic text search
error

# Field search
status: 500

# Range
response_time > 1000

# Boolean
status: 500 AND service: "payment"
NOT status: 200

# Wildcard
path: /api/*

# Phrase match
message: "connection refused"
```

---

**Q66. How do you monitor Kubernetes logs with EFK?**
```
Every K8s Node:
  Fluent Bit DaemonSet
    → reads /var/log/containers/*.log
    → adds K8s metadata (pod, namespace, labels)
    → sends to Elasticsearch

Elasticsearch Cluster:
    → stores all logs
    → creates index per day: k8s-logs-2024.01.15

Kibana:
    → search and visualize K8s logs
    → filter by namespace, pod, deployment
    → create alerts on error patterns
```

---

**Q67. What is OpenSearch?**
> OpenSearch is an open-source fork of Elasticsearch (after Elastic changed its license in 2021). AWS created it.
> - Compatible with Elasticsearch 7.x APIs
> - Includes OpenSearch Dashboards (Kibana equivalent)
> - AWS OpenSearch Service is the managed version
> Most DevOps teams now consider OpenSearch as the AWS-native alternative to Elasticsearch.

---

**Q68. How do you structure log messages for better observability?**
> Use **structured logging** (JSON format):
```json
{
  "timestamp": "2024-01-15T10:30:00Z",
  "level": "ERROR",
  "service": "payment-service",
  "trace_id": "abc123",
  "user_id": "usr_456",
  "request_id": "req_789",
  "message": "Payment processing failed",
  "error": "Connection timeout to payment gateway",
  "duration_ms": 5023,
  "amount": 99.99,
  "currency": "INR"
}
```
> Benefits:
> - Easy to filter by any field in Kibana/Grafana
> - Can correlate via trace_id
> - Machine-readable for alerting
> - Consistent across services

---

**Q69. What is log aggregation and why is it important?**
> Log aggregation collects logs from ALL servers/containers into a central place.
> Without it:
> - 100 servers → SSH to each to find error
> - Container restarts → logs lost
> - No ability to correlate across services
> With it:
> - One place to search all logs
> - Survive container restarts
> - Correlate errors across microservices

---

**Q70. What is syslog and how does it relate to modern logging?**
> Syslog is a standard protocol for sending logs to a central collector (traditional Linux logging).
> Modern approach for cloud-native:
> - Applications write to stdout/stderr
> - Container runtime captures stdout/stderr
> - Log agent (Fluent Bit/Filebeat) reads from container log files
> - Ships to centralized logging (EFK/Loki)

---

**Q71. What is log retention policy and why does it matter?**
> Log retention defines how long logs are kept before deletion.
> Considerations:
> - **Compliance:** GDPR (data minimization), PCI-DSS (1 year), HIPAA (6 years)
> - **Storage cost:** Logs are expensive to store long-term
> - **Debug value:** Recent logs (30 days) are most useful
> Typical policy:
> - Hot (fast): 7 days → Warm: 30 days → Cold (cheap): 90 days → Delete

---

**Q72. What is Jaeger?**
> Jaeger is an open-source distributed tracing platform (created by Uber):
> - Collect and store traces
> - Visualize request flows across services
> - Find bottlenecks and latency issues
> - Service dependency maps
```
Browser Request
  └─ API Gateway (15ms)
      └─ Auth Service (8ms)
      └─ Product Service (45ms)
          └─ Database Query (40ms)   ← BOTTLENECK
      └─ Cart Service (12ms)
Total: 80ms
```

---

## 5. Alerting & On-Call

**Q73. What makes a good alert?**
> A good alert is:
> - **Actionable** — someone knows what to do when it fires
> - **Urgent** — indicates something that needs attention now
> - **Accurate** — doesn't fire falsely (low false positive rate)
> Bad alerts:
> - "CPU is 75%" — so what? Not actionable.
> - Alerts that fire every night during batch jobs
> - Alerts nobody knows how to respond to
> Rule: If an alert fires and you don't know what to do → fix the alert, not the system.

---

**Q74. What is alert fatigue and how to prevent it?**
> Alert fatigue = so many alerts that on-call engineers ignore them all.
> Prevention:
> - Delete alerts nobody acts on
> - Raise alert thresholds (alert at 90% not 70%)
> - Group related alerts (Alertmanager grouping)
> - Add `for: 5m` duration (don't alert on brief spikes)
> - Route by severity (page only for critical, Slack for warnings)
> - Review and prune alerts monthly

---

**Q75. What is PagerDuty and how does it work with Prometheus?**
> PagerDuty is an incident management and on-call scheduling platform:
> - Receives alerts from Alertmanager
> - Routes to on-call engineer based on schedule
> - Escalates if not acknowledged
> - Tracks incidents and post-mortems
```yaml
# In Alertmanager
receivers:
  - name: pagerduty
    pagerduty_configs:
      - routing_key: 'YOUR_PD_ROUTING_KEY'
        description: '{{ .CommonAnnotations.summary }}'
        severity: '{{ .CommonLabels.severity }}'
```

---

**Q76. What is an Alertmanager silence?**
```bash
# Create silence via amtool (during planned maintenance)
amtool silence add \
  --alertmanager.url http://alertmanager:9093 \
  --comment "Planned maintenance window" \
  --duration 2h \
  alertname=~".*" environment="production"
```
> Suppresses alerts during planned maintenance. Removes noise when you already know something is broken.

---

**Q77. What is inhibition in Alertmanager?**
```yaml
inhibit_rules:
  - source_match:
      alertname: 'NodeDown'
    target_match:
      severity: 'warning'
    equal: ['instance']
```
> If a node is DOWN (critical), inhibit all WARNING alerts from that same node. Prevents noise from downstream effects.

---

**Q78. What is a runbook?**
> A runbook (or playbook) is a documented procedure for responding to an alert:
```markdown
# Alert: HighErrorRate

## Impact
Payment service returning 5xx errors to users

## Investigation Steps
1. Check application logs: `kubectl logs -l app=payment -n prod | grep ERROR`
2. Check database connectivity: `kubectl exec -it payment-pod -- nc -zv db:5432`
3. Check upstream service status: http://status.example.com
4. Check recent deployments: `kubectl rollout history deployment/payment`

## Resolution Steps
- If database issue: failover to read replica
- If bad deployment: `kubectl rollout undo deployment/payment`
- If upstream issue: enable circuit breaker mode

## Escalation
If not resolved in 15 minutes → page backend team lead
```
> Link runbooks in alert annotations: `runbook_url: "https://wiki.company.com/runbooks/high-error-rate"`

---

**Q79. What is an SRE (Site Reliability Engineer)?**
> SRE is a discipline applying software engineering to operations. Key concepts:
> - Define SLIs, SLOs, SLAs for services
> - Manage error budgets
> - Automate toil (repetitive manual work)
> - Post-mortems after incidents (blameless)
> - On-call rotation management
> SRE is what DevOps looks like at Google scale.

---

**Q80. What is a post-mortem / incident review?**
> A blameless post-mortem reviews an incident to prevent recurrence:
> ```
> Incident: Payment service was down for 45 minutes
>
> Timeline:
>   14:30 - Deployment started
>   14:35 - Error rate spiked
>   14:42 - Alert fired (7 min delay - issue!)
>   14:45 - Engineer paged
>   15:15 - Fix deployed, service recovered
>
> Root Cause: Memory leak in new code version
>
> Contributing Factors:
>   - Alert delay (7 minutes)
>   - No canary deployment
>   - Load test didn't catch memory leak
>
> Action Items:
>   - Implement canary deployments (Owner: Alice, Due: Jan 30)
>   - Add memory alert (Owner: Bob, Due: Jan 20)
>   - Add load test for memory (Owner: Carol, Due: Feb 1)
> ```

---

**Q81. What is MTTD, MTTR, MTTF?**
> - **MTTD (Mean Time To Detect):** Average time from incident start to detection (alert fires)
> - **MTTR (Mean Time To Resolve/Recover):** Average time from detection to resolution
> - **MTTF (Mean Time To Failure):** Average time between failures
> Goal: Minimize MTTD and MTTR. Maximize MTTF.

---

**Q82. What is synthetic monitoring?**
> Regularly test your application from the outside (simulated user actions):
```yaml
# Prometheus Blackbox Exporter
modules:
  http_2xx:
    prober: http
    timeout: 5s
    http:
      valid_http_versions: ["HTTP/1.1", "HTTP/2.0"]
      valid_status_codes: [200]

# Probe config in prometheus.yml
- job_name: 'blackbox'
  metrics_path: /probe
  params:
    module: [http_2xx]
  static_configs:
    - targets:
        - https://myapp.com
        - https://myapp.com/api/health
        - https://myapp.com/checkout
```
> Also: AWS CloudWatch Synthetics, Datadog Synthetic Tests, Grafana Cloud Synthetic Monitoring.

---

**Q83. What is Chaos Engineering?**
> Deliberately introduce failures to test system resilience:
> - Kill random pods (Netflix Chaos Monkey)
> - Add network latency
> - Fill disk space
> - Kill a random node
> Tools: **Chaos Monkey**, **LitmusChaos**, **Gremlin**, **ChaosMesh**
```yaml
# LitmusChaos experiment
apiVersion: litmuschaos.io/v1alpha1
kind: ChaosEngine
spec:
  experiments:
    - name: pod-delete
      spec:
        components:
          env:
            - name: TOTAL_CHAOS_DURATION
              value: '30'
            - name: CHAOS_INTERVAL
              value: '10'
```

---

**Q84. What is Grafana OnCall?**
> Grafana OnCall is an on-call management solution (acquired from Amixr):
> - Integrates with Grafana Alerting and Prometheus Alertmanager
> - On-call schedules and rotations
> - Escalation policies
> - Mobile app for alerts
> - Incident management
> Alternative to PagerDuty/OpsGenie.

---

## 6. Advanced & Real-World

**Q85. Design a complete monitoring stack for a production Kubernetes cluster.**
```
METRICS (Prometheus Stack)
├── kube-prometheus-stack (Helm chart)
│   ├── Prometheus (metrics storage, 30 day retention)
│   ├── Alertmanager (alert routing → Slack + PagerDuty)
│   ├── Grafana (dashboards)
│   ├── kube-state-metrics (K8s object metrics)
│   └── node-exporter (system metrics on each node)
├── Application metrics (/metrics endpoints)
└── Thanos (long-term storage to S3)

LOGS (EFK Stack)
├── Fluent Bit DaemonSet (collect from all pods)
├── Elasticsearch (store and index)
└── Kibana (search and analyze)

TRACES (Distributed Tracing)
├── Jaeger (or Tempo)
├── OpenTelemetry Collector
└── App instrumentation (OTel SDK)

ALERTS
├── Prometheus → Alertmanager → Slack (#alerts-warning)
├── Critical alerts → PagerDuty → On-call engineer
└── Dashboards → Grafana annotations for deployments
```

---

**Q86. How do you monitor application performance (APM)?**
> APM tracks:
> - Request rate, error rate, latency (RED method)
> - Slow database queries
> - External API calls
> - Background job performance
> Tools:
> - **Elastic APM** (built into ELK stack)
> - **Datadog APM**
> - **New Relic**
> - **Jaeger + OpenTelemetry** (open source)
> - **Pyroscope** (continuous profiling)

---

**Q87. What is continuous profiling?**
> Continuously collect CPU profiles and memory profiles from production applications.
> Tools: **Pyroscope** (Grafana), **Parca**, **Google Cloud Profiler**
> Shows:
> - Which functions use most CPU
> - Where memory allocations happen
> - Performance changes between versions
> Without restarting or adding instrumentation.

---

**Q88. How do you monitor cost on AWS?**
```
AWS Cost Monitoring Stack:
├── AWS Cost Explorer (built-in AWS)
├── AWS Budgets (set budget alerts)
├── CloudWatch billing alarms
├── Grafana with CloudWatch datasource
│   └── Dashboards for cost per service, per tag
├── Infracost (cost in Terraform PRs)
└── AWS Trusted Advisor (optimization recommendations)

Key metrics to track:
├── Cost by service (EC2, RDS, Data Transfer)
├── Cost by team (using Tags)
├── Cost by environment (prod vs dev)
└── Anomaly detection (cost spike alerts)
```

---

**Q89. What is the difference between Prometheus and Datadog?**
> | Feature | Prometheus | Datadog |
> |---|---|---|
> | Cost | Free (open source) | Expensive ($$$) |
> | Hosting | Self-managed | SaaS |
> | Setup | Complex | Easy |
> | Integrations | 1000+ exporters | 700+ built-in integrations |
> | Logs | Via Loki (separate) | Built-in |
> | Traces | Via Jaeger/Tempo | Built-in APM |
> | Alerting | Alertmanager | Built-in |
> Prometheus for cost-conscious teams, Datadog for ease of use with budget.

---

**Q90. How do you implement SLO-based alerting?**
```promql
# SLO: 99.9% of requests succeed
# Error budget: 0.1% = 43.8 min/month

# Burn rate alert - if you're burning budget too fast
# Fast burn: would exhaust budget in 1 hour
(
  sum(rate(http_requests_total{status=~"5.."}[5m]))
  /
  sum(rate(http_requests_total[5m]))
) > (14.4 * 0.001)   # 14.4x burn rate for 1hr window

# Slow burn: would exhaust budget in 24 hours
(
  sum(rate(http_requests_total{status=~"5.."}[1h]))
  /
  sum(rate(http_requests_total[1h]))
) > (1 * 0.001)
```

---

**Q91–Q110: Quick-fire important questions.**

**Q91. What port does Prometheus run on?**
> `9090` for Prometheus, `9091` for Pushgateway, `9093` for Alertmanager, `9100` for Node Exporter.

**Q92. What is `up` metric in Prometheus?**
> `up{job="myapp", instance="web01:8080"}` = 1 if target is scraping successfully, 0 if target is down. Alert on `up == 0` to detect target failures.

**Q93. What is metric scraping interval and how does it affect retention?**
> Default 15s interval. More frequent = more storage. Retention controlled by `--storage.tsdb.retention.time` (default 15 days). Plan storage: `#targets × #metrics × scrape_interval × retention`.

**Q94. What is `histogram_quantile` and why use it instead of average?**
> Calculates percentile from histogram buckets: `histogram_quantile(0.99, ...)` gives 99th percentile latency. Average hides outliers — p99 shows the worst 1% of users' experience.

**Q95. What is a Grafana data source?**
> A connection to a backend system where data lives. Grafana queries the data source via its API. Configured in Administration → Data Sources. Each dashboard panel points to a data source.

**Q96. What is Elastic APM and how does it work?**
> Elastic APM adds performance monitoring to the ELK stack. Install APM agents in your app (Python, Node.js, Java), they send traces and metrics to APM Server, which stores in Elasticsearch, viewable in Kibana.

**Q97. What is the difference between alerting on symptoms vs causes?**
> - **Symptom-based (better):** Alert on "error rate > 5%" — this is what users experience.
> - **Cause-based (worse):** Alert on "CPU > 80%" — CPU could be high without affecting users.
> Always prefer symptom-based alerts. Cause-based can help diagnose, not alert.

**Q98. What is SNMP and when is it used in monitoring?**
> SNMP (Simple Network Management Protocol) monitors network devices (routers, switches, load balancers). Prometheus has `snmp_exporter` for SNMP-capable devices.

**Q99. What is `logrotate` and how does it relate to monitoring?**
> logrotate manages log file rotation on Linux servers. Essential for preventing disk full from log files. Monitoring should alert on disk usage before it's too late.

**Q100. What is Grafana Mimir?**
> Mimir is Grafana's scalable long-term storage backend for Prometheus metrics. Like Thanos but more opinionated and easier to operate. Stores metrics in S3 with unlimited retention.

**Q101. How do you monitor SSL certificate expiry?**
```promql
# Alert if cert expires in less than 30 days
(probe_ssl_earliest_cert_expiry - time()) / 86400 < 30
```
> Using Blackbox Exporter to probe HTTPS endpoints and check certificate expiry.

**Q102. What is `absent_over_time()` in PromQL?**
> `absent_over_time(metric[5m])` returns 1 if the metric has been absent for 5 minutes. More reliable than `absent()` for intermittent metrics.

**Q103. What is a Grafana playlist?**
> A sequence of dashboards that auto-rotate (like a TV screen in an ops center). Useful for displaying rotating dashboards on NOC (Network Operations Center) screens.

**Q104. What is Kibana Canvas?**
> Kibana Canvas lets you create custom, pixel-perfect visualizations and infographics from Elasticsearch data. Good for executive dashboards and presentations.

**Q105. What is Prometheus remote write?**
> Prometheus can write metrics to external storage backends:
> ```yaml
> remote_write:
>   - url: https://thanos-receive:9091/api/v1/receive
>   - url: https://mimir:9009/api/v1/push
> ```
> Enables long-term storage without changing how Prometheus works.

**Q106. What is the difference between Grafana Cloud and self-hosted Grafana?**
> Grafana Cloud is the managed SaaS version — includes Grafana, Loki, Tempo, and Mimir hosted and managed by Grafana Labs. Self-hosted = you manage everything. Cloud is simpler, self-hosted is cheaper at scale.

**Q107. What is Prometheus federation vs remote write?**
> - **Federation:** Prometheus scrapes another Prometheus. Pull model. Good for hierarchical monitoring.
> - **Remote Write:** Prometheus pushes to a remote system. Push model. Good for long-term storage (Thanos, Mimir).

**Q108. How do you debug high memory usage in Prometheus?**
```bash
# Check number of time series
curl http://localhost:9090/api/v1/query?query=prometheus_tsdb_head_series
# If > 1 million, investigate high cardinality

# Find metrics with most series
curl http://localhost:9090/api/v1/label/__name__/values | jq '.data | length'

# Check TSDB stats
curl http://localhost:9090/api/v1/status/tsdb
```

**Q109. What is Grafana's alerting vs Prometheus alerting?**
> - **Prometheus + Alertmanager:** Alert on Prometheus metrics only. More powerful PromQL rules.
> - **Grafana Alerting:** Alert on any data source (Elasticsearch, CloudWatch, etc.). Unified alerting UI.
> Use both: Prometheus for metrics alerts, Grafana for cross-datasource alerting.

**Q110. Real-world scenario: Production database is slow, walk through the investigation.**
```
1. CHECK METRICS (Grafana)
   └── DB dashboard: query duration p99 spiked at 14:30
   └── CPU: normal. Disk I/O: normal. Connections: HIGH (3000/1000 max)

2. CHECK LOGS (Kibana/Loki)
   └── Filter: service=database, level=error
   └── Found: "too many connections" errors starting 14:28
   └── New app version deployed at 14:25 ← correlation!

3. CHECK TRACES (Jaeger/Tempo)
   └── Find slow requests from 14:30
   └── All slow requests go to /api/products endpoint
   └── Trace shows N+1 query problem - 500 DB queries per request

4. ROOT CAUSE
   └── New code does: for each product → query DB → N+1 problem
   └── 100 users × 500 queries = 50,000 DB queries/sec
   └── DB connection pool exhausted

5. RESOLUTION
   └── Rollback new deployment: kubectl rollout undo deployment/app
   └── Connections normalized, latency recovered

6. POST-MORTEM ACTION ITEMS
   └── Add query count per request metric + alert if > 50
   └── Add DB connection pool utilization alert
   └── Add N+1 detection to code review checklist
```

---
---