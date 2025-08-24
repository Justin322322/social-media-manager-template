# Setup Guide for Fixed Social Media Workflow

## Overview

This guide covers setting up the `social-media-workflow-fixed.json` workflow, which uses only standard n8n nodes and is compatible with all n8n versions.

## What's Fixed

✅ **Facebook Posting**: Uses HTTP Request node instead of Facebook node  
✅ **Facebook Insights**: Uses HTTP Request node instead of Facebook Insights node  
✅ **No Custom Dependencies**: Works with any n8n installation  
✅ **Full Functionality**: All original features preserved  

## Prerequisites

- n8n instance (any version)
- Google account with Google Sheets access
- Facebook Business account with page access
- Email service (SMTP or cloud provider)

## Step 1: Import the Fixed Workflow

1. In n8n, go to **Workflows**
2. Click **"Import from File"**
3. Select `social-media-workflow-fixed.json`
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
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your_email@gmail.com
SMTP_PASS=your_app_password
```

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
   - `pages_manage_posts` (for posting)
   - `pages_read_engagement` (for insights)
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
3. The HTTP Request nodes don't need separate credentials (they use environment variables)

## Step 7: Test the Workflow

### Test Facebook Posting
1. Add a test post to your Google Sheets content calendar
2. Set status to "Scheduled"
3. Set time to a few minutes from now
4. Run the workflow manually
5. Check if the post appears on Facebook
6. Verify the spreadsheet status updates to "Posted"

### Test Facebook Insights
1. Run the weekly analytics trigger manually
2. Check the execution logs for the HTTP Request node
3. Verify the response contains Facebook Insights data
4. Check if the weekly report generates in Google Sheets

## Step 8: Activate the Workflow

1. Click **"Activate"** in the workflow
2. The workflow will now run automatically:
   - **Content Check**: Every 15 minutes
   - **Weekly Report**: Every Monday at 9 AM

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

## API Endpoints Used

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

## Customization Options

### Change Posting Frequency
- Modify the cron expression in "Check Content Calendar" node
- Current: Every 15 minutes (`*/15`)
- Example: Every hour (`0 * * * *`)

### Change Weekly Report Timing
- Modify the cron expression in "Weekly Analytics Trigger" node
- Current: Every Monday at 9 AM
- Example: Every Friday at 5 PM (`0 17 * * 5`)

### Add More Facebook Metrics
- Edit the "Get Facebook Page Insights via HTTP" node
- Add more metrics to the `metric` query parameter
- Update the report generation logic accordingly

### Modify Email Report Format
- Edit the "Send Email Report to Manager" node
- Customize the HTML message template
- Add more styling or information as needed

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

## Support Resources

- [n8n Documentation](https://docs.n8n.io/)
- [Facebook Graph API Documentation](https://developers.facebook.com/docs/graph-api)
- [Google Sheets API Reference](https://developers.google.com/sheets/api)
- [n8n Community Forum](https://community.n8n.io/)

## Success Criteria

Your workflow is working correctly when:
- ✅ Posts publish automatically at scheduled times
- ✅ Facebook post IDs are captured and stored
- ✅ Weekly reports generate automatically
- ✅ Email reports are sent to managers
- ✅ No failed executions in the logs
- ✅ All nodes show green (successful) status

---

**Need Help?** Check the troubleshooting section above or consult the n8n community forum.
