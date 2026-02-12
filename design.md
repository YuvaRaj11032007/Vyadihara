# Vyadiharah — System Design Document

> **An AI-Driven Epidemiological Intelligence & Predictive Surveillance Layer for the ABDM Ecosystem**

---

## 1. Document Control

| Field               | Value                          |
|----------------------|--------------------------------|
| **Project Name**     | Vyadiharah                     |
| **Version**          | 1.0.0                         |
| **Status**           | Draft                         |
| **Date**             | 2026-02-12                    |
| **Classification**   | Confidential                  |
| **Companion Doc**    | `requirements.md` v1.0.0      |

---

## 2. Design Philosophy

Vyadiharah is designed around four guiding principles:

1. **FHIR-First** — All clinical data enters and exits the platform in HL7 FHIR R4 format. Amazon HealthLake is the single source of truth. No proprietary data silos.
2. **Consent-Native** — Every data access path is gated by the ABDM consent framework. The platform never touches patient data without a valid, purpose-scoped, time-bound consent artefact.
3. **AI-Augmented, Human-Governed** — AI (Bedrock, SageMaker) accelerates analysis but never autonomously communicates clinical conclusions to patients. A clinician or health officer is always in the loop.
4. **Event-Driven & Serverless** — The architecture favours asynchronous, event-driven patterns using AWS Lambda, EventBridge, and SQS, minimising operational overhead and scaling automatically.

---

## 3. High-Level Architecture

### 3.1 System Context Diagram (C4 — Level 1)

```
                                    +----------------------+
                                    |     CITIZENS         |
                                    |  (ABHA Holders)      |
                                    +----------+-----------+
                                               | SMS Alerts &
                                               | Web Portal
                                               v
+------------------+            +----------------------------------+           +------------------+
|                  |  FHIR R4   |                                  | Dashboards|                  |
|  CLINICS &       |----------->|         VYADIHARAH               |<----------|  HEALTH OFFICERS |
|  HOSPITALS       |            |         PLATFORM                 |           |  & EPIDEMIOL.    |
|  (HIP / HRP)    |            |                                  |           |                  |
+------------------+            |  +--------+  +----------------+  |           +------------------+
                                |  | AI     |  | Surveillance   |  |
                                |  | Engine |  | Engine         |  |
+------------------+            |  +--------+  +----------------+  |           +------------------+
|                  |  REST API  |  +--------+  +----------------+  |  Alerts   |                  |
|  PHARMACIES      |<---------->|  | Smart  |  | Consent &      |  |---------->|  REGULATORY      |
|  (POS Terminals) |            |  | Disp.  |  | Identity       |  |           |  AUTHORITIES     |
|                  |            |  +--------+  +----------------+  |           |                  |
+------------------+            +--------------+-------------------+           +------------------+
                                               |
                                               | ABDM APIs
                                               v
                                    +----------------------+
                                    |     ABDM GATEWAY     |
                                    |  (NHA Infrastructure)|
                                    +----------------------+
```

### 3.2 Container Diagram (C4 — Level 2)

```
+-----------------------------------------------------------------------------------------+
|                              VYADIHARAH PLATFORM (AWS ap-south-1)                       |
|                                                                                         |
|  +-------------------------------------------------------------------------------------+ |
|  |                           API LAYER (Amazon API Gateway)                            | |
|  |                                                                                     | |
|  |  /v1/surveillance/*    /v1/lineage/*    /v1/dispensing/*    /v1/consent/*            | |
|  +------------+---------------+------------------+----------------+---------------------+ |
|               |               |                  |                |                       |
|               v               v                  v                v                       |
|  +----------------+ +----------------+ +-----------------+ +-----------------+           |
|  |  Surveillance  | |   Lineage      | |   Dispensing    | |   Consent &     |           |
|  |  Service       | |   Service      | |   Service       | |   Identity Svc  |           |
|  |  (Lambda)      | |   (Lambda)     | |   (Lambda)      | |   (Lambda)      |           |
|  +-------+--------+ +-------+--------+ +--------+--------+ +--------+--------+           |
|          |                  |                   |                   |                     |
|          v                  v                   v                   v                     |
|  +----------------------------------------------------------------------------------+     |
|  |                         EVENT BUS (Amazon EventBridge)                           |     |
|  +----------------------------------------------------------------------------------+     |
|          |                  |                   |                   |                     |
|          v                  v                   |                   |                     |
|  +----------------+ +----------------+          |                   |                     |
|  |  SageMaker     | |  Bedrock       |          |                   |                     |
|  |  Inference     | |  (Claude 3.5)  |          |                   |                     |
|  |  Endpoint      | |                |          |                   |                     |
|  +----------------+ +----------------+          |                   |                     |
|                                                 |                   |                     |
|  +----------------------------------------------------------------------------------+     |
|  |                              DATA LAYER                                          |     |
|  |                                                                                  |     |
|  |  +------------------+  +----------------+  +----------------+  +--------------+  |     |
|  |  |  Amazon          |  |  Amazon        |  |  Amazon        |  |  Amazon S3   |  |     |
|  |  |  HealthLake      |  |  DynamoDB      |  |  ElastiCache   |  |  (Data Lake) |  |     |
|  |  |  (FHIR Store)    |  |  (Tokens,      |  |  (Redis)       |  |              |  |     |
|  |  |                  |  |   Audit Logs)  |  |                |  |              |  |     |
|  |  +------------------+  +----------------+  +----------------+  +--------------+  |     |
|  +----------------------------------------------------------------------------------+     |
|                                                                                         |
|  +----------------------------------------------------------------------------------+     |
|  |                         PRESENTATION LAYER                                       |     |
|  |                                                                                  |     |
|  |  +------------------+  +----------------+  +------------------------------------+ |     |
|  |  |  Amazon          |  |  Amazon        |  |  Secure Web Portal                | |     |
|  |  |  QuickSight      |  |  Pinpoint      |  |  (S3 + CloudFront)               | |     |
|  |  |  (Dashboards)    |  |  (SMS)         |  |                                  | |     |
|  |  +------------------+  +----------------+  +------------------------------------+ |     |
|  +----------------------------------------------------------------------------------+     |
|                                                                                         |
+-----------------------------------------------------------------------------------------+
```

