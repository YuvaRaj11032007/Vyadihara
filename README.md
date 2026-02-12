<p align="center">
  <img src="https://img.shields.io/badge/🏥-Vyadiharah-blueviolet?style=for-the-badge&logoColor=white" alt="Vyadiharah" height="50"/>
</p>

<h1 align="center">व्याधिहरः · Vyadiharah</h1>

<p align="center">
  <strong>AI-Driven Epidemiological Intelligence & Predictive Surveillance Layer for the ABDM Ecosystem</strong>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Hackathon-AI%20for%20Bharat%202026-orange?style=for-the-badge" alt="AI for Bharat 2026"/>
  <img src="https://img.shields.io/badge/Status-Prototype-yellow?style=for-the-badge" alt="Prototype"/>
  <img src="https://img.shields.io/badge/Platform-AWS-232F3E?style=for-the-badge&logo=amazonaws&logoColor=white" alt="AWS"/>
  <img src="https://img.shields.io/badge/Standard-HL7%20FHIR%20R4-red?style=for-the-badge" alt="FHIR R4"/>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/AI-Amazon%20Bedrock%20(Claude%203.5)-blue?style=flat-square" alt="Bedrock"/>
  <img src="https://img.shields.io/badge/ML-Amazon%20SageMaker-green?style=flat-square" alt="SageMaker"/>
  <img src="https://img.shields.io/badge/Data-Amazon%20HealthLake-purple?style=flat-square" alt="HealthLake"/>
  <img src="https://img.shields.io/badge/Viz-Amazon%20QuickSight-orange?style=flat-square" alt="QuickSight"/>
  <img src="https://img.shields.io/badge/Comms-Amazon%20Pinpoint-pink?style=flat-square" alt="Pinpoint"/>
</p>

---

## 🌍 The Problem

India's healthcare system generates **millions of clinical data points daily** across 770+ districts — yet this data remains siloed, underutilised, and disconnected. Outbreaks are detected **reactively**, hereditary risks go **unidentified**, and prescription fraud costs the system **billions annually**.

**There is no unified, AI-powered intelligence layer that sits atop the ABDM ecosystem to transform this fragmented data into actionable, life-saving insights.**

---

## 💡 Our Solution — Vyadiharah

**Vyadiharah** (Sanskrit: *व्याधिहरः* — *"that which removes disease"*) is a cloud-native, AI-driven **epidemiological intelligence platform** designed as a **predictive surveillance layer** on top of India's **Ayushman Bharat Digital Mission (ABDM)** infrastructure.

It transforms anonymised health records flowing through ABHA into:

| Capability | What It Does |
|---|---|
| 🗺️ **Live Outbreak Mapping** | Real-time "Health Heatmap" of India at district granularity |
| 🔮 **Predictive Alerting** | AI forecasts disease spread 14 days ahead for proactive resource deployment |
| 🧬 **Hereditary Risk Modelling** | Links family ABHA profiles to identify genetic predispositions |
| 🛡️ **Preventive Roadmaps** | Auto-generated personalised health advisories during outbreaks |
| 💊 **Smart Pharmacy Dispensing** | ABHA card swipe-to-validate with digital prescriptions |
| 🔒 **Prescription Fraud Prevention** | Cryptographic single-use tokens that burn after dispensing |

---

## 🏗️ Architecture at a Glance

```
                        +------------------+
                        |    CITIZENS       |
                        |   (ABHA Holders)  |
                        +--------+---------+
                                 | SMS Alerts / Web Portal
                                 v
+--------------+     +-----------------------------+     +------------------+
|  CLINICS &   |---->|      VYADIHARAH PLATFORM     |<----|  HEALTH OFFICERS |
|  HOSPITALS   |     |                               |     |  & EPIDEMIOL.    |
+--------------+     |  +-------+  +--------------+ |     +------------------+
                     |  |  AI   |  | Surveillance | |
+--------------+     |  |Engine |  |    Engine     | |     +------------------+
|  PHARMACIES  |<--->|  +-------+  +--------------+ |---->|  REGULATORY      |
|  (POS)       |     |  +-------+  +--------------+ |     |  AUTHORITIES     |
+--------------+     |  | Smart |  |   Consent &  | |     +------------------+
                     |  | Disp. |  |   Identity   | |
                     |  +-------+  +--------------+ |
                     +--------------+----------------+
                                    | ABDM APIs
                                    v
                        +------------------+
                        |   ABDM GATEWAY   |
                        |  (NHA Infra)     |
                        +------------------+
```

