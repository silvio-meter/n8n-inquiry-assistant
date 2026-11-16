# AI Inquiry Assistant for Small Businesses (n8n)

A self-hosted n8n workflow that handles new customer inquiries for a small business, from a web form or by email. It classifies each inquiry with an LLM, saves it, drafts a reply for the client, and sends urgent cases to Telegram so a person can review them.

Nothing is sent to the client automatically. A human always reviews the draft reply first.

## How it works

1. Two ways in:
   - **Webhook** (POST `/upit`) receives inquiries from a web form: name, email and message.
   - **Email Trigger (IMAP)** reads new emails from a mailbox folder (for example a Gmail label `Upiti`).
   - **Ulaz** (Edit Fields) turns both into the same fields: `ime`, `email`, `poruka`. For emails it takes the sender name and address, and the subject plus the plain text (or cleaned HTML) of the message.
2. **Basic LLM Chain + xAI Grok + Structured Output Parser** classifies the inquiry and returns clean JSON:
   - `vrsta` (type, e.g. webshop, website, automation)
   - `hitnost` (urgency: niska / srednja / visoka)
   - `sazetak` (one-sentence summary)
3. **Data table "Upiti"** stores every inquiry with name, email, message, type, urgency, summary and date.
4. **Basic LLM Chain1** drafts a short, polite reply in Croatian (no prices or deadlines, max 5 sentences).
5. **If** checks whether urgency is `visoka` (high).
6. **Telegram** sends urgent inquiries to the owner's phone with the summary and the draft reply.

## Screenshots

- `screenshots/workflow.png`: the full workflow
- `screenshots/data-table.png`: a saved inquiry
- `screenshots/telegram.jpg`: the urgent notification with the draft reply

## Run it yourself

1. Start n8n with Docker:
   ```
   docker run -d --name n8n -p 5678:5678 -e GENERIC_TIMEZONE=Europe/Zagreb -v n8n_data:/home/node/.n8n docker.n8n.io/n8nio/n8n
   ```
2. Open http://localhost:5678 and import `upiti-workflow.json`.
3. Add your own credentials: an xAI API key, a Telegram bot token (from @BotFather) and an IMAP account. For Gmail use host `imap.gmail.com`, port 993, SSL on and a Google App Password. Set the mailbox name in the Email Trigger (default `Upiti`).
4. Create a data table `Upiti` with columns Ime, Email, Poruka, Vrsta, Hitnost, Sazetak, Datum, and select it in the `Upiti` node.
5. Put your Telegram chat ID in the Telegram node (replace `YOUR_TELEGRAM_CHAT_ID`).
6. Click **Execute workflow** and send a test inquiry:
   ```
   curl -X POST http://localhost:5678/webhook-test/upit \
     -H "Content-Type: application/json" \
     -d '{"ime":"Marko Kovac","email":"marko@example.com","poruka":"We urgently need a webshop for a clothing store by the end of the month."}'
   ```

## Tools

n8n (self-hosted, Docker), xAI Grok, Structured Output Parser, n8n Data Tables, If logic, Telegram Bot API, webhooks, IMAP email trigger.

## Author

Silvio Meter, LOGOSOM, Osijek, Croatia. [LinkedIn](https://www.linkedin.com/in/silviometer/)