---

## 4. Component Design

### 4.1 Surveillance Service

**Responsibility:** Ingest anonymised diagnosis data, compute DDBI, generate heatmaps, trigger predictive models.

#### 4.1.1 Data Ingestion Pipeline

```
  ABDM HIE --> API Gateway --> Lambda (Ingestor) --> SQS (Buffer) --> Lambda (Transformer)
                                                                            |
                                                                            v
                                                                     Amazon HealthLake
                                                                     (FHIR R4 Store)
                                                                            |
                                                                            v
                                                                     EventBridge
                                                                     ("RecordIngested")
                                                                            |
                                                          +-----------------+-----------------+
                                                          v                 v                 v
                                                   Anonymisation     Heatmap Updater    Alert Evaluator
                                                   Pipeline          (-> S3 / QS)       (-> Pinpoint)
```

**Key Design Decisions:**

| Decision | Rationale |
|----------|-----------|
| **SQS buffer between ingestor and transformer** | Decouples ingestion rate from HealthLake write throughput; provides dead-letter queue for failed records; enables replay. |
| **Anonymisation as a separate pipeline step** | Ensures that raw PHI never reaches the heatmap or analytics data stores. Anonymisation is applied before data enters the analytics S3 bucket. |
| **EventBridge for downstream fan-out** | Allows multiple consumers (heatmap, alerting, analytics) to react independently to the same ingestion event without point-to-point coupling. |

#### 4.1.2 Heatmap Generation

```
+----------------------------------------------------------------------+
|                     HEATMAP GENERATION PIPELINE                      |
|                                                                      |
|  1. Anonymised FHIR Conditions aggregated by district + ICD-10 code |
|  2. Sliding-window computation (7-day, 14-day, 30-day)              |
|  3. District Disease Burden Index (DDBI) calculation:                |
|                                                                      |
|     DDBI = w₁·(CaseVolume/Population) + w₂·GrowthRate               |
|          + w₃·SeverityMix + w₄·(1 - CapacityUtilisation)            |
|                                                                      |
|     where w₁ + w₂ + w₃ + w₄ = 1 (configurable weights)             |
|                                                                      |
|  4. Results written to S3 (Parquet) -> QuickSight SPICE dataset      |
|  5. GeoJSON layer generated for map overlay                          |
|  6. QuickSight auto-refreshes every 15 minutes via SPICE ingestion  |
+----------------------------------------------------------------------+
```

#### 4.1.3 Predictive Alerting (SageMaker)

**Model Architecture:**

```
+-----------------------------------------------------------------+
|                  PREDICTIVE FORECASTING PIPELINE                 |
|                                                                  |
|  +------------------------------------------------------------+  |
|  | TRAINING PIPELINE (Scheduled — Weekly)                     |  |
|  |                                                            |  |
|  |  Historical Data    -->  Feature Engineering  -->  Model  |  |
|  |  (S3 Parquet)            (SageMaker Processing)   Training|  |
|  |                                                   (DeepAR)|  |
|  |                                                     |      |  |
|  |                                                     v      |  |
|  |                                              Model Registry|  |
|  |                                              (SageMaker)   |  |
|  +------------------------------------------------------------+  |
|                                                                  |
|  +------------------------------------------------------------+  |
|  | INFERENCE PIPELINE (Triggered — Every 6 hours)             |  |
|  |                                                            |  |
|  |  Latest DDBI Data  -->  SageMaker Endpoint  -->  Forecast |  |
|  |  + Covariates            (Real-time Inference)    Results  |  |
|  |   · Weather data                                    |      |  |
|  |   · Seasonal patterns                               v      |  |
|  |   · Mobility indices                          Threshold    |  |
|  |                                               Evaluator    |  |
|  |                                                     |      |  |
|  |                                          +----------+----+ |  |
|  |                                          |               | |  |
|  |                                     Alert Generated  No Alert| |
|  |                                          |                  | |
|  |                                          v                  | |
|  |                                    EventBridge              | |
|  |                                    ("PredictiveAlert")      | |
|  +------------------------------------------------------------+  |
+-----------------------------------------------------------------+
```

**Feature Vector per District per Time Step:**

| Feature                        | Source                | Type        |
|-------------------------------|-----------------------|-------------|
| Case count (by ICD-10 group)  | HealthLake (anon.)    | Numeric     |
| Growth rate (7-day, 14-day)   | Computed              | Numeric     |
| Severity distribution         | HealthLake (anon.)    | Categorical |
| Population density            | Census                | Numeric     |
| Temperature, humidity          | IMD API               | Numeric     |
| Rainfall (lagged 2 weeks)     | IMD API               | Numeric     |
| Day of week, month, holiday   | Calendar              | Categorical |
| Historical outbreak indicator | IDSP archives         | Binary      |

**Alert Thresholds (Configurable):**

