# Banking Transaction Processing and Fraud Detection Simulator

**Assembly Language Programming final project — Problem #2**  
**Target:** Intel 8086, 16-bit DOS Assembly / TASM 4.1 (GUI Turbo Assembler 5.1)

A classroom simulation that supports 10 simulated accounts, deposits, withdrawals, transfers, balance inquiries, up to 50 transaction records, account summaries, transaction validation, a daily withdrawal cap, and suspicious-withdrawal alerts.

> This is **not** a real banking or security product. Account IDs and PINs in source code must be fictitious test data only.

## Start here

1. **[ROADMAP.md](ROADMAP.md)** — one-day schedule, checkpoints, features, acceptance criteria, and GitHub checklist.
2. **[docs/INTEGRATION_CONTRACT.md](docs/INTEGRATION_CONTRACT.md)** — agreed data structures, limits, procedure interfaces, account/index rules, and transfer safety.
3. **[Member 1 instructions](docs/roles/MEMBER_1_LEAD_PROGRAMMER.txt)** — menu, accounts, login, deposits, balance inquiry, integration, Chapter 1.
4. **[Member 2 instructions](docs/roles/MEMBER_2_TRANSACTIONS.txt)** — withdrawals, transfers, validation, Chapter 2 and technical sections.
5. **[Member 3 instructions](docs/roles/MEMBER_3_FRAUD_QA.txt)** — logging, fraud detection, summaries, tests, Chapters 4–5.

## Planned source layout (to be created by the team)

```text
src/
  MAIN.ASM              <- Member 1 owns and integrates
  CORE.INC              <- Member 1: input/output, authentication, deposit
  TRANSACTIONS.INC      <- Member 2: withdrawal, transfer, transaction validation
  SECURITY.INC          <- Member 3: log, fraud alerts, history, summaries
submission/
  final-documentation.pdf
  final-source/         <- exact compiled source used for screenshots
  evidence/             <- screenshots and test evidence
```

Only the guide files and roadmap are included initially. **The `.ASM`, `.INC`, and documentation files are implementation deliverables, not prewritten programs.**

## Build plan

Using TASM from the `src` directory (or its GUI equivalent):

```dos
TASM MAIN.ASM
TLINK MAIN.OBJ
MAIN.EXE
```

`MAIN.ASM` includes the three `.INC` files inside its `.CODE` section. They must not contain a second `.MODEL`, `.DATA`, `.CODE`, `MAIN`, or `END MAIN`. Match source paths and case to your local setup. Compile evidence belongs in `submission/evidence/`.

## Required reference materials

- Instructor's **Final Project options** document: Problem #2.
- Instructor's **CSO Final Project Documentation Format** PDF: front matter, Chapters 1–5, references, appendices, and >=10 tests.

The exact classroom specifications take priority over all recommended implementation choices in these guides. Proposed numeric limits, fraud policy, and single-session "day" behavior are team design decisions, not values specified by the instructor.
