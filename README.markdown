# WhatsApp-Driven Google Drive Assistant

This n8n workflow enables a WhatsApp-driven assistant to perform Google Drive operations such as listing files, deleting files, moving files, and summarizing documents in a specified folder. It uses Twilio for WhatsApp integration, Google Drive and Google Sheets APIs, and OpenAI GPT-4o for summarization.

## Features
- **Inbound Channel**: Listens for WhatsApp messages via Twilio Sandbox, parsing commands like `/LIST /ProjectX`, `/DELETE /ProjectX/report.pdf CONFIRM`, `/MOVE /ProjectX/report.pdf /Archive`, `/SUMMARY /ProjectX`.
- **Google Drive Integration**: Uses OAuth2 to access the authenticated user's Google Drive for file operations (list, delete, move, download).
- **AI Summarization**: Uses OpenAI GPT-4o to generate concise bullet-point summaries for PDF, Docx, and TXT files.
- **Logging**: Logs all actions (command, timestamp, status) in a Google Sheet.
- **Safety**: Requires 'CONFIRM' keyword for delete operations to prevent accidental deletions.
- **Help Command**: Responds to `/help` with available command syntax.

## Prerequisites
- **Twilio Account**: For WhatsApp Sandbox.
- **Google Cloud Project**: For Google Drive and Google Sheets OAuth2 credentials.
- **OpenAI Account**: For GPT-4o API key.
- **Docker**: For running n8n locally.
- **n8n Instance**: Cloud or self-hosted.

## Setup Instructions

### 1. Twilio Setup
1. Sign up for a [Twilio account](https://www.twilio.com/try-twilio).
2. Activate the WhatsApp Sandbox in the Twilio Console (Programmable Messaging > WhatsApp).
3. Note your **Account SID**, **Auth Token**, and **WhatsApp Sandbox Number** (e.g., +14155238886).
4. Join the sandbox by sending the provided code to the sandbox number via WhatsApp.
5. Configure the sandbox webhook to point to your n8n instance (e.g., `http://<your-n8n-host>/webhook/twilio`).

### 2. Google Cloud Setup
1. Create a project in [Google Cloud Console](https://console.cloud.google.com/).
2. Enable **Google Drive API** and **Google Sheets API**.
3. Create OAuth2 credentials:
   - Set the redirect URI to `http://<your-n8n-host>/oauth2/callback`.
   - Download the client ID and secret.
4. Create a Google Sheet for logging (note the spreadsheet ID).

### 3. OpenAI Setup
1. Sign up for an [OpenAI account](https://platform.openai.com/).
2. Generate an API key for GPT-4o.

### 4. n8n Deployment with Docker
1. Install Docker and Docker Compose.
2. Create a `docker-compose.yml`:
   ```yaml
   version: '3'
   services:
     n8n:
       image: n8nio/n8n
       ports:
         - "5678:5678"
       environment:
         - N8N_HOST=localhost
         - N8N_PORT=5678
         - N8N_PROTOCOL=http
         - N8N_EDITOR_BASE_URL=http://localhost:5678
       volumes:
         - ./n8n-data:/home/node/.n8n
   ```
3. Run `docker-compose up -d` to start n8n.
4. Access n8n at `http://localhost:5678`.

### 5. n8n Configuration
1. Log in to n8n and import `workflow.json` from this repository.
2. Add credentials:
   - **Twilio**: Use Account SID and Auth Token.
   - **Google Drive OAuth2**: Use Google Cloud credentials.
   - **Google Sheets OAuth2**: Use Google Cloud credentials.
   - **OpenAI**: Use API key.
3. Update the Google Sheets node with your spreadsheet ID.
4. Activate the workflow.

### 6. Command Syntax
- `/help`: Show available commands.
- `/LIST <folder_path>`: List files in the specified folder (e.g., `/LIST /ProjectX`).
- `/DELETE <file_path> CONFIRM`: Delete a file (e.g., `/DELETE /ProjectX/report.pdf CONFIRM`).
- `/MOVE <file_path> <destination_folder>`: Move a file (e.g., `/MOVE /ProjectX/report.pdf /Archive`).
- `/SUMMARY <folder_path>`: Summarize PDF/Docx/TXT files in the folder (e.g., `/SUMMARY /ProjectX`).

### 7. Testing
1. Send a WhatsApp message to the Twilio Sandbox number with a command (e.g., `/LIST /ProjectX`).
2. Verify responses in WhatsApp and check the Google Sheet for logs.
3. Test all commands and ensure error messages are clear (e.g., missing CONFIRM for DELETE).

## Environment Variables
Create a `.env` file in the n8n data directory:
```env
N8N_AUTH_TOKEN=your_n8n_auth_token
TWILIO_ACCOUNT_SID=your_twilio_account_sid
TWILIO_AUTH_TOKEN=your_twilio_auth_token
GOOGLE_CLIENT_ID=your_google_client_id
GOOGLE_CLIENT_SECRET=your_google_client_secret
OPENAI_API_KEY=your_openai_api_key
```

## Error Handling
- Invalid commands return a message with `/help` instructions.
- Missing CONFIRM keyword for DELETE triggers a prompt to include it.
- Logs capture successes and errors for debugging.

## Security
- OAuth2 tokens are managed securely via n8n credentials.
- Twilio webhook requires a valid URL; consider adding authentication for production.
- Google Drive operations are scoped to the authenticated user’s drive.

## Extensibility
- Add new commands by extending the IF nodes with additional conditions.
- Modify the OpenAI prompt for customized summaries.
- Integrate additional AI models (e.g., Claude) by swapping the OpenAI node.

## Known Limitations
- Summarization is limited to one file at a time due to workflow simplicity; extend with a loop for multiple files.
- File path parsing assumes simple formats; enhance with regex for robustness.
- Twilio Sandbox has message rate limits; use WhatsApp Cloud API for production.

## Citations
- n8n Documentation: [Google Drive Node](https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.googledrive/)[](https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.googledrive/)
- n8n Documentation: [WhatsApp Business Cloud Node](https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.whatsappbusinesscloud/)[](https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.whatsapp/)
- n8n Workflow Templates: [WhatsApp Chatbot](https://n8n.io/workflows/1043-whatsapp-sales-agent/)[](https://n8n.io/workflows/2465-building-your-first-whatsapp-chatbot/)

## Demo Video
- [Link to demo video on YouTube/Loom] (to be added after recording)

## Blockers
- If Twilio webhook setup fails, ensure the n8n instance is publicly accessible (e.g., use ngrok for local testing).
- If Google OAuth2 fails, verify redirect URI and API scopes in Google Cloud Console.