| Level       | Condition                                                   | Action                                        |
|-------------|-------------------------------------------------------------|-----------------------------------------------|
| **Watch**   | Forecasted 14-day case growth > 50%                         | Dashboard highlight; email to DHO              |
| **Warning** | Forecasted 14-day case growth > 100% OR DDBI > 0.7         | SMS to DHO + State HO; pharmacy advisory       |
| **Emergency**| Forecasted 14-day case growth > 200% OR DDBI > 0.9        | SMS to all stakeholders; resource deployment    |

---

### 4.2 Lineage Service

**Responsibility:** Manage family ABHA linkage, invoke Bedrock for hereditary analysis, generate preventive roadmaps.

#### 4.2.1 Family Linkage & Consent Flow

```
  +---------+     +--------------------+     +----------------------+     +-------------+
  | Citizen  |---->| Vyadiharah Portal  |---->| ABDM Consent Manager |---->| Family      |
  | (ABHA)  |     | (Link Family)      |     | (Purpose: Hereditary |     | Member's    |
  |         |     |                    |     |  Risk Analysis)      |     | ABHA Consent|
  +---------+     +--------------------+     +----------+-----------+     +------+------+
                                                        |                        |
                                                        |  Consent Granted       |
                                                        v                        v
                                               +--------------------+   +----------------+
                                               | Lineage Service    |   | HealthLake     |
                                               | (Family Graph)     |<--| (Family FHIR)  |
                                               +--------+-----------+   +----------------+
                                                        |
                                                        v
                                               +--------------------+
                                               | Amazon Bedrock     |
                                               | (Claude 3.5)       |
                                               |                    |
                                               | Prompt: Analyse    |
                                               | family history for |
                                               | hereditary risks   |
                                               +--------+-----------+
                                                        |
                                                        v
                                               +--------------------+
                                               | Risk Report        |
                                               | (Stored in         |
                                               |  HealthLake as     |
                                               |  RiskAssessment    |
                                               |  FHIR Resource)    |
                                               +--------------------+
```

#### 4.2.2 Bedrock Prompt Engineering (Claude 3.5)

**System Prompt Template — Hereditary Risk Analysis:**

```
You are a clinical decision-support AI operating within India's ABDM ecosystem.

TASK: Analyse the following FHIR-formatted family health records and identify 
hereditary / genetic predispositions for the TARGET INDIVIDUAL.

CONSTRAINTS:
- Only consider conditions with established hereditary components per 
  peer-reviewed medical literature.
- Use ICD-10 codes for all condition references.
- Score each identified risk as: LOW / MODERATE / HIGH / VERY HIGH.
- Provide a brief evidence summary citing which family members' conditions 
  contributed to the risk score.
- ALWAYS include the disclaimer: "This is an AI-generated advisory report 
  and does not constitute a clinical diagnosis."

OUTPUT FORMAT: Structured JSON conforming to FHIR RiskAssessment resource schema.

FAMILY RECORDS:
{family_fhir_bundle}

TARGET INDIVIDUAL ABHA: {target_abha_id}
```

**System Prompt Template — Preventive Roadmap:**

```
You are a preventive health advisor AI within India's ABDM ecosystem.

CONTEXT:
- An outbreak of {disease_name} (ICD-10: {icd_code}) has been detected in 
  {district_name}, {state_name}.
- The following individual has a {risk_level} hereditary risk for complications 
  related to this condition.

INDIVIDUAL RISK PROFILE:
{risk_assessment_fhir}

TASK: Generate a personalised preventive roadmap including:
1. Immediate protective measures (next 7 days)
2. Recommended screenings or vaccinations
3. Lifestyle modifications
4. Medications to discuss with their physician
5. Warning signs that require immediate medical attention

CONSTRAINTS:
- Recommendations must align with Indian national clinical guidelines 
  (NMC / ICMR standards).
- Do NOT prescribe medications; recommend discussion with a physician.
- Use clear, non-technical language suitable for SMS summary 
  (≤ 160 characters) and a detailed web version.

OUTPUT FORMAT: JSON with fields: sms_summary, detailed_roadmap, urgency_level
```

#### 4.2.3 Outbreak-Triggered Preventive Workflow

```
EventBridge ("PredictiveAlert")
        |
        v
Lambda: IdentifyVulnerablePopulation
        |
        |  Query HealthLake for individuals in affected district
        |  with HIGH / VERY HIGH risk for related conditions
        |
        v
SQS: RoadmapGenerationQueue
        |
        v
Lambda: GenerateRoadmap (Bedrock Claude 3.5)
        |
        |  Per-individual roadmap generation
        |  Concurrency: 50 (rate-limited to Bedrock throughput)
        |
        v
Lambda: DeliverRoadmap
        |
        +--> Amazon Pinpoint (SMS — summary)
        |
        +--> S3 + CloudFront (Web Portal — full roadmap)
```

---

### 4.3 Dispensing Service

**Responsibility:** Pharmacy POS integration, digital prescription retrieval, token lifecycle, fraud detection.

#### 4.3.1 Swipe-to-Validate Flow

