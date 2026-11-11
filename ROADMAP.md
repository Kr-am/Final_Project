# One-day project roadmap and master checklist

**Banking Transaction Processing and Fraud Detection Simulator — Problem #2**  
**Team:** 3 members | **Suggested plan:** 12 focused development hours + 2 hours contingency  
**Target platform:** Intel 8086, 16-bit DOS, TASM 4.1 / GUI Turbo Assembler 5.1

> All checklist boxes are initially unchecked. Update them when work is actually completed. The documentation/specification is a *classroom simulation*, not an actual banking solution.

## 1. Team roles and ownership

| Member | Code owner | Documentation owner | Integration/test responsibilities |
|---|---|---|---|
| Member 1 — Lead/integrator | `src/MAIN.ASM`, `src/CORE.INC` | Front matter, Chapter 1, Chapter 3.1–3.2 | Own final merge; T01–T04 |
| Member 2 — Transactions | `src/TRANSACTIONS.INC` | Chapter 2, Chapter 3.3–3.6 (coordinate others' code explanations) | Transfers/limits; T05–T08 |
| Member 3 — Fraud/QA | `src/SECURITY.INC` | Chapter 3.7, Chapter 4, Chapter 5, references and appendices coordination | Logs/fraud/results; T09–T12 |

**Read next:** [`docs/INTEGRATION_CONTRACT.md`](docs/INTEGRATION_CONTRACT.md). Individual instructions are in [`docs/roles/`](docs/roles/).

## 2. Non-negotiable instructor requirements (Problem #2)

- [ ] Manage up to **10 simulated accounts** with unique IDs and PINs.
- [ ] Initialize each account with a specified balance.
- [ ] Support deposits, withdrawals, transfers, balance inquiries, and transaction history.
- [ ] Store up to **50 transaction records**.
- [ ] Reject operations that exceed available balances or configured limits.
- [ ] Enforce a **daily withdrawal limit per account**.
- [ ] Log transaction type, account ID(s), amount, and resulting balance.
- [ ] Flag suspicious patterns of **3 or more above-threshold withdrawals** in a short transaction sequence.
- [ ] Display account summaries and rejected transaction counts.
- [ ] Keep transfers consistent on failure (no partial balance changes).
- [ ] Attempt the advanced challenge: **two-phase transfer**, integer centavos, and overflow checks.

## 3. Proposed implementation choices — freeze in Hour 1

These are **team decisions**, NOT numeric limits specified by the project sheet. Change only by group agreement before coding, then update `docs/INTEGRATION_CONTRACT.md` and all role guides.

| Setting | Proposed setting | Reason |
|---|---|---|
| Test accounts | IDs 1001–1010 with distinct fake PINs | Easy to validate IDs and demo login |
| Initial balance | 50,000 centavos (PHP 500.00) | Fits unsigned 16-bit amount |
| User amount input | Integer centavos (e.g., 11000 means PHP 110.00) | No floating-point arithmetic |
| Per-transaction amount limit | 30,000 centavos | Testable positive amount cap |
| Daily cash-withdrawal limit | 60,000 centavos per account | Distinct from account balance constraint |
| Fraud threshold | Strictly above 10,000 centavos | Clear threshold boundary |
| Fraud detection | 3 successive **successful account transactions** each a withdrawal >10,000; successful deposit/transfer resets account streak | Deterministic short-sequence rule |
| Suspicious action | Flag + alert; do not automatically reject valid withdrawals | Simulates review, not proof |
| Transfer daily limit | Cash withdrawal limit does not include transfers | Explicit design choice to avoid ambiguity |
| Log capacity | 50 records; circular overwrite oldest | Bounded memory |
| Session day | One program run; counters reset on restart | No real clock needed |
| Balance representation | Unsigned 16-bit centavos 0–65,535 | 8086-friendly with checks |

- [ ] Members 1, 2, 3 approve values, exact fraud definition, and log policies.
- [ ] Declare shared arrays and procedure interface in `MAIN.ASM`.
- [ ] Confirm error-code list and register-preservation agreement.
- [ ] Pin this contract / link to it in the group chat.

## 4. Day plan and checkpoint gates

| Hours (elapsed) | Member 1 | Member 2 | Member 3 | Exit criterion |
|---|---|---|---|---|
| 0–1 | Repo skeleton, shared variables, input contract | Review validations + transfer pseudocode | Confirm log fields + fraud rule | Shared interface frozen |
| 1–2 | Login, menu, parsing | Withdrawal and limit logic | 50-record log writer | Procedures defined |
| 2–3 | Deposit and money printer | Two-phase transfer | Fraud streak and flags | Basic functional modules |
| 3–4 | Core functional smoke tests | Edge-case checks | History, summaries | Each module handed over |
| 4–5 | Include modules; first compile | Fix integration errors in own file | Fix integration errors in own file | TASM compiles |
| 5–6 | End-to-end demo | Transfer/limit verification | Log and fraud verification | Full menu works |
| 6–8 | T01–T04 plus Chapter 1 draft | T05–T08 plus Chapter 2/3 draft | T09–T12 plus screenshot collection | 12 actual tests recorded |
| 8–10 | Front matter/Ch.1/Ch.3.1–3.2 | Ch.2/Ch.3.3–3.6; pseudocode | Ch.3.7/Ch.4/Ch.5/appendices | Complete first report |
| 10–12 | Clean build + final merge | Cross-check code/explanations | Final screenshots + test table | Ready for submission/demo |
| 12–14 (buffer) | Fix critical remaining blockers | Assist fixes | Re-run failing tests | Contingency, not assumed complete |

### Gate A — shared interface (end of Hour 1)
- [ ] Repo folders created; GitHub write access verified for all 3.
- [ ] Shared `DW`/`DB` arrays and type names committed.
- [ ] Each procedure owner knows inputs, outputs, and globals.
- [ ] One person owns final `MAIN.ASM` to avoid merge conflicts.

### Gate B — independent working modules (end of Hour 4)
- [ ] `CORE.INC`: valid login, menu, deposit, balance, common I/O.
- [ ] `TRANSACTIONS.INC`: withdrawal + atomic transfer.
- [ ] `SECURITY.INC`: log writer + fraud check + display.
- [ ] No modules redefine `.MODEL`, `.DATA`, `.CODE`, `MAIN`, or `END`.
- [ ] All source pieces committed and available to Member 1.

### Gate C — integrated binary (end of Hour 6)
- [ ] Compile with **TASM 4.1**, not a different x86 assembler.
- [ ] Successful assembly and link; executable launches and returns to menu.
- [ ] No unresolved CALL, duplicate symbols, invalid operands, or stack corruption.
- [ ] All monetary operations call log exactly once per attempt.
- [ ] Rejected transfer leaves source and target unchanged.

### Gate D — verified deliverables (end of Hour 12)
- [ ] 12 executed test cases recorded; failures and fixes documented honestly.
- [ ] Instructor-format Chapters 1–5, front matter, references, appendices ready.
- [ ] Real build/run/screenshots captured; source matches screenshot version.
- [ ] Each person can explain their own code and how modules communicate.
- [ ] Final integrated source and report backed up; README updated.

## 5. Build order and functionality checklist

### Foundation — Member 1
- [ ] Initialize `DS` and program state.
- [ ] Create 10 fake unique accounts/PINs and initial balances.
- [ ] Validate login and reject wrong ID/PIN.
- [ ] Main menu with deposit, withdraw, transfer, inquiry, history, summary, logout, exit.
- [ ] Safe decimal input parsing with overflow detection.
- [ ] Currency printer shows `PHP 12.50` for 1250 centavos.
- [ ] Balance inquiry and deposit with capacity validation.

### Transactions — Member 2
- [ ] Withdrawal checks valid amount, funds, daily cap before updating.
- [ ] Correct daily withdrawn total increments only on success.
- [ ] Transfer validates IDs, distinct accounts, funds, amount, target capacity.
- [ ] Transfer computes both new balances before writing either.
- [ ] Failed transaction changes no bank, daily, or fraud state.

### Logging / fraud / reporting — Member 3
- [ ] 50 record arrays store type, source/dest IDs, amount, resulting balance, status.
- [ ] Both accepted and rejected monetary operations are logged.
- [ ] Record 51 overwrites the oldest; display remains chronological.
- [ ] Fraud flag triggers after 3 qualifying account transactions.
- [ ] Legitimate but suspicious operation is flagged, not automatically blocked.
- [ ] Rejected count is incremented exactly once and displayed.
- [ ] Account summaries reflect current balances, daily totals, flags.

## 6. Test matrix (12 tests; 10 minimum required by documentation)

Do NOT mark a test passed until the **actual output** has been observed and recorded. Start from a documented account state or restart the simulator to reset session state. Record exact money amounts in centavos and PHP.

| Case | Owner | Setup and action | Expected result | Status |
|---|---|---|---|---|
| T01 | M1 | Valid account ID + matching PIN | Login accepted | ☐ |
| T02 | M1 | Correct ID, incorrect PIN | Access rejected; session unchanged | ☐ |
| T03 | M1 | Balance 50,000; deposit 1,000 | Balance 51,000; success logged | ☐ |
| T04 | M1 | Balance 50,000; deposit 30,000 | Overflow rejection; balance stays 50,000 | ☐ |
| T05 | M2 | Balance 50,000; withdraw 1,000 | Balance 49,000; daily total +1,000 | ☐ |
| T06 | M2 | Reduce balance below 30,000, then withdraw more than balance but <=30,000 | Insufficient funds; no state mutation | ☐ |
| T07 | M2 | Accumulate withdrawals to daily total 60,000; replenish funds as needed, then withdraw 1 | Rejected for daily cap; daily total unchanged | ☐ |
| T08 | M2 | Two accounts with sufficient source funds and target capacity; transfer 1,000 | Source -1,000; target +1,000; one success record | ☐ |
| T09 | M3 | Source 50,000, target 60,000; attempt transfer 10,000 | Target overflow rejection; BOTH unchanged | ☐ |
| T10 | M3 | Same account; 3 successive withdrawals of 11,000; balance/day cap permit | Fraud alert on third successful withdrawal | ☐ |
| T11 | M3 | Generate 51 logged attempts | Count remains 50; oldest overwritten | ☐ |
| T12 | M3 | Make multiple known rejected attempts, inspect summary | Rejected count matches attempts exactly | ☐ |

**Important setup for T09:** With the team defaults, 60,000 is a reachable target balance after an allowed 10,000-centavo deposit. **T10:** perform any replenishment before starting the three-withdrawal streak because a successful deposit resets it. **T07:** a 60,000 daily cap can require replenishment to reach; document the setup steps.

Per test capture: `No | Initial state/setup | Input | Expected output | Actual output | Pass/Fail | Evidence file | Tester`. Member 3 merges the real results.

## 7. Documentation roadmap (from instructor's supplied CSO format)

### Front matter — Member 1
- [ ] Title page: school/logo, department, subject, project title, members, professor, school year/semester, date.
- [ ] Approval sheet.
- [ ] Table of contents.
- [ ] List of figures and tables.

### Chapter 1 — Introduction — Member 1
- [ ] 1.1 Background of the Project.
- [ ] 1.2 Project Description.
- [ ] 1.3 Statement of the Problem.
- [ ] 1.4 General and Specific Objectives.
- [ ] 1.5 Scope and Limitations.
- [ ] 1.6 Significance of the Project.

### Chapter 2 — System Analysis and Design — Member 2
- [ ] 2.1 System Overview.
- [ ] 2.2 Software and Hardware Requirements.
- [ ] 2.3 System Architecture (diagram).
- [ ] 2.4 System Flowchart.
- [ ] 2.5 Algorithm / Pseudocode.
- [ ] 2.6 Data Structure and Memory Organization.

### Chapter 3 — Assembly Implementation — shared; Member 2 coordinates technical text
- [ ] 3.1 Development Environment (Member 1).
- [ ] 3.2 Program Structure (Member 1).
- [ ] 3.3 Register Utilization (Member 2; verify actual source).
- [ ] 3.4 Assembly Instructions Used (Member 2; actual instructions only).
- [ ] 3.5 Procedures / Subroutines (everyone supplies their procedures).
- [ ] 3.6 Source Code (complete commented source + key explanations).
- [ ] 3.7 Program Execution (Member 3, real screenshots).

### Chapter 4 — Testing and Results — Member 3
- [ ] 4.1 Testing Procedures.
- [ ] 4.2 At least 10 executed test cases (team target: 12).
- [ ] 4.3 Results and Analysis.
- [ ] 4.4 Errors and Troubleshooting.
- [ ] 4.5 Final Project Output/screenshots.

### Chapter 5 — Conclusion and Recommendations — Member 3
- [ ] 5.1 Conclusion.
- [ ] 5.2 Recommendations.

### References and appendices — all, assembled by Member 3
- [ ] References in APA 7th edition, from sources actually used.
- [ ] Appendix A — Complete Assembly source.
- [ ] Appendix B — C/C++ supporting code **if applicable**; otherwise N/A.
- [ ] Appendix C — Flowchart and pseudocode.
- [ ] Appendix D — Circuit diagram **if applicable**; software-only = N/A.
- [ ] Appendix E — Additional screenshots.
- [ ] Appendix F — Test results.
- [ ] Appendix G — Actual project photographs; ask instructor if required for software-only simulation.
- [ ] Appendix H — Individual contribution report (all 3 members).
- [ ] Verify every figure/table cited in the List of Figures and Tables exists.
- [ ] Cross-check page numbers, section numbering, spelling, explanations.

**Formatting observation:** The instructor template headings show Chapter 4 sections numbered 5.x and Chapter 5 sections numbered 6.x. This roadmap uses consistent 4.x/5.x; confirm with instructor if strict original numbering is mandated.

## 8. GitHub collaboration checklist

- [ ] Member 1 creates repo and gives Members 2 and 3 access.
- [ ] One shared contract committed before parallel code starts.
- [ ] Branches: `feature/member1-core`, `feature/member2-transactions`, `feature/member3-security-qa`.
- [ ] Each member edits only their owned module; documentation can be drafted separately.
- [ ] Commit meaningful milestones instead of one giant final upload.
- [ ] Avoid committing `.OBJ`, `*.BAK`, editor temp files, debug dumps, or real PINs.
- [ ] Every PR/handoff includes changed files, procedure names, test performed, known issues.
- [ ] Member 1 integrates module changes and checks for conflicting variable names.
- [ ] After merge, build directly from `main` and re-run all 12 tests.
- [ ] Record the final Git commit/tag used for submitted screenshots.

Sample commit messages:

```text
feat(core): implement menu login and account arrays
feat(core): add deposit and centavo display
feat(transactions): validate withdrawals and daily cap
feat(transactions): implement atomic two-phase transfers
feat(security): add 50-record circular history
feat(security): flag suspicious withdrawal sequences
fix(integration): preserve registers across module calls
docs: add 12 verified tests and screenshots
```

## 9. Fast triage when something fails

1. **Assembler error** — identify offending file, line, invalid instruction/addressing mode, missing label, duplicate symbol, or unbalanced PROC/ENDP.
2. **Link error** — confirm `MAIN.ASM` includes all modules and there is one `END MAIN`.
3. **Wrong balance** — inspect amount units and `DW` index offset (2*index); verify no early commit.
4. **Wrong log** — ensure one append per attempt and proper ring-buffer pointer advancement.
5. **Fraud doesn't trigger** — check threshold uses `>` not `>=`, deposits/transfers reset streak, and there were 3 successful qualifying withdrawals.
6. **Daily limit oddities** — verify daily total is tracked separately per account and only successful cash withdrawals increase it.
7. **Program crashes** — inspect push/pop count, register preservation, DS setup, and input buffer bounds.
8. **Documentation mismatch** — trust the final tested source, correct the prose and rerun verification.

## 10. Presentation responsibilities

- **Member 1:** Purpose, architecture, accounts/PIN, menu, deposit and inquiry.
- **Member 2:** Withdrawals, daily limit, transfer two-phase rule, failed transfer demo.
- **Member 3:** History, fraud alert, account summaries, rejected totals, test results.

Each member should be prepared to explain the 8086 register and array choices and the exact `CALL`/`RET` relationships, even outside their own procedures.

## 11. Submission gate

- [ ] The program **actually assembles and runs** on the stated TASM environment.
- [ ] One final integrated, tested source code set is included.
- [ ] Requirements 1–10 from Problem #2 are satisfied or documented honestly if not.
- [ ] Advanced challenge is implemented or clearly marked incomplete.
- [ ] Report follows instructor's front matter, five chapters, references, appendices.
- [ ] 12 test cases include observed results (not assumed PASS entries).
- [ ] Compile/run/input/output/debugging screenshot evidence is included.
- [ ] README, roadmap, contract, and role assignments reflect the final implementation.
- [ ] Three contributions are distinct and traceable in commits + Appendix H.
- [ ] Backups exist; everyone can access the final commit and demonstration build.

**Rubric emphasis:** Chapter 3 Implementation = 35/100; Chapter 2 = 15; Chapter 4 = 15; Chapter 1 = 10; Chapter 5 = 10; references/appendices = 5; organization/format/presentation = 10. Prioritize a working, explained Assembly implementation over decoration.
