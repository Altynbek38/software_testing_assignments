# Money Transfer Feature: Test Documentation Pack

## 1. Test Plan Section

**Scope IN:**
* Functional testing of the money transfer feature.
* Validation of amount limits (100 – 500,000 KZT) and whole integer formatting.
* Validation of the daily limit logic (1,000,000 KZT).
* State transition and boundary testing of the SMS verification flow (timer, attempts, thresholds).

**Scope OUT:**
* UI/UX layout and cross-browser visual rendering.
* Backend transaction processing and database performance (load testing).
* Integration with real telecom providers for SMS delivery (we will use an SMS gateway emulator).

**Approach:**
* Black-box testing using Equivalence Partitioning (EP), 3-value Boundary Value Analysis (BVA), Decision Tables, and State Transition models. 
* Manual execution by QA engineers.

**Entry Criteria:**
* Build is deployed to the Test Environment.
* Smoke tests for the login and basic transfer navigation pass.
* Test data is provisioned: Accounts created with specific balances (e.g., 2,000,000 KZT) and specific daily limit histories (e.g., 0 KZT, 800,000 KZT, 900,000 KZT used).

**Exit Criteria:**
* All 12 planned test cases executed.
* 0 open Critical or High severity defects.
* Traceability matrix shows 100% coverage of agreed requirements.
* Residual risks (if any) are documented and accepted by the Product Owner in writing.

**Top 3 Product Risks:**
1. **Double-debiting (Race Condition):** The system processes the transfer twice if the user taps "Confirm" multiple times during SMS network lag.
2. **Timer Bypass:** The system silently accepts an expired SMS code if the client-side timer goes out of sync with the server.
3. **Boundary Rejections:** Off-by-one errors leading to the rejection of perfectly valid transfers (e.g., exactly 100 KZT or exactly 500,000 KZT).

---

## 2. Test Cases (12 Cases in Full Format)

*Precondition Base: Test customer TC-014 has a balance of 2,000,000 KZT.*

| ID | Traces to | Preconditions | Steps to reproduce | Expected Result | Postconditions |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **TC-01** | REQ-1, REQ-3 | Logged in. Daily limit used: 0 KZT. | 1. Open Transfers.<br>2. Enter amount: 100<br>3. Tap Continue. | Transfer executes immediately without SMS. Balance reduces by 100. | Transfer successful. |
| **TC-02** | REQ-1 | Logged in. Daily limit used: 0 KZT. | 1. Open Transfers.<br>2. Enter amount: 500000<br>3. Tap Continue. | SMS code screen is shown. | Transfer pending confirmation. |
| **TC-03** | REQ-1 | Logged in. Daily limit used: 0 KZT. | 1. Open Transfers.<br>2. Enter amount: 99<br>3. Tap Continue. | Error: "Amount must be at least 100 KZT". | Balance unchanged. |
| **TC-04** | REQ-1 | Logged in. Daily limit used: 0 KZT. | 1. Open Transfers.<br>2. Enter amount: 500001<br>3. Tap Continue. | Error: "Amount exceeds maximum 500,000 KZT". | Balance unchanged. |
| **TC-05** | REQ-1 | Logged in. Daily limit used: 0 KZT. | 1. Open Transfers.<br>2. Enter amount: 100.50<br>3. Tap Continue. | Error: "Whole tenge only". | Balance unchanged. |
| **TC-06** | REQ-2 | Logged in. Daily limit used: 900,000 KZT. | 1. Open Transfers.<br>2. Enter amount: 100000<br>3. Tap Continue. | Transfer executes immediately without SMS. | Limit used becomes 1,000,000. |
| **TC-07** | REQ-2 | Logged in. Daily limit used: 800,000 KZT. | 1. Open Transfers.<br>2. Enter amount: 300000<br>3. Tap Continue. | Error: "Daily limit of 1,000,000 KZT exceeded". | Balance unchanged. |
| **TC-08** | REQ-5 | Logged in. Daily limit used: 900,000 KZT. | 1. Open Transfers.<br>2. Enter amount: 600000<br>3. Tap Continue. | Error: "Amount exceeds maximum 500,000 KZT". (Limit error is suppressed). | Balance unchanged. |
| **TC-09** | REQ-3, REQ-4 | Logged in. Daily limit used: 0 KZT. | 1. Open Transfers.<br>2. Enter amount: 150000<br>3. Tap Continue.<br>4. Enter correct SMS code within 120s. | Transfer executes successfully. | Balance reduces by 150,000. |
| **TC-10** | REQ-4 | Logged in. Daily limit used: 0 KZT. | 1. Open Transfers.<br>2. Enter amount: 150000<br>3. Tap Continue.<br>4. Enter wrong SMS code 3 times. | Error: "Transfer blocked due to multiple invalid attempts". | Transfer canceled, balance unchanged. |
| **TC-11** | REQ-4 | Logged in. Daily limit used: 0 KZT. | 1. Open Transfers.<br>2. Enter amount: 150000<br>3. Tap Continue.<br>4. Wait 121 seconds.<br>5. Enter correct SMS code. | Error: "Code expired". Transfer is rejected. | Balance unchanged. |
| **TC-12** | REQ-4 | Logged in. Daily limit used: 0 KZT. | 1. Open Transfers.<br>2. Enter amount: 150000<br>3. Tap Continue.<br>4. Enter wrong SMS code.<br>5. Enter correct SMS code. | Transfer executes successfully. | Balance reduces by 150,000. |

---

## 3. Traceability Matrix

