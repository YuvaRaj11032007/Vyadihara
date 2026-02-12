# Vyadiharah — Requirements Specification

> **An AI-Driven Epidemiological Intelligence & Predictive Surveillance Layer for the ABDM Ecosystem**

---

## 1. Document Control

| Field               | Value                                                                 |
|----------------------|-----------------------------------------------------------------------|
| **Project Name**     | Vyadiharah                                                            |
| **Version**          | 1.0.0                                                                 |
| **Status**           | Draft                                                                 |
| **Date**             | 2026-02-12                                                            |
| **Classification**   | Confidential                                                          |
| **Domain**           | Public Health Informatics · Epidemiological Surveillance · ABDM       |

---

## 2. Executive Summary

**Vyadiharah** (Sanskrit: *व्याधिहरः* — "that which removes disease") is a cloud-native, AI-driven epidemiological intelligence platform designed to operate as a **predictive surveillance layer** atop India's **Ayushman Bharat Digital Mission (ABDM)** ecosystem. It leverages ABHA (Ayushman Bharat Health Account) identities, FHIR-formatted health records, and advanced machine-learning models to deliver:

- **Real-time district-level disease intelligence** and live outbreak mapping.
- **AI-driven hereditary risk modelling** and preventive care roadmaps.
- **Smart dispensing and pharmacy integration** with prescription fraud prevention.

The platform is built entirely on **Amazon Web Services (AWS)**, utilising HealthLake, Bedrock, SageMaker, QuickSight, and Pinpoint as core managed services.

---

## 3. Glossary & Acronyms

| Term / Acronym | Definition |
|----------------|------------|
| **ABDM**       | Ayushman Bharat Digital Mission — India's national digital health ecosystem. |
| **ABHA**       | Ayushman Bharat Health Account — a 14-digit unique health identifier for every Indian citizen. |
| **FHIR**       | Fast Healthcare Interoperability Resources (HL7 FHIR R4) — the international standard for exchanging healthcare data electronically. |
| **PHI**        | Protected Health Information. |
| **DISHA**      | Digital Information Security in Healthcare Act (proposed Indian legislation). |
| **NHA**        | National Health Authority — the apex body governing ABDM. |
| **HRP**        | Health Record Provider — any registered entity that creates or stores health records within ABDM. |
| **HIU / HIP**  | Health Information User / Health Information Provider — ABDM consent-framework roles. |
| **ICD-10**     | International Classification of Diseases, 10th Revision. |
| **SNOMED CT**  | Systematized Nomenclature of Medicine — Clinical Terms. |
| **LOINC**      | Logical Observation Identifiers Names and Codes — standard for lab & clinical observations. |
| **RBAC**       | Role-Based Access Control. |
| **PII**        | Personally Identifiable Information. |

---

## 4. Stakeholders

| # | Stakeholder                          | Role in System                                                                                     |
|---|--------------------------------------|----------------------------------------------------------------------------------------------------|
| 1 | **National Health Authority (NHA)**  | Governing body; provides ABDM APIs, ABHA registry, policy directives.                             |
| 2 | **State & District Health Officers** | Primary dashboard consumers; receive predictive alerts and outbreak intelligence.                  |
| 3 | **Epidemiologists & Public Health Researchers** | Analyse aggregated surveillance data; validate AI-generated predictions.                  |
| 4 | **Registered Pharmacies**            | Use the swipe-to-validate dispensing system; receive stockpile advisories.                         |
| 5 | **Clinics & Hospitals (HIP/HRP)**    | Source of anonymised diagnosis data; receive preventive-care recommendations for patients.         |
| 6 | **Citizens (ABHA Holders)**          | End-beneficiaries; receive SMS alerts, preventive roadmaps, and medication adherence reminders.    |
| 7 | **System Administrators**            | Manage infrastructure, IAM, deployments, and monitoring.                                           |
| 8 | **Data Protection Officer (DPO)**    | Ensures compliance with DPDPA 2023, DISHA, and ABDM consent framework.                            |

---

## 5. Business Objectives

