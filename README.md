# Social Media Content Management Workflow for n8n

This n8n workflow automates the entire social media content management process, from scheduling posts to generating weekly analytics reports.

## Table of Contents

- [Features](#features)
- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [Setup Guide](#setup-guide)
- [Workflow Overview](#workflow-overview)
- [Configuration](#configuration)
- [Troubleshooting](#troubleshooting)
- [Customization](#customization)
- [API Reference](#api-reference)
- [Support](#support)

## Features

- **Automated Post Scheduling**: Checks content calendar every 15 minutes and posts when scheduled time arrives
- **Facebook Integration**: Automatically creates posts on your Facebook page with text and media
- **Status Tracking**: Updates spreadsheet with post status and Facebook post IDs
- **Weekly Analytics**: Generates comprehensive reports with Facebook Insights data
- **Email Reporting**: Sends formatted weekly reports to managers
- **Google Sheets Integration**: Uses Google Sheets as the content calendar and reporting database

## Prerequisites

Before setting up this workflow, you'll need:

1. **n8n instance** (self-hosted or cloud)
2. **Google Sheets API** credentials
3. **Facebook App** with Page access
4. **Email service** (SMTP or cloud provider)

## Quick Start

1. **Import the workflow**: Use `social-media-workflow.json`
2. **Set environment variables** (see Configuration section)
3. **Configure credentials** for Google Sheets and Email
4. **Test the workflow** manually
5. **Activate** for automatic execution

## Setup Guide

### Step 1: Import Workflow

1. In n8n, go to **Workflows**
2. Click **"Import from File"**
3. Select `social-media-workflow.json`
4. Click **"Import"**

### Step 2: Set Environment Variables

Set these environment variables in your n8n instance:

```bash
# Google Sheets
GOOGLE_SHEETS_DOCUMENT_ID=your_document_id_here

# Facebook
FACEBOOK_PAGE_ID=your_facebook_page_id
FACEBOOK_ACCESS_TOKEN=your_facebook_access_token

# Email
MANAGER_EMAIL=manager@company.com
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your_email@gmail.com
SMTP_PASS=your_app_password
```

### Step 3: Configure Google Sheets Credentials

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project or select existing one
3. Enable Google Sheets API
4. Create Service Account credentials
5. Download JSON key file
6. In n8n: **Settings** → **Credentials** → **Add Credential** → **Google Sheets**
7. Upload the JSON key file

### Step 4: Configure Facebook App

1. Go to [Facebook Developers](https://developers.facebook.com/)
2. Create a new app or use existing one
3. Add **Facebook Login** product
4. Get **Page Access Token** with these permissions:
   - `pages_manage_posts` (for posting)
   - `pages_read_engagement` (for insights)
5. Note your **Facebook Page ID**

### Step 5: Configure Email Credentials

1. In n8n: **Settings** → **Credentials** → **Add Credential** → **Email Send**
2. Configure SMTP settings:
   - **Host**: Your SMTP server
   - **Port**: SMTP port (usually 587 or 465)
   - **User**: Your email username
   - **Password**: Your email password or app password

### Step 6: Update Workflow Credentials

1. Open the imported workflow
2. For each node, click the credential field and select your configured credentials:
   - **Google Sheets nodes**: Select your Google Sheets credential
   - **Email Send node**: Select your email credential
3. The HTTP Request nodes don't need separate credentials (they use environment variables)

### Step 7: Test the Workflow

#### Test Facebook Posting
1. Add a test post to your Google Sheets content calendar
2. Set status to "Scheduled"
3. Set time to a few minutes from now
4. Run the workflow manually
5. Check if the post appears on Facebook
6. Verify the spreadsheet status updates to "Posted"

#### Test Facebook Insights
1. Run the weekly analytics trigger manually
2. Check the execution logs for the HTTP Request node
3. Verify the response contains Facebook Insights data
4. Check if the weekly report generates in Google Sheets

### Step 8: Activate the Workflow

1. Click **"Activate"** in the workflow
2. The workflow will now run automatically:
   - **Content Check**: Every 15 minutes
   - **Weekly Report**: Every Monday at 9 AM

## Workflow Overview

### Main Content Publishing Flow
1. **Check Content Calendar** (Cron Trigger) - Runs every 15 minutes
2. **Read Content Calendar** - Fetches data from Google Sheets
3. **Filter Scheduled Posts** - Only processes posts with "Scheduled" status
4. **Check if Post Time Has Arrived** - Compares scheduled time with current time
5. **Create Facebook Post** - Posts to Facebook page via HTTP Request
6. **Update Status to Posted** - Changes status in spreadsheet
7. **Update Facebook Post ID** - Stores Facebook post ID for tracking

### Weekly Analytics Flow
1. **Weekly Analytics Trigger** (Cron Trigger) - Runs every Monday at 9 AM
2. **Get Facebook Page Insights** - Fetches analytics data via HTTP Request
3. **Read Posts from Past Week** - Gets posted content from spreadsheet
4. **Filter Weekly Posts** - Filters for posts from the past week
5. **Generate Weekly Report** - Creates report header in Google Sheets
6. **Append Post Details** - Adds post details to report
7. **Send Email Report** - Emails formatted report to manager

### Data Structure

#### Content Calendar Columns
- **A**: Date & Time (YYYY-MM-DD HH:MM format)
- **B**: Post Text (the actual post content)
- **C**: Media URL (optional image/video link)
- **D**: Status (Scheduled/Posted)
- **E**: Facebook Post ID (auto-populated after posting)

#### Weekly Report Structure
- Report header with generation date
- Page performance metrics
- Post count summary
- Detailed post information table

## Configuration

### Google Sheets Setup

Create a Google Sheet with the following structure:

**Sheet: "Content Calendar"**
| Column A | Column B | Column C | Column D | Column E |
|----------|----------|----------|----------|----------|
| Date & Time | Post Text | Media URL | Status | Facebook Post ID |
| 2024-01-15 09:00 | "Check out our new product!" | https://example.com/image.jpg | Scheduled | |
| 2024-01-16 14:30 | "Happy Monday everyone!" | | Scheduled | |

**Sheet: "Weekly Report"**
- This will be automatically populated by the workflow

### Environment Variables Reference

| Variable | Description | Example |
|----------|-------------|---------|
| `GOOGLE_SHEETS_DOCUMENT_ID` | Your Google Sheets document ID | `1BxiMVs0XRA5nFMdKvBdBZjgmUUqptlbs74OgvE2upms` |
| `FACEBOOK_PAGE_ID` | Your Facebook page ID | `123456789012345` |
| `FACEBOOK_ACCESS_TOKEN` | Your Facebook access token | `EAABwzLixnjYBO...` |
| `MANAGER_EMAIL` | Email for weekly reports | `manager@company.com` |

## Troubleshooting

### Common Issues

#### Facebook Posting Not Working
- **Error 401**: Check your access token and permissions
- **Error 403**: Ensure your app has `pages_manage_posts` permission
- **Error 400**: Verify your page ID is correct

#### Facebook Insights Not Working
- **Error 401**: Check your access token and permissions
- **Error 403**: Ensure your app has `pages_read_engagement` permission
- **No Data**: Check if your page has enough activity for insights

#### Google Sheets Errors
- **Permission Denied**: Verify service account has edit access
- **Sheet Not Found**: Check sheet names match exactly
- **Document ID Error**: Verify the document ID in environment variables

#### Email Not Sending
- **SMTP Error**: Check your email credentials and server settings
- **Authentication Failed**: Verify username/password or use app password

### Debug Steps

1. **Check Execution Logs**: Look at the detailed logs for each node
2. **Test API Calls**: Use Postman to test Facebook API calls manually
3. **Verify Credentials**: Check all credentials are properly configured
4. **Check Environment Variables**: Ensure all variables are set correctly

## Customization

### Timing Adjustments
- **Content Check Frequency**: Modify the cron expression in "Check Content Calendar" node
  - Current: Every 15 minutes (`*/15`)
  - Example: Every hour (`0 * * * *`)
- **Weekly Report Timing**: Change the cron expression in "Weekly Analytics Trigger" node
  - Current: Every Monday at 9 AM
  - Example: Every Friday at 5 PM (`0 17 * * 5`)

### Additional Platforms
To add other social media platforms:
1. Duplicate the Facebook posting nodes
2. Replace with appropriate platform nodes (Twitter, LinkedIn, Instagram)
3. Update status tracking accordingly

### Enhanced Analytics
Add more metrics by:
1. Expanding the Facebook Insights metrics array
2. Adding more data processing nodes
3. Including additional visualization options

### Modify Email Report Format
- Edit the "Send Email Report to Manager" node
- Customize the HTML message template
- Add more styling or information as needed

## API Reference

### Facebook Posting
```
POST https://graph.facebook.com/v18.0/{page-id}/feed
Headers: Authorization: Bearer {access-token}
Body: message, link, published
```

### Facebook Insights
```
GET https://graph.facebook.com/v18.0/{page-id}/insights
Headers: Authorization: Bearer {access-token}
Query: metric, period, since, until
```

### Common Facebook Metrics
- `page_impressions`: Total page views
- `page_engaged_users`: Users who engaged with your page
- `page_fan_adds`: New page followers
- `page_fan_removes`: Lost page followers
- `page_posts_impressions`: Post reach
- `page_actions_post_reactions_total`: Total reactions on posts

### Period Options
- `day`: Daily data
- `week`: Weekly data
- `days_28`: 28-day data
- `month`: Monthly data
- `quarter`: Quarterly data
- `year`: Yearly data

## Security Best Practices

1. **Store tokens securely** as environment variables
2. **Use minimal permissions** for your Facebook app
3. **Rotate access tokens** regularly
4. **Monitor API usage** to stay within Facebook's limits
5. **Audit workflow executions** regularly

## Monitoring and Maintenance

### Weekly Tasks
- Review workflow execution logs
- Check for failed executions
- Verify Facebook posts are publishing correctly
- Review weekly reports for accuracy

### Monthly Tasks
- Check Facebook app permissions
- Verify access tokens haven't expired
- Review and optimize workflow performance
- Update documentation if needed

### Quarterly Tasks
- Review Facebook API usage limits
- Assess workflow efficiency
- Plan any major modifications
- Backup workflow configurations

## Support

### Resources
- [n8n Documentation](https://docs.n8n.io/)
- [Google Sheets API Reference](https://developers.google.com/sheets/api)
- [Facebook Graph API Documentation](https://developers.facebook.com/docs/graph-api)
- [n8n Community Forum](https://community.n8n.io/)

### Getting Help
For issues or questions:
1. Check the troubleshooting section above
2. Review n8n execution logs
3. Consult the n8n community forum
4. Review API documentation for specific services

### Success Criteria

Your workflow is working correctly when:
- ✅ Posts publish automatically at scheduled times
- ✅ Facebook post IDs are captured and stored
- ✅ Weekly reports generate automatically
- ✅ Email reports are sent to managers
- ✅ No failed executions in the logs
- ✅ All nodes show green (successful) status

---

**Note**: This workflow is designed for production use but should be tested thoroughly in a development environment first. Always backup your data before implementing automation workflows.
