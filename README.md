# OD/EL Data Modelling and Regulatory Report Automation

End-to-end data modelling and automation for overdraft and entrusted loan regulatory reporting

---

## The Problem

OD — overdraft facilities — and EL — entrusted loans — are routine product lines. Both carry daily, monthly, and quarterly reporting obligations across two regulators: NFRA and PBOC, each with their own field definitions, granularity requirements, and submission windows.

When I inherited the function, the person who had run it was gone. There was no handover document. No SOP. No data dictionary. The data lived in a collection of manually maintained spreadsheets, and it wasn't always clear what the numbers actually represented.

Every monthly close was a six-day scramble: locating source data, cleaning it by hand, reconciling it against other teams' summaries, and generating submission files through a chain of manual steps that changed slightly every cycle. Errors surfaced late. The checker had little to work with.

The process hadn't been designed. It had been survived.

---

## The Solution

Before touching any tooling, I spent the first weeks mapping the data: what it represented, how OD and EL related to each other, and what the regulatory reporting requirements actually were at each granularity level.

The mapping exercise produced two things: a five-layer data model — corporate customer, loan contract, loan note, loan movement, guarantee contract — and a workflow design that moved data cleaning from month-end to every working day.

OD activity happens daily. The redesigned workflow processed it daily: download the source file, run the Alteryx automation, validate against the data model, sync. By the time the monthly trigger fired, the data had already been cleaned incrementally across thirty working days. The monthly close went from six days to thirty minutes — not because the automation was clever, but because the work was already done.

**Built with:** `Alteryx` `Excel`

→ [Architecture & Design Decisions](ARCHITECTURE.md)

---

## The Methodology

The core principle was deceptively simple: do today's work today.

Regulatory reporting fails at month-end because month-end is where accumulated problems finally have nowhere to hide. The default response — batch everything, fix it all at once, in the same window as the submission deadline — makes the problem structural. Every cycle ends the same way: a race against time, with errors that should have been caught weeks earlier.

The alternative is to stop batching the problem. OD data arrives daily. Process it daily. Validate it daily. By the time the submission window opens, the data is already clean — not because month-end is better managed, but because month-end has almost nothing left to manage.

This is not a tooling insight. It is a workflow design insight. The automation came last, after the data model was stable and the daily process was running cleanly. Automating a dirty process produces faster dirty output. Automating a clean one produces something that runs without attention.

→ [Philosophy: Daily Hygiene and What It Revealed](PHILOSOPHY.md)

---

## Outcomes

| Metric | Result |
|---|---|
| Monthly close time | 30 minutes (from 6 days) |
| Report types automated | 6+ across NFRA and PBOC |
| Data model layers | 5 unified layers |
| Regulators served | NFRA and PBOC |
| IT resources required | 0 |
| Month-end data surprises | 0 |

The checker's workload changed character as well as volume. Instead of hunting for errors in a six-day output, they were reviewing a working paper generated from a model that had already been validated incrementally. The review became confirmation rather than discovery.

---

## What This Project Revealed

The handover on departure exposed a gap that the automation hadn't addressed. Colleagues who inherited the system could run it fluently. When something broke or needed adapting, they couldn't. The system had been built to outlast its creator. It hadn't been built to be understood by the people who followed.

That gap — between a system that runs and a team that can evolve it — became the question that shaped every project that followed. It is the origin of a broader framework for sustainable digitalisation, developed and applied at scale in the project that came next.

→ [China Regulatory Reporting Automation](https://github.com/jonathanzhuxh/CRRA-China-Regulatory-Reporting-Automation)