```
  +------------------+                    +-----------------------------------+
  |  PHARMACY POS    |                    |       VYADIHARAH DISPENSING SVC   |
  |  TERMINAL        |                    |                                   |
  |                  |   1. ABHA ID       |                                   |
  |  [ABHA Card      |------------------> |  +------------------------------+ |
  |   Swipe/QR/NFC]  |                    |  | a. Validate ABHA via         | |
  |                  |                    |  |    ABDM Registry              | |
  |                  |   2. Active Rx     |  | b. Fetch active prescriptions| |
  |                  |<------------------ |  |    from HealthLake            | |
  |                  |                    |  | c. Check drug interactions    | |
  |  Pharmacist      |                    |  |    & allergy alerts           | |
  |  selects items   |   3. Dispense Req  |  | d. Generate dispensing token  | |
  |  & confirms      |------------------> |  +------------------------------+ |
  |                  |                    |                                   |
  |                  |   4. Confirmation  |  +------------------------------+ |
  |                  |<------------------ |  | e. Burn token (DynamoDB)     | |
  |                  |      + Receipt     |  | f. Write audit log           | |
  |                  |                    |  | g. Update HealthLake         | |
  |                  |                    |  |    (MedicationDispense)      | |
  |                  |                    |  +------------------------------+ |
  +------------------+                    +-----------------------------------+
```

#### 4.3.2 Token Data Model (DynamoDB)

```json
{
  "TableName": "DispensingTokens",
  "KeySchema": {
    "PartitionKey": "token_id (UUID v4)",
    "SortKey": "prescription_id"
  },
  "Attributes": {
    "token_id":           "string  — UUID v4, globally unique",
    "prescription_id":    "string  — FHIR MedicationRequest ID",
    "abha_id":            "string  — 14-digit ABHA number (encrypted)",
    "prescriber_id":      "string  — NMC registration number",
    "medication_code":    "string  — SNOMED CT / RxNorm code",
    "medication_name":    "string  — human-readable drug name",
    "dosage":             "string  — FHIR Dosage string",
    "quantity_prescribed":"number  — units prescribed",
    "quantity_dispensed": "number  — units actually dispensed",
    "status":             "string  — ACTIVE | PARTIALLY_DISPENSED | BURNED | EXPIRED | REVOKED",
    "created_at":         "string  — ISO 8601 timestamp",
    "burned_at":          "string  — ISO 8601 timestamp (null if not burned)",
    "burned_by_pharmacy": "string  — Pharmacy registration ID",
    "expiry_at":          "string  — ISO 8601 (auto-expire after Rx validity period)",
    "ttl":                "number  — DynamoDB TTL for auto-archival"
  },
  "GSI": [
    { "name": "abha-index",       "pk": "abha_id",       "sk": "created_at" },
    { "name": "prescriber-index", "pk": "prescriber_id", "sk": "created_at" },
    { "name": "pharmacy-index",   "pk": "burned_by_pharmacy", "sk": "burned_at" },
    { "name": "status-index",     "pk": "status",        "sk": "expiry_at" }
  ]
}
```

#### 4.3.3 Token State Machine

```
                   +-----------------+
                   |                 |
        Rx Created |     ACTIVE      |
        ---------->|                 |
                   +--------+--------+
                            |
              +-------------+-------------+
              |             |             |
              v             v             v
   +--------------+  +-----------+  +----------+
   |  PARTIALLY   |  |  BURNED   |  | EXPIRED  |
   |  DISPENSED   |  | (Terminal) |  |(Terminal)|
   |              |  |           |  |          |
   +------+-------+  +-----------+  +----------+
          |
          | Remaining items dispensed
          v
   +-----------+
   |  BURNED   |
   | (Terminal) |
   +-----------+

  Additional transition:
  ANY non-terminal state --> REVOKED (by prescriber or regulatory action)
```

#### 4.3.4 Fraud Detection Patterns

| Pattern                                       | Detection Method                                  | Action                                   |
|-----------------------------------------------|---------------------------------------------------|------------------------------------------|
| Same prescription redeemed at multiple pharmacies | Token status check (already BURNED)           | Reject + audit log + alert               |
| Prescriber generates > N tokens/day (anomaly) | GSI query on `prescriber-index` + statistical threshold | Alert to regulatory authority        |
| Patient fills controlled substance at > M pharmacies in 30 days | Cross-pharmacy query via `abha-index` | Flag for review                      |
| Token used after expiry                       | `expiry_at` check at validation time              | Reject + audit log                       |
| High-frequency token generation & immediate burn | Time-delta analysis (`created_at` vs `burned_at`) | Flag for investigation              |

---

### 4.4 Consent & Identity Service

**Responsibility:** ABHA authentication, ABDM consent gateway integration, consent lifecycle management.

#### 4.4.1 Consent Flow

```
+--------------------------------------------------------------------------+
|                     ABDM CONSENT FRAMEWORK INTEGRATION                   |
|                                                                          |
|  1. Vyadiharah (as HIU) sends Consent Request to ABDM Consent Manager   |
|     - Purpose: e.g., "Hereditary Risk Analysis"                         |
|     - Data scope: Conditions, FamilyMemberHistory, Observations         |
|     - Date range:  e.g., last 10 years                                  |
|     - Expiry: e.g., 90 days                                             |
|                                                                          |
|  2. ABDM Consent Manager notifies the citizen (via ABHA app / SMS)      |
|                                                                          |
|  3. Citizen GRANTS or DENIES the consent request                         |
|                                                                          |
|  4. If GRANTED:                                                          |
|     a. ABDM returns a signed Consent Artefact (JWT)                     |
|     b. Vyadiharah uses the artefact to fetch data from HIP via ABDM HIE|
|     c. Data is processed within the scope & expiry of the consent        |
|     d. Upon expiry, all non-anonymised data derived under this consent   |
|        is purged                                                         |
|                                                                          |
|  5. Citizen can REVOKE consent at any time via ABHA app                  |
|     -> Vyadiharah receives revocation callback                            |
|     -> Triggers immediate data purge for that consent scope               |
|                                                                          |
+--------------------------------------------------------------------------+
```

#### 4.4.2 Identity Verification for Pharmacy

