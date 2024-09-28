
                                        # Prometheus  & Grafana

Great! Let's begin with the basics of Prometheus and Grafana. These two tools are widely used for monitoring and visualization, especially in production environments where observability is crucial.

Prometheus Basics: Introduction
What is Prometheus?
Prometheus is an open-source monitoring and alerting toolkit designed primarily for reliability and scalability. It is used to collect and store metrics as time-series data, where metrics are stored with timestamps and optional key-value pairs called labels.

Key Features:

Multi-dimensional data model: Prometheus organizes data using labels, which are flexible key-value pairs.
Time-series storage: It stores metrics in a time-series database (TSDB), with the ability to handle large amounts of data.
Powerful query language (PromQL): Prometheus uses PromQL, a query language for selecting and aggregating time-series data.
Pull-based model: Prometheus scrapes (pulls) metrics from defined targets.
Alerting: It has a powerful alerting system via the Alertmanager.
Service discovery: Supports integration with different environments, such as Kubernetes, EC2, GCP, etc.
Setting up Prometheus in Production
1. Basic Configuration:

Prometheus works by scraping metrics from targets (applications, servers, etc.). You define the targets to scrape in a configuration file prometheus.yml.

Example prometheus.yml configuration:

yaml
Copy code
global:
  scrape_interval: 15s     # How often to scrape targets
  evaluation_interval: 15s # How often to evaluate rules

scrape_configs:
  - job_name: 'node_exporter'  # Name of the job (or target group)
    static_configs:
      - targets: ['localhost:9100'] # Target to scrape
scrape_interval: Defines how often Prometheus collects metrics.
evaluation_interval: Defines how often Prometheus evaluates rules (e.g., alerting).
scrape_configs: Specifies which services to scrape, identified by their IP/port.
Basic Metrics Collection with Node Exporter
To monitor system metrics such as CPU, memory, and disk usage, we can use a Prometheus exporter like node_exporter. This exporter exposes system metrics in a Prometheus-compatible format.

Example Steps:

Install node_exporter on your server.
Configure Prometheus to scrape the metrics by adding the following to your prometheus.yml file:
yaml
Copy code
scrape_configs:
  - job_name: 'node'
    static_configs:
      - targets: ['localhost:9100']  # Assuming node_exporter is running locally
Start both Prometheus and node_exporter. Prometheus will scrape metrics exposed by node_exporter at localhost:9100/metrics.

Open Prometheus UI at http://localhost:9090 and use PromQL to query the metrics.

Basic PromQL Example:

promql
Copy code
node_cpu_seconds_total
This query will return the total time spent by CPUs.

PromQL Queries:
PromQL (Prometheus Query Language) is used to select and aggregate time-series data.

Basic Queries:

up: Returns whether the target is up (1) or down (0).
node_memory_MemFree_bytes: Shows the free memory in bytes.
rate(node_cpu_seconds_total[1m]): Calculates the per-second rate of CPU usage over the last minute.
Tricky PromQL Question: How would you use PromQL to calculate the percentage of CPU usage over time?

promql
Copy code
100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
This query calculates the percentage of time the CPU was not idle, averaged over 5 minutes, for each instance.

Service Discovery:
In production, services often scale dynamically (e.g., in Kubernetes). Instead of hardcoding targets, Prometheus can use service discovery to automatically find services to scrape.

Example with Kubernetes service discovery:

yaml
Copy code
scrape_configs:
  - job_name: 'kubernetes-apiservers'
    kubernetes_sd_configs:
      - role: endpoints
    relabel_configs:
      - source_labels: [__meta_kubernetes_namespace]
        action: keep
        regex: default  # Only keep default namespace services
Prometheus Alerting:
Prometheus supports alerting by defining rules in the prometheus.yml file, and using Alertmanager to send notifications.

Example Alerting Rule:

yaml
Copy code
groups:
  - name: example
    rules:
      - alert: HighCPUUsage
        expr: avg(node_cpu_seconds_total) > 0.9
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Instance {{ $labels.instance }} has high CPU usage"
          description: "CPU usage is above 90% for more than 5 minutes."
Follow-up Tricky Questions:
What would happen if your scrape interval is significantly lower than the evaluation interval?

This could result in missing certain events or having delayed alerts, since the scrape interval should generally be at least as frequent as the evaluation interval.
How would you optimize Prometheus's storage in a high-scale environment?

Use remote storage for long-term data retention.
Downsample old data to reduce the size.
Set a retention period and compaction settings to control the size of stored data.