| Requirement ID | Requirement Description | Test Cases | Status |
| :--- | :--- | :--- | :--- |
| **REQ-1** | Amount 100 to 500,000 KZT, whole tenge | TC-01, TC-02, TC-03, TC-04, TC-05 | Pending Execution |
| **REQ-2** | Daily limit 1,000,000 KZT | TC-06, TC-07 | Pending Execution |
| **REQ-3** | Transfer > 100,000 KZT requires SMS | TC-01, TC-09 | Pending Execution |
| **REQ-4** | Code valid for 120s, 3 wrong attempts cancel | TC-09, TC-10, TC-11, TC-12 | Pending Execution |
| **REQ-5** | Amount error overrides limit error | TC-08 | Pending Execution |
| **REQ-6** | *Insufficient balance behavior* | **—** | **NOT COVERED**<br>*(Reason: The brief does not state what happens if amount and limits are valid, but account balance is insufficient. Escalated to Analyst).* |

---

## 4. Release Checklist: SMS Code Flow

[ ] SMS arrives on the device within 30 seconds.
[ ] Screen accepts exactly 6 digits.
[ ] "Confirm" button is disabled if less than 6 digits are entered.
[ ] Entering a correct code executes the transfer.
[ ] Code expires exactly after 120 seconds.
[ ] Entering a correct code *after* 120 seconds is rejected.
[ ] 3rd incorrect code completely cancels the transfer flow.
[ ] State resets: A new transfer after a cancellation gives 3 fresh attempts.
[ ] Double-tapping "Confirm" with a correct code debits the account only once.
[ ] SMS text is correctly localized in KZ, RU, and EN.

---

## 5. Defect Reports

### Defect 1: Off-by-one boundary rejection
**Title:** Transfers: 500,000 KZT amount is incorrectly rejected as exceeding the limit.
**Environment:** iOS 18.2, App version 4.11.0, Test Env 2.
**Preconditions:** Test customer TC-014, balance 2,000,000 KZT, daily total 0.
**Steps to reproduce:**
1. Open Transfers $\rightarrow$ Between my accounts.
2. Enter amount: 500,000.
3. Tap Continue.
**Expected result:** SMS code screen is shown (500,000 is a valid upper boundary).
**Actual result:** Error shown: "Amount exceeds maximum 500,000 KZT".
**Reproducibility:** 5 of 5 attempts.
**Severity:** High (Legitimate transfers at the max limit are blocked).
**Priority:** Medium.
**Evidence:** Screenshot `amount_500k_error.png`, logs ID `req-77812`.
**Traces to:** REQ-1 / TC-02.

### Defect 2: Missing requirement implementation
**Title:** Transfers: Silent failure with no error message when account balance is insufficient.
**Environment:** Android 14, App version 4.11.0, Test Env 2.
**Preconditions:** Test customer TC-015, balance 50,000 KZT, daily total 0.
**Steps to reproduce:**
1. Open Transfers $\rightarrow$ Between my accounts.
2. Enter amount: 100,001.
3. Tap Continue.
4. Enter correct SMS code.
**Expected result:** Missing from brief, but realistically an "Insufficient balance" error should appear.
**Actual result:** Screen loads indefinitely, no error is shown, transfer remains in "Processing" state.
**Reproducibility:** 3 of 3 attempts.
**Severity:** High (Core flow hangs, user is left confused).
**Priority:** High.
**Evidence:** Screen recording `infinite_load_balance.mp4`.
**Traces to:** REQ-6 (Not Covered).

### Defect 3: Critical State Transition failure (Double Debit)
**Title:** Transfers: Account is debited twice if "Confirm" is tapped rapidly on the SMS screen.
**Environment:** Web App (Chrome 120), Test Env 2.
**Preconditions:** Test customer TC-014, balance 1,000,000 KZT, daily total 0. Network throttled to "Slow 3G".
**Steps to reproduce:**
1. Open Transfers $\rightarrow$ Between my accounts.
2. Enter amount: 200,000.
3. Tap Continue.
4. Enter correct SMS code.
5. Tap "Confirm" rapidly 3 times while the network is loading.
**Expected result:** Transfer succeeds once, balance reduces to 800,000 KZT (API is idempotent).
**Actual result:** Transfer executes twice, balance reduces to 600,000 KZT.
**Reproducibility:** 4 of 5 attempts under throttled network.
**Severity:** Critical (Loss of customer money).
**Priority:** High (Fix Now - Stop Release).
**Evidence:** Video `double_debit.mp4`, transaction IDs `tx_881`, `tx_882`.
**Traces to:** REQ-4 / SMS Code State Table (Confirmed + Correct Code transition).

---

## 6. AI Appendix (Level 1)

**Prompts Used:**
> "I need to create a test documentation pack based on a money transfer brief. Generate 12 test cases, a traceability matrix, a 10-item checklist, and 3 defect reports. Use only facts from the brief. Do not invent business logic."

**Raw Output (What the AI generated initially):**
*   The AI generated the 12 test cases but grouped invalid conditions together (e.g., *Test Case 4: Enter amount 600,000 with letters "abc"*).
*   The AI wrote a defect report stating: *"Severity: Critical. Priority: Critical. The app crashes when 500,000 is entered."*
*   The AI created an exit criterion: *"All bugs are fixed."*

**What I Changed and Why:**
1.  **Test Cases:** I manually separated the invalid conditions (One invalid partition per test) to prevent *error masking*, enforcing the rule learned in Week 3. I updated the preconditions to include exact starting balances (Self-contained property).
2.  **Defect Reports:** I deleted the AI's hallucinated "app crash" and replaced it with realistic defects derived from our Week 3 State Transition table (the Double Debit bug) and Boundary Value Analysis (the 500,000 off-by-one bug). I also ensured Severity and Priority were split into separate concepts as per Week 4 rules.
3.  **Exit Criteria:** I rewrote the AI's naive "All bugs fixed" to the honest version taught in class: *"0 open Critical/High defects; residual risks accepted by the PO."*
