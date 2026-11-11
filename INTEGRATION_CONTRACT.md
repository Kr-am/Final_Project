# Integration contract — read before programming

## 0. Why this matters

The three members must not independently invent different variable names, account formats, transaction rules, or procedure parameters. Member 1 owns `src/MAIN.ASM` and the common `.DATA` definitions. Members 2 and 3 own standalone `.INC` procedure files. Everyone follows the contract below. Changes require agreement in the group chat and a single coordinated GitHub update.

## 1. Requirements versus agreed implementation decisions

**Instructor-required (Problem #2):** up to 10 unique accounts and PINs; initial balances; deposit, withdrawal, transfer, balance inquiry, history; 50 transaction records; balance/limit validation; daily withdrawal limit per account; log type/IDs/amount/resulting balance; detection of 3+ suspicious high withdrawals in a short sequence; summaries and rejected counts; failed transfers preserve consistency. Advanced challenge: two-phase transfers, integer centavos, and arithmetic-overflow handling.

**Team-proposed settings (may be adjusted before coding):**

| Setting | Agreed default |
|---|---|
| Account count | Exactly 10 seeded *simulation* accounts, IDs 1001 through 1010 |
| PINs | Unique 4-digit demo PINs; publish test credentials only, never actual PINs |
| Starting balance | 50,000 centavos per account = PHP 500.00 |
| Amount input | Positive whole **centavos** (e.g., 1250 means PHP 12.50) |
| Maximum amount per deposit / withdrawal / transfer | 30,000 centavos = PHP 300.00 |
| Daily cash-withdrawal cap, per account | 60,000 centavos = PHP 600.00 |
| High-withdrawal threshold | Strictly **greater than** 10,000 centavos = PHP 100.00 |
| Fraud rule | 3 consecutive successful account transactions that are each withdrawals over 10,000 centavos. Other successful transactions involving that account reset its streak. Rejected attempts do not change its streak. |
| Fraud alert behavior | Flag account and increment global alert count only when streak first reaches 3; never block a valid transaction solely because it is flagged |
| Day definition | One simulated day per program run; daily totals reset on restart; no clock/date integration |
| Outgoing transfers and daily cash cap | Transfers have per-transaction cap but **do not** count toward the cash-withdrawal daily cap (explicit team choice) |
| Log policy | Store accepted **and** rejected deposit/withdrawal/transfer attempts; balance inquiry is not included as a transaction |
| Full log policy | 50-slot circular buffer: the oldest entry is overwritten at record #51 |
| Numeric bounds | All balances and transaction amounts are 16-bit unsigned words (0..65,535 centavos) |

These recommended amounts intentionally allow straightforward 16-bit 8086 arithmetic. Do not silently change them after Member 2 or Member 3 begins writing procedures.

## 2. Core variables: Member 1 defines once in `.DATA`

Suggested names and representations; syntax below is **illustrative**, not a complete program:

```asm
ACCOUNT_COUNT EQU 10
MAX_LOGS      EQU 50
MAX_AMOUNT    EQU 30000
DAILY_LIMIT   EQU 60000
FRAUD_LIMIT   EQU 10000

AccountIDs       DW 1001,1002,1003,1004,1005,1006,1007,1008,1009,1010
AccountPINs      DW 1111,2222,3333,4444,5555,6666,7777,8888,9999,1234  ; Fake test PINs
Balances         DW 10 DUP(50000)
DailyWithdrawn   DW 10 DUP(0)
FraudStreak      DB 10 DUP(0)
FraudFlags       DB 10 DUP(0)
RejectedCount    DW 0
FraudAlertCount  DW 0

CurrentIndex     DB 0FFh            ; 0..9 after login; FFh means none
TargetIndex      DB 0FFh
InputValue       DW 0
InputValid       DB 0
RequestedAmount  DW 0
TxnType          DB 0               ; 'D', 'W', or 'T'
TxnSourceID      DW 0
TxnDestID        DW 0               ; Zero for deposit/withdrawal
TxnResultBalance DW 0               ; Balance AFTER success, or unchanged after failure
TxnStatus        DB 0               ; 1 accepted, 0 rejected
TxnError         DB 0

LogType          DB 50 DUP(0)
LogSrc           DW 50 DUP(0)
LogDst           DW 50 DUP(0)
LogAmount        DW 50 DUP(0)
LogBalance       DW 50 DUP(0)
LogStatus        DB 50 DUP(0)
LogError         DB 50 DUP(0)
LogCount         DB 0               ; 0..50
LogNext          DB 0               ; 0..49
```

Place explanatory comments in the documentation/source when required by the instructor. These names are an agreed interface; if changed, update this file **before** implementation continues.

**Array indexing:** Account positions are `0..9`, **not** IDs `1001..1010`. A `DW` element is two bytes: `word offset = index * 2` (e.g., using `SHL`). A `DB` element uses `index`. Log record positions are `0..49`. Never mix a byte index with a word byte offset.

## 3. Procedure rules for all three files

- Use `PROC NEAR` and `ENDP` in included files. `MAIN.ASM` alone declares `.MODEL SMALL`, `.STACK`, `.DATA`, `.CODE`, `MAIN PROC`, and `END MAIN`.
- `MAIN.ASM` should `INCLUDE CORE.INC`, `INCLUDE TRANSACTIONS.INC`, and `INCLUDE SECURITY.INC` **after** the main `ENDP` and **before** `END MAIN`, inside the single `.CODE` section. Procedure order can be adjusted if required by assembler resolution.
- Each procedure must leave `DS` and `SS` intact; preserve `BX`, `CX`, `DX`, `SI`, `DI`, and `BP` when used. `AX` and FLAGS may change. Nested calls must not accidentally destroy inputs. Use the stack symmetrically (`PUSH`/`POP`).
- Pass complex inputs and results through the named globals instead of making incompatible register-calling conventions. Document any additional globals.
- DOS text I/O through INT 21h (e.g., AH=09h for `$`-terminated text, AH=02h for a character, AH=0Ah for buffered input) is allowed. Prompt strings printed by AH=09h must end in `$`.
- Do not assume a `CALL` preserves FLAGS. Check results through `InputValid`, `TxnStatus`, or `TxnError`.
- Numeric parsing rejects blank, non-digit, zero amount, and 16-bit overflow; reject amount > 30,000 for every money-changing operation. PIN parsing may accept leading zeroes if the 4-digit test PIN format needs it.
- Avoid memory-to-memory `MOV`, invalid scaled addressing, and direct `MOV` to segment register using an immediate. Check TASM 4.1 syntax, not modern x86 syntax.

## 4. Named procedure interface and responsibility

| Procedure | Owner | Required behavior / globals |
|---|---|---|
| `ReadUnsignedInput` | M1 | Parse decimal digits to `InputValue`; write `InputValid` (1/0), check overflow |
| `PrintUnsigned` | M1 | Display unsigned decimal value supplied in `AX` (or document global alternative) |
| `PrintMoney` | M1 | Display unsigned centavos as PHP pesos with exactly two decimal places |
| `FindAccount` | M1 | Look up supplied account ID in agreed input global; returns index / found flag (add names to contract) |
| `LoginAccount` | M1 | Set `CurrentIndex` only after ID+PIN match; reject missing/wrong ID/PIN |
| `ShowBalance` | M1 | Read balance at `CurrentIndex` and print it |
| `DoDeposit` | M1 | Validate amount, ensure recipient balance won't overflow, commit, set transaction context, then log and fraud-reset |
| `DoWithdrawal` | M2 | Validate amount, available balance and daily total, commit or reject atomically, log and fraud-update |
| `DoTransfer` | M2 | Validate source/destination, distinct accounts, amount, source balance, destination capacity; commit both only if all checks pass |
| `AppendTransaction` | M3 | Save Txn* context to 50-slot circular log and update `LogCount`, `LogNext` |
| `UpdateFraudAfterSuccess` | M3 | Update per-account streak(s) after a successful operation, trigger flag/alert on streak 3 |
| `ShowHistory` | M3 | Show log records chronologically; optionally filter for the logged-in account |
| `ShowAccountSummaries` | M3 | Display account IDs, balances, withdrawal totals, fraud flags, rejected count |

**Before coding**, Member 1 must also define shared lookup globals, e.g. `LookupID DW`, `LookupIndex DB`, `LookupFound DB`; Member 2 and Member 3 need those names. Add them to `.DATA` and this contract. Implement `TxnError` numbers consistently (e.g., 0 none, 1 invalid amount, 2 invalid account, 3 insufficient funds, 4 daily cap exceeded, 5 balance overflow, 6 same-account transfer, 7 invalid authentication). The display of the error text can be owned by M1 or the caller, but numerical meanings must not conflict.

## 5. Transaction lifecycle: fixed sequence

1. Read account / amount input; validate format and range; prepare `TxnType`, `TxnSourceID`, `TxnDestID`, `RequestedAmount`.
2. Validate all business rules without modifying balances; for transfers validate **both accounts and both balance arithmetic outcomes**.
3. On rejection: leave all balances and daily totals unchanged; set `TxnStatus=0`, meaningful `TxnError`, `TxnResultBalance` to unchanged source/current balance, increment `RejectedCount`, then call `AppendTransaction`. Do **not** call fraud updater.
4. On success: update relevant account balances (and daily cash total for withdrawals); set `TxnStatus=1`, `TxnError=0`, resulting balance, then call `AppendTransaction` and `UpdateFraudAfterSuccess`.
5. Report success or rejection clearly. Do not log a successful transfer until **both** balances are committed.

**Failed transfer example:** Source 1001 has PHP 500.00, target 1002 has PHP 600.00. Transferring PHP 100.00 would make target PHP 700.00 = 70,000 centavos, which exceeds 65,535. Reject without changing either balance, then log the rejected attempt.

**Overflow check:** Before `ADD balance, amount`, ensure `balance <= 65535 - amount` (or check carry flag immediately and never retain mutated state on failure). Before `ADD daily, amount`, ensure `daily <= 60000 - amount`. Use unsigned comparisons for balances.

## 6. Fraud state updates

- For a successful withdrawal on index `i`, increment `FraudStreak[i]` if `RequestedAmount > 10000`; otherwise set it to 0.
- For a successful deposit on `i`, set its streak to 0.
- For a successful transfer from `i` to `j`, set both `FraudStreak[i]` and `[j]` to 0.
- Rejected attempts never affect streak.
- On transition from streak 2 to 3: set `FraudFlags[i]=1`, increment `FraudAlertCount`, display suspicious-activity notice. At 4+, keep the flag; do not increment the alert count again for the same streak. A later qualifying new streak may trigger another event if the team chooses; the flag itself remains set for the session.
- Fraud detection is a **flag only**, not a transaction block; do not claim criminal detection accuracy.

## 7. Source integration checklist

- [ ] All shared names agree with contract.
- [ ] Every `PROC` referenced by `CALL` exists exactly once.
- [ ] Every `.INC` compiles when included in the single program.
- [ ] No duplicate `.MODEL`, `.CODE`, `MAIN`, or `END`.
- [ ] No procedure accidentally changes `DS` or the stack balance.
- [ ] IDs versus array indices are never confused.
- [ ] Currency is represented consistently in centavos.
- [ ] Success/rejection logging happens once per attempt.
- [ ] Failure does not partially update account/daily/fraud state.
- [ ] Build and test from a fresh checkout, not only someone's local setup.
