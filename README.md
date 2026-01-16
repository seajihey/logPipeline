# 📜 로그 파이프라인 실습

본 문서는 **Filebeat → Logstash → Elasticsearch + RDBMS** 구조의  
로그 수집·가공·저장 파이프라인을 직접 구축하고 실습한 내용을 정리한 문서입니다.

서버에 생성되는 로그를 실시간으로 수집하고,  
가공 후 목적지에 따라 **Elasticsearch(분석·시각화)** 와  
**RDBMS(영구 저장)** 로 분산 저장하는 전체 흐름을 다룹니다.

---

## 🧩 1. 전체 아키텍처 개요

<img width="100%" alt="architecture" src="https://github.com/user-attachments/assets/d5c841f8-9199-4753-b194-047ef06e0e3c" />

- ✅ 애플리케이션에서 발생한 로그는 `app.logs` 파일에 기록됩니다.
- ✅ Filebeat가 로그 파일을 실시간으로 감시하며 로그를 수집합니다.
- ✅ 수집된 로그는 Logstash로 전달되어 파싱 및 데이터 가공을 거칩니다.
- ✅ 가공된 로그는 Elasticsearch에 저장되며 Kibana를 통해 시각화·분석됩니다.

---

## ⚙️ 2. 구성 요소별 역할

### 📦 Filebeat

- 로그 수집 에이전트
- 서버에 생성되는 로그 파일을 실시간 감시
- 로그를 이벤트 단위로 Logstash에 전송

**특징**
- 가볍고 빠른 구조
- 데이터 가공 및 저장 기능 없음
- 로그 생성 서버에 상주하며 자동 수집 수행

---

### 🛠 Logstash

- 로그 중앙 처리 시스템
- 수집된 로그를 가공 및 분기 처리

**주요 기능**
- 로그 파싱
- 필드 분리 및 타입 변환
- 불필요한 로그 제거
- 목적지별 라우팅 (Elasticsearch, RDBMS)

<img width="70%" alt="logstash-flow" src="https://github.com/user-attachments/assets/bf07da5a-6c64-4f65-befd-eafdc126e3ea" />

*본 이미지는 AI로 제작되었습니다.

---

### 🔍 Elasticsearch

- 대용량 로그 데이터 실시간 검색·집계 엔진
- Kibana를 통한 시각화 및 모니터링 수행

**사용 목적**
- 로그 검색
- 실시간 집계
- 대시보드 기반 분석

| 항목 | RDBMS | Elasticsearch |
|----|----|----|
| 대용량 로그 | ❌ | ⭕ |
| 전문 검색 | ❌ | ⭕ |
| 실시간 집계 | 느림 | 매우 빠름 |
| 스키마 유연성 | 낮음 | 높음 |

---

## 🧪 3. 실습 단계

### 📝 Step 1. 로그 생성

- Filebeat가 감시할 로그 파일을 준비합니다.
- 애플리케이션 로그는 테스트 목적으로 AI를 활용해 생성했습니다.

<img width="100%" alt="log-sample" src="https://github.com/user-attachments/assets/24b7ac80-a8ed-414b-a4e7-591eb917d26e" />

---

### 🗄 Step 2. RDBMS 테이블 생성

- HTTP 로그를 영구 저장하기 위해 Oracle RDBMS에  
  <app_http_logs> 테이블을 생성합니다.
- Logstash를 통해 가공된 로그 데이터가 저장됩니다.

<img width="100%" alt="db-table" src="https://github.com/user-attachments/assets/c7b5c7f3-2521-4005-9939-d659f757b096" />

- 시퀀스와 트리거를 사용하여 `id` 값을 자동 생성하도록 구성했습니다.

---

### 🧾 Step 3. Filebeat 설정

- <filebeat.yml>에서 로그 파일 경로를 지정하고  
  로그 발생 시 Logstash로 전송되도록 설정합니다.

<table>
  <tr>
    <td><img src="https://github.com/user-attachments/assets/06222aea-09af-48ef-a318-78ee7fb3fa16" width="100%"/></td>
    <td><img src="https://github.com/user-attachments/assets/21a9c03c-3769-4ad4-b539-4e90b83e34c3" width="100%"/></td>
    <td><img src="https://github.com/user-attachments/assets/d2541a6e-b931-45a2-acbd-6fc961ac7c18" width="100%"/></td>
  </tr>
</table>

---

### 🔧 Step 4. Logstash 설정

- <input → filter → output> 구조로 구성합니다.
- Filebeat 로그 수신 → 파싱 및 가공 → Elasticsearch 전송 흐름입니다.

<img width="100%" alt="logstash-config" src="https://github.com/user-attachments/assets/e581ae5d-9c43-40a6-b3cf-083339eaa0cb" />

---

### ▶️ Step 5. 실행

<img width="100%" alt="run" src="https://github.com/user-attachments/assets/996ddbb9-b531-420f-8a15-c352d9fe2cac" />

---

### 📊 Step 6. 결과 확인

<table>
  <tr>
    <td><img src="https://github.com/user-attachments/assets/eddbb368-1908-45ac-8240-d338355ac269" width="100%"/></td>
    <td><img src="https://github.com/user-attachments/assets/3b20b59e-939c-4a09-b6e1-37045f59c2ad" width="100%"/></td>
  </tr>
</table>

---

## ✅ 6. 정리

- **Filebeat**: 로그 수집 전용
- **Logstash**: 로그 가공 및 분기 처리
- **Elasticsearch**: 실시간 분석 및 시각화
- **RDBMS**: 정합성 있는 장기 저장

본 실습을 통해 대용량 로그를 안정적으로 수집하고, 가공된 데이터를 목적에 맞게 저장·분석하는  
로그 파이프라인 구조를 이해할 수 있었습니다.
