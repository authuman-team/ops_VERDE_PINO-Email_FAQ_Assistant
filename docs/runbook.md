# Runbook — Verde Pino Email FAQ Assistant

## Normal operation

### Expected daily behavior

The workflow monitors the configured Gmail mailbox.

For each new email:

1. The workflow extracts sender and message content.
2. The classifier determines whether it is a guest/customer/potential customer request.
3. The responder checks whether the FAQ fully supports an answer.
4. If the responder returns `tem_contexto = true`, the workflow replies automatically.
5. After replying, the workflow marks the message as read and applies the processed label.
6. If the responder returns `tem_contexto = false`, the workflow leaves the email for human handling by marking/leaving it unread.

### Human mailbox operator responsibilities

The mailbox operator should:

- Review unread messages daily.
- Manually answer messages not handled by the assistant.
- Report wrong replies, missed replies, or unsafe attempted replies to Authuman.
- Request FAQ updates only when the business rule is stable and approved.
- Avoid manually editing workflow prompts in n8n.

### Authuman responsibilities

Authuman should:

- Maintain workflow JSON.
- Maintain prompt versions.
- Maintain the FAQ configuration file.
- Review failed executions during pilot.
- Monitor LLM usage logs.
- Apply changes through GitHub-controlled updates.

## Re-processing behavior

Re-processing is allowed during pilot.

Before re-processing an email:

1. Confirm whether the workflow already replied to the thread.
2. Check whether the processed label exists.
3. Check Gmail thread history.
4. If an automated reply was already sent, do not re-run unless duplicate reply risk has been removed.
5. If the message was not answered and remains unread, re-processing may be performed manually.

Recommended pilot rule:

- Re-processing is allowed for debugging and missed messages.
- Re-processing must not create duplicate customer replies.
- Manual re-processing must be documented in the execution notes or issue tracker when it affects a real customer.

## Common failure scenarios

### 1. Workflow does not trigger

Symptoms:

- New emails arrive but no n8n execution starts.

Checks:

- Confirm workflow is active.
- Confirm Gmail trigger credential is valid.
- Confirm Gmail trigger polling is enabled.
- Confirm the correct mailbox is connected.
- Check n8n execution history.

Resolution:

- Reconnect Gmail OAuth credential if expired.
- Activate workflow if inactive.
- Run a manual test only with a safe test email.

### 2. Classifier returns invalid JSON

Symptoms:

- Gemini gateway throws JSON parsing error.
- Parent workflow stops before classifier output.

Likely causes:

- Gemini returned non-JSON text.
- Schema mismatch.
- Prompt or response schema changed incorrectly.
- Provider issue.

Resolution:

- Inspect raw Gemini response.
- Revert latest prompt/schema change if applicable.
- Keep temperature at `0`.
- Keep response schema strict.
- Do not add free-text instructions after the JSON-only instruction.

### 3. Responder returns `SEM_INFO`

Symptoms:

- Email is left unread.
- No automatic reply is sent.

This is expected behavior when:

- FAQ is insufficient.
- Request involves availability, prices, payment, complaints, invoices, cancellation, damage, lost property, or a decision requiring the owner.
- Message is ambiguous.
- Message is not from a guest/customer/potential customer.

Resolution:

- Human operator replies manually.
- If the same question repeats often and has a stable approved answer, update the FAQ through GitHub.

### 4. Email is wrongly answered

Symptoms:

- Assistant sends a reply that should have been handled manually.

Immediate action:

1. Disable the workflow or Gmail trigger.
2. Notify Authuman.
3. Manually follow up with the customer if needed.
4. Capture the n8n execution ID, Gmail thread, prompt version, and generated answer.
5. Add a changelog entry and prompt/config issue before reactivation.

Likely causes:

- FAQ was too broad.
- Responder prompt allowed partial context.
- Full email context was missing because only snippet was used.
- Classifier incorrectly allowed processing.

Resolution:

- Tighten FAQ.
- Tighten responder prompt.
- Add explicit exclusion.
- Add tests for the failing message pattern.

### 5. Duplicate reply risk

Symptoms:

- Same thread receives more than one automated reply.

Likely causes:

- Manual re-run.
- Gmail thread triggering behavior.
- Processed label not checked before replying.

Resolution:

