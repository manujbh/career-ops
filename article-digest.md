# Article Digest — AI & Compliance Engineering Proof Points
# Source: /Users/manujbh/Library/Mobile Documents/iCloud~md~obsidian/Documents/MEMORY/RESEARCH+WORK
# Last updated: 2026-05-10

---

## AI Compliance by Design (Dematic)

### RAG Security Agent on GCP — Compliance Automation via Generative AI

**What:** Architected a Retrieval-Augmented Generation (RAG) security agent on Google Cloud (Vertex AI + Gemini) to automate GRC compliance Q&A. The agent retrieves grounded answers from a curated policy corpus — NIST 800-53 Rev 5, ISO 27001:2022, NIS2, EU CRA, internal policies — and generates citation-grounded responses without hallucinating from training data.

**Scope of implementation:**
- 4-phase, 16-week implementation roadmap: corpus design, behavioral configuration, ground truth evaluation, operationalization
- Full NIST 800-53 control mapping for the AI system itself (14 control families: AC, AU, CA, CM, IA, IR, PL, PM, RA, SA, SC, SI, AT, MA)
- Ground truth evaluation infrastructure: 200+ queries, 5-dimension scoring (citation accuracy, completeness, scope adherence, factual correctness, actionability), automated Vertex AI Evaluation Service
- Corpus management with document versioning, staleness detection, and regulatory calendar automation (NIST updates, EU CRA September 2026 obligations)
- AI governance controls: version-controlled system prompts, hallucination prevention, grounding threshold enforcement, prompt injection detection

**Compliance alignment:** NIST AI RMF, ISO 27001 A.8.25, EU AI Act Art. 13, OWASP AI Maturity Assessment

**Stack:** GCP (Vertex AI Agent Builder, Vertex AI Search, Gemini Pro/Flash, Cloud Run, BigQuery, Cloud Logging, Cloud Scheduler, Secret Manager, VPC Service Controls)

---

### AI Security Posture Management (AI-SPM)

**What:** Designed AI-SPM framework extending CSPM practices to AI/ML assets -- applying the same discovery-and-remediation loop used to reduce cloud misconfigs from 3,000 to 15 to the AI asset layer.

**Components:**
- AI Asset Registry: inventory of all AI assets by type (LLM API, self-hosted, embedding model, agentic workflow, vector store) with risk tiering (Critical/High/Medium/Low)
- Shadow AI Scanner: detects unauthorized AI usage via package dependency scan (`openai`, `anthropic`, `langchain`, `transformers`), network/DNS log analysis, and environment variable pattern matching
- AI Supply Chain Scanner: scans model cards for training data documentation, bias evaluation, data poisoning risk; checks ML framework CVEs (torch, transformers, langchain)
- Posture Score: composite risk score (0-100) weighting shadow AI, unapproved production deployments, supply chain CVEs, and control coverage gaps
- Executive brief format: posture label (GOOD / NEEDS ATTENTION / AT RISK), critical findings, coverage KPIs

**Compliance alignment:** ISO 27001 A.5.23, NIST SSDF PS.1.1/PS.3.2, EU AI Act Art. 9, NIST AI RMF GOVERN 6.1

---

### LLM Firewalls and AI Guardrails

**What:** Designed LLM Firewall (input/output security control plane) and composable AI Guardrail pipeline for production AI systems.

**LLM Firewall layers:**
- Prompt injection detection: multi-tier (regex pattern matching + semantic embedding similarity + classifier scoring), targeting instruction override, persona hijack, delimiter injection, data exfiltration
- PII detection and redaction: risk-tiered (SSN/CCN/keys → block; email/phone → redact and allow), deterministic pseudonymization for referential integrity
- Output safety scoring: toxicity, bias, hallucination risk, confidentiality leakage -- with configurable thresholds per deployment context

**AI Guardrail pipeline:**
- Topic classifier for scope enforcement (functional boundary control -- same pattern as CODEOWNERS for AI)
- Composable pipeline: fail-fast block on critical violations, accumulating warn on lower-severity signals
- Signal quality metrics: block rate, precision (TP/total alerts), false positive rate -- same formula as AWS detection engineering work

**Metrics alignment:** Block rate < 5%, signal quality > 85%, FP rate < 10%, P99 latency < 50ms

**Compliance alignment:** NIST 800-53 SI-3 (malicious code protection), SI-10 (input validation), AU-12 (audit record generation), SC-8 (transmission confidentiality)

---

## Policy-as-Code / NIST 800-53 Automation (Dematic)

### OPA/Rego Policy-as-Code Library — 19 NIST 800-53 Control Families

**What:** Designed and implemented OPA/Rego policy-as-code library translating NIST 800-53 "-1" (Policy and Procedures) controls into machine-enforced reality across three layers: pre-merge CI/CD gates, GKE admission controls, and GCP runtime enforcement.

**Architecture:**
```
Developer Commit
→ GitLab MR Pipeline (SAST + SCA + IaC OPA + Compliance story check)
→ Jenkins Deploy Pipeline (artifact provenance + OPA bundle + Terraform Sentinel)
→ GCP Runtime (OPA Gatekeeper + Org Policy Constraints + Security Command Center)
→ Evidence Pipeline (BigQuery + Looker dashboards)
```