| ID     | Objective                                                                                                  | Success Metric                                                              |
|--------|------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------|
| BO-01  | Reduce the average time from outbreak onset to government response.                                        | ≤ 48 hours from first-signal detection to district-level alert issuance.    |
| BO-02  | Enable predictive resource deployment for pharmacies and hospitals ahead of outbreak peaks.                 | ≥ 70% forecast accuracy at a 14-day horizon for tracked pathogens.          |
| BO-03  | Identify hereditary health risks for citizens through family ABHA linkage.                                 | ≥ 80% of linked families receive a personalised risk profile within 30 days.|
| BO-04  | Eliminate prescription fraud and handwriting-based dispensing errors at pharmacies.                         | Zero duplicate-token redemptions; < 0.1% dispensing-mismatch rate.          |
| BO-05  | Improve medication adherence through proactive citizen communication.                                      | ≥ 20% improvement in refill compliance rates within pilot districts.        |
| BO-06  | Achieve full interoperability with ABDM standards (FHIR R4, ABHA, consent framework).                     | 100% conformance with NHA integration test suites.                          |

---

## 6. Functional Requirements

### 6.1 Use Case 1 — Real-Time District Disease Intelligence

#### 6.1.1 Live Outbreak Mapping

| ID       | Requirement                                                                                                                                       | Priority   |
|----------|---------------------------------------------------------------------------------------------------------------------------------------------------|------------|
| FR-1.1   | The system SHALL ingest anonymised diagnosis tags (ICD-10 / SNOMED CT) from registered clinics and hospitals via ABDM Health Information Exchange. | **Must**   |
| FR-1.2   | The system SHALL store all ingested records in **Amazon HealthLake** in FHIR R4 format, ensuring full interoperability.                           | **Must**   |
| FR-1.3   | The system SHALL generate a real-time **Health Heatmap of India** at district granularity, updated at least every **15 minutes**.                 | **Must**   |
| FR-1.4   | The heatmap SHALL support filtering by disease category, date range, severity (mild / moderate / severe / critical), and age cohort.              | **Must**   |
| FR-1.5   | The system SHALL display the heatmap on interactive **Amazon QuickSight** dashboards accessible to authorised health officers.                    | **Must**   |
| FR-1.6   | The system SHALL support drill-down from national → state → district → facility levels.                                                           | **Should** |
| FR-1.7   | The system SHALL compute a **District Disease Burden Index (DDBI)** — a composite score reflecting case volume, growth rate, severity mix, and healthcare-capacity utilisation. | **Should** |
| FR-1.8   | The system SHALL allow export of heatmap data in CSV, GeoJSON, and PDF report formats.                                                            | **Could**  |

#### 6.1.2 Predictive Alerting

| ID       | Requirement                                                                                                                                       | Priority   |
|----------|---------------------------------------------------------------------------------------------------------------------------------------------------|------------|
| FR-1.9   | The system SHALL use **Amazon SageMaker** to train and serve time-series forecasting models (e.g., DeepAR, Prophet) on historical outbreak data.  | **Must**   |
| FR-1.10  | The system SHALL generate **predictive alerts** when a forecasted metric (case count, growth rate) exceeds configurable thresholds.                | **Must**   |
| FR-1.11  | Predictive alerts SHALL be routed to relevant District Health Officers and state-level authorities via dashboard notifications and **Amazon Pinpoint** SMS. | **Must** |
| FR-1.12  | The system SHALL provide resource-deployment recommendations (e.g., "Stockpile 10,000 units of Oseltamivir in District X within 7 days").          | **Should** |
| FR-1.13  | The system SHALL allow epidemiologists to adjust model parameters (look-back window, seasonality, confidence interval) through a configuration UI. | **Should** |
| FR-1.14  | The system SHALL log all predictions with timestamps and actuals for retrospective accuracy analysis and model improvement.                        | **Must**   |

---

### 6.2 Use Case 2 — AI-Driven Lineage & Preventive Care

#### 6.2.1 Hereditary Risk Modelling

