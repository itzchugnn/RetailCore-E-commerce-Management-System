---
name: retailcore-devops-deployment
description: >-
  Use when containerizing, configuring CI/CD pipelines, orchestrating with Docker Compose, and setting up monitoring for RetailCore (Milestone 8).
  Guides Docker multi-stage builds, SQL Server initialization, GitHub Actions workflows, Prometheus metrics, and Grafana dashboards.
---

# RetailCore DevOps, Containerization & Observability Skill

Bộ quy chuẩn đóng gói Docker, tự động hóa CI/CD và giám sát hệ thống trong **RetailCore** (Milestone 8).

---

## 1. KIẾN TRÚC ORCHESTRATION VỚI DOCKER COMPOSE

1. **Khởi tạo SQL Server 2022 An toàn (Tránh lỗi `Msg 4060` & Vi phạm `sa`):**
   * Sử dụng `MSSQL_PID=Developer` cho môi trường phát triển/kiểm thử/demo (không dùng `Enterprise` không giấy phép).
   * Bắt buộc có container chạy một lần `sqlserver-init` thực thi script tạo `RetailCoreDB`, các Filegroups vật lý và 2 tài khoản `flyway_migrator` / `retailcore_app` **trước khi** container `backend` khởi động!
   * `backend` kết nối runtime bằng `retailcore_app` (bảo toàn hiệu lực của `REVOKE DML` theo `CORR-04`) và chạy Flyway bằng `flyway_migrator`.

```yaml
services:
  sqlserver:
    image: mcr.microsoft.com/mssql/server:2022-latest
    container_name: retailcore-sqlserver
    environment:
      ACCEPT_EULA: "Y"
      MSSQL_PID: "Developer"
      MSSQL_SA_PASSWORD: "${MSSQL_SA_PASSWORD}"
    ports:
      - "1433:1433"
    volumes:
      - sqlserver-data:/var/opt/mssql
    healthcheck:
      test: ["CMD-SHELL", "/opt/mssql-tools18/bin/sqlcmd -S localhost -U sa -P \"$${MSSQL_SA_PASSWORD}\" -C -Q 'SELECT 1' || exit 1"]
      interval: 10s
      timeout: 5s
      retries: 10

  sqlserver-init:
    image: mcr.microsoft.com/mssql/server:2022-latest
    depends_on:
      sqlserver:
        condition: service_healthy
    volumes:
      - ./database/01_database:/init:ro
    entrypoint: ["/bin/bash", "-c", "/opt/mssql-tools18/bin/sqlcmd -S sqlserver -U sa -P \"$${MSSQL_SA_PASSWORD}\" -C -i /init/00_bootstrap_db_and_logins.sql"]
    environment:
      MSSQL_SA_PASSWORD: "${MSSQL_SA_PASSWORD}"

  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: retailcore-backend
    environment:
      SPRING_PROFILES_ACTIVE: "docker"
      SPRING_DATASOURCE_URL: "jdbc:sqlserver://sqlserver:1433;databaseName=RetailCoreDB;encrypt=true;trustServerCertificate=true"
      SPRING_DATASOURCE_USERNAME: "retailcore_app"
      SPRING_DATASOURCE_PASSWORD: "${DB_APP_PASSWORD}"
      SPRING_FLYWAY_USER: "flyway_migrator"
      SPRING_FLYWAY_PASSWORD: "${DB_MIGRATOR_PASSWORD}"
    ports:
      - "8080:8080"
    depends_on:
      sqlserver-init:
        condition: service_completed_successfully

  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    container_name: retailcore-frontend
    ports:
      - "3000:80"
    depends_on:
      - backend

  prometheus:
    image: prom/prometheus:latest
    container_name: retailcore-prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./docker/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml:ro
      - prometheus-data:/prometheus

  grafana:
    image: grafana/grafana:latest
    container_name: retailcore-grafana
    ports:
      - "3001:3000"
    volumes:
      - grafana-data:/var/lib/grafana
    depends_on:
      - prometheus

volumes:
  sqlserver-data:
  prometheus-data:
  grafana-data:
```

---

## 2. DOCKERFILE MULTI-STAGE & CI/CD PIPELINE

* **Backend:** Maven/Gradle builder $\to$ `eclipse-temurin:21-jre-alpine` chạy bằng non-root user.
* **Frontend:** Node 20+ Vite builder $\to$ `nginx:alpine` với `try_files $uri $uri/ /index.html`.
* **GitHub Actions CI/CD:** Lint (ESLint/TypeScript/Checkstyle) $\to$ Testcontainers SQL Server 2022 Integration Tests (kiểm tra Flyway + Constraints + SPs + `REVOKE DML`) $\to$ Build Docker Images & Trivy Scan.

---

## 3. GIÁM SÁT HỆ THỐNG & METRICS SQL SERVER (OBSERVABILITY)

* **Spring Boot Micrometer (`/actuator/prometheus`):** Đo HikariCP pool (`hikaricp.connections.active`, `pending`), JVM memory, HTTP P95/P99 latency và custom Business Counters (Orders/min, Reservation Expiry, Flash Sale Out-of-Stock).
* **SQL Server Engine Metrics:** Thu thập chỉ số Lock Waits, Deadlocks/sec và Buffer Cache Hit Ratio thông qua `prometheus-mssql-exporter` (hoặc custom Micrometer MeterBinder đọc `sys.dm_os_performance_counters` bằng quyền `VIEW SERVER STATE`).
