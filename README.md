# 📊 Сравнение dbt-core и SQLMesh  
## Для стека: HDFS → ClickHouse → Airflow → Superset

> **Контекст**:  
> - Источник данных: **HDFS**  
> - Загрузка: **HDFS → ClickHouse** (через Spark/Airflow)  
> - Трансформации: агрегации, view, доп. таблицы в **ClickHouse**  
> - Оркестрация: **Airflow**  
> - Визуализация: **Superset**  
> - Цель: надёжный, автоматизированный, эффективный data pipeline с CI/CD

---

## 🆚 Сравнительная таблица
<img width="1225" height="217" alt="image" src="https://github.com/user-attachments/assets/72cd0913-d572-4ebb-a839-4e29dbba91a2" />



## 📄 Пример SQL-модели для SQLMesh

Файл: `models/events_daily.sql`

```sql
MODEL (
    name mart.events_daily,
    kind INCREMENTAL_BY_TIME_RANGE (
        time_column event_date
    ),
    start '2024-01-01',
    cron '@daily',
    columns (
        event_date DATE,
        event_count INT
    )
);

SELECT
    toDate(event_time) AS event_date,
    count(*) AS event_count
FROM raw.events
WHERE
    event_time BETWEEN @start_date AND @end_date
GROUP BY event_date
```

### Пояснение:
- **`name`**: таблица будет создана как `mart.events_daily` в ClickHouse  
- **`kind INCREMENTAL_BY_TIME_RANGE`**: пересчёт только за указанный день  
- **`@start_date` / `@end_date`**: автоматически подставляются SQLMesh (например, из Airflow `execution_date`)  
- **Фильтрация по дате**: обязательна для корректной инкрементальности  

---

## 🛠️ Архитектура вашего pipeline с SQLMesh

```
[HDFS]
   ↓ (Airflow: SparkSubmitOperator)
[ClickHouse.raw.events]
   ↓ (Airflow: SQLMeshAirflowOperator)
[ClickHouse.mart.events_daily] ← SQLMesh модель
   ↓
[Superset] ← подключён к mart-схеме
```

---

## ✅ Заключение

**SQLMesh — оптимальный выбор** для вашего стека, потому что он:
- Нативно поддерживает **ClickHouse**
- Интегрируется с **Airflow** "из коробки"
- Обеспечивает **безопасные, инкрементальные трансформации**
- Позволяет строить **полностью автоматизированный CI/CD**
- Работает прозрачно с **Superset**