| ID       | Requirement                                                                                                                                       | Priority   |
|----------|---------------------------------------------------------------------------------------------------------------------------------------------------|------------|
| FR-2.1   | The system SHALL securely link family members' ABHA profiles with **explicit, granular consent** obtained via the ABDM consent framework.          | **Must**   |
| FR-2.2   | The system SHALL use **Amazon Bedrock (Claude 3.5)** to analyse linked family health records and identify hereditary/genetic predispositions.      | **Must**   |
| FR-2.3   | The AI engine SHALL produce a **Hereditary Risk Report** per individual, covering conditions such as diabetes mellitus (Type 2), cardiovascular disease, certain cancers, thalassemia, sickle cell disease, and other conditions prevalent in Indian demographics. | **Must** |
| FR-2.4   | Risk scores SHALL be graded on a standardised scale (e.g., Low / Moderate / High / Very High) with supporting evidence citations from family records. | **Must** |
| FR-2.5   | The system SHALL allow clinicians to review, annotate, and override AI-generated risk assessments before they are communicated to the patient.      | **Must**   |
| FR-2.6   | All AI-generated reports SHALL include a disclaimer that they are advisory and do not constitute a clinical diagnosis.                              | **Must**   |

#### 6.2.2 Preventive Roadmap Generation

| ID       | Requirement                                                                                                                                       | Priority   |
|----------|---------------------------------------------------------------------------------------------------------------------------------------------------|------------|
| FR-2.7   | During a detected local outbreak (e.g., flu spike), the system SHALL automatically identify high-vulnerability individuals based on hereditary risk profiles. | **Must** |
| FR-2.8   | The system SHALL generate personalised **Preventive Roadmaps** containing recommended vaccinations, screening tests, lifestyle modifications, and medication adjustments. | **Must** |
| FR-2.9   | Preventive roadmaps SHALL be delivered to citizens via **Amazon Pinpoint** SMS alerts with links to a secure web portal for full details.           | **Must**   |
| FR-2.10  | High-priority warnings SHALL be sent within **1 hour** of outbreak threshold being crossed for individuals in the affected district.                | **Should** |
| FR-2.11  | The system SHALL support multi-language SMS delivery (Hindi, English, and at least 5 regional languages).                                           | **Should** |
| FR-2.12  | The system SHALL track acknowledgement/read status of preventive roadmap messages and escalate unacknowledged high-priority alerts.                 | **Could**  |

---

### 6.3 Use Case 3 — Smart Dispensing & Pharmacy Integration

#### 6.3.1 Swipe-to-Validate System

| ID       | Requirement                                                                                                                                       | Priority   |
|----------|---------------------------------------------------------------------------------------------------------------------------------------------------|------------|
| FR-3.1   | The system SHALL support ABHA card validation at pharmacy point-of-sale terminals via a secure API (card swipe, QR scan, or NFC tap).              | **Must**   |
| FR-3.2   | Upon card validation, the system SHALL retrieve and display the patient's **active digital prescriptions** from HealthLake.                        | **Must**   |
| FR-3.3   | The displayed prescription SHALL include: prescribing physician name & registration number, medication name (generic + brand), dosage, frequency, duration, quantity, and special instructions. | **Must** |
| FR-3.4   | The pharmacist SHALL confirm dispensing by selecting items from the digital prescription; partial dispensing SHALL be supported.                    | **Must**   |

#### 6.3.2 Error Reduction

| ID       | Requirement                                                                                                                                       | Priority   |
|----------|---------------------------------------------------------------------------------------------------------------------------------------------------|------------|
| FR-3.5   | The system SHALL display prescriptions in a clear, standardised digital format, completely eliminating reliance on handwritten prescriptions.       | **Must**   |
| FR-3.6   | The system SHALL perform **drug-drug interaction checks** and **allergy cross-referencing** against the patient's HealthLake record at the point of dispensing. | **Should** |
| FR-3.7   | The system SHALL flag dosage anomalies (e.g., paediatric patient prescribed an adult dose) and require pharmacist acknowledgement before proceeding. | **Should** |

#### 6.3.3 Illegal Purchase Prevention

