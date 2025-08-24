# Workflow Overview

How the Social Media Content Management Workflow operates and processes data.

## Workflow Structure

The workflow consists of two main flows that run independently:

1. **Content Publishing Flow** - Automatically publishes scheduled posts every 15 minutes
2. **Weekly Analytics Flow** - Generates and sends comprehensive weekly reports every Monday at 9:00 AM

## Main Content Publishing Flow

### 1. Content Calendar Check (Cron Trigger)
- **Frequency**: Every 15 minutes (`*/15`)
- **Purpose**: Triggers the workflow to check for scheduled content ready to publish
- **Node Type**: Cron Trigger
- **Position**: [300, 300]

### 2. Read Google Sheets Calendar
- **Action**: Fetches all scheduled content from the "Content Calendar" sheet
- **Range**: Columns A-E (ID, Date & Time, Post Text, Status, Facebook Post ID)
- **Options**: Unformatted values for accurate data processing
- **Node Type**: Google Sheets (Read)
- **Position**: [520, 300]

### 3. Filter Scheduled Posts
- **Condition**: Status equals "Scheduled"
- **Purpose**: Only processes posts that are ready to be published
- **Logic**: Ignores drafts and already published posts
- **Node Type**: If (Conditional)
- **Position**: [740, 300]

### 4. Check Publishing Time
- **Condition**: Scheduled time ≤ Current time
- **Purpose**: Ensures posts are only published when their scheduled time has arrived
- **Logic**: Compares timestamps for accurate timing
- **Node Type**: If (Conditional)
- **Position**: [960, 300]

### 5. Publish Facebook Post
- **Action**: Posts content to Facebook page via Graph API
- **Method**: POST to Facebook Graph API v18.0
- **Endpoint**: `/{{ $env.FACEBOOK_PAGE_ID }}/feed`
- **Data**: Post text, media URL, published status
- **Authentication**: Bearer token via HTTP header
- **Node Type**: HTTP Request
- **Position**: [1180, 300]

### 6. Update Status to Posted
- **Action**: Changes status from "Scheduled" to "Posted" in Google Sheets
- **Target**: Status column (D) with dynamic row reference
- **Purpose**: Tracks successful publications
- **Node Type**: Google Sheets (Update)
- **Position**: [1400, 300]

### 7. Store Facebook Post ID
- **Action**: Captures and stores the Facebook post ID for tracking
- **Target**: Facebook Post ID column (E) with dynamic row reference
- **Purpose**: Enables post tracking, analytics, and future reference
- **Node Type**: Google Sheets (Update)
- **Position**: [1620, 300]

### 8. Success Logging
- **Action**: Logs successful completion of content publishing workflow
- **Purpose**: Provides execution confirmation and debugging information
- **Node Type**: Code
- **Position**: [1840, 300]

## Weekly Analytics Flow

### 1. Weekly Analytics Trigger (Cron Trigger)
- **Frequency**: Every Monday at 9:00 AM
- **Cron Expression**: Week 1, Monday, 9:00 AM
- **Purpose**: Triggers comprehensive weekly report generation
- **Node Type**: Cron Trigger
- **Position**: [300, 800]

### 2. Get Facebook Page Insights
- **Action**: Fetches comprehensive analytics data from Facebook
- **Method**: GET from Facebook Insights API
- **Endpoint**: `/{{ $env.FACEBOOK_PAGE_ID }}/insights`
- **Metrics**: Page impressions, engaged users, follower changes
- **Period**: Weekly data for the past 7 days
- **Node Type**: HTTP Request
- **Position**: [520, 800]

### 3. Read Weekly Post History
- **Action**: Retrieves all posted content from the Content Calendar
- **Range**: All columns (A-E) for complete post data
- **Purpose**: Gathers comprehensive data for weekly report
- **Node Type**: Google Sheets (Read)
- **Position**: [740, 800]

### 4. Filter Weekly Posts
- **Conditions**: 
  - Date ≥ 7 days ago (past week)
  - Status = "Posted"
- **Purpose**: Identifies posts published in the last week for accurate reporting
- **Node Type**: If (Conditional)
- **Position**: [960, 800]

### 5. Generate Weekly Report
- **Action**: Creates comprehensive report header in Google Sheets
- **Sheet**: "Weekly Report" tab
- **Content**: Report title, generation date, performance metrics summary
- **Structure**: Formatted table with metrics and headers
- **Node Type**: Google Sheets (Create)
- **Position**: [1180, 800]

### 6. Append Post Details
- **Action**: Adds detailed post information to the report
- **Data**: Date & Time, Post Text, Facebook Post ID
- **Format**: Tabular data for easy reading and analysis
- **Dynamic**: Maps all filtered posts to report rows
- **Node Type**: Google Sheets (Append)
- **Position**: [1400, 800]