| Method          | Implementation                                  | Fallback                          |
|-----------------|--------------------------------------------------|-----------------------------------|
| **ABHA Card Swipe** | Magnetic stripe / chip read -> ABHA ID extracted | Manual ABHA number entry          |
| **QR Code Scan**    | ABHA QR from citizen's app / printed card       | Manual entry                      |
| **NFC Tap**         | NFC-enabled ABHA card (future)                  | QR / manual entry                 |
| **Biometric**       | Aadhaar-linked fingerprint (via ABHA e-KYC)     | OTP-based authentication          |

---

## 5. Data Architecture

### 5.1 FHIR Resource Model

```
+-----------------------------------------------------------------------------+
|                        HEALTHLAKE FHIR RESOURCE MAP                         |
|                                                                             |
|  +--------------+      +-----------------+      +-------------------+      |
|  |   Patient     |-----| FamilyMember     |      | Practitioner      |      |
|  |   (ABHA ID)  |      | History          |      | (NMC Reg. No.)   |      |
|  +------+-------+      +-----------------+      +--------+----------+      |
|         |                                                 |                 |
|    +----+--------------+--------------+-------------------+|                 |
|    |    |              |              |                   ||                 |
|    v    v              v              v                   vv                 |
|  +----------+  +------------+  +-----------+  +------------------+         |
|  |Condition |  |Observation |  |Allergy     |  |MedicationRequest |         |
|  |(ICD-10)  |  |(LOINC)     |  |Intolerance |  |(Prescription)    |         |
|  +----------+  +------------+  +-----------+  +--------+---------+         |
|                                                         |                   |
|                                                         v                   |
|                                                +------------------+         |
|                                                |MedicationDispense|         |
|                                                |(Pharmacy Event)  |         |
|                                                +------------------+         |
|                                                                             |
|  +-------------------+      +------------------------+                      |
|  |  RiskAssessment   |      | CommunicationRequest   |                      |
|  |  (AI-generated    |      | (Pinpoint SMS/Alert)   |                      |
|  |   hereditary risk)|      |                        |                      |
|  +-------------------+      +------------------------+                      |
|                                                                             |
|  +-------------------+                                                      |
|  |  CarePlan          |                                                      |
|  |  (Preventive       |                                                      |
|  |   Roadmap)         |                                                      |
|  +-------------------+                                                      |
+-----------------------------------------------------------------------------+
```

### 5.2 Data Flow Summary

| Flow                          | Source              | Destination         | Format       | Transport             | Frequency       |
|-------------------------------|---------------------|---------------------|--------------|-----------------------|-----------------|
| Diagnosis ingestion           | HIP (via ABDM)     | HealthLake          | FHIR Bundle  | HTTPS (ABDM HIE)      | Near real-time  |
| Anonymised aggregation        | HealthLake          | S3 (analytics)      | Parquet      | Lambda                | Every 15 min    |
| Heatmap rendering data         | S3                  | QuickSight SPICE    | Parquet      | SPICE ingestion       | Every 15 min    |
| Predictive model training     | S3 (historical)     | SageMaker           | CSV/Parquet  | SageMaker Training Job| Weekly          |
| Predictive inference          | S3 (latest + covariates) | SageMaker Endpoint | JSON    | HTTPS                 | Every 6 hours   |
| Hereditary risk analysis      | HealthLake (family) | Bedrock             | FHIR JSON    | Bedrock API           | On-demand       |
| Preventive roadmap gen.       | Risk profiles       | Bedrock             | JSON         | Bedrock API           | Event-triggered |
| SMS delivery                  | Lambda              | Pinpoint            | JSON         | Pinpoint API          | Event-triggered |
| Prescription retrieval        | Pharmacy POS        | HealthLake          | FHIR R4      | API Gateway + Lambda  | On-demand       |
| Token lifecycle events        | Dispensing Service  | DynamoDB            | JSON         | DynamoDB SDK          | Real-time       |
| Audit log entries             | All services        | DynamoDB + CloudWatch | JSON       | SDK                   | Real-time       |

---

## 6. Security Architecture

### 6.1 Network Topology

```
+-----------------------------------------------------------------------------+
|                           AWS VPC (10.0.0.0/16)                             |
|                           ap-south-1 (Mumbai)                               |
|                                                                             |
|  +----------------------------------------------------------------------+   |
|  |  PUBLIC SUBNET (10.0.1.0/24 & 10.0.2.0/24 — Multi-AZ)              |   |
|  |                                                                      |   |
|  |  +--------------+  +---------------+  +------------------------+    |   |
|  |  | NAT Gateway  |  | ALB           |  | CloudFront             |    |   |
|  |  | (outbound)   |  | (API routing) |  | (Web Portal CDN)       |    |   |
|  |  +--------------+  +---------------+  +------------------------+    |   |
|  +----------------------------------------------------------------------+   |
|                                                                             |
|  +----------------------------------------------------------------------+   |
|  |  PRIVATE SUBNET (10.0.10.0/24 & 10.0.11.0/24 — Multi-AZ)           |   |
|  |                                                                      |   |
|  |  +--------------+  +---------------+  +------------------------+    |   |
|  |  | Lambda       |  | ElastiCache   |  | SageMaker              |    |   |
|  |  | Functions    |  | (Redis)       |  | Endpoints              |    |   |
|  |  +--------------+  +---------------+  +------------------------+    |   |
|  +----------------------------------------------------------------------+   |
|                                                                             |
|  +----------------------------------------------------------------------+   |
|  |  ISOLATED SUBNET (10.0.20.0/24 & 10.0.21.0/24 — Multi-AZ)          |   |
|  |                                                                      |   |
|  |  +--------------+  +---------------+                                |   |
|  |  | HealthLake   |  | DynamoDB      |  (VPC Endpoints only)          |   |
|  |  | (FHIR Store) |  | (Tokens/Audit)|                                |   |
|  |  +--------------+  +---------------+                                |   |
|  +----------------------------------------------------------------------+   |
|                                                                             |
|  +----------------------------------------------------------------------+   |
|  |  VPC ENDPOINTS (PrivateLink)                                         |   |
|  |  · S3 Gateway Endpoint                                               |   |
|  |  · DynamoDB Gateway Endpoint                                         |   |
|  |  · HealthLake Interface Endpoint                                     |   |
|  |  · SageMaker Interface Endpoint                                      |   |
|  |  · Bedrock Interface Endpoint                                        |   |
|  |  · KMS Interface Endpoint                                            |   |
|  +----------------------------------------------------------------------+   |
|                                                                             |
|  +----------------------------------------------------------------------+   |
|  |  WAF v2 (attached to ALB & CloudFront)                               |   |
|  |  · Rate limiting (per-IP & per-ABHA)                                 |   |
|  |  · SQL injection & XSS protection                                    |   |
|  |  · Geo-restriction (India only for PHI endpoints)                    |   |
|  |  · Bot mitigation                                                    |   |
|  +----------------------------------------------------------------------+   |
|                                                                             |
+-----------------------------------------------------------------------------+
```