| ID       | Requirement                                                                                                                                       | Priority   |
|----------|---------------------------------------------------------------------------------------------------------------------------------------------------|------------|
| FR-3.8   | Each digital prescription SHALL be tokenised with a unique, cryptographic, single-use **Dispensing Token**.                                        | **Must**   |
| FR-3.9   | Upon successful dispensing, the token SHALL be **burned** (irreversibly invalidated) in real time, preventing reuse.                                | **Must**   |
| FR-3.10  | The system SHALL reject any attempt to dispense against an already-burned token with a clear error message and audit log entry.                     | **Must**   |
| FR-3.11  | The system SHALL maintain a tamper-evident **Dispensing Ledger** recording every dispensing event (token ID, pharmacy ID, timestamp, items dispensed). | **Must** |
| FR-3.12  | The system SHALL generate alerts to regulatory authorities when patterns indicative of prescription fraud are detected (e.g., high-frequency token generation from a single prescriber). | **Should** |

---

## 7. Non-Functional Requirements

### 7.1 Performance

| ID       | Requirement                                                                                   | Target                          |
|----------|-----------------------------------------------------------------------------------------------|---------------------------------|
| NFR-01   | Heatmap data refresh latency                                                                  | ≤ 15 minutes end-to-end        |
| NFR-02   | ABHA card validation and prescription retrieval at pharmacy POS                               | ≤ 3 seconds (P99)              |
| NFR-03   | Token burn confirmation                                                                       | ≤ 1 second (P99)               |
| NFR-04   | Predictive model inference (per-district forecast)                                            | ≤ 30 seconds                   |
| NFR-05   | Preventive roadmap generation (per individual, via Bedrock)                                   | ≤ 10 seconds                   |
| NFR-06   | SMS delivery (Pinpoint to telecom handoff)                                                    | ≤ 5 seconds (P95)              |

### 7.2 Scalability

| ID       | Requirement                                                                                   | Target                          |
|----------|-----------------------------------------------------------------------------------------------|---------------------------------|
| NFR-07   | Peak concurrent FHIR record ingestion rate                                                    | ≥ 50,000 records/minute        |
| NFR-08   | Total ABHA profiles supported                                                                 | ≥ 500 million                  |
| NFR-09   | Concurrent pharmacy POS sessions                                                              | ≥ 100,000                      |
| NFR-10   | QuickSight concurrent dashboard viewers                                                       | ≥ 10,000                       |

### 7.3 Availability & Reliability

| ID       | Requirement                                                                                   | Target                          |
|----------|-----------------------------------------------------------------------------------------------|---------------------------------|
| NFR-11   | Overall system availability                                                                   | ≥ 99.95% (monthly)             |
| NFR-12   | Pharmacy dispensing subsystem availability                                                    | ≥ 99.99% (critical path)       |
| NFR-13   | Recovery Point Objective (RPO) for HealthLake data                                            | ≤ 1 hour                       |
| NFR-14   | Recovery Time Objective (RTO) for critical subsystems                                         | ≤ 4 hours                      |

### 7.4 Security & Compliance

| ID       | Requirement                                                                                                                                       | Priority   |
|----------|---------------------------------------------------------------------------------------------------------------------------------------------------|------------|
| NFR-15   | All data in transit SHALL be encrypted using TLS 1.3.                                                                                              | **Must**   |
| NFR-16   | All data at rest SHALL be encrypted using AES-256 (AWS KMS managed keys).                                                                          | **Must**   |
| NFR-17   | Access to PHI SHALL be governed by RBAC with the principle of least privilege.                                                                      | **Must**   |
| NFR-18   | The system SHALL comply with India's **Digital Personal Data Protection Act (DPDPA) 2023**.                                                         | **Must**   |
| NFR-19   | The system SHALL implement the **ABDM Consent Manager** protocol for all patient data access.                                                       | **Must**   |
| NFR-20   | The system SHALL maintain immutable audit logs of all data access, consent events, and dispensing transactions for a minimum of **7 years**.         | **Must**   |
| NFR-21   | The platform SHALL undergo annual **CERT-In empanelled** security audits and penetration testing.                                                   | **Must**   |
| NFR-22   | AI-generated outputs (risk reports, roadmaps) SHALL be explainable; the system SHALL log the input features and reasoning chain that led to each output. | **Should** |

### 7.5 Interoperability