**Control families automated (19):**
AC-1, AT-1, AU-1, CA-1, CM-1, CP-1, IA-1, IR-1, MA-1, MP-1, PE-1, PL-1, PM-1, PS-1, RA-1, SA-1, SC-1, SI-1, SR-1

**Key implementations:**
- AC-1: OPA blocks wildcard IAM, owner-level bindings, and static service account keys; Terraform Sentinel enforces approved role registry
- AU-1: OPA enforces Cloud Run structured logging, GCS bucket access logging; GKE Gatekeeper requires log-collector sidecar in prod namespaces
- CA-1: Jenkins gate queries GRC API for ATO token before prod deployment; expired ATO = deployment blocked; evidence written to BigQuery
- CM-1: Drift detection via daily Terraform plan against live GCP; diff = BigQuery drift event + Slack alert; resource change tracking labels enforced
- IR-1: Cloud Run webhook routes SCC alerts by severity (CRITICAL = PagerDuty + 15min MTTD SLA), logs MTTD/MTTR to BigQuery
- SI-1: Veracode SAST gate blocks Critical findings; OPA warns on High findings > 30 days old; SAST results archived to BigQuery

**Evidence pipeline (unified schema):** All policy events write to `dematic-security.compliance.policy_events` (BigQuery), powering real-time Looker dashboards per control family.

**Outcome:** Compliance assertions automated from manual evidence gathering to continuous, queryable, real-time control telemetry across 19 NIST 800-53 families.

---

### CI/CD to NIST 800-53 Mapping — Five Layers

**What:** Mapped DevSecOps pipeline layers to NIST 800-53 Moderate baseline controls with precise control citations and audit evidence artifacts.

| Layer | Tool | Lead Control | Evidence |
|-------|------|-------------|---------|
| Pre-commit secret detection | TruffleHog | IA-5 (Authenticator Management) | SARIF + BigQuery |
| PR/merge gates (SAST + SCA) | Semgrep + dependency-review | SA-11 (Developer Testing) | SARIF + BigQuery |
| IaC scanning | Checkov | CM-2 (Baseline Configuration) | SARIF + BigQuery |
| Container scanning | Trivy | SI-2 (Flaw Remediation) | SBOM (SPDX/CycloneDX) |
| Continuous posture monitoring | GCP SCC | CA-7 (Continuous Monitoring) | Looker dashboards |

**Cross-cutting:** AU-12 (audit record generation), AU-9 (audit protection via immutable SARIF), AU-10 (non-repudiation via signed commits + commit SHA-linked SARIF)

---

## AI Governance Framework (Dematic)

### Enterprise AI Governance Plan — NIST AI RMF + OWASP AIMA

**What:** Authored enterprise AI governance strategy integrating NIST AI RMF (Govern, Map, Measure, Manage), OWASP AI Maturity Assessment (AIMA), and AgentOps evaluation pyramid.

**Key components:**
- AI Council governance structure with defined maturity levels (Level 1 Initial → Level 2 Defined → Level 3 Integrated)
- AI system inventory with model version tracking, data provenance documentation, and deactivation protocols
- Data quality pipeline with toxicity/hallucination filters, LLM-specific data curation
- Evaluation pyramid: Component (unit tests), Trajectory (reasoning traces), Outcome (LLM-as-judge), System (production monitoring)
- AI velocity paradox resolution: path from ad-hoc experimentation to "AI Factory" model with standardized POC guides

**Board-level metrics defined:**
- Governance Coverage Rate: % of AI systems inventoried and risk-classified per NIST AI RMF
- Compliance Adherence Score: % of AI events complying with governance/residency policies (target > 95%)
- Incident Rate per User: AI policy violations per user per month (target < 0.1%)
- Audit Readiness: % of models with current documentation, version control, model cards

**Compliance alignment:** NIST AI RMF 1.0, EU AI Act, GDPR, ISO 27001 A.8.25, OWASP AI Maturity Assessment

---

## DevSecOps Implementation (Dematic)

### Security Automation Implementation

**What:** Directed DevSecOps transformation at Dematic, learning developer experience end-to-end to identify security embedding points.

**Implementations:**
- GitLab security features: Protected branches, MR approval rules (four-eyes), CODEOWNERS for sensitive paths (/infrastructure, /auth, /iam), signed commits
- GitLab security scanning: SAST, SCA, IaC scanning, secret detection -- from point-in-time reports to continuous pipeline integration
- SAST via Veracode with policy-backed gating (critical/very-high flaws, known-vuln CVEs, audit trail of who approved risk and when)
- Checkov for Terraform/IaC: prevent public buckets, overly broad IAM, missing encryption, insecure network exposure
- OPA (Rego) planned for unified policy language across Checkov, Veracode, and GitLab built-ins
- SRE/security overlap: DORA metrics + security signals in SRE dashboards (deploy frequency, security gate failure rate, critical vuln aging, privileged IAM changes, drift detections, incident classes)

**Outcome:** Continuous security reporting aligned to development velocity; 240+ daily CI/CD security scans embedded across 5 platforms.