---

## 🛠️ Technology Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| **Data Backbone** | Amazon HealthLake | FHIR R4 compliant health record storage ensuring interoperability across all Indian health providers |
| **AI Engine** | Amazon Bedrock (Claude 3.5) | Complex hereditary lineage analysis, clinical summarisation, and preventive roadmap generation |
| **Predictive ML** | Amazon SageMaker (DeepAR / Prophet / Claude) | Time-series outbreak forecasting and predictive alerting |
| **Analytics & Viz** | Amazon QuickSight | District-level interactive dashboards with drill-down (national -> state -> district -> facility) |
| **Communication** | Amazon Pinpoint | Automated multi-language SMS alerts for medication adherence & outbreak warnings |
| **Token Ledger** | Amazon DynamoDB | Cryptographic dispensing token lifecycle management with fraud detection |
| **Event Bus** | Amazon EventBridge | Asynchronous, event-driven orchestration across all services |
| **Compute** | AWS Lambda (Python 3.12) | Serverless microservices for all business logic |
| **API** | Amazon API Gateway | RESTful API layer (OpenAPI 3.1) for pharmacy POS and third-party integrations |
| **Security** | AWS KMS, WAF v2, IAM, GuardDuty | End-to-end encryption (TLS 1.3 / AES-256), RBAC, threat detection |
| **IaC & CI/CD** | AWS CDK (TypeScript), CodePipeline | Infrastructure as Code with automated canary deployments |

---

## 🎯 Use Cases — Deep Dive

### 1️⃣ Real-Time District Disease Intelligence

<table>
<tr>
<td width="50%">

**Live Outbreak Mapping**
- Aggregates anonymised diagnosis tags (ICD-10 / SNOMED CT) from registered clinics
- Generates a real-time **Health Heatmap** at district granularity, refreshed every **15 minutes**
- Computes a **District Disease Burden Index (DDBI)** — a composite score of case volume, growth rate, severity mix, and healthcare-capacity utilisation

</td>
<td width="50%">

**Predictive Alerting**
- SageMaker-trained DeepAR models forecast outbreak trajectories **14 days ahead**
- Features include case counts, weather data, population density, seasonal patterns, and historical outbreak indicators
- Three alert tiers: **Watch** (>50% growth) -> **Warning** (>100%) -> **Emergency** (>200%)
- Automated resource-deployment recommendations to pharmacies and hospitals

</td>
</tr>
</table>

### 2️⃣ AI-Driven Lineage & Preventive Care

<table>
<tr>
<td width="50%">

**Hereditary Risk Modelling**
- Securely links family ABHA profiles with **explicit consent** via the ABDM Consent Manager
- Amazon Bedrock (Claude 3.5) analyses multi-generational family health records
- Produces structured **Hereditary Risk Reports** covering diabetes, cardiovascular disease, thalassemia, sickle cell disease, and more
- All reports require **clinician review** before reaching patients

</td>
<td width="50%">

**Preventive Roadmap Generation**
- During a detected local outbreak, the system identifies high-vulnerability individuals from hereditary risk profiles
- Generates personalised **Preventive Roadmaps**: vaccinations, screenings, lifestyle modifications, and warning signs
- Delivered via **multi-language SMS** (Hindi, English, 5+ regional languages) within **1 hour** of outbreak threshold crossing
- Full detailed roadmap available on a secure web portal

</td>
</tr>
</table>

### 3️⃣ Smart Dispensing & Pharmacy Integration

<table>
<tr>
<td width="33%">

