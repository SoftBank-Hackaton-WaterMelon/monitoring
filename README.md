Container Monitoring Stack (Prometheus + Grafana + Loki)

📘 개요

이 프로젝트는 Docker 기반 모니터링 환경으로,
컨테이너의 리소스 사용량과 로그 데이터를 통합적으로 수집하고 시각화하기 위해 구성되었습니다.

모니터링은 두 가지 트랙으로 이루어집니다:

📊 리소스 모니터링: cAdvisor → Prometheus → Grafana

📜 로그 모니터링: Promtail → Loki → Grafana

🏗️ 아키텍처 개요

+---------------------+
|       Grafana       |  ← 시각화 (메트릭 + 로그)
+----------+----------+
           ↑
┌──────────┴──────────┐
│                     │
v                     v
Prometheus           Loki
(Metrics)           (Logs)
    ↑                     ↑
    |                     |
cAdvisor             Promtail
(Export Metrics)     (Ship Logs)


🧱 구성 서비스

서비스

포트

역할

cAdvisor

8081

Docker 컨테이너 리소스 메트릭 수집

Prometheus

9090

cAdvisor에서 메트릭 수집 및 저장

Loki

3100

로그 데이터 저장 (Promtail 수집 로그)

Promtail

-

컨테이너 로그 수집 및 Loki로 전달

Grafana

3000

시각화 대시보드 (Prometheus + Loki 연동)

⚙️ 파일 구조

📂 monitoring-stack/
├── docker-compose.yml
├── prometheus.yml
├── loki-config.yml
└── promtail-config.yml


🚀 실행 방법

1️⃣ 저장소 클론

git clone [https://github.com/](https://github.com/)<YOUR_REPO>/monitoring-stack.git
cd monitoring-stack


2️⃣ 컨테이너 실행

docker-compose up -d


3️⃣ 실행 확인

서비스

주소

Grafana

http://localhost:3000

Prometheus

http://localhost:9090

cAdvisor

http://localhost:8081

🧭 Grafana 설정

1. 데이터 소스 추가

Prometheus: http://prometheus:9090

Loki: http://loki:3100

2. 대시보드 불러오기

Grafana → Dashboards → “Import via grafana.com”

예시 Dashboard ID: 193 (cAdvisor 컨테이너 모니터링)

🧩 Prometheus 설정 (prometheus.yml)

Prometheus는 cAdvisor에서 메트릭을 수집하도록 설정되어 있습니다.

scrape_configs:
  - job_name: "cadvisor"
    static_configs:
      - targets: ["cadvisor:8080"]


📜 Promtail 설정 (promtail-config.yml)

Promtail은 Docker 로그를 수집하여 Loki로 전송합니다.

clients:
  - url: http://loki:3100/loki/api/v1/push

positions:
  filename: /tmp/positions.yaml

scrape_configs:
  - job_name: containers
    static_configs:
      - targets:
          - localhost
        labels:
          job: varlogs
          __path__: /var/lib/docker/containers/*/*.log


🧰 Loki 설정 (loki-config.yml)

Loki는 로그 데이터를 효율적으로 저장하기 위한 기본 구성입니다.

server:
  http_listen_port: 3100

ingester:
  lifecycler:
    address: 127.0.0.1
  chunk_idle_period: 5m
  max_chunk_age: 1h

schema_config:
  configs:
    - from: 2024-01-01
      store: boltdb-shipper
      object_store: filesystem
      schema: v11
      index:
        prefix: index_
        period: 24h


💡 Tip

수집 주기: Prometheus 기본 15초 → 필요 시 scrape_interval 수정 가능

데이터 보존 기간: Prometheus/Loki 설정 파일에서 조정

알림 설정: Grafana Alerting 기능을 이용해 Slack, Email 연동 가능

🧹 컨테이너 종료

docker-compose down


Note: 필요 시 데이터 영구 보존을 위해 docker-compose.yml의 volumes: 항목 수정 또는 마운트 경로 지정이 가능합니다.

📎 참고

Prometheus Docs

Grafana Docs

Loki Docs

cAdvisor GitHub

Promtail GitHub
