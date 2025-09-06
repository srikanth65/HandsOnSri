 # How to Answer: "What Metrics Do You Scrape with Prometheus?"

## Interview Answer Structure

### 1. Start with Overview (30 seconds)
"In our organization, we use Prometheus to monitor our entire stack across infrastructure, applications, and
business metrics. Let me break down our monitoring strategy by category."

### 2. Infrastructure Metrics (2 minutes)

#### **Node/Server Metrics (Node Exporter)**
yaml
# Key metrics we collect:
- node_cpu_seconds_total          # CPU usage per core
- node_memory_MemAvailable_bytes  # Available memory
- node_filesystem_avail_bytes     # Disk space available
- node_load1, node_load5, node_load15  # System load
- node_network_receive_bytes_total # Network I/O
- node_disk_io_time_seconds_total # Disk I/O latency


"We monitor CPU utilization, memory usage, disk space, network I/O, and system load across all our EC2 instances
and on-premise servers."

#### **Kubernetes Metrics (kube-state-metrics)**
yaml
# Container and Pod metrics:
- kube_pod_status_phase           # Pod lifecycle states
- kube_deployment_status_replicas # Deployment health
- container_cpu_usage_seconds_total # Container CPU
- container_memory_working_set_bytes # Container memory
- kube_node_status_condition      # Node health


"For our Kubernetes clusters, we track pod states, resource utilization, node health, and deployment status."

### 3. Application Metrics (3 minutes)

#### **HTTP/API Metrics**
yaml
# Custom application metrics:
- http_requests_total{method, status, endpoint}  # Request count
- http_request_duration_seconds                  # Response time
- http_requests_in_flight                        # Concurrent requests
- api_errors_total{service, error_type}          # Error tracking


"We instrument our microservices to track request rates, response times, error rates, and concurrent connections.
This follows the RED method - Rate, Errors, Duration."

#### **Database Metrics**
yaml
# PostgreSQL metrics (postgres_exporter):
- pg_stat_database_tup_returned    # Queries executed
- pg_stat_database_tup_inserted    # Insert operations
- pg_locks_count                   # Lock contention
- pg_stat_activity_count           # Active connections
- pg_stat_bgwriter_buffers_clean   # Buffer performance

# Redis metrics (redis_exporter):
- redis_connected_clients          # Client connections
- redis_memory_used_bytes          # Memory usage
- redis_keyspace_hits_total        # Cache hit rate
- redis_commands_processed_total   # Command throughput


#### **Message Queue Metrics**
yaml
# RabbitMQ metrics:
- rabbitmq_queue_messages          # Queue depth
- rabbitmq_queue_messages_ready    # Messages ready for delivery
- rabbitmq_queue_consumers         # Consumer count
- rabbitmq_channel_messages_published # Message publish rate


### 4. Business Metrics (1 minute)

#### **Custom Business Metrics**
yaml
# Application-specific metrics:
- user_registrations_total         # New user signups
- orders_completed_total           # Successful transactions
- payment_processing_duration      # Payment latency
- active_users_gauge              # Current active users
- revenue_total{product_type}      # Revenue tracking


"We also track business KPIs like user registrations, order completion rates, and revenue metrics to correlate
technical performance with business impact."

### 5. Alerting Metrics (1 minute)

#### **SLA/SLO Metrics**
yaml
# Service Level Indicators:
- sli_availability_ratio           # Service uptime
- sli_latency_p99                 # 99th percentile latency
- sli_error_rate                  # Error budget consumption
- sli_throughput                  # Request throughput


"We define SLIs for availability (99.9%), latency (p95 < 200ms), and error rate (< 0.1%) to track our SLOs and
trigger alerts when we're at risk of breaching SLAs."

### 6. Monitoring Stack Integration (1 minute)

#### **Complete Observability Stack**
yaml
# Our monitoring ecosystem:
Prometheus → Grafana (Visualization)
     ↓
