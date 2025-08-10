# WhatsApp-Driven Google Drive Assistant (n8n Workflow)

## 🚀 Project Overview

This project creates an intelligent WhatsApp bot that can manage your Google Drive files through simple text commands. Built with n8n workflow automation platform.

### What You Can Do:
- **LIST** `/ProjectX` - List all files in a folder
- **DELETE** `/ProjectX/report.pdf` - Delete a specific file
- **MOVE** `/ProjectX/report.pdf /Archive` - Move files between folders
- **SUMMARY** `/ProjectX` - Get AI-powered summaries of all documents in a folder

## 📋 Table of Contents
1. [What is n8n?](#what-is-n8n)
2. [Project Architecture](#project-architecture)
3. [Setup Instructions](#setup-instructions)
4. [Command Reference](#command-reference)
5. [Security & Safety](#security--safety)
6. [Troubleshooting](#troubleshooting)

## 🤖 What is n8n?

**n8n** is a visual workflow automation tool that connects different services together. Think of it like IFTTT but much more powerful and self-hosted.

### Key Concepts:

#### 1. **Nodes** 🔗
- Each service/action is a "node" (like Lego blocks)
- Examples: Twilio node, Google Drive node, OpenAI node
- Nodes have inputs and outputs

#### 2. **Workflows** 🔄
- A sequence of connected nodes
- Data flows from one node to the next
- Can have multiple paths and conditions

#### 3. **Triggers** ⚡
- How workflows start
- Examples: Webhook received, schedule, manual trigger

#### 4. **Executions** ▶️
- Each time a workflow runs
- Can see logs, data, and errors

## 🏗️ Project Architecture

```
WhatsApp Message → Twilio → n8n Webhook → Parse Command → Google Drive Action → AI Summary → Response → WhatsApp
```

### Workflow Components:

1. **Webhook Trigger** - Receives WhatsApp messages from Twilio
2. **Command Parser** - Extracts command and parameters
3. **Google Drive Operations** - LIST, DELETE, MOVE, READ files
4. **AI Summarization** - OpenAI GPT-4 integration
5. **Response Handler** - Sends results back to WhatsApp
6. **Error Handler** - Manages failures gracefully
7. **Audit Logger** - Records all operations

## 🛠️ Setup Instructions

### Prerequisites
- Windows 10/11
- Docker Desktop
- Google Account
- Twilio Account (free tier available)
- OpenAI API key

### Step 1: Install Docker Desktop

1. Download from: https://www.docker.com/products/docker-desktop/
2. Install and start Docker Desktop
3. Verify installation:
   ```powershell
   docker --version
   ```

### Step 2: Setup n8n with Docker

1. Create environment file:
   ```powershell
   cp .env.example .env
   ```

2. Edit `.env` with your credentials (see [Environment Variables](#environment-variables))

3. Start n8n:
   ```powershell
   docker-compose up -d
   ```

4. Access n8n at: http://localhost:5678

### Step 3: Import Workflow

1. Open n8n web interface
2. Click "Import from File"
3. Select `whatsapp-drive-workflow.json`
4. Configure credentials for each service

### Step 4: Setup Twilio WhatsApp Sandbox

1. Sign up at: https://www.twilio.com/
2. Go to Console > Messaging > Try it out > Send a WhatsApp message
3. Follow sandbox setup instructions
4. Note your webhook URL: `http://your-domain:5678/webhook/whatsapp`

### Step 5: Setup Google Drive API

1. Go to: https://console.cloud.google.com/
2. Create new project or select existing
3. Enable Google Drive API
4. Create OAuth2 credentials
5. Add your credentials to n8n

### Step 6: Setup OpenAI API

1. Get API key from: https://platform.openai.com/
2. Add to n8n credentials

## 📱 Command Reference

### Basic Commands

#### LIST Command
```
LIST /folder-path
```
**Example:** `LIST /ProjectX`
**Response:** Lists all files and folders in the specified path

#### DELETE Command
```
DELETE /path/to/file.ext
```
**Example:** `DELETE /ProjectX/report.pdf`
**Response:** Confirms deletion or asks for confirmation

#### MOVE Command
```
MOVE /source/file.ext /destination/
```
**Example:** `MOVE /ProjectX/report.pdf /Archive`
**Response:** Confirms file moved successfully

#### SUMMARY Command
```
SUMMARY /folder-path
```
**Example:** `SUMMARY /ProjectX`
**Response:** AI-generated summary of all documents in folder

### Advanced Commands

#### HELP Command
```
HELP
```
Shows available commands and syntax

#### STATUS Command
```
STATUS
```
Shows your Google Drive usage and recent operations

## 🔒 Security & Safety

### Built-in Protections:

1. **Confirmation Required**: DELETE operations require typing "CONFIRM" 
2. **Audit Logging**: All operations logged to Google Sheets
3. **Rate Limiting**: Max 10 operations per minute
4. **Path Validation**: Prevents access outside user's Drive
5. **Error Handling**: Graceful failure messages

### Security Best Practices:

- Keep your `.env` file private
- Use strong API keys
- Regular audit log reviews
- Set up webhook authentication

## 🚨 Environment Variables

Create a `.env` file with these variables:

```env
# n8n Configuration
N8N_BASIC_AUTH_ACTIVE=true
N8N_BASIC_AUTH_USER=admin
N8N_BASIC_AUTH_PASSWORD=your-secure-password

# Twilio Credentials
TWILIO_ACCOUNT_SID=your-twilio-sid
TWILIO_AUTH_TOKEN=your-twilio-token
TWILIO_PHONE_NUMBER=your-twilio-whatsapp-number

# Google Drive API
GOOGLE_CLIENT_ID=your-google-client-id
GOOGLE_CLIENT_SECRET=your-google-client-secret

# OpenAI API
OPENAI_API_KEY=your-openai-api-key

# Webhook Security
WEBHOOK_SECRET=your-random-secret-key
```

## 🔧 How the Workflow Works

### 1. Message Reception
- Twilio receives WhatsApp message
- Forwards to n8n webhook endpoint
- n8n validates sender and extracts message

### 2. Command Parsing
- JavaScript function parses command structure
- Validates syntax and parameters
- Routes to appropriate handler

### 3. Google Drive Operations
- Uses Google Drive API nodes
- Implements proper error handling
- Respects user permissions

### 4. AI Processing
- Reads document content
- Sends to OpenAI for summarization
- Formats response for WhatsApp

### 5. Response Delivery
- Formats results for mobile display
- Sends via Twilio back to WhatsApp
- Logs operation in audit sheet

## 🧩 n8n Workflow Breakdown

### Node Types Used:

1. **Webhook Node** - Receives HTTP requests from Twilio
2. **Function Node** - JavaScript code for parsing and logic
3. **Google Drive Nodes** - File operations (list, delete, move, read)
4. **OpenAI Node** - AI summarization
5. **HTTP Request Node** - Send responses to Twilio
6. **Google Sheets Node** - Audit logging
7. **Switch Node** - Route commands to handlers
8. **Set Node** - Data transformation

### Workflow Structure:
```
Webhook → Parse Command → Switch → [LIST|DELETE|MOVE|SUMMARY] → Format Response → Send to WhatsApp
                                        ↓
                                   Audit Logger
```

## 🎯 Testing Your Setup

### Test Commands:

1. **Basic Test:**
   ```
   HELP
   ```

2. **List Root:**
   ```
   LIST /
   ```

3. **Create Test File:**
   - Upload a PDF to your Drive
   - Try: `LIST /` to see it
   - Try: `SUMMARY /` to get AI summary

## 🐛 Troubleshooting

### Common Issues:

1. **Webhook Not Receiving Messages**
   - Check Twilio webhook URL
   - Verify n8n is accessible from internet
   - Check firewall settings

2. **Google Drive Authentication Failed**
   - Re-authenticate in n8n credentials
   - Check API permissions
   - Verify OAuth consent screen

3. **AI Summaries Not Working**
   - Verify OpenAI API key
   - Check credit balance
   - Ensure proper node configuration

4. **Commands Not Recognized**
   - Check command syntax
   - Verify case sensitivity
   - Review parsing logic in Function node

## 📊 Monitoring & Maintenance

### Logs to Monitor:
- n8n execution logs
- Twilio webhook logs
- Google API quota usage
- OpenAI API usage

### Regular Tasks:
- Review audit logs weekly
- Update API keys before expiration
- Monitor storage usage
- Backup workflow configuration

## 🚀 Future Enhancements

Possible improvements:
- Natural language processing
- File upload/download support
- Collaborative folder management
- Integration with other cloud storage
- Voice message support

## 📞 Support

If you encounter issues:
1. Check the troubleshooting section
2. Review n8n execution logs
3. Verify all credentials are correct
4. Test each component individually

---

**Built with ❤️ using n8n workflow automation**
