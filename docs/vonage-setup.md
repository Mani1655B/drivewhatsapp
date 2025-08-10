# 🔧 Setup Guide: Vonage Messages API (WhatsApp)

This guide walks you through setting up Vonage Messages API for WhatsApp integration with your n8n workflow.

## 📋 Prerequisites
- Vonage Developer Account (free signup)
- Business verification for WhatsApp Business API (for production)
- n8n instance with webhook capability
- ngrok or similar tunneling service for development

## 🚀 Step-by-Step Setup

### 1. Create Vonage Account
1. Go to [Vonage Dashboard](https://dashboard.nexmo.com/)
2. Sign up for a free account
3. Complete email verification
4. Add your phone number for account verification

### 2. Create Vonage Application
1. In the Vonage Dashboard, go to **Applications**
2. Click "Create an application"
3. Fill in application details:
   - **Application name**: WhatsApp Drive Assistant
   - **Select capabilities**: Messages
4. Configure Messages capability:
   - **Inbound URL**: `https://your-domain.ngrok.io/webhook/whatsapp`
   - **Status URL**: `https://your-domain.ngrok.io/webhook/whatsapp/status`
5. Click "Create Application"
6. **Save the Application ID** - you'll need this!

### 3. Generate Private Key
1. In your application details page
2. Click "Generate public and private key pair"
3. Download the private key file
4. Save it securely as `vonage_private.key`

### 4. Get API Credentials
1. In Vonage Dashboard, go to **Getting Started**
2. Note your **API Key** and **API Secret**
3. These will be used for authentication

## 📱 WhatsApp Configuration

### Development (Sandbox)
For development and testing:

1. **WhatsApp Sandbox**:
   - Go to **Messages and Dispatch** → **Sandbox**
   - Follow instructions to configure WhatsApp sandbox
   - Add your phone number to allowlist
   - Note the sandbox number provided

### Production (Business API)
For production use:

1. **Business Verification**:
   - Requires business verification with Meta/WhatsApp
   - Submit business documentation
   - Can take several days to weeks for approval

2. **Phone Number**:
   - Provide business phone number for WhatsApp
   - Number will be verified by WhatsApp
   - Cannot be personal WhatsApp number

## 🔒 Environment Variables

Add these to your `.env` file:

```env
# Vonage API Credentials
VONAGE_API_KEY=your_vonage_api_key
VONAGE_API_SECRET=your_vonage_api_secret
VONAGE_APPLICATION_ID=your_application_id

# Private Key (file path or content)
VONAGE_PRIVATE_KEY_PATH=./credentials/vonage_private.key

# Webhook URLs (update with your ngrok or domain)
VONAGE_WEBHOOK_URL=https://your-domain.ngrok.io/webhook/whatsapp
```

## 🔗 Configure n8n Credentials

### 1. Create Vonage Credentials in n8n
Since n8n might not have built-in Vonage credentials, we'll use HTTP Request nodes with authentication:

#### Option A: HTTP Basic Auth
1. Go to **Settings** → **Credentials**
2. Click "Create new credential"
3. Search for "HTTP Basic Auth"
4. Fill in:
   - **Username**: Your Vonage API Key
   - **Password**: Your Vonage API Secret

#### Option B: Custom Headers
1. Create credential for "HTTP Header Auth"
2. Add headers:
   - **Authorization**: `Basic base64(api_key:api_secret)`

### 2. Store Private Key
1. Create folder: `credentials/` in your project
2. Copy the downloaded private key to: `credentials/vonage_private.key`
3. Ensure proper file permissions (600)

## 🌐 Webhook Setup

### 1. Expose Local n8n with ngrok
```bash
# Install ngrok if not already installed
ngrok http 5678

# Note the HTTPS URL (e.g., https://abc123.ngrok.io)
```

### 2. Update Vonage Application
1. Go back to your Vonage Application
2. Update webhook URLs:
   - **Inbound URL**: `https://your-ngrok-url.ngrok.io/webhook/whatsapp`
   - **Status URL**: `https://your-ngrok-url.ngrok.io/webhook/whatsapp/status`
3. Save changes

### 3. Configure n8n Webhook
In your n8n workflow:
1. **Webhook Trigger** node settings:
   - **HTTP Method**: POST
   - **Path**: whatsapp
   - **Response Mode**: Respond Immediately
   - **Response Code**: 200

## 📨 Message Format

### Receiving Messages (Webhook Payload)
Vonage sends webhooks in this format:
```json
{
  "message_uuid": "unique-message-id",
  "to": "your-whatsapp-number",
  "from": "sender-whatsapp-number", 
  "timestamp": "2025-01-11T10:00:00Z",
  "message_type": "text",
  "text": "User message content",
  "channel": "whatsapp"
}
```

### Sending Messages (API Request)
Send messages using this format:
```json
{
  "from": "your-whatsapp-number",
  "to": "recipient-whatsapp-number",
  "message_type": "text",
  "text": "Response message content",
  "channel": "whatsapp"
}
```

## 🔧 n8n Integration

### HTTP Request Node Configuration
For sending messages:

1. **Method**: POST
2. **URL**: `https://api.nexmo.com/v1/messages`
3. **Authentication**: HTTP Basic Auth (using Vonage credentials)
4. **Headers**:
   ```
   Content-Type: application/json
   ```
5. **Body**:
   ```json
   {
     "from": "{{$env.VONAGE_WHATSAPP_NUMBER}}",
     "to": "{{$json.whatsapp.from}}",
     "message_type": "text", 
     "text": "{{$json.response}}",
     "channel": "whatsapp"
   }
   ```

### Authentication Setup
1. Use HTTP Basic Auth credential
2. Username: Vonage API Key
3. Password: Vonage API Secret

## 🧪 Testing Your Setup

### 1. Test Webhook Reception
1. Send a message to your WhatsApp sandbox number
2. Check n8n execution logs
3. Verify webhook payload is received correctly

### 2. Test Message Sending
1. Create a test workflow
2. Use HTTP Request node to send a message
3. Verify message appears in WhatsApp

### 3. Test Complete Flow
1. Send command via WhatsApp: "HELP"
2. Verify n8n workflow processes command
3. Check response appears in WhatsApp

## ⚠️ Common Issues

### Webhook Not Receiving Messages
- Check ngrok is running and URL is correct
- Verify webhook URL in Vonage application matches ngrok
- Check firewall/port settings
- Ensure n8n is accessible from internet

### Authentication Errors
- Verify API Key and Secret are correct
- Check Basic Auth encoding
- Ensure credentials are saved properly in n8n

### Message Delivery Failures
- Check phone numbers are in correct format (+1234567890)
- Verify WhatsApp Business number is approved
- Check message content doesn't violate WhatsApp policies
- Review Vonage dashboard for error logs

### Sandbox Limitations
- Limited to pre-approved phone numbers
- Cannot send to all users
- May have message limits
- Some features unavailable in sandbox

## 🔒 Security Best Practices

### 1. Webhook Security
- Use HTTPS for all webhook URLs
- Validate webhook signatures (if implemented)
- Implement rate limiting
- Log and monitor webhook activity

### 2. Credential Security
- Store API keys in environment variables
- Never commit credentials to version control
- Rotate API keys regularly
- Use least privilege access

### 3. Message Content
- Validate user input
- Sanitize responses
- Implement content filtering
- Respect user privacy

## 📊 Rate Limits and Quotas

### Vonage Limits:
- **Free tier**: Limited messages per month
- **API rate limits**: Varies by endpoint
- **WhatsApp limits**: Set by Meta/WhatsApp

### Best Practices:
- Implement exponential backoff
- Monitor usage in Vonage dashboard
- Cache responses when possible
- Batch operations when appropriate

## 🚀 Production Considerations

### 1. Business Verification
- Complete WhatsApp Business verification
- Submit required business documentation
- Wait for approval (can take weeks)

### 2. Phone Number Setup
- Use dedicated business phone number
- Verify number with WhatsApp
- Configure number in Vonage dashboard

### 3. Scaling
- Monitor message volumes
- Upgrade Vonage plan as needed
- Implement proper error handling
- Set up monitoring and alerting

## 🎯 Next Steps

Once Vonage is configured:
1. ✅ Test basic message sending/receiving
2. ✅ Import the complete n8n workflow
3. ✅ Update webhook URLs in workflow
4. ✅ Test all WhatsApp commands
5. ✅ Set up monitoring and logging

## 📞 Support Resources

- [Vonage Messages API Documentation](https://developer.vonage.com/messaging/overview)
- [WhatsApp Business API Guide](https://developer.vonage.com/messages/whatsapp)
- [Vonage Dashboard](https://dashboard.nexmo.com/)
- [WhatsApp Business Platform](https://business.whatsapp.com/)
