# Changelog — Verde Pino Email FAQ Assistant

## 0.1.0 — Initial pilot import

Date: 2026-05-25

Imported initial pilot workflow assets:

- `workflows/verde-pino-email-faq-assistant.n8n.json`
- `workflows/llm-gemini-json-gateway.n8n.json`

Initial capabilities:

- Gmail trigger for incoming mailbox messages.
- Message sender and snippet extraction.
- LLM-based customer/guest email classification.
- Structured classifier output schema.
- Static FAQ context embedded in n8n.
- FAQ-grounded LLM response generation.
- Structured responder output schema.
- Automatic Gmail reply when responder confirms sufficient context.
- Mark answered messages as read.
- Apply processed Gmail label after reply.
- Mark/leave unanswered messages unread for human handling.
- Gemini request construction using JSON response schema.
- Shared Gemini JSON gateway integration.
- LLM usage logging to n8n Data Table.
- Token usage and estimated cost calculation.

Known pilot limitations:

- Workflow currently uses Gmail snippet instead of full email body.
- FAQ is hard-coded inside workflow.
- Responder prompt expression appears to start with `=={{` and should be corrected.
- Responder is called even when classifier output may indicate the email should not be processed.
- False branch does not yet apply a dedicated human-review label.
- Duplicate-reply guard should be added before pilot operation.
- Several nodes still include invoice-related names and should be renamed.
- Workflow context uses `environment = dev`; pilot deployment should use `pilot`.
