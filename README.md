'''mermaid
flowchart LR
  %% Direction
  %% Core layout
  subgraph SRC[Источники данных]
    A1[(OLTP БД/логи этапов)]:::src
    A2[(Файлы XLSX/CSV\nинциденты, справочники)]:::src
    A3[(BPMN/XML\nрепозиторий)]:::src
  end

  subgraph ING[Загрузка и оркестрация]
    B1[Коннекторы\nJDBC/ODBC + SQL]:::ing
    B2[Файловый приёмник\n(S3/MinIO/FTP/Shared)]::: ing
    B3[Парсер BPMN/XML]:::ing
    B4[Планировщик\nAirflow/Cron]:::sched
    B5[[Очередь задач\n(опц. Rabbit/Kafka)]]:::ing
  end

  subgraph STG[Хранилище «сырых» данных (staging)]
    C1[(raw_events)]:::stg
    C2[(raw_incidents)]:::stg
    C3[(raw_bpmn)]:::stg
    C4[(audit_logs)]:::stg
  end

  subgraph PROC[Обработка и расчёт метрик]
    D1[Нормализатор схемы\nмаппинг колонок, валидация времени]:::proc
    D2[Калькулятор длительностей\nstart/end → duration_sec]:::proc
    D3[Классификатор инцидентов\nregex/DSL по error_trace]:::proc
    D4[Оценка SLA\nпо справочнику задач]:::proc
    D5[Агрегатор\nпочас/день/неделя (p50,p95,mean,count)]:::proc
  end

  subgraph DWH[Хранилище метрик (DWH)]
    E1[(fact_events)]:::dwh
    E2[(fact_incidents)]:::dwh
    E3[(dim_elements)]:::dwh
    E4[(agg_hourly/agg_daily/agg_weekly)]:::dwh
    E5[(experiments_baseline\n& releases)]:::dwh
  end

  subgraph SVC[Сервисный слой]
    F1[Rules Engine\nпороги/условия алертов]:::svc
    F2[Report Builder\n(HTML/PDF/CSV)]:::svc
    F3[Metrics API\n(FastAPI)]:::svc
  end

  subgraph OUT[Потребление]
    G1[[BI/Дашборд\n(Streamlit/Metabase)]]:::out
    G2[[Отчёты по расписанию\nemail/Teams/Slack]]:::out
    G3[[Алерты при сбоях/SLA miss\nemail/webhook]]:::out
    G4[[Сравнение «до/после»\nи A/B-эксперименты]]:::out
  end

  %% Flows
  A1 --> B1
  A2 --> B2
  A3 --> B3
  B1 -.ETL.-> B4
  B2 -.ETL.-> B4
  B3 -.ETL.-> B4
  B4 -->|расписание| B5
  B5 --> C1
  B5 --> C2
  B5 --> C3
  B5 --> C4

  C1 --> D1 --> D2 --> E1
  C2 --> D1 --> D3 --> E2
  C3 --> D1 --> E3
  E1 --> D4 --> E1
  E1 --> D5 --> E4
  E2 --> D5

  E3 --> F3
  E4 --> F3
  E1 --> F3
  E2 --> F3
  E5 <-->|baseline/variant| F3

  F3 --> G1
  F2 --> G2
  F1 --> G3
  F3 --> F2
  F3 --> F1
  F3 --> G4

  classDef src fill:#eef7ff,stroke:#5b8def,stroke-width:1px
  classDef ing fill:#f2f7f2,stroke:#6ab17d,stroke-width:1px
  classDef sched fill:#e8fff0,stroke:#2ea043,stroke-width:1px
  classDef stg fill:#fff6e6,stroke:#f59e0b,stroke-width:1px
  classDef proc fill:#f3e8ff,stroke:#8b5cf6,stroke-width:1px
  classDef dwh fill:#e6f7fb,stroke:#0891b2,stroke-width:1px
  classDef svc fill:#fff0f3,stroke:#ec4899,stroke-width:1px
  classDef out fill:#f8fafc,stroke:#64748b,stroke-width:1px
'''