- Add a pre-reply guard checking for processed label or previous Authuman reply.
- Use Gmail labels to exclude already processed messages.
- Do not re-run answered messages without manual inspection.

### 6. LLM usage logging fails

Symptoms:

- Reply flow may continue, but usage log is missing.

Current behavior:

- Gateway logging is configured to continue on regular output.

Resolution:

- Check `llm_usage_log` Data Table availability.
- Check schema compatibility.
- Check gateway execution.
- Do not block customer replies only because usage logging fails during pilot, unless cost governance requires strict blocking.

### 7. Gemini API request fails

Symptoms:

- HTTP Request node fails after retries.

Current behavior:

- Gemini call has retry enabled with up to 5 tries and 5 seconds between tries.

Resolution:

- Check Gemini credential.
- Check model name.
- Check request body.
- Check provider availability.
- Re-run only after confirming no duplicate reply risk.

## Kill switch

Primary kill switch:

- Deactivate the main workflow `verde-pino-email-faq-assistant.n8n.json`.

Secondary kill switches:

- Disable Gmail trigger.
- Remove or invalidate Gmail credential.
- Remove Gemini HTTP credential.
- Add Gmail trigger filter preventing new messages from matching.
- Temporarily change the responder branch to always route to human handling.

Emergency rule:

- If a wrong automatic reply is sent to a real customer, deactivate the workflow first and investigate second.

## Change management rules

All production/pilot changes must be made through GitHub first.

Required GitHub updates for each change:

- Workflow JSON updated in `/workflows`.
- Relevant doc updated in `/docs`.
- FAQ/config updated in `/config` when business knowledge changes.
- `docs/changelog.md` updated.
- `docs/decision-log.md` updated when architecture or business rules change.
- `docs/prompts.md` updated when prompt or schema changes.

Prompt changes:

- Must be versioned.
- Must preserve JSON-only output.
- Must preserve conservative fallback.
- Must not expand automatic reply scope without an approved FAQ update.
- Must be tested with examples covering:
  - safe FAQ question,
  - unavailable FAQ question,
  - non-client email,
  - complaint,
  - availability/pricing request,
  - ambiguous multi-question email.

FAQ changes:

- Must be approved by Verde Pino owner or designated operational owner.
- Must be specific.
- Must avoid vague policy language.
- Must not imply guaranteed availability unless the business confirms that guarantee.

Workflow changes:

- Must not store credentials or secrets in JSON.
- Must not expose personal data in comments, docs, or test fixtures.
- Must maintain human fallback.

## 3C hardening status

### Already compliant

- JSON schema used for both LLM outputs.
- Temperature set to `0`.
- Conservative responder prompt.
- Explicit `SEM_INFO` fallback.
- Messages without sufficient FAQ context are not automatically answered.
- Automated replies are followed by read status and processed label.
- LLM usage is logged through a shared gateway.
- Gateway estimates cost and logs token usage.
- No OCR or attachment processing detected.
- No automatic correction of business data detected.

### Small changes recommended inside n8n

1. Replace Gmail snippet with full email body retrieval.
2. Fix responder prompt expression from `=={{ ... }}` to `={{ ... }}`.
3. Add an IF after classifier:
   - If `deve_processar = true`, continue to responder.
   - If `deve_processar = false`, skip responder and route to no-action/human-review path.
4. Add a "Needs human reply" Gmail label to the false branch.
5. Add a duplicate-reply guard before the Gmail reply node.
6. Rename:
   - `Extract Invoice Data via Gateway` → `Parse Classifier Output via Gateway`
   - `Extract Invoice Data via Gateway1` → `Parse Responder Output via Gateway`
7. Change workflow context environment from `dev` to `pilot` for pilot deployments.
8. Move FAQ content to `/config/faq.verde-pino.md` and keep the n8n Set node synchronized.
9. Add workflow sticky notes documenting:
   - safe-answer boundary,
   - no availability/pricing decisions,
   - no complaints/payments/invoices,
   - human fallback rule.

### Explicitly not to do

- Do not allow the model to answer from general knowledge.
- Do not ask the model to infer prices, availability, or exceptions.
- Do not process suppliers, newsletters, platforms, banks, tax, accounting, spam, or internal messages as customers.
- Do not process attachments.
- Do not add OCR unless a separate OCR path is designed, documented, and approved.
- Do not store credentials in GitHub.
- Do not rely on prompt wording alone for business policy changes.
