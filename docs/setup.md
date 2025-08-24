# Setup Guide

Complete step-by-step setup instructions for the Social Media Content Management Workflow.

## Prerequisites

Before setting up this workflow, you'll need:

1. **n8n instance** (self-hosted or cloud)
2. **Google account** with Google Sheets access
3. **Facebook Business account** with page access
4. **Email service** (SMTP or cloud provider)

## Step 1: Import Workflow

1. In n8n, go to **Workflows**
2. Click **"Import from File"**
3. Select `social-media-workflow.json`
4. Click **"Import"**

## Step 2: Set Environment Variables

Set these environment variables in your n8n instance:

```bash
# Google Sheets
GOOGLE_SHEETS_DOCUMENT_ID=your_document_id_here

# Facebook
FACEBOOK_PAGE_ID=your_facebook_page_id
FACEBOOK_ACCESS_TOKEN=your_facebook_access_token

# Email
MANAGER_EMAIL=manager@company.com
```

**Note**: The workflow uses n8n's built-in email functionality, so additional SMTP configuration is handled through n8n's credential system.

## Step 3: Configure Google Sheets Credentials

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project or select existing one
3. Enable Google Sheets API
4. Create Service Account credentials
5. Download JSON key file
6. In n8n: **Settings** → **Credentials** → **Add Credential** → **Google Sheets**
7. Upload the JSON key file

## Step 4: Configure Facebook App

1. Go to [Facebook Developers](https://developers.facebook.com/)
2. Create a new app or use existing one
3. Add **Facebook Login** product
4. Get **Page Access Token** with these permissions:
   - `pages_manage_posts` (for posting content)
   - `pages_read_engagement` (for reading insights)
   - `read_insights` (for analytics data)
5. Note your **Facebook Page ID**

## Step 5: Configure Email Credentials

1. In n8n: **Settings** → **Credentials** → **Add Credential** → **Email Send**
2. Configure SMTP settings:
   - **Host**: Your SMTP server
   - **Port**: SMTP port (usually 587 or 465)
   - **User**: Your email username
   - **Password**: Your email password or app password

## Step 6: Update Workflow Credentials

1. Open the imported workflow
2. For each node, click the credential field and select your configured credentials:
   - **Google Sheets nodes**: Select your Google Sheets credential
   - **Email Send node**: Select your email credential
3. The HTTP Request nodes use environment variables for authentication

## Step 7: Set Up Google Sheets Structure

### Content Calendar Sheet
Create a sheet named "Content Calendar" with these columns:

| Column | Header | Description | Format |
|--------|--------|-------------|---------|
| A | ID | Unique identifier | Auto-increment |
| B | Date & Time | Scheduled posting time | YYYY-MM-DD HH:MM |
| C | Post Text | Content to post | Plain text |
| D | Status | Post status | Scheduled/Posted |
| E | Facebook Post ID | Facebook's post identifier | Auto-populated |

### Weekly Report Sheet
Create a sheet named "Weekly Report" - this will be automatically populated by the workflow.

## Step 8: Test the Workflow

### Test Facebook Posting
1. Add a test post to your Google Sheets content calendar
2. Set status to "Scheduled"
3. Set time to a few minutes from now
4. Run the workflow manually
5. Check if the post appears on Facebook
6. Verify the spreadsheet status updates to "Posted"
7. Confirm Facebook Post ID is captured

### Test Facebook Insights
1. Run the weekly analytics trigger manually
2. Check the execution logs for the HTTP Request node
3. Verify the response contains Facebook Insights data
4. Check if the weekly report generates in Google Sheets
5. Verify email delivery to manager

## Step 9: Activate the Workflow

1. Click **"Activate"** in the workflow
2. The workflow will now run automatically:
   - **Content Check**: Every 15 minutes
   - **Weekly Report**: Every Monday at 9:00 AM

## Success Criteria

Your workflow is working correctly when:
- ✅ Posts publish automatically at scheduled times
- ✅ Facebook post IDs are captured and stored
- ✅ Weekly reports generate automatically
- ✅ Email reports are sent to managers
- ✅ No failed executions in the logs
- ✅ All nodes show green (successful) status
- ✅ Status columns update correctly in Google Sheets

## Workflow Features

### Content Publishing
- **Automated Scheduling**: Checks every 15 minutes for scheduled posts
- **Time Validation**: Only publishes when scheduled time arrives
- **Status Tracking**: Updates post status from "Scheduled" to "Posted"
- **Post ID Storage**: Captures Facebook post IDs for analytics

### Weekly Analytics
- **Comprehensive Reporting**: Generates detailed weekly reports
- **Performance Metrics**: Tracks impressions, engagement, and followers
- **Post History**: Includes all posts from the past week
- **Professional Email**: Sends formatted HTML reports to managers

### Error Handling
- **Graceful Failures**: Handles edge cases without workflow interruption
- **Comprehensive Logging**: Provides detailed execution information
- **Status Preservation**: Failed posts remain in "Scheduled" status
- **Retry Logic**: Automatic retries for failed operations

## Next Steps

- Review the [Workflow Overview](workflow-overview.md) to understand how it works
- Check [Configuration](configuration.md) for detailed setup options
- Set up your [Google Sheets Template](google-sheets-template.md)
- Learn about [Customization](customization.md) options
