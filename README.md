# Distributed Systems Docker Stack

This repository contains a `docker-compose.yml` file to spin up a **production-ready distributed systems stack** with observability, databases, messaging, auth, and DevOps tools.
All data is persisted in `C:\Dhruba\Work\DIp\volumes`.

---

---

## 📁 Setup

### 1. Prerequisites
- [Docker Desktop](https://www.docker.com/products/docker-desktop/) (Windows)
- At least **16GB RAM** allocated to Docker (recommended for all services)
- **WSL 2** enabled (for better performance on Windows)

---

### 2. Clone or Download
- Clone this repo or download the `docker-compose.yml` and `.env` files.

---

### 3. Configure Environment
- Create a `.env` file in the same directory as `docker-compose.yml` with the following content:
  ```VOLUMES_BASE_PATH=C:\Dhruba\Work\DIp\volumes```
- Create Directory
  ```New-Item -ItemType Directory -Path "C:\Dhruba\Work\DIp\volumes"```
  
### 4. Create Config Files
Some services require config files. Create the following files in their respective directories:
- Prometheus

Create C:\Dhruba\Work\DIp\volumes\prometheus\prometheus.yml:
```global:
  scrape_interval: 15s
scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
```
- Fluent Bit

Create C:\Dhruba\Work\DIp\volumes\fluent-bit\fluent-bit.conf:
```
[SERVICE]
    Flush        1
    Log_Level    info
    Daemon       off

[INPUT]
    Name        tail
    Path        /var/log/containers/*.log
    Tag         docker.*

[OUTPUT]
    Name        stdout
    Match       *
```
- OpenTelemetry Collector

Create C:\Dhruba\Work\DIp\volumes\otel-collector\config.yaml:
```
receivers:
  otlp:
    protocols:
      grpc:
      http:

processors:
  batch:

exporters:
  logging:
    loglevel: debug

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [batch]
      exporters: [logging]
    metrics:
      receivers: [otlp]
      processors: [batch]
      exporters: [logging]
    logs:
      receivers: [otlp]
      processors: [batch]
      exporters: [logging]
```

### Start stack
1. Open a PowerShell or Command Prompt in the directory containing docker-compose.yml.
2. Run:
```
docker-compose up -d
```
3. Check status:
```
docker-compose ps
```

###  Accessing Services
Use the following URLs and credentials to access each service:
| Service | URL | Credentials | Notes |
| --- | --- | --- | --- |
| Nginx | [http://localhost](http://localhost) | - | Reverse proxy |
| Kong Admin | [http://localhost:8001](http://localhost:8001) | - | API Gateway Admin UI |
| PostgreSQL | localhost:5432 | User: postgres, Pass: postgres | Use psql or pgAdmin |
| pgAdmin | [http://localhost:5050](http://localhost:5050) | Email: admin@admin.com, Pass: admin | PostgreSQL UI |
| MongoDB | localhost:27017 | User: root, Pass: root | Use MongoDB Compass or CLI |
| Cassandra | localhost:9042 | - | Use cqlsh or DataStax DevCenter |
| Redis | localhost:6379 | - | Use redis-cli or RedisInsight |
| RabbitMQ | [http://localhost:15672](http://localhost:15672) | User: admin, Pass: admin | Management UI |
| Kafka | localhost:9092 | - | Use kafka-console-producer/consumer |
| Keycloak | [http://localhost:8080](http://localhost:8080) | Admin: admin, Pass: admin | Auth Server UI |
| Vault | [http://localhost:8200](http://localhost:8200) | Root Token: root | Secrets Management UI |
| Consul | [http://localhost:8500](http://localhost:8500) | - | Service Discovery UI |
| Prometheus | [http://localhost:9090](http://localhost:9090) | - | Metrics UI |
| Grafana | [http://localhost:3000](http://localhost:3000) | User: admin, Pass: admin | Dashboards UI |
| Jaeger | [http://localhost:16686](http://localhost:16686) | - | Distributed Tracing UI |
| MailHog | [http://localhost:8025](http://localhost:8025) | - | SMTP Testing UI |
| Portainer | [http://localhost:9000](http://localhost:9000) | - | Docker Management UI |
| SonarQube | [http://localhost:9001](http://localhost:9001) | User: admin, Pass: admin | Code Quality UI |
| Jenkins | [http://localhost:8081](http://localhost:8081) | - (Check logs for initial password) | CI/CD UI |
| LocalStack | [http://localhost:4566](http://localhost:4566) | - | AWS Emulation UI |

### Stopping the Stack

To stop all containers:
```
docker-compose down
docker-compose down -v(to remove persisted data)
```
Troubleshooting:
- Port Conflicts
If a port is already in use, stop the conflicting service or change the port in docker-compose.yml.
Permission Issues (Windows)
- Ensure Docker has access to C:\Dhruba\Work\DIp\volumes.
In Docker Desktop:
Go to Settings > Resources > File Sharing.
Add C:\Dhruba\Work\DIp\volumes to the list of shared drives.
- Out of Memory
Allocate more RAM to Docker in Settings > Resources.
Reduce the number of services running simultaneously.
- Slow Performance
Enable WSL 2 in Docker Desktop settings.
Allocate more CPU cores to Docker.

Service-Specific Issues

Kafka: May take a while to start. Check logs with:
```
docker-compose logs kafka
docker-compose logs jenkins
```








































