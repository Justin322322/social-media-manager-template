# Quick Setup Checklist

## Pre-Setup Requirements
- [ ] n8n instance running (self-hosted or cloud)
- [ ] Google account with access to Google Sheets
- [ ] Facebook Business account with page access
- [ ] Email service (Gmail, Outlook, or SMTP provider)

## Step 1: Google Sheets Setup
- [ ] Create new Google Sheet named "Social Media Content Calendar"
- [ ] Create "Content Calendar" tab with columns: Date & Time, Post Text, Media URL, Status, Facebook Post ID
- [ ] Create "Weekly Report" tab
- [ ] Set up Google Cloud Project and enable Sheets API
- [ ] Create Service Account and download JSON key
- [ ] Share sheet with service account email (Editor access)
- [ ] Copy document ID from URL

## Step 2: Facebook App Setup
- [ ] Go to [Facebook Developers](https://developers.facebook.com/)
- [ ] Create new app or use existing one
- [ ] Add Facebook Login product
- [ ] Get Page Access Token
- [ ] Note your Facebook Page ID

## Step 3: Email Setup
- [ ] Choose email provider (Gmail, Outlook, etc.)
- [ ] Get SMTP credentials or API keys
- [ ] Test email sending capability

## Step 4: n8n Configuration
- [ ] Set environment variables:
  - [ ] `GOOGLE_SHEETS_DOCUMENT_ID`
  - [ ] `FACEBOOK_PAGE_ID`
  - [ ] `FACEBOOK_ACCESS_TOKEN`
  - [ ] `MANAGER_EMAIL`
  - [ ] SMTP settings
- [ ] Import workflow JSON file
- [ ] Configure credentials for each service
- [ ] Update document ID in workflow nodes

## Step 5: Testing
- [ ] Test Google Sheets connection
- [ ] Test Facebook posting (use test page first)
- [ ] Test email sending
- [ ] Add test post to calendar
- [ ] Run workflow manually to verify
- [ ] Check all status updates

## Step 6: Go Live
- [ ] Activate workflow
- [ ] Monitor first few executions
- [ ] Verify posts are publishing correctly
- [ ] Check weekly report generation
- [ ] Set up monitoring and alerts

## Step 7: Optimization
- [ ] Review execution logs
- [ ] Optimize timing if needed
- [ ] Add error handling
- [ ] Set up backup procedures
- [ ] Document any customizations

## Security Checklist
- [ ] Store credentials securely
- [ ] Use minimal required permissions
- [ ] Enable audit logging
- [ ] Regular credential rotation
- [ ] Monitor access logs

## Mobile Setup
- [ ] Install Google Sheets app
- [ ] Test mobile editing
- [ ] Set up mobile notifications
- [ ] Train team on mobile usage

## Troubleshooting Prepared
- [ ] Know how to check n8n logs
- [ ] Have API documentation bookmarked
- [ ] Know how to reset credentials
- [ ] Have backup workflow ready
- [ ] Know emergency contact procedures

## Documentation
- [ ] Save workflow JSON backup
- [ ] Document custom configurations
- [ ] Create team training materials
- [ ] Set up knowledge base
- [ ] Document troubleshooting steps

---

## Success Criteria
- [ ] Posts publish automatically at scheduled times
- [ ] Status updates correctly in spreadsheet
- [ ] Weekly reports generate automatically
- [ ] Email reports sent to manager
- [ ] No failed executions in first week
- [ ] Team can easily add/edit content

## Maintenance Schedule
- [ ] Weekly: Review execution logs
- [ ] Monthly: Check credential expiration
- [ ] Quarterly: Review and optimize workflow
- [ ] Annually: Update documentation and procedures

---

**Need Help?** Check the README.md file for detailed instructions or consult the n8n community forum.
