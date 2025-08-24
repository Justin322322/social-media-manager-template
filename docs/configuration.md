# Configuration

Detailed configuration guide for environment variables, credentials, and Google Sheets setup.

## Environment Variables

Set these environment variables in your n8n instance:

### Required Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `GOOGLE_SHEETS_DOCUMENT_ID` | Your Google Sheets document ID | `1BxiMVs0XRA5nFMdKvBdBZjgmUUqptlbs74OgvE2upms` |
| `FACEBOOK_PAGE_ID` | Your Facebook page ID | `123456789012345` |
| `FACEBOOK_ACCESS_TOKEN` | Your Facebook access token | `EAABwzLixnjYBO...` |
| `MANAGER_EMAIL` | Email for weekly reports | `manager@company.com` |

**Note**: The workflow uses n8n's built-in email functionality, so SMTP configuration is handled through n8n's credential system rather than environment variables.

## Google Sheets Setup

### 1. Create New Google Sheet

1. Go to [Google Sheets](https://sheets.google.com)
2. Click "Blank" to create a new spreadsheet
3. Rename it to "Social Media Content Calendar"

### 2. Set Up Content Calendar Tab

1. Rename the first tab to "Content Calendar"
2. Add the column headers in row 1:
   - **A**: ID
   - **B**: Date & Time
   - **C**: Post Text
   - **D**: Status
   - **E**: Facebook Post ID
3. Format headers with bold text and background color
4. Freeze the first row for easy navigation

### 3. Set Up Weekly Report Tab

1. Click the "+" button to add a new tab
2. Rename it to "Weekly Report"
3. This will be automatically populated by the workflow

### 4. Configure Data Validation (Optional)

For the Status column:
1. Select column D (Status)
2. Go to Data → Data validation
3. Set criteria to "List of items"
4. Enter: `Scheduled,Posted,Draft`
5. Check "Show validation help text"

### 5. Format Date Column

1. Select column B (Date & Time)
2. Go to Format → Number → Date time
3. Choose format: "12/31/2023 23:59"

### 6. Set Up Conditional Formatting (Optional)

For the Status column:
1. Select column D
2. Go to Format → Conditional formatting
3. Add rule: "Text is exactly 'Scheduled'" → Blue background
4. Add rule: "Text is exactly 'Posted'" → Green background

## Google Cloud Console Setup

### 1. Create Project

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Click "Select a project" → "New Project"
3. Enter project name: "Social Media Workflow"
4. Click "Create"

### 2. Enable APIs

1. Go to "APIs & Services" → "Library"
2. Search for "Google Sheets API"
3. Click on it and click "Enable"

### 3. Create Service Account

1. Go to "APIs & Services" → "Credentials"
2. Click "Create Credentials" → "Service Account"
3. Enter service account name: "n8n-workflow"
4. Click "Create and Continue"
5. Skip role assignment (click "Continue")
6. Click "Done"

### 4. Generate JSON Key

1. Click on your service account
2. Go to "Keys" tab
3. Click "Add Key" → "Create new key"
4. Choose "JSON" format
5. Click "Create"
6. Download the JSON file

### 5. Share Google Sheet

1. Open your Google Sheet
2. Click "Share" button in top right
3. Add your service account email (from the JSON file)
4. Give "Editor" permissions
5. Uncheck "Notify people"

## Facebook App Setup

### 1. Create Facebook App

1. Go to [Facebook Developers](https://developers.facebook.com/)
2. Click "My Apps" → "Create App"
3. Choose "Business" type
4. Enter app name and contact email
5. Click "Create App"

### 2. Add Facebook Login

1. In your app dashboard, click "Add Product"
2. Find "Facebook Login" and click "Set Up"
3. Choose "Web" platform
4. Enter your website URL (can be placeholder)
5. Click "Save and Continue"

### 3. Configure App Settings

1. Go to "App Settings" → "Basic"
2. Note your App ID and App Secret
3. Add your domain to "App Domains"
4. Save changes

### 4. Get Page Access Token

1. Go to "Tools" → "Graph API Explorer"
2. Select your app from dropdown
3. Click "Generate Access Token"
4. Request these permissions:
   - `pages_manage_posts` (for posting content)
   - `pages_read_engagement` (for reading insights)
   - `read_insights` (for analytics data)
5. Click "Generate Access Token"
6. Copy the token

### 5. Get Page ID

1. Go to your Facebook page
2. Click "About" in the left sidebar
3. Scroll down to find "Page ID"
4. Copy the number

## Email Configuration

### Gmail Setup

1. Enable 2-factor authentication on your Google account
2. Generate an App Password:
   - Go to Google Account settings
   - Security → 2-Step Verification → App passwords
   - Generate password for "Mail"
3. Use this app password in your n8n email credential configuration

### Outlook Setup

1. Use these settings:
   - Host: `smtp-mail.outlook.com`
   - Port: `587`
   - Security: `STARTTLS`
   - Username: Your email address
   - Password: Your account password

### Custom SMTP

1. Get SMTP settings from your email provider
2. Common ports: 25, 465, 587
3. Test connection before using in production

## Workflow Configuration

### Cron Triggers

The workflow includes two automated triggers:

1. **Content Publishing Trigger**
   - Frequency: Every 15 minutes
   - Expression: `*/15 * * * *`
   - Purpose: Check for scheduled posts

2. **Weekly Analytics Trigger**
   - Frequency: Every Monday at 9:00 AM
   - Expression: `0 9 * * 1`
   - Purpose: Generate weekly reports

### Node Positions

The workflow is organized with logical positioning:
- **Content Flow**: Nodes positioned around [300-1840, 300]
- **Analytics Flow**: Nodes positioned around [300-1840, 800]
- **Documentation**: Sticky notes positioned for easy reference

## Security Best Practices

### 1. Store Credentials Securely

- Use environment variables for sensitive data
- Never commit credentials to version control
- Use n8n's built-in credential management
- Store Facebook tokens securely

### 2. Minimal Permissions

- Facebook app: Only request needed permissions
- Google service account: Only give access to specific sheets
- Email: Use app passwords instead of main passwords

### 3. Regular Rotation

- Rotate Facebook access tokens monthly
- Update Google service account keys quarterly
- Change email passwords regularly

### 4. Monitor Access

- Enable audit logging in n8n
- Monitor Facebook app usage
- Check Google Cloud Console logs
- Review workflow execution history

## Testing Configuration

### 1. Test Google Sheets

1. Try to read from your sheet
2. Verify you can write to the sheet
3. Check that sheet names match exactly
4. Test column structure matches requirements

### 2. Test Facebook API

1. Use Graph API Explorer to test endpoints
2. Verify your access token works
3. Test posting to a test page first
4. Verify insights API access

### 3. Test Email

1. Send a test email through n8n
2. Verify it arrives correctly
3. Check spam folders if needed
4. Test HTML formatting

## Troubleshooting

### Common Issues

- **Permission Denied**: Check service account permissions
- **Invalid Token**: Verify Facebook access token hasn't expired
- **Sheet Not Found**: Ensure sheet names match exactly
- **SMTP Error**: Check firewall and port settings
- **Column Mismatch**: Verify column structure matches workflow

### Debug Steps

1. Check n8n execution logs
2. Test API calls manually
3. Verify all environment variables are set
4. Check credential configurations
5. Review workflow node connections

## Next Steps

- Complete the [Setup Guide](setup.md) to import and configure the workflow
- Review [Workflow Overview](workflow-overview.md) to understand the flow
- Set up your [Google Sheets Template](google-sheets-template.md)