### 6.2 Encryption Strategy

| Layer            | Mechanism                                                    |
|------------------|--------------------------------------------------------------|
| **In Transit**   | TLS 1.3 for all external connections; mutual TLS for ABDM Gateway communication. |
| **At Rest**      | AES-256 via AWS KMS. Separate CMKs for: HealthLake, DynamoDB, S3, ElastiCache. |
| **Application**  | Field-level encryption for ABHA IDs stored in DynamoDB (client-side encryption using KMS). |
| **Key Rotation** | Automatic annual rotation for all KMS CMKs.                  |

### 6.3 IAM & RBAC Design

| Role                        | Permissions                                                                | MFA Required |
|-----------------------------|-----------------------------------------------------------------------------|-------------|
| **SystemAdmin**             | Full infrastructure access; no direct PHI access.                          | Yes         |
| **DataEngineer**            | S3 (analytics bucket), SageMaker, Glue; no HealthLake direct access.      | Yes         |
| **Epidemiologist**          | QuickSight dashboards (read-only); anonymised data only.                   | Yes         |
| **DistrictHealthOfficer**   | QuickSight dashboards (district-scoped); alert management.                 | Yes         |
| **ClinicalReviewer**        | HealthLake (read); RiskAssessment (read/write); annotation capabilities.   | Yes         |
| **PharmacyPOS**             | Dispensing API (invoke); scoped to own pharmacy ID.                        | No*         |
| **AIServiceRole**           | Bedrock (invoke); HealthLake (read, consent-gated); S3 (read/write).      | N/A (service)|

*_Pharmacy POS uses API key + device certificate; biometric/OTP used for pharmacist login._

### 6.4 Audit Logging

```
+-----------------------------------------------------------------------+
|                         AUDIT LOG ARCHITECTURE                        |
|                                                                       |
|  Every API call, data access, and state change produces an audit      |
|  event with the following structure:                                   |
|                                                                       |
|  {                                                                    |
|    "event_id":        "uuid",                                         |
|    "timestamp":       "ISO 8601",                                     |
|    "actor":           "IAM principal / ABHA ID / system",             |
|    "action":          "READ | WRITE | DELETE | INVOKE | AUTH",        |
|    "resource_type":   "Patient | Token | RiskAssessment | ...",       |
|    "resource_id":     "FHIR resource ID / token ID",                  |
|    "consent_id":      "ABDM consent artefact ID (if applicable)",     |
|    "source_ip":       "IP address",                                   |
|    "user_agent":      "client identifier",                            |
|    "outcome":         "SUCCESS | FAILURE | DENIED",                   |
|    "detail":          "human-readable description"                    |
|  }                                                                    |
|                                                                       |
|  Storage:                                                             |
|    · Real-time: DynamoDB (AuditLogs table) — 90 days hot             |
|    · Archive:   S3 (audit-archive bucket) — 7 years cold             |
|    · Monitoring: CloudWatch Logs (real-time streaming for alerts)     |
|    · Integrity:  S3 Object Lock (WORM) for tamper evidence           |
|                                                                       |
+-----------------------------------------------------------------------+
```

---

## 7. Infrastructure & Deployment

### 7.1 Infrastructure as Code

| Aspect              | Technology                                    |
|----------------------|-----------------------------------------------|
| **IaC Framework**    | AWS CDK (TypeScript)                          |
| **Environment Mgmt** | CDK Stacks per environment (dev / staging / prod) |
| **CI/CD**            | AWS CodePipeline + CodeBuild                  |
| **Artifact Registry**| Amazon ECR (Lambda container images)          |
| **Config Mgmt**      | AWS Systems Manager Parameter Store + Secrets Manager |

### 7.2 Deployment Pipeline

```
+----------+     +----------+     +--------------+     +--------------+     +---------+
|  GitHub   |---->| CodeBuild|---->|  CDK Deploy  |---->|  Integration |---->|  PROD   |
|  (main)   |     |  (Build  |     |  (Staging)   |     |  Tests       |     | Deploy  |
|           |     |  + Test) |     |              |     |  (Staging)   |     |         |
+----------+     +----------+     +--------------+     +--------------+     +---------+
                                                                              |
                                                              Manual Approval ^
                                                              (Production)    |
                                                                              |
                                                        +---------------------+
                                                        | Canary Deployment
                                                        | (10% -> 50% -> 100%)
                                                        +---------------------
```

