# Workflow Overview

Comprehensive overview of all automation workflows in the n8n workspace.

## Available Workflows

The workspace contains three main automation workflows:

### 1. Social Media Content Management
**File**: `social-media-workflow.json`
- **Content Publishing Flow** - Automatically publishes scheduled posts every hour
- **Weekly Analytics Flow** - Generates comprehensive weekly reports every Monday

### 2. Lead Capture & Enrichment
**File**: `lead-capture-enrichment-workflow.json`
- **Lead Processing Flow** - Automatically captures and enriches lead information from Google Forms

### 3. Social Media Monitoring & Alerts
**File**: `social-media-monitoring-alerts-workflow.json`
- **Sentiment Analysis Flow** - Monitors social media mentions and alerts on negative sentiment

## Social Media Content Management Workflow

### Content Publishing Flow

#### 1. Content Calendar Check (Cron Trigger)
- **Frequency**: Every hour
- **Purpose**: Triggers the workflow to check for scheduled content ready to publish
- **Node Type**: Cron Trigger
- **Position**: [-1392, -144]

#### 2. Read Google Sheets Calendar
- **Action**: Fetches all scheduled content from the "Content Calendar" sheet
- **Range**: Columns A-H (ID, Date & Time, Post Text, Status, Facebook Post ID, Media URL, Posted Date, Post URL)
- **Options**: Unformatted values for accurate data processing
- **Node Type**: Google Sheets (Read)
- **Position**: [-1168, -144]

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

### 6. Update Status and Track Posting
- **Action**: Updates multiple columns in Google Sheets
- **Updates**:
  - Status column (D): Changes from "Scheduled" to "Posted"
  - Posted Date column (G): Records the publication timestamp
- **Target**: Dynamic row reference based on post content
- **Purpose**: Tracks successful publications with timestamps
- **Node Type**: Google Sheets (Update)
- **Position**: [-272, -336]

### 7. Store Facebook Post ID and URL
- **Action**: Captures and stores Facebook post ID and generates post URL
- **Updates**:
  - Facebook Post ID column (E): Stores the post identifier
  - Post URL column (H): Generates Facebook post URL for easy access
- **Target**: Dynamic row reference based on post content
- **Purpose**: Enables post tracking, analytics, and direct links
- **Node Type**: Google Sheets (Update)
- **Position**: [-48, -336]

### 8. Success Logging
- **Action**: Logs successful completion of content publishing workflow
- **Purpose**: Provides execution confirmation and debugging information
- **Node Type**: Code
- **Position**: [1840, 300]

## Weekly Analytics Flow

#### 1. Weekly Analytics Trigger (Cron Trigger)
- **Frequency**: Every Monday
- **Purpose**: Triggers comprehensive weekly report generation
- **Node Type**: Cron Trigger
- **Position**: [-1392, 272]

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

#### 5. Generate Analytics Report
- **Action**: Processes Facebook insights and post data into structured format
- **Data Processing**:
  - Combines Facebook page metrics with weekly post data
  - Calculates engagement metrics and follower changes
  - Structures data for Google Sheets storage
- **Node Type**: Code
- **Position**: [-496, 176]

#### 6. Store Weekly Summary in Google Sheets
- **Action**: Saves comprehensive weekly analytics to Google Sheets
- **Sheet**: "Weekly Report" tab (gid=1)
- **Data**: Week date range, total posts, impressions, engaged users, followers
- **Format**: Structured data for analysis and reporting
- **Node Type**: Google Sheets (Append or Update)
- **Position**: [-272, 176]

#### 7. Send Email Notification
- **Action**: Sends simple email notification about report generation
- **Format**: Plain text with summary statistics
- **Content**: Report availability confirmation and key metrics
- **Recipient**: Configured manager email address
- **Node Type**: Email Send
- **Position**: [-48, 176]

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
| F | Media URL | Media attachment URL | URL | Optional media content |
| G | Posted Date | Actual posting timestamp | ISO 8601 | Publication tracking |
| H | Post URL | Facebook post URL | URL | Direct post access |

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
- **Check Frequency**: Every hour
- **Posting**: Immediate when scheduled time arrives
- **Status Updates**: Real-time after successful posting
- **Additional Tracking**: Posted date and direct post URL generation
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

