# Verde Pino — Email FAQ Assistant

## What the solution does

This solution monitors a Gmail mailbox used by Verde Pino and assists with guest/customer email handling.

The workflow:

1. Detects new emails in the configured Gmail mailbox.
2. Extracts the sender and message content.
3. Uses an LLM classifier to decide whether the message is from a guest, customer, or potential customer.
4. Classifies the type of request, subtype, priority, sentiment, summary, tags, and customer name when available.
5. Uses a FAQ-grounded LLM responder to generate a reply only when the answer is clearly supported by the approved FAQ.
6. Sends an email reply only when the responder confirms that sufficient context exists.
7. Marks answered messages as read and applies a processed label.
8. Leaves unanswered or unsafe-to-answer messages unread for human handling.
9. Logs LLM usage and estimated cost through the shared Gemini JSON gateway.

The solution is designed for pilot operation, not fully autonomous production without monitoring.

## Scope

In scope:

- Guest/customer email classification.
- FAQ-based answers for simple operational questions.
- Portuguese Portugal reply style.
- Conservative fallback to human handling.
- LLM usage logging.
- Gmail read/unread status updates.
- Gmail processed label after automated reply.

Currently supported FAQ topics:

- Check-in time.
- Early check-in subject to availability.
- Check-out time.
- Late check-out may have additional cost.
- Outdoor pool opening hours.
- Free parking subject to availability.
- Breakfast serving hours.

## Out of scope

Out of scope:

- Reservation creation.
- Reservation modification.
- Reservation cancellation.
- Real availability checks.
- Price quotation or final pricing.
- Payment handling.
- Invoice handling.
- Complaint resolution.
- Damage claims.
- Lost property handling.
- Supplier, bank, accounting, tax, platform, newsletter, spam, or internal email processing.
- Attachment processing.
- OCR.
- Automatic extraction from PDFs or images.
- Sending replies when FAQ coverage is partial or ambiguous.
- Auto-correction or inference of missing business data.

## High-level flow

1. Gmail receives a new email.
2. n8n extracts the Gmail snippet and sender.
3. Classifier LLM decides whether the message should be processed.
4. FAQ context is attached.
5. Responder LLM decides whether there is enough FAQ context.
6. If `tem_contexto = true`, Gmail sends a reply.
7. If `tem_contexto = false`, the message is left unread for the owner/team.
8. LLM usage is logged through the shared gateway.

## Inputs

Primary input:

- New Gmail message.

Current extracted fields:

- `Mensagem`: Gmail snippet.
- `Email Cliente`: Gmail `From` field.

LLM classifier output:

- `deve_processar`
- `motivo_nao_processar`
- `tipo_solicitacao`
- `subtipo`
- `prioridade`
- `sentimento`
- `resumo`
- `tags`
- `nome_do_cliente`

LLM responder output:

- `resposta`
- `tem_contexto`
- `confianca_resposta`
- `motivo`

## Outputs

Automated outputs:

- Gmail reply, only when `tem_contexto = true`.
- Message marked as read after successful reply.
- Processed label added after successful reply.
- Message marked/unmarked as unread for human handling when no safe answer is available.
- LLM usage log entry through `llm_gemini_json_gateway`.

## Assumptions and constraints

- Pilot environment.
- Re-processing is allowed during pilot.
- The workflow must not answer unless the FAQ clearly supports the answer.
- Human handling remains required for anything involving real availability, pricing, payments, invoices, complaints, exceptions, or operational decisions.
- The FAQ is currently embedded in the workflow and should be externalised into `/config/faq.verde-pino.md`.
- The workflow currently uses the Gmail snippet, not the full email body. This must be hardened before relying on the workflow for production-grade pilot use.
- No OCR is used.
- No attachments are processed.
- Credentials and secrets must remain in n8n credentials, never in GitHub.
- LLM responses are constrained by JSON schemas and temperature `0`.
- LLM usage is logged for cost visibility.

## Ownership

Business owner:

- Verde Pino owner / designated mailbox operator.

Delivery owner:

- Authuman.

Technical owner:

- Authuman automation maintainer.

Knowledge owner:

- Verde Pino owner for FAQ content.
- Authuman for prompt and workflow implementation.

Source of truth:

- GitHub repository for workflow JSON, documentation, configuration, prompts, changelog, and decisions.
- Notion page for descriptive client-facing overview only.
