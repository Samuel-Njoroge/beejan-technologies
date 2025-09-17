# Business Scenario

You are working as a Data Engineer at a telecom company called **Beejan Technologies**. Every day, thousands of customers complain about issues like poor network, incorrect billing, or bad customer service. These complaints come through different channels: social media, call center log files, SMS, and website forms.  

The management is frustrated. Data is stored in different formats. The reporting team manually compiles spreadsheets. No single pipeline exists for this data flow. Reports are delayed. Teams work in silos.  

They ask you to design a solution to bring all this data together, clean it, enrich it and make it ready to provide actionable insights.  

Your first job is to design a **conceptual end-to-end data pipeline** to solve this problem.  

This conceptual data pipeline will lay the foundation for building the actual pipeline. 

## Steps and Questions

| Step                          | Questions                                                                 |
|-------------------------------|---------------------------------------------------------------------------|
| **1. Source Identification**  | - What are the data sources? <br> - What formats and frequency (batch or streaming or both)? |
| **2. Ingestion Strategy**     | - Will you use API ingestion, file uploads, or streaming? <br> - How will real-time data (e.g. Twitter) be handled? |
| **3. Processing/Transformation** | - How will you clean and standardize the data? <br> - How will you classify complaints into categories? |
| **4. Storage Options**        | - Do you need a data lake? A warehouse? Both? <br> - What format will the cleaned data be stored in (Parquet, JSON, etc.)? |
| **5. Serving**                | - How will the data be queried? <br> - How will the downstream users use this data? |
| **6. Orchestration & Monitoring** | - How often will this pipeline run? <br> - How will failures be detected or notified? |
| **7. DataOps**                | - Where will the pipeline run? <br> - How will you make it available in production? |


## Conceptual Pipeline Architecture.

### 1. Source Identification
**Sources**
- Social Media (public posts + DMs)
- Call center logs (voice transcripts + metadata)
- SMS
- Website/contact forms

**Formats & Frequency** 
- Streaming (social media public stream, live chat)
- Near‑real‑time streaming for some partner feeds.
- Batch uploads for daily call logs and periodic exports from legacy systems.

### 2. Ingestion Strategy

**Hybrid ingestion**
- Streaming collectors for real‑time channels.
- Batch upload endpoints for files (CSV, JSON, XML, log files).

**Real‑time handling**
- Social media public stream and live chat event streams --> buffered --> validated --> appended to raw storage.

### 3. Processing & Transformation

**Cleaning & Standardization Stages**

- Schema detection & field mapping for inputs.

- Timestamp normalization to UTC.

- Deduplication by content hash + customer/time window.
  
- Normalization of contact identifiers (phone, email) using canonicalizers.

**Enrichment**

- NLP layer: language detection, sentiment scoring, named‑entity recognition (locations, product names, account numbers), key‑phrase extraction.

- Intent classification to detect complaint types (network, billing, service, handset, others).

- Confidence scores attached for downstream filtering.

**Classification rules:**  Deterministic rules (keywords, regexes, matching to known billing codes) --> ML classifier --> categories and subcategories.

**Data quality:** Automated validation rules with quarantining of malformed records for manual review.

**Privacy:** PII detection and masking or tokenization before moving to curated stores.

### 4. Storage Options

**Zones**

- Raw Zone holding original payloads.

- Processed/Curated Zone with cleaned, enriched, columnar storage preferably Parquet format partitioned by date & channel for fast access.

- Serving/Analytical Store optimized for low‑latency queries and dashboards.

** File Formats** 
- Columnar files (Parquet) for analytics and ML training artifacts.
- JSON or Avro for event interchange where schema evolution matters.

### 5. Serving

**Querying** 
- Analytical layer that supports SQL queries.
- Ad‑hoc exploration and scheduled aggregates for KPIs (MTTR, volume by category, SLA violations).

**Downstream usage**

- Dashboards for operations and management with near‑real‑time KPIs.

- Alerting & routing rules to operations teams.

- ML model training datasets (historical labeled complaints) for improving classification and churn/prediction models.

- APIs for customer care systems to fetch consolidated complaint history per customer.

### 6. Orchestration & Monitoring

- **Orchestration:** Streaming pipelines run continuously; batch jobs for legacy systems run on a schedule (hourly/daily). Reconciliation jobs run daily to ensure completeness.

- **Observability:** Metrics (throughput, lag, error rates), data quality dashboards (nulls, schema changes, duplicate rates), and lineage for traceability.

- **Failure handling:** Automatic retries with exponential backoff, dead‑letter queues for poisoned messages, and alerting (email/IM/pager) for sustained failures or SLA breaches.

### 7. DataOps & Productionization

- **Deployment:** Pipelines are packaged and deployed to an orchestration environment with versioned artifacts and configuration per environment (dev/stage/prod).

- **Access & governance:** Role‑based access to raw vs curated zones; audit logging for data access; schemas and contracts versioned. Data retention policies applied per regulatory needs.

### Design choices

1. Hybrid streaming + batch covers both real‑time and legacy systems.

2. Immutable raw storage preserves original inputs for audits & reprocessing.

3. Columnar curated storage optimizes analytics and ML training.

4. Hybrid classification (rules + ML) balances precision and explainability.

5. Strong observability and DLQ patterns reduce operational risk.

### Assumptions

1. Customer identifiers are present in at least some channels or can be resolved via matching heuristics.

2. Volume is large but manageable with partitioning and time‑based retention.

3. Legal/regulatory constraints require PII minimization and auditability.

4. Teams will adopt APIs and shared datasets rather than continue manual spreadsheets.

### Challenges

1. Identity resolution across channels may be challenging (phone, email, anonymized social handles).

2. Labeling training data for ML classifiers may require human‑in‑the‑loop to reach high accuracy.

3. Handling multimedia (call recordings, images) needs additional processing and storage considerations.

4. Defining exact SLAs for near‑real‑time reporting vs. daily reconciliation.