## Lead Capture & Enrichment Workflow

### Overview
Automatically processes new leads from Google Forms, enriches company email addresses with Clearbit data, and notifies the team via Slack.

### Workflow Steps

#### 1. Form Response Trigger
- **Trigger**: Google Sheets row addition
- **Purpose**: Detects new form submissions
- **Node Type**: Google Sheets Trigger
- **Position**: [300, 300]

#### 2. Company Email Check
- **Condition**: Email contains "@company.com"
- **Purpose**: Determines enrichment path
- **Node Type**: If (Conditional)
- **Position**: [600, 300]

#### 3. Clearbit Enrichment (Company Emails)
- **Action**: Fetches company and person data from Clearbit
- **Data**: Name, company, title, location
- **Purpose**: Enriches lead information
- **Node Type**: Clearbit
- **Position**: [900, 200]

#### 4. Save Enriched Lead
- **Action**: Stores enriched lead data in Google Sheets
- **Fields**: Name, Email, Company, Title, Location, Enriched status
- **Node Type**: Google Sheets (Append or Update)
- **Position**: [1200, 200]

#### 5. Save Basic Lead (Personal Emails)
- **Action**: Stores basic lead data without enrichment
- **Fields**: Name, Email, Company, basic tracking
- **Node Type**: Google Sheets (Append or Update)
- **Position**: [1200, 400]

#### 6. Slack Notification
- **Action**: Sends formatted notification to team
- **Content**: Lead details and enrichment status
- **Node Type**: Slack
- **Position**: [1500, 300]

## Social Media Monitoring & Alerts Workflow

### Overview
Monitors social media mentions, analyzes sentiment using OpenAI, and alerts the team about negative mentions via Slack and Notion.

### Workflow Steps

#### 1. Hourly Trigger
- **Frequency**: Every hour
- **Purpose**: Regular monitoring cycle
- **Node Type**: Cron Trigger
- **Position**: [-592, 128]

#### 2. Fetch Twitter Mentions
- **Action**: Searches for brand mentions on Twitter
- **Limit**: 5 most recent mentions
- **Node Type**: Twitter
- **Position**: [-352, 128]

#### 3. Process Individual Tweets
- **Action**: Splits tweet results for individual processing
- **Purpose**: Enables per-tweet sentiment analysis
- **Node Type**: Split In Batches
- **Position**: [-96, 128]

#### 4. Sentiment Analysis
- **Action**: Analyzes tweet sentiment using OpenAI GPT-3.5
- **Classification**: Positive, negative, or neutral
- **Node Type**: OpenAI
- **Position**: [160, 128]

#### 5. Negative Sentiment Check
- **Condition**: Sentiment equals "negative"
- **Purpose**: Identifies tweets requiring attention
- **Node Type**: If (Conditional)
- **Position**: [416, 128]

#### 6. Slack Alert
- **Action**: Sends immediate alert to team
- **Content**: Tweet text, sentiment, author, timestamp
- **Node Type**: Slack
- **Position**: [656, 32]

#### 7. Log to Notion
- **Action**: Creates tracking entry in Notion database
- **Fields**: Tweet text, sentiment, date, author, status
- **Node Type**: Notion
- **Position**: [656, 224]

#### 8. Aggregate Results
- **Action**: Combines all processed results
- **Purpose**: Provides summary of monitoring cycle
- **Node Type**: Aggregate
- **Position**: [912, 128]

## Next Steps

- Review the [Setup Guide](setup.md) to get started
- Check [Configuration](configuration.md) for setup options
- Learn about [Customization](customization.md) possibilities
- Set up your [Google Sheets Template](google-sheets-template.md)
