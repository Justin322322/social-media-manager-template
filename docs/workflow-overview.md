# Workflow Overview

How the Social Media Content Management Workflow operates and processes data.

## Workflow Structure

The workflow consists of two main flows that run independently:

1. **Content Publishing Flow** - Automatically publishes scheduled posts
2. **Weekly Analytics Flow** - Generates and sends weekly reports

## Main Content Publishing Flow

### 1. Check Content Calendar (Cron Trigger)
- **Frequency**: Every 15 minutes
- **Purpose**: Triggers the workflow to check for new content
- **Node Type**: Cron Trigger

### 2. Read Content Calendar
- **Action**: Fetches data from Google Sheets "Content Calendar" tab
- **Range**: Columns A-D (Date & Time, Post Text, Media URL, Status)
- **Node Type**: Google Sheets (Read)

### 3. Filter Scheduled Posts
- **Condition**: Status equals "Scheduled"
- **Purpose**: Only processes posts that are ready to be published
- **Node Type**: If (Conditional)

### 4. Check if Post Time Has Arrived
- **Condition**: Scheduled time ≤ Current time
- **Purpose**: Ensures posts are only published when their time has come
- **Node Type**: If (Conditional)

### 5. Create Facebook Post
- **Action**: Posts content to Facebook page via HTTP Request
- **Method**: POST to Facebook Graph API
- **Data**: Post text, media URL, published status
- **Node Type**: HTTP Request

### 6. Update Status to Posted
- **Action**: Changes status from "Scheduled" to "Posted" in Google Sheets
- **Column**: Status column (D)
- **Node Type**: Google Sheets (Update)

### 7. Update Facebook Post ID
- **Action**: Stores the Facebook post ID for tracking
- **Column**: Facebook Post ID column (E)
- **Purpose**: Enables post tracking and analytics
- **Node Type**: Google Sheets (Update)

## Weekly Analytics Flow

### 1. Weekly Analytics Trigger (Cron Trigger)
- **Frequency**: Every Monday at 9 AM
- **Purpose**: Triggers weekly report generation
- **Node Type**: Cron Trigger

### 2. Get Facebook Page Insights
- **Action**: Fetches analytics data from Facebook
- **Method**: GET from Facebook Insights API
- **Metrics**: Page impressions, engaged users, follower changes
- **Node Type**: HTTP Request

### 3. Read Posts from Past Week
- **Action**: Retrieves all posted content from the past week
- **Range**: All columns (A-E)
- **Purpose**: Gathers data for weekly report
- **Node Type**: Google Sheets (Read)

### 4. Filter Weekly Posts
- **Conditions**: 
  - Date ≥ 7 days ago
  - Status = "Posted"
- **Purpose**: Identifies posts published in the last week
- **Node Type**: If (Conditional)

### 5. Generate Weekly Report
- **Action**: Creates report header in Google Sheets
- **Sheet**: "Weekly Report" tab
- **Content**: Report title, date, performance metrics
- **Node Type**: Google Sheets (Create)

### 6. Append Post Details
- **Action**: Adds detailed post information to the report
- **Data**: Date, post text, Facebook post ID
- **Format**: Tabular data for easy reading
- **Node Type**: Google Sheets (Append)

### 7. Send Email Report
- **Action**: Emails formatted report to manager
- **Format**: HTML with tables and styling
- **Content**: Performance metrics and post details
- **Node Type**: Email Send

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
   Check Post Time
         ↓
   Create Facebook Post
         ↓
   Update Status & ID
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
   Read Past Week Posts
         ↓
   Filter Weekly Posts
         ↓
   Generate Report
         ↓
   Append Details
         ↓
   Send Email Report
```

## Data Structure

### Content Calendar Columns

| Column | Name | Description | Format |
|--------|------|-------------|---------|
| A | Date & Time | Scheduled posting time | YYYY-MM-DD HH:MM |
| B | Post Text | Content to post | Plain text |
| C | Media URL | Image/video link | URL or empty |
| D | Status | Post status | Scheduled/Posted |
| E | Facebook Post ID | Facebook's post identifier | Auto-populated |

### Weekly Report Structure

The weekly report is automatically generated with:

1. **Report Header**
   - Report title and generation date
   - Page performance summary

2. **Performance Metrics**
   - Page impressions
   - Engaged users
   - New followers
   - Lost followers

3. **Post Summary**
   - Total posts published
   - Post details table

## Timing and Scheduling

### Content Publishing
- **Check Frequency**: Every 15 minutes
- **Posting**: Immediate when scheduled time arrives
- **Status Updates**: Real-time after posting

### Weekly Reports
- **Generation**: Every Monday at 9 AM
- **Data Period**: Previous 7 days
- **Delivery**: Email sent immediately after generation

## Error Handling

### Automatic Retries
- Failed API calls are retried automatically
- Google Sheets operations have built-in retry logic

### Status Tracking
- Failed posts remain in "Scheduled" status
- Successful posts are marked as "Posted"
- Facebook post IDs are captured for tracking

### Monitoring
- Execution logs show success/failure for each step
- Failed executions can be reviewed and debugged

## Performance Considerations

### API Rate Limits
- Facebook Graph API has rate limits
- Google Sheets API has quotas
- Workflow includes appropriate delays

### Resource Usage
- Minimal memory footprint
- Efficient data processing
- Optimized for production use

## Security Features

### Credential Management
- Uses n8n's built-in credential system
- Environment variables for sensitive data
- No hardcoded secrets

### Access Control
- Minimal required permissions
- Service account isolation
- Audit logging enabled

## Customization Points

### Timing Adjustments
- Modify cron expressions for different frequencies
- Change weekly report day/time
- Adjust content check intervals

### Content Processing
- Add content validation rules
- Implement approval workflows
- Extend to multiple platforms

### Reporting
- Customize email templates
- Add more metrics
- Include visualizations

## Next Steps

- Review the [Setup Guide](setup.md) to get started
- Check [Configuration](configuration.md) for setup options
- Learn about [Customization](customization.md) possibilities
- Set up your [Google Sheets Template](google-sheets-template.md)