### 7. Send Email Report
- **Action**: Emails beautifully formatted HTML report to manager
- **Format**: Professional HTML with tables, styling, and metrics
- **Content**: Performance metrics, post details, and visual elements
- **Recipient**: Configured manager email address
- **Node Type**: Email Send
- **Position**: [1620, 800]

### 8. Analytics Completion Log
- **Action**: Logs successful completion of weekly analytics workflow
- **Purpose**: Confirms analytics generation and email delivery
- **Node Type**: Code
- **Position**: [1840, 800]

## Error Handling and Edge Cases

### No Scheduled Posts
- **Handling**: Workflow continues gracefully without errors
- **Action**: Logs informational message
- **Result**: No action required, workflow completes successfully

### Post Time Not Ready
- **Handling**: Waits for scheduled publication time
- **Action**: Logs waiting status
- **Result**: Will retry on next 15-minute cycle

### No Weekly Posts
- **Handling**: Still generates basic analytics report
- **Action**: Shows zero post count
- **Result**: Sends email with available page metrics

## Data Flow Diagram

```
Content Calendar (Google Sheets)
         ↓
   Cron Trigger (15 min)
         ↓
   Read Content Calendar
         ↓
   Filter Scheduled Posts
         ↓
   Check Publishing Time
         ↓
   Publish Facebook Post
         ↓
   Update Status & ID
         ↓
   Success Logging
         ↓
   Facebook Page
```

```
Weekly Analytics (Google Sheets)
         ↓
   Cron Trigger (Monday 9 AM)
         ↓
   Get Facebook Insights
         ↓
   Read Post History
         ↓
   Filter Weekly Posts
         ↓
   Generate Report
         ↓
   Append Post Details
         ↓
   Send Email Report
         ↓
   Analytics Logging
```

## Data Structure

### Content Calendar Columns

| Column | Name | Description | Format | Purpose |
|--------|------|-------------|---------|---------|
| A | ID | Unique identifier | Auto-increment | Row reference |
| B | Date & Time | Scheduled posting time | YYYY-MM-DD HH:MM | Timing control |
| C | Post Text | Content to post | Plain text | Post content |
| D | Status | Post status | Scheduled/Posted | Workflow control |
| E | Facebook Post ID | Facebook's post identifier | Auto-populated | Tracking & analytics |

### Weekly Report Structure

The weekly report is automatically generated with:

1. **Report Header**
   - Report title and generation date
   - Page performance summary with key metrics

2. **Performance Metrics**
   - Page impressions
   - Engaged users
   - New followers
   - Lost followers

3. **Post Summary**
   - Total posts published in the week
   - Detailed post information table
   - Date, content preview, and Facebook IDs

## Timing and Scheduling

### Content Publishing
- **Check Frequency**: Every 15 minutes
- **Posting**: Immediate when scheduled time arrives
- **Status Updates**: Real-time after successful posting
- **Error Handling**: Graceful failure with logging

### Weekly Reports
- **Generation**: Every Monday at 9:00 AM
- **Data Period**: Previous 7 days
- **Delivery**: Email sent immediately after generation
- **Format**: Professional HTML with comprehensive data

## Error Handling

### Automatic Retries
- Failed API calls are retried automatically by n8n
- Google Sheets operations have built-in retry logic
- HTTP requests include proper error handling

### Status Tracking
- Failed posts remain in "Scheduled" status
- Successful posts are marked as "Posted"
- Facebook post IDs are captured for tracking
- Comprehensive logging for debugging

### Monitoring
- Execution logs show success/failure for each step
- Failed executions can be reviewed and debugged
- Success confirmations for both workflows

## Performance Considerations

### API Rate Limits
- Facebook Graph API has rate limits (handled gracefully)
- Google Sheets API has quotas (optimized usage)
- Workflow includes appropriate error handling

### Resource Usage
- Minimal memory footprint
- Efficient data processing
- Optimized for production use
- Graceful handling of edge cases

## Security Features

### Credential Management
- Uses n8n's built-in credential system
- Environment variables for sensitive data
- No hardcoded secrets or tokens
- Secure token storage and rotation

### Access Control
- Minimal required permissions for Facebook API
- Service account isolation for Google Sheets
- Audit logging enabled for all operations
- Secure email delivery

## Customization Points

### Timing Adjustments
- Modify cron expressions for different frequencies
- Change weekly report day/time
- Adjust content check intervals
- Customize business hours scheduling

### Content Processing
- Add content validation rules
- Implement approval workflows
- Extend to multiple platforms
- Add content categorization

### Reporting
- Customize email templates
- Add more metrics and visualizations
- Include performance trends
- Customize report recipients

## Next Steps

- Review the [Setup Guide](setup.md) to get started
- Check [Configuration](configuration.md) for setup options
- Learn about [Customization](customization.md) possibilities
- Set up your [Google Sheets Template](google-sheets-template.md)
