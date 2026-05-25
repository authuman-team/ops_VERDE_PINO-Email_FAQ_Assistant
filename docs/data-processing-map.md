# Data Processing Map — Verde Pino Email FAQ Assistant

## Purpose

The solution processes incoming mailbox messages to classify customer/guest requests and, when safe, generate FAQ-based email replies.

This document maps personal data categories, processing activities, subprocessors, retention assumptions, and access controls for GDPR readiness.

This is not a legal opinion.

## Processing activities

| Activity | Description | System |
|---|---|---|
| Email ingestion | New Gmail messages are detected by n8n | Gmail, n8n |
| Message extraction | Sender and message text are mapped into workflow fields | n8n |
| Classification | LLM classifies whether the email is a customer/guest request | Gemini |
| FAQ-grounded response | LLM generates a reply only when FAQ context is sufficient | Gemini |
| Email reply | Gmail sends a reply in the original thread | Gmail |
| Mailbox status update | Message is marked as read or unread and labelled | Gmail |
| Usage logging | Token counts, model, workflow metadata, and estimated cost are logged | n8n Data Table |

## Data categories

### Customer / guest data

Potentially processed:

- Sender email address.
- Sender display name.
- Message content.
- Name if included in email or sender field.
- Stay-related questions.
- Reservation-related context if included by the sender.
- Arrival/departure dates if included by the sender.
- Issue/complaint content if included by the sender.
- Any other personal data voluntarily included in the email body.

### Business operational data

Potentially processed:

- FAQ content.
- Email classification.
- Request category and subtype.
- Priority.
- Sentiment.
- Tags.
- Short summary.
- Generated reply.
- Human handoff reason.

### Technical metadata

Processed:

- n8n workflow ID.
- n8n execution ID.
- Workflow name.
- Node name.
- LLM provider.
- LLM model.
- Token counts.
- Estimated cost.
- Prompt character count.
- Response character count.
- Provider request/response metadata where available.

## Special category data

The workflow is not designed to process special category data.

However, customers may include sensitive data in free-text emails. The solution does not intentionally request or require this information.

If sensitive information appears in an email, it may be temporarily processed by Gmail, n8n, and the LLM provider as part of normal message handling.

## Subprocessors / systems involved

| System | Role | Data processed |
|---|---|---|
| Google Gmail / Google Workspace | Mailbox provider | Email content, sender, thread metadata, labels, read/unread state |
| n8n | Workflow automation platform | Email fields, workflow data, execution metadata |
| Google Gemini API | LLM provider | Prompts containing email text and FAQ context; structured response output |
| n8n Data Tables | Usage logging | LLM usage metadata, model, estimated cost, execution references |
| GitHub | Source of truth | Workflow JSON, docs, configuration, prompts; no runtime customer email data should be stored |
| Notion | Client control layer | Descriptive solution overview; no runtime customer email data should be stored |

## Data flow

1. Email arrives in Gmail.
2. n8n detects the email.
3. n8n extracts sender and message text.
4. n8n sends message text and prompt instructions to Gemini.
5. Gemini returns structured JSON.
6. n8n evaluates whether a reply is allowed.
7. If allowed, Gmail sends a reply.
8. n8n logs usage metadata in the LLM usage log.
9. If not allowed, message remains available for human handling.

## Retention assumptions

These are pilot assumptions and must be validated with the client before production.

| Data | Recommended pilot assumption |
|---|---|
| Gmail messages | Retained according to Verde Pino mailbox policy |
| n8n execution data | Retain only as long as needed for debugging and pilot monitoring |
| LLM usage logs | Retain for billing, cost monitoring, and operational audit during pilot |
| GitHub docs/config | Retained as source of truth |
| Notion overview | Retained while solution is active |
| Test emails | Remove or anonymise when no longer needed |

No retention commitment is made by this document.

## Access control assumptions

### Verde Pino

Should access:

- Gmail mailbox.
- Notion client page.
- Operational outputs.

Should not access by default:

- n8n credentials.
- Gemini API credentials.
- Internal cost gateway implementation unless agreed.

### Authuman

Should access:

- n8n workflow.
- n8n execution logs required for support.
- GitHub repository.
- Notion documentation.
- LLM usage logs.

Should not store:

- Gmail passwords.
- API keys in GitHub.
- Customer email content in documentation.
- Real customer examples unless anonymised.

## Data minimisation

Current minimisation measures:

- Only sender and message text are needed for classification/reply.
- No attachments are processed.
- No OCR is used.
- LLM output is structured and limited.
- The responder is instructed not to answer unless FAQ context is sufficient.

Recommended improvement:

- Use full email body for correctness, but strip quoted history, signatures, tracking text, and irrelevant HTML before sending to the LLM.
- Avoid sending attachments to the LLM.
- Avoid logging full prompts in permanent logs unless explicitly required and approved.

## GDPR risk notes

Main risks:

- Free-text emails can contain unexpected personal data.
- Automated replies can affect customer experience.
- Prompt injection may be attempted through email content.
- Incorrect classification may send non-customer content to the responder.
- Snippet-only processing may miss important context.

Mitigations:

- Conservative responder rules.
- `SEM_INFO` fallback.
- Structured JSON schema.
- Temperature `0`.
- Human handling for insufficient context.
- Recommended full-body sanitisation before pilot.
- Recommended classifier gate before responder.
