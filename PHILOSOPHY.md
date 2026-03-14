# Philosophy: Daily Hygiene and What It Revealed

---

## Chapter 1 — Do Today's Work Today

This project didn't start with a framework or a methodology. It started with a problem that wouldn't go away.

Every month-end was the same: six days of frantic data work, errors discovered too late to trace properly, submission files assembled under deadline pressure from data that hadn't been properly cleaned. The volume of work wasn't the issue. The timing was.

All of it — the cleaning, the reconciliation, the anomaly resolution — happened at the same moment. The moment when there was least time to do any of it well.

The obvious fix was automation. The instinct in most operations teams when they encounter a painful manual process is to look for a tool that removes the manual work. But I held off. Not because the tooling wasn't there — Alteryx was available and I was learning it — but because I wasn't yet sure what I was automating. The data was undocumented. The process had never been written down. Automating an undocumented process produces a faster undocumented process. That wasn't an improvement.

So before I touched a tool, I mapped the data.

### What mapping revealed

OD and EL looked like two separate reporting problems because they had always been handled as two separate reporting problems. In practice, they shared the same underlying structure. Both involved customers, contracts, loan notes, movements, and guarantees. Both reported to the same two regulators. Both required the same granularity dimensions.

That insight — that the apparent complexity was the product of accumulated handling, not inherent in the data — changed the design completely. Instead of two parallel workflows, a single five-layer data model. Instead of separate reconciliation logic for each product line, a shared validation framework. Instead of separate month-end processes for NFRA and PBOC, a unified trigger that served both.

The map made the model possible. The model made the automation straightforward.

### The temporal redesign

With the model in place, the second insight followed from the first.

OD generates activity every working day. The old approach batched it all to month-end because that was when it needed to be reported. But that logic confuses the reporting cycle with the cleaning cycle. Reports are submitted monthly. Problems should be resolved daily.

The daily workflow was built on that principle: every working day, download the OD source file, process it, validate it, sync it to the model. Any anomaly surfaces the same day it occurs — when the transaction is fresh, when tracing it back is easy, when there is no deadline bearing down.

By the time the monthly trigger fires, there is almost nothing left to do. The data has been cleaned incrementally across thirty working days. The submission file is generated in minutes from a model that has already been validated. The checker reviews a working paper rather than hunting for errors in raw output.

**The lesson is not about automation. It is about when work gets done.**

Month-end compression is not a natural consequence of monthly reporting cycles. It is the consequence of deferring daily work until the last possible moment. Move the work upstream — to the day it arises — and month-end stops being a crisis. It becomes a formality.

This is the principle I think of as daily hygiene. Not a dramatic intervention. Not a technology transformation. Just the discipline of not letting today's work become next month's problem.

---

## Chapter 2 — What the Handover Revealed

The system worked. The daily workflow ran. The monthly close took thirty minutes. The checker's job became easier.

When I left, I handed it over.

What the handover exposed was a gap I hadn't designed for. Colleagues who inherited the system could run it. They understood the daily steps. They knew how to trigger the monthly automation. What they couldn't do was change it. When an upstream data format changed, when a reporting dimension was added, when something broke in a way that wasn't obvious — they were stuck. The system that had been built to outlast its creator hadn't been built to be understood by the people who followed.

This wasn't their failure. It was a design failure. I had built a system that was operationally robust but intellectually opaque — not through any deliberate choice, but because I had focused on making it work and hadn't thought enough about making it legible.

### The gap between running and understanding

There is a meaningful difference between a system that can be operated and a system that can be owned.

Operation is following a procedure. Ownership is understanding the logic well enough to adapt when the procedure no longer fits. A system that can only be operated creates a dependency on whoever understands it deeply. A system that can be owned is genuinely transferable.

The handover taught me that transferability is not automatic. It requires deliberate design: documentation that explains the why, not just the what; tooling choices that non-specialists can read; training that targets ownership, not just operation.

Automation without capability transfer is not sustainable digitalisation. It is a dependency with a lower headcount.

### What came next

That gap — between a system that runs and a team that can evolve it — became the central design constraint for every project that followed.

The next project was built from the start with that constraint in mind: not just a robust architecture, but a visible one; not just trained operators, but colleagues who understood enough to push back when requirements didn't make sense; not just documentation for record-keeping, but documentation for handover.

The result was a system that outlived its creator, then outlived its platform, and left behind a methodology that spread to other teams without being mandated.

That is the project this one was the prequel to.

→ [China Regulatory Reporting Automation](https://github.com/jonathanzhuxh/CRRA-China-Regulatory-Reporting-Automation)

---

*[Back to Architecture & Design Decisions](ARCHITECTURE.md)*
