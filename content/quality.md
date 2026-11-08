---
title: 'Quality: criteria before claims'
---

Software quality must be specified, measured, and verified. Choosing a framework does not establish quality by itself.

| Attribute | Planned criterion | Planned evidence |
|---|---|---|
| Functional suitability | All ten API contracts behave as specified | Shared acceptance runner, actual responses |
| Data integrity | One active loan per physical item | Concurrent checkout test and database inspection |
| Access control | Borrowers cannot mutate staff catalog or another borrower's loans | Missing/invalid token, forbidden role and ownership tests |
| Maintainability | A policy change has a limited and explainable impact | Before/after diff, affected/unaffected modules, regression results |
| Reproducibility | A reviewer can start and test each stage | Clean-environment setup instructions and recorded verification |

## Process, design, automation

Use small vertical slices to learn early: database, endpoint, browser interaction, and acceptance test for one usable behavior. Use modules and interfaces to hide persistence details and isolate business rules. Use Docker and scripts for repeated setup, and GitHub Actions for documentation publication.

## Planned comparison

Stage 1 makes routing, PDO, validation, and token checks explicit. Stage 2 delegates common mechanisms to Laravel. Record the repetitive work removed, new dependency/setup costs, and whether the same acceptance assertions still pass. Use actual evidence rather than assuming the framework is better.

## Limitations

No API implementation, security audit, load test, peer review, or production deployment has occurred. The examples use synthetic data. Passing future tests will support only the cases and environment tested.

## Sources

- [CSC 640 course materials](https://github.com/nkuase/csc640)
- [ASE 230: PHP, Laravel and automation](https://github.com/nkuase/ase230)
- [ASE 285: software process](https://github.com/nkuase/ase285/tree/main/module1)
- [ASE 420: software design](https://github.com/nkuase/ase420/tree/main/module1)
