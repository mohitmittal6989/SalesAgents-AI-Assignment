# Approach Note: Home Credit LAP Voice Agent

## 1. Problem in one line
A voice agent that verifies the customer, pitches a pre-approved Loan Against Property offer (up to ₹75 lakh), captures 7 eligibility data points, disqualifies instantly on a failed criterion, routes existing-loan/EMI-reduction cases to a transfer specialist, and hands qualified leads to a senior loan expert, all while sounding like an advisor and not a questionnaire.

## 2. How I structured the solution
The prompt implements five layers in plain language:

| Layer | Where in the prompt |
|---|---|
| Conversation | Stages A to D (greet and verify, offer, checklist, handoff), voice-style rules |
| Business logic | Eligibility table, disqualification (§8), loan-limit flow (§8.4), transfer switch (§9) |
| State / memory | Internal qualification state (§11A), rebuilt every turn from the conversation history |
| Diversions | Question, objection and interruption handling (§10), RAG rules (§11B) |
| Final decision | Handoff gate (§6), call-termination rules (§11C) |

## 3. Core algorithm
**Extract first, validate second, update state third, ask the next missing question fourth.**
Every reply: pull every fact from the utterance (including out-of-order ones) → check disqualifiers → check transfer triggers → handle any customer question → ask only the first still-unknown checklist item. The handoff is allowed only when all 7 items are answered and every criterion has passed.

## 4. Key design decisions
- **No invented rules.** No minimum market value, no extra questions. Interest rates, EMI and fees are never quoted; the senior expert provides them. Product facts come only from `additional_context_from_rag`.
- **Disqualification is final and polite.** Agricultural property, no original documents, cash income, tenure outside 3–15 years → one courteous message, then the call ends. No bargaining.
- **Loan above ₹75 lakh is not a rejection.** Explain the cap, ask whether to proceed with ₹75 lakh; if the customer refuses, close politely.
- **Transfer is its own branch.** An existing loan on the property or a wish to reduce the EMI ends the fresh-loan flow immediately with the specialist message. The agent never asks about existing loans on its own, so the checklist stays at exactly 7 items.
- **Hedged answers are never treated as yes.** "I think the papers are probably somewhere" triggers one clarifying question.
- **Availability, not possession,** is the test for original documents.
- **Voice-first style.** Short turns, one question at a time, numbers spoken in words, Devanagari for Hindi, correct gendered verb forms from `agent_gender`.
- **Honesty and compliance.** Admits being an AI when sincerely asked, never asks for OTPs, PINs or card/Aadhaar/PAN details, never promises approval.

## 5. Testing
25 scenarios were planned (see `lap_test_scenarios_and_setup.md`): happy path, each disqualifier, loan limit (accept and refuse), both transfer triggers, busy customer, out-of-order answers, fillers and interruptions, corrections, contradictions, ambiguous answers, unrelated questions, and a Hindi run.

| # | Scenario | Result | Notes / prompt fix made |
|---|---|---|---|
| | *(fill in after your real calls)* | | |

## 6. Known assumptions to flag
- Tenure outside 3–15 years is treated as an immediate disqualification, exactly as the assignment states.
- If a disqualifier and a transfer trigger appear in the same sentence, disqualification takes precedence.
- Mixed bank/cash income: the primary mode decides eligibility.
