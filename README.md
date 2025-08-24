# Social Media Content Management Workflow for n8n

This n8n workflow automates the entire social media content management process, from scheduling posts to generating weekly analytics reports.

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

## Setup Instructions

### 1. Google Sheets Setup

Create a Google Sheet with the following structure:

**Sheet: "Content Calendar"**
| Column A | Column B | Column C | Column D | Column E |
|----------|----------|----------|----------|----------|
| Date & Time | Post Text | Media URL | Status | Facebook Post ID |
| 2024-01-15 09:00 | "Check out our new product!" | https://example.com/image.jpg | Scheduled | |
| 2024-01-16 14:30 | "Happy Monday everyone!" | | Scheduled | |

**Sheet: "Weekly Report"**
- This will be automatically populated by the workflow

### 2. Environment Variables

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

### 3. n8n Credentials Setup

#### Google Sheets Credentials
1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project or select existing one
3. Enable Google Sheets API
4. Create Service Account credentials
5. Download JSON key file
6. In n8n: Settings → Credentials → Add Credential → Google Sheets
7. Upload the JSON key file

#### Facebook Credentials
1. Go to [Facebook Developers](https://developers.facebook.com/)
2. Create a new app
3. Add Facebook Login product
4. Get Page Access Token
5. In n8n: Settings → Credentials → Add Credential → Facebook
6. Enter your access token

#### Email Credentials
1. In n8n: Settings → Credentials → Add Credential → Email Send
2. Configure SMTP settings or use cloud provider

### 4. Import Workflow

1. In n8n, go to Workflows
2. Click "Import from File"
3. Select the `social-media-workflow-fixed.json` file (this version uses only standard n8n nodes)
4. Update the credentials for each node
5. Set your environment variables (see below)

**Important**: Use the `social-media-workflow-fixed.json` file as it's compatible with all n8n versions and doesn't require additional node packages.

## Fixed Workflow Compatibility

This workflow has been updated to use only standard n8n nodes that are available in all versions:

- **Facebook Posting**: Uses HTTP Request node to call Facebook Graph API directly
- **Facebook Insights**: Uses HTTP Request node to call Facebook Insights API directly
- **No Custom Nodes Required**: Works with any n8n installation

### Files Available

- **`social-media-workflow-fixed.json`**: Main workflow file (use this one)
- **`social-media-workflow.json`**: Original version (requires Facebook nodes)
- **`social-media-workflow-updated.json`**: Intermediate version

## Workflow Flow

### Main Content Publishing Flow
1. **Check Content Calendar** (Cron Trigger) - Runs every 15 minutes
2. **Read Content Calendar** - Fetches data from Google Sheets
3. **Filter Scheduled Posts** - Only processes posts with "Scheduled" status
4. **Check if Post Time Has Arrived** - Compares scheduled time with current time
5. **Create Facebook Post** - Posts to Facebook page
6. **Update Status to Posted** - Changes status in spreadsheet
7. **Update Facebook Post ID** - Stores Facebook post ID for tracking

### Weekly Analytics Flow
1. **Weekly Analytics Trigger** (Cron Trigger) - Runs every Monday at 9 AM
2. **Get Facebook Page Insights** - Fetches analytics data
3. **Read Posts from Past Week** - Gets posted content from spreadsheet
4. **Filter Weekly Posts** - Filters for posts from the past week
5. **Generate Weekly Report** - Creates report header in Google Sheets
6. **Append Post Details** - Adds post details to report
7. **Send Email Report** - Emails formatted report to manager

## Data Structure

### Content Calendar Columns
- **A**: Date & Time (YYYY-MM-DD HH:MM format)
- **B**: Post Text (the actual post content)
- **C**: Media URL (optional image/video link)
- **D**: Status (Scheduled/Posted)
- **E**: Facebook Post ID (auto-populated after posting)

### Weekly Report Structure
- Report header with generation date
- Page performance metrics
- Post count summary
- Detailed post information table

## Customization Options

### Timing Adjustments
- **Content Check Frequency**: Modify the cron expression in "Check Content Calendar" node
- **Weekly Report Timing**: Change the cron expression in "Weekly Analytics Trigger" node

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

## Troubleshooting

### Common Issues

1. **Posts not publishing**
   - Check Facebook credentials and permissions
   - Verify page ID is correct
   - Ensure access token has page posting permissions

2. **Google Sheets errors**
   - Verify service account has edit access to the sheet
   - Check sheet names match exactly
   - Ensure document ID is correct

3. **Email not sending**
   - Verify SMTP credentials
   - Check firewall/network restrictions
   - Ensure email service allows SMTP access

### Debug Mode
Enable debug mode in n8n to see detailed execution logs and identify issues.

## Monitoring

### Workflow Execution
- Monitor workflow execution in n8n dashboard
- Check execution history for failed runs
- Review logs for error details

### Performance Metrics
- Track execution time for each node
- Monitor API rate limits
- Check resource usage

## Security Considerations

1. **Access Tokens**: Store sensitive credentials securely
2. **API Permissions**: Use minimal required permissions
3. **Data Privacy**: Ensure compliance with data protection regulations
4. **Audit Logging**: Monitor access and changes

## Additional Resources

- [n8n Documentation](https://docs.n8n.io/)
- [Google Sheets API Reference](https://developers.google.com/sheets/api)
- [Facebook Graph API Documentation](https://developers.facebook.com/docs/graph-api)
- [n8n Community Forum](https://community.n8n.io/)

## Support

For issues or questions:
1. Check the troubleshooting section above
2. Review n8n execution logs
3. Consult the n8n community forum
4. Review API documentation for specific services

---

**Note**: This workflow is designed for production use but should be tested thoroughly in a development environment first. Always backup your data before implementing automation workflows.
