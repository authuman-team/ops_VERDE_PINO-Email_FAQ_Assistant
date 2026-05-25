# Architecture — Verde Pino Email FAQ Assistant

## Architecture summary

The solution is composed of two n8n workflows:

1. `verde-pino-email-faq-assistant.n8n.json`
   - Client-facing workflow.
   - Polls Gmail.
   - Extracts message data.
   - Calls Gemini for classification.
   - Adds FAQ context.
   - Calls Gemini for FAQ-grounded response generation.
   - Replies or leaves message for human handling.

2. `llm-gemini-json-gateway.n8n.json`
   - Internal shared gateway.
   - Parses Gemini responses.
   - Enforces JSON parsing.
   - Logs token usage and estimated cost.
   - Returns parsed JSON to the parent workflow.

## Components

### Gmail Trigger

The Gmail trigger polls the configured mailbox every minute.

Current behavior:

- Detects incoming email.
- Passes Gmail message metadata downstream.
- Current extraction uses `snippet`.

Hardening requirement:

- Replace snippet-only processing with full email body retrieval before pilot operation.

### Email pre-processing

The workflow maps:

- Gmail `snippet` → `Mensagem`
- Gmail `From` → `Email Cliente`

Current limitation:

- Gmail snippets can truncate content and remove relevant context.
- Long emails, multi-question emails, and forwarded messages may be misclassified or answered incompletely.

### Classifier LLM

The classifier decides whether the email should be processed by the assistant.

It returns structured JSON with:

- processing decision,
- non-processing reason,
- request category,
- subtype,
- priority,
- sentiment,
- short summary,
- tags,
- customer name when clearly available.

The classifier uses:

- provider: Gemini
- model: `gemini-2.5-flash-lite`
- temperature: `0`
- structured JSON schema

### FAQ context

FAQ context is currently stored in a Set node inside the n8n workflow.

Current FAQ:

- Check-in from 15:00.
- Early check-in subject to availability.
- Check-out until 11:00.
- Late check-out may have additional cost.
- Outdoor pool open from 08:00 to 20:00.
- Free parking subject to availability.
- Breakfast from 07:00 to 10:30.

Recommended architecture:

- Move FAQ text to `/config/faq.verde-pino.md`.
- Keep the n8n node synchronized with that file.
- Treat FAQ changes as versioned business changes.

### FAQ-grounded responder LLM

The responder receives:

- classifier output,
- FAQ context,
- customer message.

It returns:

- answer text,
- whether there is enough context,
- confidence,
- reason.

Only messages with `tem_contexto = true` may be answered automatically.

### Gmail response branch

If `tem_contexto = true`:

1. Reply to the Gmail message.
2. Mark the message as read.
3. Add processed label.

If `tem_contexto = false`:

1. Leave or mark the message unread.
2. Human operator handles the message manually.

Recommended hardening:

- Add a dedicated "Needs human reply" label to the false branch.
- Add an IF immediately after the classifier so non-client messages do not continue into the responder.

### Gemini JSON gateway

The gateway receives LLM metadata and raw Gemini response JSON from the parent workflow.

It:

1. Parses Gemini text output as JSON.
2. Extracts token usage metadata.
3. Estimates cost using an internal price book.
4. Logs usage to `llm_usage_log`.
5. Returns only the parsed `output` object to the parent workflow.

Log failure does not block the parent workflow because the logging node continues on regular output.

## Mermaid diagram

```mermaid
flowchart TD
    A[Gmail Trigger] --> B[Extract sender and message]
    B --> C[Set classifier LLM context]
    C --> D[Build Gemini classifier request]
    D --> E[Call Gemini classifier]
    E --> F[Prepare logging payload]
    F --> G[Gemini JSON Gateway]
    G --> H[Parsed classifier output]

    H --> I[Prepare FAQ responder input]
    I --> J[Attach FAQ context]
    J --> K[Set responder LLM context]
    K --> L[Build Gemini responder request]
    L --> M[Call Gemini responder]
    M --> N[Prepare logging payload]
    N --> O[Gemini JSON Gateway]
    O --> P[Parsed responder output]

    P --> Q{tem_contexto = true?}
    Q -->|Yes| R[Reply in Gmail]
    R --> S[Mark as read]
    S --> T[Add processed label]

    Q -->|No| U[Mark / leave unread]
    U --> V[Human handling]

    subgraph Internal Shared Component
      G
      O
      W[LLM usage log]
      G --> W
      O --> W
    end
