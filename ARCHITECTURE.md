# Architecture & Design Decisions

This document covers the data model and workflow architecture for OD/EL regulatory reporting automation. The system processes daily, monthly, and quarterly reports across NFRA and PBOC frameworks, for two product lines: overdraft facilities (OD) and entrusted loans (EL).

This is not a record of what was built. It is a record of why it was built this way.

---

## Contents

- [1. The Starting Point](#1-the-starting-point)
- [2. The Data Model](#2-the-data-model)
- [3. The Workflow Design](#3-the-workflow-design)
- [4. The Automation Layer](#4-the-automation-layer)
- [5. Outcomes](#5-outcomes)
- [6. Q&A](#6-qa)

---

## 1. The Starting Point

The inherited state had three defining characteristics.

**No documentation.** The person who had run the function was gone. There was no SOP, no data dictionary, no process record. What existed were spreadsheets — manually maintained, inconsistently structured, with field names that didn't always correspond to a clear business definition.

**No data model.** OD and EL data were managed as separate, unrelated files. The relationship between customer records, loan contracts, and regulatory reporting dimensions had never been made explicit. Each submission cycle involved rediscovering which numbers fed which reports.

**No temporal design.** All data work happened at month-end: locate the data, clean it, reconcile it, generate the output, submit. Every problem arrived at the same moment — the one moment when there was least time to fix it properly.

These three problems were connected. The absence of a data model made daily processing difficult to design. The absence of documentation made the data model hard to build. And both made the month-end batch approach feel safer than it was — because nobody had enough visibility to see a better alternative.

The correct response was not to automate. It was to map.

---

## 2. The Data Model

Before any workflow was designed or any tool was opened, the first deliverable was a complete map of the data.

OD and EL are structurally different products: OD generates daily transaction activity; EL involves infrequent, longer-duration loan arrangements. But from a regulatory reporting perspective, they share a single underlying structure. Both require reporting at customer level, contract level, and transaction level. Both are governed by the same two regulators. Both draw on the same categories of business data.

The data model reflects that shared structure across five layers:

**Corporate Customer** — One record per legal entity. Covers industry classification, enterprise scale, unified credit code, and geography. Shared across OD and EL.

**Loan Contract** — One record per loan arrangement, linked to the customer record. Covers product type, principal, tenor, counterparty, and contractual terms.

**Loan Note** — Transaction-level granularity. One record per individual drawdown or note under a contract. Covers outstanding balance, interest rate, drawdown date, and five-tier classification.

**Loan Movement** — Change records. Captures amendments, repayments, rollovers, and classification changes at the note level. The basis for movement analysis in monthly and quarterly reports.

**Guarantee Contract** — Collateral and guarantee arrangements linked to the loan contract. Covers guarantee type, guarantor, and coverage amount.

Every regulatory report is a different view of this structure. The model does not change when reporting requirements change. The processing logic does.

### Data Sources

**OD source data** is downloaded daily as a structured file from the bank's internal systems. The format is consistent enough to be processed programmatically by the Alteryx daily workflow, which reads the file and writes the results into the data model.

**EL source data** has no machine-readable export from the upstream system. Data is entered manually into the model — directly into the relevant layers — as EL transactions occur. This was a constraint, not a design choice. Rather than block the automation on an uncontrollable upstream dependency, the architecture treated manual EL entry as a valid input layer: entered once, in the right structure, consumed automatically by all downstream processes.

The distinction matters. Manual entry is not the same as unstructured entry. EL data entered directly into the five-layer model is as usable as OD data processed automatically — the only difference is how it arrives.

---

## 3. The Workflow Design

The workflow design is the core of this project. The automation came later. The design came first.

### The daily cycle

OD generates activity every working day. The daily workflow runs on the same schedule:

1. Download the OD source file
2. Process through the Alteryx daily workflow
3. Validate the output against the data model
4. Sync validated data into the relevant model layers
5. Flag any anomalies for same-day resolution

Step 5 is where the temporal design change matters most. Anomalies caught during daily processing are caught when the relevant transaction is one day old, context is fresh, and there is no deadline pressure. Anomalies caught at month-end are caught when the transaction may be weeks old, the submission window is already open, and there is no time to trace the root cause properly.

The daily workflow does not add work. It redistributes it — away from the deadline and toward the days when it can be handled well.

### The monthly and quarterly trigger

When the monthly trigger fires, it reads from the accumulated daily data in the model. At that point, the data has already been validated across every working day in the period. The trigger generates the submission file and working paper, and exits. There is no error-handling logic for data quality problems because by the time the trigger runs, those problems have already been resolved.

This is why the monthly close time dropped from six days to thirty minutes. Not because the monthly automation is fast. Because the daily workflow had already done the work.

---

## 4. The Automation Layer

The automation was built in Alteryx — the low-code tool available at the time, being promoted across the firm as a response to the tech team's reduced capacity during COVID.

The choice of Alteryx was made by availability rather than selection. What mattered was what it enforced: every transformation step visible, every logic traceable, no proprietary scripting required to follow the flow. A colleague without a technical background could read the workflow and understand what it was doing. That property was important — not for the build phase, but for everything that came after it.

The automation is organised into three layers:

**Daily OD workflow** — Ingests the source file, applies cleaning and validation logic, calculates the daily report figures, and syncs to the data model. Runs every working day.

**Data model sync** — Manages the state of the five-layer model: resolving conflicts between daily inputs, maintaining consistency across the customer, contract, note, movement, and guarantee layers, and flagging records that require manual review.

**Monthly / quarterly trigger** — Reads from the accumulated model, applies the relevant regulatory calculation logic for each report type, and generates the submission file and working paper. Runs at the start of each reporting cycle.

The EL layer has no automated workflow for data entry. EL records are entered manually into the model. All downstream automation — validation, monthly trigger, report generation — treats EL data identically to OD data once it is in the model.

---

## 5. Outcomes

| Metric | Result |
|---|---|
| Monthly close time | 30 minutes (from 6 days) |
| Report types automated | 6+ across NFRA and PBOC |
| Data model layers | 5 unified layers |
| Regulators served | NFRA and PBOC |
| IT resources required | 0 |
| Month-end data surprises | 0 |

Beyond the numbers, two things stand out.

**The checker's role changed.** The working paper generated by the monthly trigger is produced from a model that has already been validated daily. The checker is reviewing a clean output, not hunting for errors in a six-day manual process. The review became confirmation rather than discovery.

**Problems started surfacing earlier.** Data quality issues that previously appeared at month-end — because that was the only moment anyone looked — began appearing during daily processing, days or weeks before they could affect the submission. Many were resolved before they became problems at all.

---

## 6. Q&A

**Why map before building anything?**

The instinct when inheriting a broken process is to start fixing. This is usually the wrong instinct. Without a map of what the data represents and how the reporting requirements relate to it, any fix is provisional — it addresses the immediate symptom without understanding the structure.

The mapping exercise produced two things that made everything afterwards faster: a data model that reflected the actual business logic, and a workflow design that followed from it. Both would have been worse if built before the mapping was complete.

**Why build a data model in Excel rather than a database?**

The data volumes didn't require a database. OD and EL are not high-frequency products — the model holds hundreds of records, not millions. Excel provided a structure that the business users who would eventually inherit the system could read, audit, and where necessary amend directly, without IT involvement or specialist tooling.

The goal was not the most technically capable solution. It was the most maintainable one for the people who had to live with it.

**Why accept manual EL entry rather than pushing for automation?**

Automating EL source data would have required IT involvement that wasn't available. The tech department was effectively paralysed by COVID at the time. Blocking the entire project on that dependency would have meant another year of six-day month-end closes.

The architectural response was to treat manual entry as a valid input layer rather than a problem to be solved. EL data entered into the correct model structure is just as usable as automatically processed data. The distinction between how data arrives and how data is structured is important to keep clear.

**Why daily processing rather than weekly?**

OD generates daily activity. Processing it daily means the lag between a transaction occurring and that transaction being validated is one day. Processing weekly means the lag is up to five days, and the weekly processing session inherits a week's worth of accumulated anomalies.

The practical difference is response time. A discrepancy in a day-old transaction is easy to trace. A discrepancy in a transaction that happened four days ago, in a batch with forty others, is not.

**Why not Python instead of Alteryx?**

At the time, Alteryx was the available tool. It was also the right one for this context. The visual workflow format meant that any colleague could follow the logic without programming knowledge. The system needed to be maintainable by people who were finance professionals, not engineers.

This turned out to matter more than anticipated — see [PHILOSOPHY.md](PHILOSOPHY.md).

---

*[Philosophy: Daily Hygiene and What It Revealed](PHILOSOPHY.md)*