AlertManager → PagerDuty/Slack (Alerting)
     ↓
Jaeger (Distributed Tracing)
     ↓
ELK Stack (Logging)


"We use Grafana for visualization, AlertManager for notifications, and correlate metrics with logs from our ELK
stack and traces from Jaeger."

### 7. Specific Configuration Examples (2 minutes)

#### **Prometheus Configuration**
yaml
# Sample scrape config:
scrape_configs:
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
    - role: pod
    relabel_configs:
    - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
      action: keep
      regex: true
    - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
      action: replace
      target_label: __metrics_path__
      regex: (.+)

  - job_name: 'node-exporter'
    static_configs:
    - targets: ['node-exporter:9100']
    scrape_interval: 15s


#### **Custom Metrics Implementation**
go
// Example application instrumentation:
var (
    httpRequestsTotal = prometheus.NewCounterVec(
        prometheus.CounterOpts{
            Name: "http_requests_total",
            Help: "Total HTTP requests",
        },
        []string{"method", "endpoint", "status"},
    )

    httpDuration = prometheus.NewHistogramVec(
        prometheus.HistogramOpts{
            Name: "http_request_duration_seconds",
            Help: "HTTP request duration",
            Buckets: prometheus.DefBuckets,
        },
        []string{"method", "endpoint"},
    )
)


### 8. Operational Insights (1 minute)

#### **Key Learnings**
"Some important practices we've learned:

Cardinality Management:
• We limit label cardinality to prevent memory issues
• Use recording rules for expensive queries
• Regular cleanup of stale metrics

Performance Optimization:
• 15-second scrape intervals for most metrics
• 1-minute intervals for expensive exporters
• Retention of 15 days for detailed metrics, 1 year for aggregated data

Alert Fatigue Prevention:
• Meaningful alert thresholds based on historical data
• Alert grouping and routing based on severity
• Runbooks for every alert with clear remediation steps"

## Sample Follow-up Questions & Answers

Q: "How do you handle high cardinality metrics?"
A: "We use recording rules to pre-aggregate high-cardinality metrics, implement label dropping for unnecessary
dimensions, and use metric relabeling to normalize label values. For user-specific metrics, we sample or use
separate time series databases like InfluxDB."

Q: "What's your alerting strategy?"
A: "We follow a tiered approach: P1 for service-down scenarios (immediate PagerDuty), P2 for SLO violations (
Slack during business hours), P3 for capacity planning (email). Each alert includes runbooks and is tied to
specific SLIs."

Q: "How do you ensure Prometheus itself is highly available?"
A: "We run Prometheus in HA mode with multiple replicas, use Thanos for long-term storage and global querying,
implement federation for scaling, and have dedicated monitoring for our monitoring stack."

Q: "What challenges have you faced with Prometheus?"
A: "Main challenges include managing storage growth (solved with Thanos), handling high cardinality (recording
rules and sampling), and ensuring consistent labeling across teams (solved with governance and automated
validation)."

## Pro Tips for the Interview

1. Be Specific: Mention actual metric names and values, not just concepts
2. Show Progression: Start with basic metrics, then explain how you evolved
3. Mention Tools: Reference specific exporters and integrations you've used
4. Discuss Challenges: Show you understand operational complexities
5. Business Context: Connect technical metrics to business outcomes
6. Best Practices: Demonstrate understanding of cardinality, retention, and performance
7. Real Numbers: "We monitor 500+ services with 10M+ metrics" shows scale

## Key Metrics Categories to Cover

✅ Infrastructure: CPU, memory, disk, network
✅ Application: Request rate, errors, duration (RED method)
✅ Database: Connections, queries, locks, performance
✅ Business: User actions, revenue, conversions
✅ SLI/SLO: Availability, latency, error budgets
✅ Security: Failed logins, suspicious activities
✅ Cost: Resource utilization, cloud spend