| ID       | Requirement                                                                                                  | Priority   |
|----------|--------------------------------------------------------------------------------------------------------------|------------|
| NFR-23   | The system SHALL conform to **HL7 FHIR R4** for all health data exchange.                                    | **Must**   |
| NFR-24   | The system SHALL integrate with **ABDM Gateway APIs** (registration, consent, health information exchange).   | **Must**   |
| NFR-25   | Diagnosis coding SHALL support **ICD-10**, **SNOMED CT**, and **LOINC**.                                     | **Must**   |
| NFR-26   | The system SHALL expose RESTful APIs conforming to **OpenAPI 3.1** specification for third-party integration. | **Should** |

### 7.6 Usability

| ID       | Requirement                                                                                                  | Priority   |
|----------|--------------------------------------------------------------------------------------------------------------|------------|
| NFR-27   | QuickSight dashboards SHALL be accessible on desktop and tablet form factors.                                | **Must**   |
| NFR-28   | Pharmacy POS interface SHALL be operable with minimal training (≤ 2 hours onboarding).                       | **Must**   |
| NFR-29   | All user-facing interfaces SHALL comply with **WCAG 2.1 Level AA** accessibility standards.                  | **Should** |
| NFR-30   | SMS messages SHALL be concise (≤ 160 characters for alerts) with opt-in deep links for detailed information.  | **Must**   |

---

## 8. Data Requirements

### 8.1 Data Sources

| Source                              | Data Type                                      | Ingestion Method                        | Frequency            |
|-------------------------------------|------------------------------------------------|-----------------------------------------|----------------------|
| ABDM Health Information Exchange    | FHIR Bundles (Conditions, Observations, etc.)  | ABDM Gateway Push / NDHM Sandbox Pull   | Near real-time       |
| ABHA Registry                       | Patient demographics, family linkage           | ABDM API                                | On-demand            |
| Pharmacy POS Terminals              | Dispensing events, token lifecycle              | REST API (HTTPS)                        | Real-time            |
| State IDSP / IHIP Feeds             | Notifiable disease reports                     | Batch (CSV/JSON) / sFTP                 | Daily                |
| Indian Meteorological Department    | Weather data (for vector-borne disease models) | Public API                              | Hourly               |
| Census / District Demographics      | Population data, SES indicators                | Batch load (periodic)                   | Annual               |

### 8.2 Data Retention

| Data Category                       | Retention Period | Storage Tier                    |
|-------------------------------------|------------------|---------------------------------|
| Health records (FHIR)               | 10 years         | HealthLake (Hot) → S3 Glacier   |
| Audit logs                          | 7 years          | CloudWatch Logs → S3 Glacier    |
| Dispensing ledger                   | 7 years          | DynamoDB → S3 Glacier           |
| Predictive model artefacts          | 5 years          | S3 Standard                     |
| SMS delivery logs                   | 3 years          | S3 Standard-IA                  |
| Anonymised epidemiological datasets | Indefinite       | S3 Intelligent-Tiering          |

### 8.3 Data Anonymisation & Privacy

| ID       | Requirement                                                                                                                                       |
|----------|---------------------------------------------------------------------------------------------------------------------------------------------------|
| DR-01    | All data used for heatmap generation and epidemiological analysis SHALL be **de-identified** in compliance with DPDPA 2023 anonymisation standards. |
| DR-02    | Family linkage for hereditary risk modelling SHALL only be established with **explicit, purpose-limited consent** from all linked ABHA holders.     |
| DR-03    | Re-identification risk SHALL be assessed quarterly using k-anonymity (k ≥ 5) and l-diversity metrics.                                              |
| DR-04    | A **Data Protection Impact Assessment (DPIA)** SHALL be conducted before production launch and reviewed annually.                                   |

---

## 9. Integration Requirements

### 9.1 ABDM Integration Points

