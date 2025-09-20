![img](./logo/logo-title.png)

<div align="center">
  <a href="./README.md">English</a> |
  <a href="./readme/README_CN.md">中文</a>
</div>
<br>

<div align="center">

![Docker](https://img.shields.io/badge/-Docker-2496ED?style=flat-square&logo=docker&logoColor=white) [![License: GPL-3.0](https://img.shields.io/badge/License-GPL%203.0-4CAF50?style=flat-square)](https://github.com/Heavrnl/mail2telegram/blob/master/LICENSE) 

</div>

# Mail2Telegram

Mail2Telegram is a real-time email monitoring service that forwards emails from multiple email accounts to Telegram. It includes advanced features like verification code extraction and clipboard synchronization for enhanced productivity.

## ✨ Features

- **Multi-Account Support**: Monitor multiple email accounts simultaneously
- **Real-Time Forwarding**: Instant email notifications to Telegram
- **Smart Filtering**: Separate handling for inbox and junk mail
- **Verification Code Extraction**: Automatically extract and sync verification codes to clipboard
- **HTML Content Processing**: Clean conversion of HTML emails to readable text
- **Robust Error Handling**: Automatic reconnection and retry mechanisms
- **Timezone Support**: Configurable timezone display
- **Docker Support**: Easy deployment with Docker Compose

## 📋 Prerequisites

Before you begin, ensure you have the following:

- Docker and Docker Compose installed
- A Telegram bot token and chat ID
- Email accounts with IMAP access enabled
- Application passwords for email accounts (if 2FA is enabled)

## 🚀 Quick Start

### Step 1: Prepare Your Email Accounts

#### For Gmail:
1. **Enable IMAP Access**:
   - Go to Gmail Settings → Forwarding and POP/IMAP
   - Enable IMAP access
   
2. **Generate App Password** (if 2FA is enabled):
   - Go to Google Account settings
   - Navigate to Security → 2-Step Verification → App passwords
   - Generate a new app password for "Mail"
   - Use this password in the configuration (not your regular Gmail password)

#### For Other Email Providers:
- **Yahoo Mail**: Enable IMAP in Settings → More Settings → Mailboxes
- **Outlook/Hotmail**: Currently not supported due to Microsoft's authentication changes
- **QQ Mail**: Enable IMAP/SMTP service in Settings → Account
- **163/126 Mail**: Enable IMAP service and generate authorization code

### Step 2: Set Up Telegram Bot

1. **Create a Telegram Bot**:
   - Message [@BotFather](https://t.me/botfather) on Telegram
   - Send `/newbot` and follow the instructions
   - Save the bot token (format: `123456789:ABCdefGHIjklMNOpqrsTUVwxyz`)

2. **Get Your Chat ID**:
   - Start a conversation with your bot
   - Send any message to the bot
   - Visit `https://api.telegram.org/bot<YOUR_BOT_TOKEN>/getUpdates`
   - Find your chat ID in the response (usually a number like `123456789`)

### Step 3: Clone and Configure

1. **Clone the Repository**:
```bash
git clone https://github.com/Heavrnl/Mail2Telegram.git
cd Mail2Telegram
```

2. **Create Configuration File**:
```bash
cp config-template.py config.py
```

3. **Edit Configuration** (`config.py`):
```python
EMAILS = [
    {
        'EMAIL': 'your-email@gmail.com',
        'PASSWORD': 'your-app-password',  # Use app password, not regular password
        'IMAP_SERVER': 'imap.gmail.com',
        'IMAP_SERVER_PORT': 993,
    },
    {
        'EMAIL': 'another-email@qq.com',
        'PASSWORD': 'your-password-or-auth-code',
        'IMAP_SERVER': 'imap.qq.com',
        'IMAP_SERVER_PORT': 993,
    },
    # Add more email accounts as needed
]

# Telegram Configuration
TELEGRAM_BOT_TOKEN = 'your-bot-token-here'
TELEGRAM_CHAT_ID = 'your-chat-id-here'  # Main chat for regular emails
TELEGRAM_JUNK_CHAT_ID = 'your-junk-chat-id-here'  # Optional: separate chat for junk mail

# Connection Settings
RETRY_LIMIT = 5  # Number of retry attempts after failure
RETRY_DELAY = 5  # Delay between retries (seconds)
RECONNECT_INTERVAL = 1800  # Proactive reconnection interval (seconds)
RETRY_PAUSE = 600  # Pause after multiple failures (seconds)
```

### Step 4: Configure Environment (Optional)

Edit `docker-compose.yml` to customize settings:

```yaml
environment:
  - LANGUAGE=English  # or Chinese
  - TIMEZONE=America/New_York  # Set your timezone
  - ENABLE_LOGGING=true  # Enable detailed logging
  - ENABLE_EVC=false  # Enable verification code extraction (see advanced features)
```

### Step 5: Deploy

1. **Start the Service**:
```bash
docker-compose up -d
```

2. **Verify Deployment**:
```bash
docker-compose logs -f mail2telegram
```

3. **Test the Setup**:
   - Send a test email to one of your configured accounts
   - You should receive a "Successfully logged in" message from your Telegram bot
   - The test email should appear in your Telegram chat

## 🔧 Advanced Features

### Verification Code Extraction & Clipboard Sync

This feature automatically extracts verification codes from emails and syncs them to your clipboard across devices.

#### Step 1: Deploy Clipboard Sync Service

1. **Set up SyncClipboard**:
   - Follow the deployment guide at [Jeric-X/SyncClipboard](https://github.com/Jeric-X/SyncClipboard)
   - Note down your sync URL, username, and token

#### Step 2: Deploy Verification Code Extraction Service

1. **Clone the extraction service**:
```bash
git clone https://github.com/Heavrnl/ExtractVerificationCode
cd ExtractVerificationCode
```

2. **Configure the service**:
```bash
cp .env.example .env
```

3. **Edit `.env` file**:
```ini
# API Configuration - Choose one
API_TYPE=gemini  # Options: azure (GitHub Models) or gemini

# Local Processing (optional)
USE_LOCAL=false  # Try local regex first, then API if it fails

# Prompt Template
PROMPT_TEMPLATE=Extract the verification code from the following text. Output only the code, without any other text. If there is no verification code, output 'None'.\n\nText: {input_text}\n\nCode:

# GitHub Models (Azure) Configuration
AZURE_ENDPOINT=https://models.inference.ai.azure.com
AZURE_MODEL_NAME=gpt-4o-mini
GITHUB_TOKEN=your-github-token-here

# Gemini Configuration
GEMINI_API_KEY=your-gemini-api-key-here
GEMINI_MODEL=gemini-1.5-flash

# Clipboard Sync Configuration
SYNC_URL=your-syncclipboard-url
SYNC_USERNAME=your-syncclipboard-username
SYNC_TOKEN=your-syncclipboard-token

# Debug Mode
DEBUG_MODE=false
```

4. **Start the extraction service**:
```bash
docker-compose up -d
```

#### Step 3: Enable in Mail2Telegram

1. **Update Mail2Telegram configuration**:
   - Edit `docker-compose.yml`
   - Change `ENABLE_EVC=false` to `ENABLE_EVC=true`

2. **Configure service connection** (if needed):
   - Edit `tools/extract_verification_code.py`
   - Update the URL if your extraction service runs on a different host/port:
   ```python
   url = 'http://your-extraction-service:5788/evc'
   ```

3. **Restart Mail2Telegram**:
```bash
docker-compose up -d
```

## 📊 Monitoring and Logs

### View Logs
```bash
# View real-time logs
docker-compose logs -f mail2telegram

# View logs from specific time
docker-compose logs --since="2024-01-01T00:00:00" mail2telegram
```

### Log Configuration
Logs are automatically rotated with the following defaults:
- Maximum file size: 5MB
- Number of backup files: 5
- Location: `./log/email_checker.log`

## 🔍 Troubleshooting

### Common Issues

1. **"Login failed" errors**:
   - Verify IMAP is enabled for your email account
   - Use app passwords instead of regular passwords for accounts with 2FA
   - Check if your email provider requires specific security settings

2. **"Connection timeout" errors**:
   - Verify IMAP server and port settings
   - Check firewall settings
   - Ensure Docker has internet access

3. **Telegram messages not received**:
   - Verify bot token is correct
   - Ensure chat ID is correct (try sending `/start` to your bot first)
   - Check if bot is blocked or chat is deleted

4. **Verification code extraction not working**:
   - Ensure the extraction service is running (`docker ps`)
   - Check API keys are valid
   - Verify network connectivity between services

### Debug Mode

Enable detailed logging by setting `ENABLE_LOGGING=true` in docker-compose.yml:

```yaml
environment:
  - ENABLE_LOGGING=true
  - DEBUG_MODE=true  # For extraction service
```

## 🔒 Privacy and Security

### Email Content Processing
- HTML emails are converted to clean text
- Sensitive information is filtered before AI processing
- Only verification code-related content is sent to AI models
- Full email content is never stored permanently

### AI Processing (for verification codes)
- Content is desensitized before sending to AI models
- Only relevant text snippets are processed
- You can use local regex matching to avoid AI processing entirely
- Consider self-hosting AI models for maximum privacy

### Data Storage
- No email content is permanently stored
- Logs can be disabled if needed
- All processing happens in real-time

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## 📄 License

This project is licensed under the GNU General Public License v3.0 - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- [Jeric-X/SyncClipboard](https://github.com/Jeric-X/SyncClipboard) - Cross-platform clipboard synchronization
- [imaplib2](https://github.com/jazzband/imaplib2) - Enhanced IMAP client library

## ☕ Support

If you find this project helpful, consider buying me a coffee:

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/0heavrnl)

## 📞 Support

For issues and questions:
- Create an issue on GitHub
- Check the troubleshooting section above
- Review the logs for error details

---

**Note**: This project is designed for personal use. Please ensure you comply with your email provider's terms of service and local privacy laws.