### 7.3 Environments

| Environment  | Purpose                              | Data                        | Scale        |
|--------------|---------------------------------------|-----------------------------|--------------|
| **dev**      | Developer sandbox                    | Synthetic FHIR data         | Minimal      |
| **staging**  | Pre-production validation            | Anonymised production subset| 10% of prod  |
| **prod**     | Live production                      | Real data (consent-gated)   | Full scale   |
| **dr**       | Disaster recovery (passive)          | Replicated from prod        | Warm standby |

---

## 8. Monitoring & Observability

### 8.1 Monitoring Stack

```
+------------------------------------------------------------------+
|                     OBSERVABILITY ARCHITECTURE                    |
|                                                                  |
|  +------------------------------------------------------------+  |
|  |  METRICS (CloudWatch Metrics)                              |  |
|  |  · Lambda: invocations, errors, duration, throttles       |  |
|  |  · API Gateway: 4xx/5xx rates, latency P50/P95/P99        |  |
|  |  · HealthLake: read/write latency, throttled requests     |  |
|  |  · DynamoDB: RCU/WCU consumption, throttled requests      |  |
|  |  · SageMaker: inference latency, model errors             |  |
|  |  · Pinpoint: delivery rate, bounce rate                   |  |
|  +------------------------------------------------------------+  |
|                                                                  |
|  +------------------------------------------------------------+  |
|  |  LOGS (CloudWatch Logs)                                    |  |
|  |  · Structured JSON logs from all Lambda functions          |  |
|  |  · API Gateway access logs                                 |  |
|  |  · Audit event stream                                      |  |
|  |  · Retention: 90 days hot, 7 years archived to S3          |  |
|  +------------------------------------------------------------+  |
|                                                                  |
|  +------------------------------------------------------------+  |
|  |  TRACES (AWS X-Ray)                                        |  |
|  |  · End-to-end request tracing across Lambda, API GW,       |  |
|  |    HealthLake, DynamoDB, Bedrock, SageMaker                |  |
|  |  · Sampling rate: 5% (normal), 100% (on-error)            |  |
|  +------------------------------------------------------------+  |
|                                                                  |
|  +------------------------------------------------------------+  |
|  |  ALARMS (CloudWatch Alarms -> SNS -> PagerDuty)             |  |
|  |  · P1: Dispensing API 5xx rate > 1% for 5 min              |  |
|  |  · P1: HealthLake write failures > 0 for 10 min            |  |
|  |  · P2: Heatmap staleness > 30 min                          |  |
|  |  · P2: Bedrock invocation error rate > 5%                  |  |
|  |  · P3: SageMaker inference latency P99 > 60s               |  |
|  +------------------------------------------------------------+  |
|                                                                  |
|  +------------------------------------------------------------+  |
|  |  DASHBOARDS                                                |  |
|  |  · Ops Dashboard (CloudWatch) — system health overview     |  |
|  |  · Epidemiology Dashboard (QuickSight) — disease intel     |  |
|  |  · Dispensing Dashboard (QuickSight) — pharmacy metrics    |  |
|  |  · AI Performance Dashboard — model accuracy over time     |  |
|  +------------------------------------------------------------+  |
+------------------------------------------------------------------+
```

### 8.2 Key SLIs / SLOs

| Service Level Indicator (SLI)                   | Service Level Objective (SLO) | Measurement Window |
|-------------------------------------------------|-------------------------------|--------------------|
| Dispensing API availability                     | ≥ 99.99%                      | Monthly            |
| Dispensing API latency (P99)                    | ≤ 3 seconds                   | Rolling 7 days     |
| Heatmap data freshness                          | ≤ 15 minutes                  | Continuous         |
| Predictive alert delivery latency               | ≤ 5 minutes from trigger      | Per event          |
| SMS delivery success rate                       | ≥ 95%                         | Daily              |
| HealthLake ingestion success rate               | ≥ 99.9%                       | Daily              |
| Bedrock invocation success rate                 | ≥ 99%                         | Daily              |

---

## 9. Disaster Recovery & Business Continuity

### 9.1 DR Strategy

| Component        | Strategy                        | RPO      | RTO      |
|------------------|---------------------------------|----------|----------|
| HealthLake       | Cross-region replication to ap-south-2 (Hyderabad) | 1 hour  | 4 hours |
| DynamoDB         | Global Tables (ap-south-1 <-> ap-south-2) | Near 0  | < 1 min |
| S3               | Cross-region replication        | 15 min   | < 1 hour |
| Lambda / API GW  | Multi-region deployment (active-passive) | N/A    | 30 min  |
| SageMaker        | Model artefacts replicated to S3 in DR region | 6 hours | 2 hours |
| QuickSight       | Recreate from replicated SPICE data | 1 hour  | 2 hours |
| Pinpoint         | Multi-region (native)           | N/A      | N/A      |

### 9.2 Backup Schedule

| Data Store   | Backup Type          | Frequency  | Retention |
|--------------|----------------------|------------|-----------|
| HealthLake   | FHIR export to S3    | Daily      | 90 days   |
| DynamoDB     | Point-in-time recovery (PITR) | Continuous | 35 days  |
| S3           | Versioning + lifecycle | Continuous | Per policy |
| Secrets Mgr  | Automatic rotation    | 90 days    | N/A       |

---

## 10. Cost Estimation Framework

### 10.1 Primary Cost Drivers

