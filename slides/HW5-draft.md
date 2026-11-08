---
marp: true
paginate: true
---

# Building Software That Is Safe to Change
Campus Equipment Loan API · CSC 640

HW1 planning draft — no implementation results yet

---

## The problem
- Borrowers need reliable equipment availability.
- Staff need controlled catalog updates and loan records.
- Keep the scope small; use synthetic data.

---

## Two stages, one contract
- Stage 1: PHP + PDO + MySQL.
- Stage 2: Laravel + Eloquent + Sanctum.
- Preserve ten API contracts and shared acceptance assertions.

---

## Quality needs evidence
- Specify the expected behavior.
- Measure actual behavior with tests.
- Compare results with the criterion and explain limitations.
- Add actual HW4 results before final submission.

---

## Manage complexity
- Process: small deliveries and feedback.
- Design: narrow interfaces and clear responsibilities.
- Automation: tests, Docker, scripts, and documentation deployment.

---

## Planned evidence
- Same assertions against both stages.
- One explained design change and regression results.
- Comparison of benefits and costs.
- Peer feedback and verified fixes.

---

## Sources and assistance
Instructor materials: nkuase/csc640, ASE, ase230, ase285, ase420.
OpenAI Codex helped draft this outline. Student review and project-specific evidence are required.
