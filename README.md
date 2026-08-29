# Aktriva Medical Device Security Tools

Free, browser-based tools for medical device security teams. No account, and
everything runs in your browser — nothing you enter is transmitted to Aktriva
or stored anywhere.

| Tool | Live at | What it does |
|------|---------|--------------|
| **CVSS 3.1 Vulnerability Scoring** | <https://app.aktriva.com/cvss/> | Score vulnerabilities with CVSS v3.1 base metrics, or a guided questionnaire adapted from MITRE's *"Rubric for Applying CVSS to Medical Devices."* Build a findings table and export it to CSV. |
| **CVSS 4.0 Vulnerability Scoring** | <https://app.aktriva.com/cvss4/> | Score vulnerabilities with CVSS v4.0: direct base metrics, a guided v4.0 rubric, a v3.1‑to‑v4.0 converter, and an AI‑assisted draft score. Build a findings table and export it to CSV. |
| **VEX Generator** | <https://app.aktriva.com/vex/> | Produce standards‑compliant VEX (Vulnerability Exploitability eXchange) documents in CycloneDX 1.6 format through a guided five‑step wizard, with schema validation and an HTML / PDF report. |

This repository hosts the documentation and the public issue tracker for these
three tools. The tools themselves are developed and hosted by
[Aktriva](https://aktriva.com).

---

## Reporting bugs & feedback

**Use this repository's [Issues](https://github.com/aktriva/tools/issues) to
report a problem or request a change for any of the three tools.**

When filing an issue, please include:

- **Which tool** and the page URL (e.g. `https://app.aktriva.com/vex/`)
- **What you did** — the steps to reproduce
- **What you expected** vs. **what actually happened**
- **Browser and operating system** (e.g. Firefox 128 on Windows 11)
- Screenshots if the problem is visual

Please do **not** paste confidential vulnerability details, proprietary device
information, or PHI into a public issue. Everything you enter into the tools
stays in your browser, so the maintainers never see it — a GitHub issue is
public. Use a minimal, non‑sensitive example instead.

For anything security‑sensitive about the tools themselves, contact us through
<https://aktriva.com/contact/> rather than opening a public issue.

---

## Common behavior

Things that are true of all three tools.

- **No account, no database.** Nothing you enter is written to a server or
  shared with anyone.
  - The CVSS tools keep your findings in the browser **session** only; they are
    cleared when the session ends.
  - The VEX Generator keeps everything in **browser memory** until you download
    it — reloading the page discards it.
- **Outbound requests.** The tools run in your browser, with three exceptions
  that call external services:
  - **NVD lookup** — sends the CVE ID to the [NVD REST API](https://services.nvd.nist.gov/)
    to pull a description and CVSS metrics.
  - **CISA KEV check** — checks the CVE against CISA's Known Exploited
    Vulnerabilities catalog to show the "Listed in CISA KEV Catalog" badge. The
    catalog is cached for 12 hours.
  - **AI estimate** (CVSS 4.0 only) — sends the description, CVE, vector and any
    reference URL you provide to Anthropic's API.
- **Severity bands** (standard CVSS): `0.0` None · `< 4.0` Low · `< 7.0` Medium
  · `< 9.0` High · otherwise Critical. Base scores use the CVSS round‑up
  (nearest 0.1, rounding up).
- **Light/dark theme.** The header toggle remembers your choice; with no choice
  saved the page follows your operating system setting.
- **Requirements.** A modern browser with JavaScript enabled.
- **Not regulatory advice.** The output is a decision‑support aid only and does
  not by itself satisfy FDA premarket, EU MDR, postmarket, or quality‑system
  requirements. Review all output before relying on it.

---

## CVSS 3.1 Vulnerability Scoring

Score medical device vulnerability findings with CVSS v3.1 base metrics and
build an exportable findings table.

### Modes

Switch with the tab toggle at the top; the choice is remembered for the
session.

**Basic CVSS** — pick a value for each of the eight base metrics directly, per
the [FIRST.org v3.1 specification](https://www.first.org/cvss/specification-document).
Best if you already know CVSS. A live preview panel shows the score and vector
as you choose. Metrics: Attack Vector (AV), Attack Complexity (AC), Privileges
Required (PR), User Interaction (UI), Scope (S), Confidentiality (C), Integrity
(I), Availability (A).

**MITRE CVSS Rubric** — a guided questionnaire adapted from MITRE's *"Rubric
for Applying CVSS to Medical Devices"* (Base Metric Group). It walks through
medical‑device‑specific questions that resolve to the **same** CVSS values as
Basic mode.

- A progress bar tracks "*X* of 8 metrics answered". The submit button unlocks
  only when all are resolved.
- The **Impact Metrics** section expands into per‑data‑type and per‑function
  questions for C, I and A. Use *"Mark remaining unanswered as None"* to fill in
  data types the device doesn't handle. Answered categories collapse
  automatically; click a heading to collapse or expand it.
- Answers that indicate a **Potential Impact to Patient Safety (PIPS)** raise a
  banner and set the finding's PIPS flag, which appears as its own column in the
  findings table and CSV.
- This implementation is not produced, reviewed, or endorsed by MITRE. While
  the MITRE rubric document is an FDA‑qualified Medical Device Development Tool
  (MDDT), this software implementation is **not** separately qualified.

### Per-finding fields

- **Vulnerability ID** — auto‑assigned as `VULN-001`, `VULN-002`, … The
  auto‑numbering follows whatever prefix and zero‑padding width the previous row
  used, so a custom scheme (e.g. `ACME-0007`) continues correctly. Editable.
- **CVE ID** *(optional)* — enables the NVD lookup when it matches
  `CVE-YYYY-NNNN[NNN]`.
- **Description** *(optional)* — free text; prefilled by an NVD lookup.

### CVE lookup (NVD + KEV)

Click **Look up in NVD** once the CVE ID is well‑formed:

- Prefills the description and, in **Basic** mode, the eight metric dropdowns
  from NVD's primary CVSS v3.1 (or v3.0) data. **Review before adding** — NVD's
  score is not always right for a device context.
- In **MITRE** mode the lookup prefills the **description only**; metrics still
  come from the questions.
- Shows a **CISA KEV** badge: listed (with date added), not listed, or hidden
  if the KEV check itself was unavailable (it never blocks the NVD lookup).
- Trailing "CVSS Vector: (…)" / "CVSS x Base Score y" sentences that some
  advisories append to the description are stripped.
- If NVD has a description but no CVSS v3.x data, the description is still
  filled and you enter metrics manually.

### Findings table & export

Columns: Vulnerability ID · CVE ID · Description (truncated, full text on
hover) · Vector · Score · Severity badge · PIPS · Remove.

- **Clear All** empties the table.
- **Export CSV** downloads `cvss-findings-YYYY-MM-DD.csv` with every metric in
  its own column plus the vector string and PIPS flags. Values that could be
  read as spreadsheet formulas are escaped to defend against CSV injection.

---

## CVSS 4.0 Vulnerability Scoring

The same workflow as the 3.1 tool, updated for the CVSS v4.0 metric set, plus
two ways to bootstrap a score when you don't have full v4.0 inputs. It keeps its
own findings table, separate from the 3.1 tool.

### Modes

**Direct Entry** — pick a value for each of the eleven base metrics directly,
per the [FIRST.org v4.0 specification](https://www.first.org/cvss/v4-0/). Live
preview of score and vector. Metrics — Exploitability: Attack Vector (AV),
Attack Complexity (AC), **Attack Requirements (AT)**, Privileges Required (PR),
User Interaction (UI). Vulnerable System Impact: VC, VI, VA. Subsequent System
Impact: SC, SI, SA.

**Guided Rubric** — a question‑by‑question wizard transcribed from FIRST.org's
[CVSS v4.0 User Guide](https://www.first.org/cvss/v4-0/cvss-v40-user-guide.pdf).
Each metric fills in automatically as its question resolves; the submit button
unlocks when all 11 are set. Progress bar: "*X* of 11 metrics answered". The
three **Subsequent System** impact metrics default to **None** — work through
their questions only if the vulnerability also affects systems beyond the
vulnerable component. Hover the "i" tooltips for extra guidance.

### Bootstrapping a v4.0 score (Guided Rubric only)

**Convert from a CVSS 3.1 vector** — paste a v3.1 vector into the *CVSS 3.1
Vector* field and click **Estimate CVSS 4.0**. No CVE required; the field
auto‑fills from an NVD lookup when one is available. There is no official
FIRST.org / NVD cross‑version conversion, so this fills in only what the metric
definitions make unambiguous and leaves the rest for the questions. The finding
is tagged **Estimated**. If the 3.1 Scope was *Changed*, the impact questions
are left unanswered (rather than defaulted) because the 3.1 vector can't say
which side was worse.

**Estimate via AI** — with a description entered (a CVE is not required), click
**Estimate via AI**. The request goes to an Anthropic model with structured
JSON output and, if a **Reference URL** is given, the model may fetch that one
page. It returns a value for each metric **or nothing** where the context
doesn't support a judgment (it won't guess), plus a rationale and optional
notes. The finding is tagged **AI‑Estimated**.

- Editing any metric afterward clears the Estimated / AI‑Estimated tag for that
  finding — you're overriding the seeded value.
- **AI output is a suggestion, not authoritative.** Review every metric before
  relying on it.
- The feature is rate‑limited: a short per‑session cooldown, plus per‑IP and
  global daily caps that reset at midnight. The reference‑URL fetch path is cut
  off before the metrics‑only path when the daily budget runs low.

### CVE lookup (NVD + KEV)

**Look up in NVD** prefills the description and, when NVD has published **CVSS
v4.0** metrics, the eleven dropdowns. NVD's v4.0 coverage is still limited:

- If NVD has only a **CVSS 3.1** score, Direct Entry offers a link to switch to
  Guided Rubric with the CVE carried over; the lookup re‑runs automatically and
  the 3.1 vector is filled so you can click **Estimate CVSS 4.0**.
- If NVD has no CVSS data at all, the description is still filled and you answer
  the questions manually.
- The **CISA KEV** badge behaves exactly as in the 3.1 tool.

### Findings table & export

Columns: Vulnerability ID · CVE ID · Description · Vector (with an *Estimated*
or *AI‑Estimated* pill when applicable) · Score · Severity · Remove. There is no
PIPS column.

**Export CSV** downloads `cvss4-findings-YYYY-MM-DD.csv` with all eleven metrics
as columns, the vector string, and an **Estimation Source** column (`No` /
`CVSS 3.1` / `AI`). Same CSV‑injection escaping as the 3.1 tool.

---

## VEX Generator

Produce a standards‑compliant **VEX** (Vulnerability Exploitability eXchange)
document in **CycloneDX 1.6** format through a five‑step wizard. Unlike the CVSS
tools there is no findings table — each pass through the wizard produces one
document, and all state lives in browser memory until you download it.

### Step 1 — Vulnerability Information

- **CVE ID** + **Look up in NVD**: prefills Title, Description, CVSS Score,
  CVSS Vector, Published Date, and the **Affected Component** (name and version)
  from NVD's data. Shows the **CISA KEV** badge. Everything stays editable
  afterward.
- **CVSS Score** → **Severity** is derived automatically (same bands as the
  CVSS tools) and shown as a badge; there is no separate severity input.
- **Affected Component** — the specific vulnerable dependency named by the CVE
  (usually a library): name, version, and CycloneDX component **type** (one of
  application, framework, library, container, platform, operating‑system,
  device, device‑driver, firmware, file, machine‑learning‑model, data,
  cryptographic‑asset; defaults to *library*).

### Step 2 — Product Information

- **Your organization name** *(required)* — you are the publisher of the VEX
  document; this becomes its publisher / organization identity. Optional
  organization URL.
- One or more **products** you ship that embed the affected component. Each:
  name\*, version\*, supplier / manufacturer, and type (defaults to
  *application*). Use **+ Add another product** for more.

### Step 3 — Assessment

Up to five yes / no / unknown questions. Status and justification are derived
automatically, and a later question only appears when the answer so far isn't
conclusive:

| # | Question | Conclusive answers |
|---|----------|--------------------|
| 1 | Does the product contain the vulnerable component? | **No** → Not Affected (*component not present*) · **Unknown** → Under Investigation |
| 2 | Is the vulnerable code included? | **No** → Not Affected (*vulnerable code not present*) · **Unknown** → Under Investigation |
| 3 | Can the vulnerable functionality be executed? | **No** → Not Affected (*not in execute path*) |
| 4 | Are mitigations already in place? | **Yes** → Not Affected (*inline mitigation exists*) |
| 5 | Has a fix been released? | **Yes** → Fixed (reveals a **Fixed Version** field) · **No** → Affected |

- Optional **Rationale** and **Recommendation** free‑text fields. Recommendation
  is disabled when the status is *Not Affected* (nothing to remediate).
- **Per‑product overrides** let individual products carry a different status or
  justification from the default.

### Step 4 — Review

A consolidated summary. Products that share the same status, justification,
fixed version, rationale and recommendation are grouped together.

### Step 5 — Generate & Download

- **CycloneDX VEX JSON** — CycloneDX 1.6. **Validate Against Official Schema**
  runs a bundled validator against the official CycloneDX 1.6 schema. **Copy**
  or **Download** the `.json`.
- **Human‑readable report** — **Print / Save as PDF**, or **Download HTML**.

Internal status → CycloneDX `analysis.state`: Not Affected → `not_affected`,
Affected → `exploitable`, Fixed → `resolved`, Under Investigation → `in_triage`.
Justifications map to CycloneDX's nine‑value `analysis.justification` enum on a
best‑effort basis (the vocabularies aren't 1:1).

---

## Troubleshooting

| Symptom | Cause / fix |
|---------|-------------|
| "Could not reach NVD" / "NVD rate limit reached" | NVD's public API is rate‑limited and occasionally slow. Wait a moment and retry, or enter values manually. |
| KEV badge doesn't appear | The CISA feed couldn't be loaded and there's no cached copy yet. The badge is suppressed rather than showing a wrong "not listed". |
| "AI estimation is not configured" | The AI‑estimate feature isn't enabled on this deployment. |
| "You've reached today's AI estimation limit" / "at capacity for today" | A per‑IP or global daily cap was hit; it resets at midnight. Continue manually. |
| Findings disappeared | Expected — the CVSS tools store findings in the browser session only, and the VEX Generator in memory only. Export before closing. |
| Metrics from an NVD lookup look wrong for the device | NVD's score reflects a generic context. Adjust the metrics (or use the MITRE / Guided Rubric mode) for the device's actual use. |

---

## Disclaimer

These tools are decision‑support aids only. Their output does not constitute
regulatory, legal, clinical, or compliance advice and does not by itself
satisfy FDA premarket cybersecurity requirements, postmarket surveillance
obligations, EU MDR, or any other regulatory or quality‑system requirement.
Scores and documents are generated from the information you provide and are not
independently verified. Aktriva LLC makes no warranty as to their accuracy,
completeness, or fitness for any purpose and disclaims liability for decisions
made in reliance on their output. Your organization remains solely responsible
for its risk‑assessment methodology and regulatory compliance.

## Attribution

- **CVSS** is owned by [FIRST.Org, Inc.](https://www.first.org/cvss/)
- The **"Rubric for Applying CVSS to Medical Devices"** is © 2019 The MITRE
  Corporation, developed under contract to the FDA. The guided‑rubric mode is an
  independent implementation, not produced, reviewed, or endorsed by MITRE.
- **CycloneDX** is a specification of the [OWASP Foundation](https://cyclonedx.org/)
  (Apache License 2.0). The VEX Generator also bundles the JSON Signature Format
  (JSF) schema (Apache License 2.0) and the Ajv JSON Schema validator (MIT
  License).

---

© Aktriva LLC. Built from our medical device cybersecurity consulting work —
see <https://aktriva.com>.