| Service         | Cost Driver                          | Estimated Scale (Pilot: 10 Districts) |
|-----------------|--------------------------------------|---------------------------------------|
| HealthLake      | FHIR resource storage + R/W requests | ~5M resources, ~100K req/day          |
| Bedrock (Claude)| Input/output tokens                  | ~50K reports/month                    |
| SageMaker       | Training hours + inference endpoint  | 1 ml.m5.xlarge (24/7)                |
| QuickSight      | Reader sessions                      | ~500 users                            |
| Pinpoint         | SMS messages                        | ~2M SMS/month                         |
| DynamoDB        | RCU/WCU + storage                    | ~1M tokens/month                      |
| Lambda          | Invocations + duration               | ~10M invocations/month                |
| S3              | Storage + requests                   | ~500 GB/month growth                  |
| CloudWatch      | Logs ingestion + storage             | ~100 GB/month                         |

### 10.2 Cost Optimisation Strategies

1. **HealthLake** — Use FHIR search indices judiciously; archive old records to S3 Glacier.
2. **Bedrock** — Cache common risk-profile patterns; batch non-urgent roadmap generation.
3. **SageMaker** — Use Serverless Inference for bursty workloads; Spot Instances for training.
4. **Lambda** — Right-size memory allocations using AWS Lambda Power Tuning.
5. **DynamoDB** — On-demand capacity for unpredictable workloads; TTL for auto-archival.
6. **Pinpoint** — Consolidate messages; avoid redundant alerts via frequency capping.

---

## 11. Technology Stack Summary

```
+-----------------------------------------------------------------------------+
|                         VYADIHARAH TECHNOLOGY STACK                          |
|                                                                             |
|  +---------------------------------------------------------------------+    |
|  |                        PRESENTATION LAYER                           |    |
|  |  Amazon QuickSight | Amazon Pinpoint (SMS) | CloudFront + S3 Portal |    |
|  +---------------------------------------------------------------------+    |
|                                                                             |
|  +---------------------------------------------------------------------+    |
|  |                        APPLICATION LAYER                            |    |
|  |  API Gateway (REST) | AWS Lambda (Python 3.12) | EventBridge | SQS  |    |
|  +---------------------------------------------------------------------+    |
|                                                                             |
|  +---------------------------------------------------------------------+    |
|  |                        AI / ML LAYER                                |    |
|  |  Amazon Bedrock (Claude 3.5) | Amazon SageMaker (DeepAR / Prophet) |    |
|  +---------------------------------------------------------------------+    |
|                                                                             |
|  +---------------------------------------------------------------------+    |
|  |                        DATA LAYER                                   |    |
|  |  Amazon HealthLake (FHIR R4) | DynamoDB | ElastiCache | S3 (Lake)  |    |
|  +---------------------------------------------------------------------+    |
|                                                                             |
|  +---------------------------------------------------------------------+    |
|  |                        SECURITY & GOVERNANCE                        |    |
|  |  IAM | KMS | WAF v2 | Secrets Manager | CloudTrail | GuardDuty     |    |
|  +---------------------------------------------------------------------+    |
|                                                                             |
|  +---------------------------------------------------------------------+    |
|  |                        OPERATIONS                                   |    |
|  |  CloudWatch | X-Ray | CDK (IaC) | CodePipeline | CodeBuild         |    |
|  +---------------------------------------------------------------------+    |
|                                                                             |
|  +---------------------------------------------------------------------+    |
|  |                        EXTERNAL INTEGRATIONS                        |    |
|  |  ABDM Gateway | ABHA Registry | ABDM Consent Manager | ABDM HIE    |    |
|  +---------------------------------------------------------------------+    |
|                                                                             |
+-----------------------------------------------------------------------------+
```

---

## 12. References

| #  | Reference                                                                                    |
|----|----------------------------------------------------------------------------------------------|
| 1  | [ABDM Architecture Document](https://abdm.gov.in/publications)                              |
| 2  | [HL7 FHIR R4 Specification](https://hl7.org/fhir/R4/)                                       |
| 3  | [NHA FHIR India Implementation Guide](https://nrces.in/ndhm/fhir/r4/index.html)             |
| 4  | [AWS HealthLake Documentation](https://docs.aws.amazon.com/healthlake/)                      |
| 5  | [Amazon Bedrock — Claude Model Card](https://docs.aws.amazon.com/bedrock/)                   |
| 6  | [Amazon SageMaker DeepAR](https://docs.aws.amazon.com/sagemaker/latest/dg/deepar.html)      |
| 7  | [Digital Personal Data Protection Act 2023](https://www.meity.gov.in/dpdpa)                  |
| 8  | [CERT-In Directions 2022](https://www.cert-in.org.in/)                                       |
| 9  | `requirements.md` — Vyadiharah Requirements Specification v1.0.0                             |

---
## Important Note & Disclaimer

### Prototype Status:
- Vyadiharah is currently a prototype model and conceptual demonstration for the AI for Bharat Hackathon 2026. It is designed as an intelligence layer to sit on top of the existing ABHA (Ayushman Bharat Health Account) infrastructure and is not an official government-affiliated service.

### Data Integrity:
- In compliance with hackathon safety guidelines, all data utilized for this prototype is entirely synthetic or derived from publicly available datasets. No real-world Protected Health Information (PHI) or private government databases have been accessed or stored.

### Liability:
- This system is for informational and research purposes only. The AI-generated outputs, including disease risk assessments and preventive roadmaps, should not be construed as clinical diagnoses or professional medical advice.

---

*End of System Design Document — Vyadiharah v1.0*
