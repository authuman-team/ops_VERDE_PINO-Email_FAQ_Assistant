# Decision Log — Verde Pino Email FAQ Assistant

## DEC-001 — Classify before answering

Date: 2026-05-25  
Status: Accepted

Decision:

The workflow first classifies whether an email is from a guest/customer/potential customer before attempting to generate a reply.

Rationale:

The client mailbox receives mixed messages, including suppliers, platforms, newsletters, spam, administrative topics, and internal messages. The assistant must not behave as a general mailbox responder.

Implication:

Non-client messages should not be answered automatically.

Hardening note:

Add an explicit IF after classifier so `deve_processar = false` skips the responder entirely.

---

## DEC-002 — FAQ-grounded answers only

Date: 2026-05-25  
Status: Accepted

Decision:

The responder may only answer when the answer is clearly supported by the approved FAQ context.

Rationale:

The system must avoid hallucinated or unsupported operational commitments.

Implication:

If FAQ context is insufficient, the responder returns `SEM_INFO` and `tem_contexto = false`.

---

## DEC-003 — Human fallback over risky automation

Date: 2026-05-25  
Status: Accepted

Decision:

When uncertain, the workflow leaves the message for human handling instead of sending a reply.

Rationale:

For hospitality operations, wrong replies about availability, price, payments, cancellation, or exceptions create operational and reputational risk.

Implication:

The assistant should under-answer rather than over-answer.

---

## DEC-004 — Recall-first classification, precision-first answering

Date: 2026-05-25  
Status: Accepted

Decision:

The classifier should identify potentially relevant guest/customer messages broadly enough not to miss operational requests. The responder should answer narrowly and only when FAQ support is explicit.

Rationale:

Missing a customer request is recoverable through human review. Sending an unsupported automatic answer is more damaging.

Implication:

Classifier and responder have different risk profiles.

---

## DEC-005 — Re-processing allowed during pilot

Date: 2026-05-25  
Status: Accepted

Decision:

Manual re-processing is allowed during pilot.

Rationale:

Pilot delivery requires debugging, prompt tuning, FAQ tuning, and operational validation.

Constraints:

- Do not re-process threads that already received an automated reply unless duplicate-reply risk has been removed.
- Check Gmail thread and labels before re-processing.
- Document re-processing when it affects a real customer.

---

## DEC-006 — No OCR or attachment processing

Date: 2026-05-25  
Status: Accepted

Decision:

The workflow does not process attachments or OCR.

Rationale:

The solution intent is email FAQ answering, not document extraction.

Implication:

Attachments must be ignored unless a separate documented feature is added.

---

## DEC-007 — No auto-correction of data

Date: 2026-05-25  
Status: Accepted

Decision:

The assistant must not correct, infer, or complete missing business information.

Rationale:

Business commitments must come from approved FAQ or human decision.

Implication:

The model must not infer prices, availability, policies, exceptions, names, booking status, or dates.

---

## DEC-008 — GitHub is source of truth

Date: 2026-05-25  
Status: Accepted

Decision:

GitHub is the source of truth for workflow JSON, prompts, configuration, changelog, and architecture.

Rationale:

n8n is the execution layer; Notion is descriptive; GitHub controls versioned delivery assets.

Implication:

Changes made directly in n8n must be exported back to GitHub before being considered controlled.

---

## DEC-009 — Notion is descriptive control layer

Date: 2026-05-25  
Status: Accepted

Decision:

Notion should present the client-facing overview, scope, operating rules, and links to source-of-truth assets.

Rationale:

Notion is easier for client visibility but should not duplicate technical implementation details.

Implication:

Detailed architecture, prompts, runbook, security, and changelog remain in GitHub.

---

## DEC-010 — LLM usage must be logged

Date: 2026-05-25  
Status: Accepted

Decision:

All LLM calls should pass through the shared Gemini JSON gateway for parsing, usage logging, and cost visibility.

Rationale:

Authuman needs repeatable cost governance across client solutions.

Implication:

Parent workflows should set LLM context consistently before calling the gateway.

---

## DEC-011 — Current FAQ storage is temporary

Date: 2026-05-25  
Status: Accepted

Decision:

The embedded FAQ node is acceptable for initial pilot import but should not remain the only source of truth.

Rationale:

Business knowledge must be reviewed and versioned.

Implication:

Move canonical FAQ to `/config/faq.verde-pino.md`.

---

## DEC-012 — Required hardening before production-grade pilot

Date: 2026-05-25  
Status: Accepted

Decision:

The following small n8n changes are required before relying on the workflow with real customers:

1. Use full Gmail body instead of snippet.
2. Fix responder prompt expression from `=={{` to `={{`.
3. Add classifier IF before responder.
4. Add dedicated human-review label.
5. Add duplicate-reply guard.
6. Rename invoice-related gateway nodes.
7. Align environment value to `pilot`.

Rationale:

These are low-effort changes that materially reduce operational risk.