**Swipe-to-Validate**
- ABHA cards function as secure swipe cards at pharmacy POS terminals (swipe / QR / NFC)
- Retrieves and displays exact digital prescriptions from HealthLake
- Supports partial dispensing

</td>
<td width="33%">

**Error Reduction**
- Digital prescriptions eliminate handwriting errors entirely
- Real-time **drug-drug interaction** and **allergy cross-referencing** at point of sale
- Dosage anomaly detection (e.g., adult dose for a paediatric patient)

</td>
<td width="33%">

**Fraud Prevention**
- Each prescription tokenised with a cryptographic **single-use Dispensing Token**
- Tokens are **burned** (irreversibly invalidated) upon dispensing
- Pattern detection flags suspicious activity (high-frequency prescribing, multi-pharmacy fills)

</td>
</tr>
</table>

---

## 📂 Repository Structure

```
Vyadiharah/
+-- README.md              <- You are here
+-- requirements.md        <- Detailed functional & non-functional requirements (MoSCoW prioritised)
+-- design.md              <- System design: architecture, component designs, data models,
                              security, deployment, DR, cost estimation, and rollout plan
```

---

## 📊 Key Metrics & Targets

| Metric | Target |
|--------|--------|
| Outbreak Detection -> Alert | ≤ 48 hours |
| 14-Day Forecast Accuracy | ≥ 70% |
| Heatmap Refresh Latency | ≤ 15 minutes |
| Prescription Validation (P99) | ≤ 3 seconds |
| Token Burn Confirmation (P99) | ≤ 1 second |
| SMS Delivery Success Rate | ≥ 95% |
| Platform Availability | ≥ 99.95% |
| Dispensing System Availability | ≥ 99.99% |

---

## 🔐 Security & Compliance

- **Encryption**: TLS 1.3 (transit) + AES-256 via AWS KMS (rest) + field-level encryption for ABHA IDs
- **Consent**: Full ABDM Consent Manager integration — purpose-scoped, time-bound, revocable
- **Privacy**: k-anonymity (k ≥ 5), Data Protection Impact Assessments (DPIA)
- **Audit**: Immutable audit logs with S3 Object Lock (WORM) — 7-year retention
- **Compliance**: DPDPA 2023 · ABDM Health Data Management Policy · IT Act 2000 · CERT-In Directions 2022
- **Data Residency**: All data processed and stored exclusively in AWS ap-south-1 (Mumbai)

---

## 🏆 AI for Bharat Hackathon 2026

This project is our submission for the **AI for Bharat Hackathon 2026**. Vyadiharah demonstrates how India's ABDM infrastructure can be **supercharged with AI** to:

- 🩺 **Save lives** through early outbreak detection and predictive alerting
- 🧬 **Prevent disease** through hereditary risk identification and proactive care
- 💊 **Secure pharmacies** through digital prescriptions and fraud prevention
- 🇮🇳 **Scale nationally** across 770+ districts and 500M+ ABHA profiles

---

## ⚠️ Important Note & Disclaimer

> **Prototype Status:** Vyadiharah is currently a **prototype model and conceptual demonstration** for the AI for Bharat Hackathon 2026. It is designed as an intelligence layer to sit on top of the existing ABHA (Ayushman Bharat Health Account) infrastructure and is **not an official government-affiliated service**.
>
> **Data Integrity:** In compliance with hackathon safety guidelines, **all data utilized for this prototype is entirely synthetic** or derived from publicly available datasets. No real-world Protected Health Information (PHI) or private government databases have been accessed or stored.
>
> **Liability:** This system is for **informational and research purposes only**. The AI-generated outputs, including disease risk assessments and preventive roadmaps, should not be construed as clinical diagnoses or professional medical advice.

---

## 👥 Team

Built with ❤️ for India's public health future.

---

<p align="center">
  <strong>व्याधिहरः</strong> — <em>"That which removes disease"</em>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Made%20in-India%20🇮🇳-success?style=for-the-badge" alt="Made in India"/>
</p>
