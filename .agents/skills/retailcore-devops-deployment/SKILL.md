---
name: retailcore-devops-deployment
description: >-
  Use when containerizing, configuring CI/CD pipelines, orchestrating with Docker Compose, and setting up monitoring for RetailCore (Milestone 8).
  Guides Docker multi-stage builds, GitHub Actions workflows, Prometheus metrics, and Grafana dashboards.
---

# RetailCore DevOps, Containerization & Observability Skill

Bộ quy chuẩn đóng gói Docker, tự động hóa CI/CD và giám sát hệ thống trong **RetailCore** (Milestone 8).

---

## 1. KIẾN TRÚC ORCHESTRATION VỚI DOCKER COMPOSE

File `docker-compose.yml` điều phối toàn bộ hệ sinh thái dịch vụ:

```yaml
version: '3.8'

services:
  sqlserver:
    image: mcr.microsoft.com/mssql/server:2022-latest
    container_name: retailcore-sqlserver
    environment:
      - ACCEPT_EULA=Y
      - MSSQL_SA_PASSWORD=${MSSQL_SA_PASSWORD}
      - MSSQL_PID=Enterprise
    ports:
      - "1433:1433"
    volumes:
      - sqlserver-data:/var/opt/mssql
    healthcheck:
      test: ["CMD-SHELL", "/opt/mssql-tools18/bin/sqlcmd -S localhost -U sa -P \"$${MSSQL_SA_PASSWORD}\" -C -Q 'SELECT 1' || exit 1"]
      interval: 10s
      timeout: 5s
      retries: 5

  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: retailcore-backend
    environment:
      - SPRING_PROFILES_ACTIVE=prod
      - SPRING_DATASOURCE_URL=jdbc:sqlserver://sqlserver:1433;databaseName=RetailCoreDB;encrypt=true;trustServerCertificate=true
      - SPRING_DATASOURCE_USERNAME=sa
      - SPRING_DATASOURCE_PASSWORD=${MSSQL_SA_PASSWORD}
    ports:
      - "8080:8080"
    depends_on:
      sqlserver:
        condition: service_healthy
    healthcheck:
      test: ["CMD-SHELL", "wget -q --spider http://localhost:8080/actuator/health || exit 1"]
      interval: 15s
      timeout: 5s
      retries: 3

  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    container_name: retailcore-frontend
    ports:
      - "80:80"
    depends_on:
      backend:
        condition: service_healthy

  prometheus:
    image: prom/prometheus:latest
    container_name: retailcore-prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./docker/prometheus.yml:/etc/prometheus/prometheus.yml
    depends_on:
      - backend

  grafana:
    image: grafana/grafana:latest
    container_name: retailcore-grafana
    ports:
      - "3000:3000"
    depends_on:
      - prometheus

volumes:
  sqlserver-data:
```

---

## 2. QUY CHUẨN DOCKERFILE MULTI-STAGE BUILDS

1. **Backend (Java Spring Boot):**
   * Stage 1 (`builder`): Maven/Gradle build tạo file `.jar` tối ưu.
   * Stage 2 (`runner`): Base image `eclipse-temurin:21-jre-alpine` siêu nhẹ, chạy bằng user non-root để đảm bảo an ninh.
2. **Frontend (React + Vite):**
   * Stage 1 (`builder`): Node.js 20+ cài đặt dependencies và chạy `npm run build`.
   * Stage 2 (`runner`): Nginx Alpine nhẹ (dưới 25MB), cấu hình `try_files $uri $uri/ /index.html` để hỗ trợ React Router SPA.

---

## 3. QUY TRÌNH TỰ ĐỘNG HÓA CI/CD PIPELINE (GITHUB ACTIONS)

Luồng kiểm thử và đóng gói tự động trên mỗi Pull Request và commit vào nhánh `main`:
1. **Lint & Static Code Analysis:** Kiểm tra cú pháp TypeScript, ESLint, Checkstyle cho Java.
2. **Database Migration Verification:** Chạy kiểm tra Dry-Run toàn bộ script Flyway SQL để chống lỗi cú pháp T-SQL.
3. **Automated Testing:**
   * Unit tests.
   * Integration tests với Testcontainers MSSQL Server.
4. **Docker Image Build & Security Scan:** Quét lỗ hổng bảo mật với Trivy trước khi đẩy image lên Container Registry.
5. **Deployment:** Tự động kích hoạt deploy lên môi trường Staging/Production khi tests pass 100%.

---

## 4. QUY CHUẨN GIÁM SÁT HỆ THỐNG & METRICS (OBSERVABILITY)

Theo dõi chặt chẽ qua Prometheus và Grafana Dashboard:
* **Hạ tầng CSDL & Connection Pool (HikariCP):**
  * `hikaricp.connections.active` & `hikaricp.connections.pending`: Phát hiện sớm nguy cơ nghẽn kết nối khi lượng truy cập tăng vọt.
  * SQL Server Lock Waits & Deadlocks per second.
* **Hiệu năng Ứng dụng Backend:**
  * Throughput (RPS) và Response Time (P95, P99 latency).
  * Mã lỗi HTTP: Tỷ lệ lỗi 5xx phải luôn $< 0.01\%$.
* **Chỉ số Nghiệp vụ Trọng yếu (Business Metrics):**
  * Tốc độ tạo đơn hàng thành công (Orders/minute).
  * Tỷ lệ giải phóng reservation do hết hạn thanh toán (Reservation Expiry Rate).
  * Số lượng lỗi hết hàng Flash Sale (Out-of-Stock events).