```
┌──────────────────────────────────────────────────────────────────┐
│                          ABDM GATEWAY                            │
│                                                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────────────────┐  │
│  │ ABHA Registry│  │ Consent Mgr  │  │ Health Info Exchange    │  │
│  └──────┬───────┘  └──────┬───────┘  └──────────┬─────────────┘  │
│         │                 │                      │                │
└─────────┼─────────────────┼──────────────────────┼────────────────┘
          │                 │                      │
          ▼                 ▼                      ▼
┌─────────────────────────────────────────────────────────────────┐
│                       VYADIHARAH PLATFORM                       │
│                                                                 │
│  ┌────────────┐  ┌──────────────┐  ┌─────────────────────────┐  │
│  │ Identity   │  │ Consent      │  │ FHIR Ingestion Engine   │  │
│  │ Service    │  │ Service      │  │ (→ Amazon HealthLake)   │  │
│  └────────────┘  └──────────────┘  └─────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

### 9.2 AWS Service Dependencies

| AWS Service          | Purpose in Vyadiharah                                      | Region Requirement       |
|----------------------|------------------------------------------------------------|--------------------------|
| **Amazon HealthLake**| FHIR R4 data store for all health records                  | ap-south-1 (Mumbai)      |
| **Amazon Bedrock**   | Claude 3.5 for lineage analysis, risk reports, roadmaps    | ap-south-1 (Mumbai)      |
| **Amazon SageMaker** | Predictive outbreak forecasting models                     | ap-south-1 (Mumbai)      |
| **Amazon QuickSight**| District-level dashboards and data visualisation            | ap-south-1 (Mumbai)      |
| **Amazon Pinpoint**  | SMS alerts (adherence, outbreak warnings, roadmaps)        | ap-south-1 (Mumbai)      |
| **Amazon DynamoDB**  | Dispensing token ledger, session state                      | ap-south-1 (Mumbai)      |
| **Amazon S3**        | Data lake, model artefacts, archival storage                | ap-south-1 (Mumbai)      |
| **AWS Lambda**       | Event-driven microservice compute                           | ap-south-1 (Mumbai)      |
| **Amazon API Gateway**| RESTful API exposure for pharmacy POS and third parties    | ap-south-1 (Mumbai)      |
| **AWS KMS**          | Encryption key management                                   | ap-south-1 (Mumbai)      |
| **Amazon CloudWatch**| Monitoring, logging, alerting                               | ap-south-1 (Mumbai)      |
| **AWS IAM**          | Identity and access management, RBAC                        | Global                   |

---

## 10. Constraints & Assumptions

### 10.1 Constraints

| ID   | Constraint                                                                                                          |
|------|---------------------------------------------------------------------------------------------------------------------|
| C-01 | All infrastructure MUST be deployed in the **AWS Asia Pacific (Mumbai) — ap-south-1** region to comply with data residency requirements under DPDPA 2023. |
| C-02 | The platform MUST NOT store any health data in regions outside India.                                                |
| C-03 | Bedrock model selection is limited to models available in ap-south-1 at the time of deployment.                     |
| C-04 | SMS delivery is subject to TRAI DND (Do Not Disturb) regulations; the system cannot override DND preferences.       |
| C-05 | The platform operates as a **HIU (Health Information User)** under ABDM; it does not create health records.          |

### 10.2 Assumptions

| ID   | Assumption                                                                                                          |
|------|---------------------------------------------------------------------------------------------------------------------|
| A-01 | Clinics and hospitals participating in the pilot are already ABDM-registered and capable of sharing FHIR records.   |
| A-02 | Citizens have active ABHA IDs and have registered mobile numbers for SMS delivery.                                  |
| A-03 | Pharmacies in pilot districts have internet connectivity and POS hardware capable of card swipe / QR scan.          |
| A-04 | The NHA will grant Vyadiharah the necessary API credentials and sandbox access for development and testing.         |
| A-05 | State IDSP/IHIP feeds are available in a structured format (CSV/JSON) and shared under a data-sharing agreement.    |

---

## 11. Acceptance Criteria (High-Level)

| Criterion                                                                                         | Verification Method        |
|---------------------------------------------------------------------------------------------------|----------------------------|
| Heatmap updates within 15 minutes of diagnosis data ingestion.                                    | Automated latency test     |
| Predictive alerts issued ≥ 7 days before outbreak peak in retrospective validation on 3+ historical outbreaks. | Backtesting report |
| Hereditary risk report generated for a 3-generation linked family within 10 seconds.              | Performance test           |
| Prescription token burned within 1 second; re-use attempt rejected with audit log entry.          | Functional test + audit    |
| SMS alert delivered to ≥ 95% of targeted recipients within 5 minutes of trigger.                  | Pinpoint delivery report   |
| All ABDM integration test suites pass at 100%.                                                    | NHA certification          |
| Zero critical/high vulnerabilities in pre-launch penetration test.                                | Security audit report      |

---

## 12. Risks & Mitigations

| #  | Risk                                                        | Impact | Likelihood | Mitigation                                                                                               |
|----|-------------------------------------------------------------|--------|------------|----------------------------------------------------------------------------------------------------------|
| R1 | ABDM API changes break integration.                         | High   | Medium     | Abstract ABDM interactions behind an adapter layer; subscribe to NHA changelogs; maintain a sandbox env.  |
| R2 | Bedrock / Claude 3.5 produces clinically inaccurate output. | High   | Medium     | Mandatory clinician review before patient-facing reports; bias auditing pipeline; human-in-the-loop.     |
| R3 | Data breach exposing PHI.                                   | Critical| Low       | Encryption at rest + in transit; VPC isolation; WAF; quarterly pen-tests; incident response playbook.     |
| R4 | Low ABHA adoption in pilot districts limits data volume.    | Medium | Medium     | Partner with NHA awareness campaigns; support offline ABHA registration at pharmacies.                   |
| R5 | SMS fatigue causes citizens to ignore alerts.               | Medium | Medium     | Frequency capping; personalisation; allow preference management; A/B test message content via Pinpoint.  |
| R6 | SageMaker model drift degrades forecast accuracy.           | High   | Medium     | Automated model monitoring (SageMaker Model Monitor); scheduled retraining pipeline; accuracy dashboards. |
| R7 | Pharmacy POS connectivity failures in rural districts.      | High   | High       | Offline-capable POS mode with store-and-forward sync; SMS-based fallback for token validation.           |

---

## 13. Regulatory & Compliance Checklist

| Regulation / Standard                                  | Applicability                                  | Status      |
|--------------------------------------------------------|------------------------------------------------|-------------|
| **Digital Personal Data Protection Act (DPDPA) 2023**  | All personal and health data processing        | Required    |
| **ABDM Health Data Management Policy**                 | FHIR data exchange, consent management         | Required    |
| **IT Act 2000 (Section 43A, 72A)**                     | Sensitive personal data; breach notification    | Required    |
| **DISHA (when enacted)**                               | Electronic health records governance            | Anticipated |
| **ISO 27001:2022**                                     | Information Security Management System          | Recommended |
| **ISO 27799:2016**                                     | Health informatics — information security       | Recommended |
| **HIPAA (for international interop)**                  | If/when data shared with international bodies   | Conditional |
| **CERT-In Directions 2022**                            | Incident reporting within 6 hours              | Required    |

---

## 14. Out of Scope (v1.0)

The following are explicitly **out of scope** for the initial release:

1. **Telemedicine integration** — video/audio consultations via ABHA.
2. **Insurance claim processing** — integration with PMJAY or private insurers.
3. **Genomic sequencing data** — the hereditary risk model operates on clinical diagnostic history, not raw genomic data.
4. **International data sharing** — cross-border health data exchange (e.g., IHR reporting to WHO).
5. **Mobile application** — citizen-facing mobile app (v1.0 relies on SMS + secure web portal).
6. **Wearable / IoT device integration** — real-time vitals from smartwatches or medical devices.

---

## 15. Appendices

### Appendix A — Referenced ABDM Standards

- [ABDM Sandbox Documentation](https://sandbox.abdm.gov.in/docs/)
- [NHA FHIR Implementation Guide](https://nrces.in/ndhm/fhir/r4/index.html)
- [ABHA Number Guidelines](https://abdm.gov.in/abha-number)
- [Health Data Management Policy](https://abdm.gov.in/publications/health_data_management_policy)

### Appendix B — MoSCoW Priority Legend

| Priority   | Meaning                                                       |
|------------|---------------------------------------------------------------|
| **Must**   | Non-negotiable for v1.0 launch.                               |
| **Should** | High value; include if feasible within timeline.              |
| **Could**  | Desirable enhancement; defer if necessary.                    |
| **Won't**  | Explicitly excluded from v1.0 (may appear in future roadmap). |

---

*End of Requirements Specification — Vyadiharah v1.0*
