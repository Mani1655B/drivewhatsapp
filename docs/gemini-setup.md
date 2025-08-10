# 🔧 Setup Guide: Google Gemini API

This guide walks you through setting up Google Gemini API for AI-powered document summarization in your WhatsApp Google Drive Assistant.

## 📋 Prerequisites
- Google account
- n8n instance running
- Basic understanding of API keys

## 🚀 Step-by-Step Setup

### 1. Get Gemini API Key
1. Go to [Google AI Studio](https://makersuite.google.com/app/apikey)
2. Sign in with your Google account
3. Click "Create API key"
4. Select "Create API key in new project" (or choose existing project)
5. **Copy and save the API key** - you'll only see it once!

### 2. Alternative: Using Google Cloud Console
If you prefer using Google Cloud Console:

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create or select a project
3. Enable the "Generative Language API"
4. Go to **APIs & Services** → **Credentials**
5. Click "Create Credentials" → "API key"
6. Copy the API key
7. (Optional) Restrict the API key to specific APIs

## 🔒 Environment Variables

Add this to your `.env` file:

```env
# Google Gemini API Configuration
GOOGLE_GEMINI_API_KEY=your_gemini_api_key_here
GEMINI_MODEL=gemini-1.5-flash-latest
GEMINI_MAX_TOKENS=500
GEMINI_TEMPERATURE=0.3
```

## 🔗 Configure n8n Gemini Credentials

### Option A: Google Gemini(PaLM) API Credential
If n8n has built-in Gemini support:

1. Go to **Settings** → **Credentials**
2. Click "Create new credential"
3. Search for "Google Gemini" or "Google PaLM API"
4. Fill in:
   - **Credential Name**: Google Gemini(PaLM) Api account
   - **API Key**: Your Gemini API key
5. Save the credential
6. **Note the Credential ID** for the workflow

### Option B: HTTP Request with API Key
If using HTTP Request nodes:

1. Create "HTTP Header Auth" credential
2. Add header:
   - **Name**: `Authorization`
   - **Value**: `Bearer YOUR_API_KEY`

Or use query parameter authentication:
1. No credential needed
2. Add API key as query parameter in the request

## 🔧 API Configuration

### Gemini API Endpoint
```
https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash-latest:generateContent
```

### Request Format
```json
{
  "contents": [
    {
      "parts": [
        {
          "text": "Your prompt text here"
        }
      ]
    }
  ],
  "generationConfig": {
    "temperature": 0.3,
    "maxOutputTokens": 500,
    "topP": 0.8,
    "topK": 40
  }
}
```

### Response Format
```json
{
  "candidates": [
    {
      "content": {
        "parts": [
          {
            "text": "Generated response text"
          }
        ],
        "role": "model"
      },
      "finishReason": "STOP",
      "index": 0
    }
  ]
}
```

## 🎯 n8n Integration

### HTTP Request Node Configuration
For AI summarization:

1. **Method**: POST
2. **URL**: `https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash-latest:generateContent`
3. **Authentication**: Predefined Credential Type → Google Gemini(PaLM) API
4. **Send Query Parameters**: True
5. **Query Parameters**:
   - **Name**: `key`
   - **Value**: `{{$credentials.googlePalmApi.apiKey}}`
6. **Headers**:
   ```
   Content-Type: application/json
   ```
7. **JSON Body**:
   ```json
   {
     "contents": [{
       "parts": [{
         "text": "Please provide a concise summary of this document content:\n\n{{$json.data}}\n\nSummary should be 2-3 sentences highlighting key points."
       }]
     }],
     "generationConfig": {
       "temperature": 0.3,
       "maxOutputTokens": 500
     }
   }
   ```

### Processing Gemini Response
Add a "Process Gemini Response" Function node:

```javascript
// Process Gemini API response and extract the generated text
const response = $input.first().json;

// Gemini API returns response in format: { candidates: [{ content: { parts: [{ text: "..." }] } }] }
let summary = '';

if (response.candidates && response.candidates.length > 0) {
  const candidate = response.candidates[0];
  if (candidate.content && candidate.content.parts && candidate.content.parts.length > 0) {
    summary = candidate.content.parts[0].text || '';
  }
}

// If no summary found, provide fallback
if (!summary) {
  summary = 'Unable to generate summary from the document content.';
}

// Return in format expected by downstream nodes
return {
  json: {
    summary: summary.trim(),
    model: 'gemini-1.5-flash',
    timestamp: new Date().toISOString()
  }
};
```

## 🧪 Testing Your Setup

### 1. Test API Access
Create a simple test workflow:

1. **Manual Trigger** node
2. **HTTP Request** node with Gemini configuration
3. **Set** node with test data:
   ```json
   {
     "data": "This is a test document about artificial intelligence and machine learning."
   }
   ```

### 2. Test Document Summarization
Test with actual document content:
- Upload a PDF to Google Drive
- Extract text content
- Send to Gemini for summarization
- Verify response quality

### 3. Test Complete Integration
1. Send WhatsApp command: "SUMMARY test.pdf"
2. Verify document is processed
3. Check Gemini generates summary
4. Confirm response sent to WhatsApp

## ⚠️ Common Issues

### API Key Authentication Errors
- Verify API key is correct and active
- Check API key restrictions in Google Cloud Console
- Ensure Generative Language API is enabled
- Try creating a new API key

### Request Format Errors
- Verify JSON body structure matches Gemini API format
- Check content encoding (UTF-8)
- Ensure proper escaping of special characters
- Validate JSON syntax

### Response Processing Errors
- Check response structure in n8n execution logs
- Verify candidate array exists and has content
- Handle cases where no response is generated
- Implement proper error handling

### Token Limit Exceeded
- Reduce document content size before sending
- Implement text chunking for large documents
- Adjust maxOutputTokens parameter
- Monitor token usage in requests

### Rate Limiting
- Implement exponential backoff
- Add delays between requests
- Monitor usage quotas
- Consider upgrading API plan

## 📊 Usage and Limits

### Free Tier Limits:
- **Requests per minute**: 60
- **Tokens per minute**: 32,000
- **Requests per day**: 1,500

### Model Capabilities:
- **Input context**: Up to 1M tokens
- **Output tokens**: Up to 8,192 tokens
- **Supported formats**: Text only
- **Languages**: 100+ languages supported

### Best Practices:
- Keep prompts concise and clear
- Use appropriate temperature settings (0.1-0.7)
- Implement proper error handling
- Cache responses when possible

## 🔒 Security Best Practices

### 1. API Key Security
- Store API keys in environment variables
- Never commit keys to version control
- Restrict API key access in Google Cloud Console
- Rotate keys regularly

### 2. Content Security
- Validate input content before sending
- Sanitize user-provided text
- Implement content filtering
- Respect privacy and confidentiality

### 3. Rate Limiting
- Implement client-side rate limiting
- Monitor API usage patterns
- Set up alerting for unusual activity
- Use appropriate retry strategies

## 🎛️ Configuration Options

### Generation Config Parameters:
- **temperature**: Controls randomness (0.0-1.0)
- **maxOutputTokens**: Maximum response length
- **topP**: Nucleus sampling parameter
- **topK**: Top-k sampling parameter
- **stopSequences**: Custom stop sequences

### Model Variants:
- **gemini-1.5-flash**: Fast, efficient for most tasks
- **gemini-1.5-pro**: More capable, slower
- **gemini-1.0-pro**: Legacy model

## 🚀 Advanced Features

### Custom Prompts
Tailor prompts for different document types:

```javascript
const getPromptForFileType = (mimeType, content) => {
  switch(mimeType) {
    case 'application/pdf':
      return `Summarize this PDF document, focusing on key findings and conclusions:\n\n${content}`;
    case 'application/vnd.google-apps.document':
      return `Provide a brief summary of this Google Doc, highlighting main points:\n\n${content}`;
    default:
      return `Summarize this document in 2-3 sentences:\n\n${content}`;
  }
};
```

### Error Handling
Implement robust error handling:

```javascript
try {
  const response = await makeGeminiRequest(content);
  return processSummary(response);
} catch (error) {
  if (error.status === 429) {
    return { error: 'Rate limit exceeded. Please try again later.' };
  } else if (error.status === 400) {
    return { error: 'Invalid content format. Please check document.' };
  }
  return { error: 'Unable to generate summary. Please try again.' };
}
```

## 🎯 Next Steps

Once Gemini API is configured:
1. ✅ Test basic text generation
2. ✅ Integrate with document processing workflow
3. ✅ Test with various document types
4. ✅ Optimize prompts for better summaries
5. ✅ Set up monitoring and error handling

## 📞 Support Resources

- [Google AI Studio](https://makersuite.google.com/)
- [Gemini API Documentation](https://ai.google.dev/docs)
- [Generative AI API Reference](https://ai.google.dev/api)
- [Google Cloud Generative AI](https://cloud.google.com/ai/generative-ai)
