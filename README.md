<div align="center">
<a href="./README.md">中文</a> |
<a href="./readme/README_EN.md">English</a>
</div>
<br>

<div align="center">

</div>

Mail2Telegram
Mail2Telegram can monitor multiple email accounts in real-time and forward emails to Telegram. An extended feature supports extracting verification codes from emails and sending them to the clipboard.

Quick Start
Prerequisites
Using Gmail as an example:

Log in to Gmail and enable IMAP access in the settings.

If you have 2FA enabled, please refer to this link to get an app password.

After obtaining the app password, enter it for the PASSWORD in config.py.

(The process is similar for other email providers; please go to your email settings to enable IMAP access.)

Note: Due to changes in how Microsoft handles Outlook connections, Outlook email accounts cannot currently be used with this project. If needed, you can set up email forwarding from your Outlook account to an email account used by this project.

Deployment Steps
Clone the repository and enter the project directory:

git clone [https://github.com/Heavrnl/Mail2Telegram.git](https://github.com/Heavrnl/Mail2Telegram.git)
cd ./Mail2Telegram

Configure config.py:

Copy config-template.py and rename it to config.py.

Fill in the necessary configuration information.

EMAILS = [
    {
        'EMAIL': 'example@gmail.com',
        'PASSWORD': 'password/application password',
        'IMAP_SERVER': 'imap.gmail.com',
        'IMAP_SERVER_PORT': 993,
    },
    {
        'EMAIL': 'example@qq.com',
        'PASSWORD': 'password/application password',
        'IMAP_SERVER': 'imap.qq.com',
        'IMAP_SERVER_PORT': 993,
    },
    # You can add more email configurations...
]
TELEGRAM_BOT_TOKEN = 'BOT_TOKEN'
TELEGRAM_CHAT_ID = 'CHAT_ID'  # The chat_id where primary emails are forwarded, can be your own user_id
TELEGRAM_JUNK_CHAT_ID = 'CHAT_ID' # The chat_id where junk emails are forwarded. If not set (TELEGRAM_JUNK_CHAT_ID=''), junk email forwarding will be skipped.
RETRY_LIMIT = 5               # Number of retry attempts after a failure
RETRY_DELAY = 5               # Delay between retry attempts
RECONNECT_INTERVAL = 1800     # Interval for active disconnection and reconnection, in seconds
RETRY_PAUSE = 600             # Pause duration after multiple failed retries, in seconds

Start the service:

docker-compose up -d

When you receive a "Login successful" message from the Telegram bot, it means the service is running successfully.

Extended Features
Extract Email Verification Codes and Send to Clipboard
This supports extracting verification codes using local regex matching and AI (GitHub Models/Gemini). Specific configuration details are provided below.

Deploy the clipboard synchronization service Jeric-X/SyncClipboard. Please refer to that project for deployment instructions.

Deploy the verification code extraction service Heavrnl/ExtractVerificationCode.

git clone [https://github.com/Heavrnl/ExtractVerificationCode](https://github.com/Heavrnl/ExtractVerificationCode)

cd ExtractVerificationCode

Configure the .env file, filling in the relevant settings for the clipboard synchronization service you just deployed:

cp .env.example .env

# Select the API type to use: azure (GitHub Models) or gemini
API_TYPE=gemini

# Enable local regex matching for verification code extraction (If enabled, local matching is prioritized; the API is used as a fallback)
USE_LOCAL=false

# Prompt template
PROMPT_TEMPLATE=Extract the verification code from the following text. Output only the verification code, with no other text. If there is no verification code, output only 'None'.\n\nText: {input_text}\n\nVerification Code:

# Azure API related configuration
AZURE_ENDPOINT=[https://models.inference.ai.azure.com](https://models.inference.ai.azure.com)
AZURE_MODEL_NAME=gpt-4o-mini
# Azure API authentication token (authenticate using a GitHub Token)
GITHUB_TOKEN=

# Gemini API related configuration
GEMINI_API_KEY=
GEMINI_MODEL=gemini-1.5-flash

# Clipboard synchronization configuration
SYNC_URL=your_sync_url
SYNC_USERNAME=your_username
SYNC_TOKEN=your_token

# Enable debug mode (true/false)
DEBUG_MODE=false

Note: For the most accurate verification code extraction, it is recommended to use an AI model. Local regex matching may have inaccuracies.

Start the service:

docker-compose up -d

In our main project's docker-compose.yml file, change ENABLE_EVC=false to ENABLE_EVC=true.

Configure the tools/send_code.py file:

If the verification code extraction service is deployed on the same server as this project and uses the default port (5788), no changes are needed.

Otherwise, you need to modify the service address and port.

# Replace with the actual address of your ExtractVerificationCode application
url = 'http://evc:5788/evc'

Start:

docker-compose up -d

Regarding Privacy
The ExtractVerificationCode project takes the following security measures when processing email content:

Email Text Sanitization: Sensitive information is automatically removed before the text is sent to the AI model.

Content Filtering: Only emails containing content related to verification codes are sent to the AI model; not all emails are sent.

If you are still concerned about AI providers accessing your information, you can deploy a large language model locally or use only the local regex matching for verification code extraction.

Acknowledgments
Jeric-X/SyncClipboard - A cross-platform clipboard synchronization tool.

Donation
If you find this project helpful, feel free to buy me a coffee through the following link:
