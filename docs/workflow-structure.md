# Workflow Structure

Detailed breakdown of the Social Media Content Management Workflow nodes, connections, and configuration.

## Workflow Overview

The workflow consists of **24 nodes** organized into two main flows:
- **Content Publishing Flow**: 8 nodes for automated content publishing
- **Weekly Analytics Flow**: 8 nodes for comprehensive reporting
- **Documentation**: 8 sticky notes with setup and maintenance information

## Node Details

### Content Publishing Flow

#### 1. TRIGGER - Content Calendar Check
- **Type**: `n8n-nodes-base.cron`
- **Position**: [300, 300]
- **Configuration**:
  ```json
  {
    "rule": {
      "interval": [{ "field": "minute", "expression": "*/15" }]
    }
  }
  ```
- **Purpose**: Triggers workflow every 15 minutes to check for scheduled content
- **Notes**: Runs continuously to ensure timely content publishing

#### 2. READ - Google Sheets Calendar
- **Type**: `n8n-nodes-base.googleSheets`
- **Position**: [520, 300]
- **Configuration**:
  ```json
  {
    "operation": "read",
    "documentId": "={{ $env.GOOGLE_SHEETS_DOCUMENT_ID }}",
    "sheetName": "Content Calendar",
    "range": "A:E",
    "options": { "valueRenderMode": "UNFORMATTED_VALUE" }
  }
  ```
- **Purpose**: Fetches all scheduled content from the Content Calendar sheet
- **Data Range**: Columns A-E (ID, Date & Time, Post Text, Status, Facebook Post ID)

#### 3. FILTER - Scheduled Posts
- **Type**: `n8n-nodes-base.if`
- **Position**: [740, 300]
- **Configuration**:
  ```json
  {
    "conditions": {
      "conditions": [
        {
          "id": "status-check",
          "leftValue": "={{ $json.Status || '' }}",
          "rightValue": "Scheduled",
          "operator": { "type": "string", "operation": "equals" }
        }
      ],
      "combinator": "and"
    }
  }
  ```
- **Purpose**: Filters posts with "Scheduled" status, ignoring drafts and published posts
- **Logic**: Only processes posts ready for publishing

#### 4. CHECK - Publishing Time
- **Type**: `n8n-nodes-base.if`
- **Position**: [960, 300]
- **Configuration**:
  ```json
  {
    "conditions": {
      "conditions": [
        {
          "id": "time-check",
          "leftValue": "={{ new Date($json['Date & Time'] || 0).getTime() }}",
          "rightValue": "={{ new Date().getTime() }}",
          "operator": { "type": "number", "operation": "lte" }
        }
      ],
      "combinator": "and"
    }
  }
  ```
- **Purpose**: Verifies if the scheduled publication time has arrived
- **Logic**: Compares scheduled time with current time

#### 5. PUBLISH - Facebook Post
- **Type**: `n8n-nodes-base.httpRequest`
- **Position**: [1180, 300]
- **Configuration**:
  ```json
  {
    "url": "https://graph.facebook.com/v18.0/{{ $env.FACEBOOK_PAGE_ID }}/feed",
    "method": "POST",
    "authentication": "genericCredentialType",
    "genericAuthType": "httpHeaderAuth",
    "sendHeaders": true,
    "headerParameters": {
      "parameters": [{ "name": "Authorization", "value": "Bearer {{ $env.FACEBOOK_ACCESS_TOKEN }}" }]
    },
    "sendBody": true,
    "bodyParameters": {
      "parameters": [
        { "name": "message", "value": "={{ $json['Post Text'] || '' }}" },
        { "name": "link", "value": "={{ $json['Media URL'] || '' }}" },
        { "name": "published", "value": "true" }
      ]
    }
  }
  ```
- **Purpose**: Posts content to Facebook using Graph API v18.0
- **Features**: Supports media URLs and automatic publishing
- **Authentication**: Bearer token via HTTP header

#### 6. UPDATE - Mark as Posted
- **Type**: `n8n-nodes-base.googleSheets`
- **Position**: [1400, 300]
- **Configuration**:
  ```json
  {
    "operation": "update",
    "documentId": "={{ $env.GOOGLE_SHEETS_DOCUMENT_ID }}",
    "sheetName": "Content Calendar",
    "range": "D{{ $json.rowIndex || $json.ID }}",
    "values": [["Posted"]]
  }
  ```
- **Purpose**: Updates post status from "Scheduled" to "Posted"
- **Target**: Status column (D) with dynamic row reference

#### 7. STORE - Facebook Post ID
- **Type**: `n8n-nodes-base.googleSheets`
- **Position**: [1620, 300]
- **Configuration**:
  ```json
  {
    "operation": "update",
    "documentId": "={{ $env.GOOGLE_SHEETS_DOCUMENT_ID }}",
    "sheetName": "Content Calendar",
    "range": "E{{ $json.rowIndex || $json.ID }}",
    "values": [["={{ $('PUBLISH - Facebook Post').first()?.json?.id || '' }}"]]
  }
  ```
- **Purpose**: Stores the Facebook post ID for tracking and analytics
- **Target**: Facebook Post ID column (E) with dynamic row reference

#### 8. SUCCESS - Content Published
- **Type**: `n8n-nodes-base.code`
- **Position**: [1840, 300]
- **Configuration**:
  ```json
  {
    "jsCode": "console.log('Content publishing workflow completed successfully');\nreturn $json;"
  }
  ```
- **Purpose**: Logs successful completion of content publishing workflow
- **Output**: Confirmation message and original data

### Weekly Analytics Flow

#### 1. TRIGGER - Weekly Analytics
- **Type**: `n8n-nodes-base.cron`
- **Position**: [300, 800]
- **Configuration**:
  ```json
  {
    "rule": {
      "interval": [
        { "field": "week", "expression": "1" },
        { "field": "weekday", "expression": "1" },
        { "field": "hour", "expression": "9" },
        { "field": "minute", "expression": "0" }
      ]
    }
  }
  ```
- **Purpose**: Triggers weekly report generation every Monday at 9:00 AM
- **Frequency**: Weekly on Mondays

#### 2. ANALYTICS - Facebook Insights
- **Type**: `n8n-nodes-base.httpRequest`
- **Position**: [520, 800]
- **Configuration**:
  ```json
  {
    "url": "https://graph.facebook.com/v18.0/{{ $env.FACEBOOK_PAGE_ID }}/insights",
    "method": "GET",
    "authentication": "genericCredentialType",
    "genericAuthType": "httpHeaderAuth",
    "sendHeaders": true,
    "headerParameters": {
      "parameters": [{ "name": "Authorization", "value": "Bearer {{ $env.FACEBOOK_ACCESS_TOKEN }}" }]
    },
    "sendQuery": true,
    "queryParameters": {
      "parameters": [
        { "name": "metric", "value": "page_impressions,page_engaged_users,page_fan_adds,page_fan_removes" },
        { "name": "period", "value": "week" },
        { "name": "since", "value": "={{ new Date(Date.now() - 7 * 24 * 60 * 60 * 1000).toISOString() }}" },
        { "name": "until", "value": "={{ new Date().toISOString() }}" }
      ]
    }
  }
  ```
- **Purpose**: Retrieves comprehensive page analytics for the past week
- **Metrics**: Impressions, engaged users, follower changes
- **Period**: Weekly data with dynamic date range

#### 3. READ - Weekly Post History
- **Type**: `n8n-nodes-base.googleSheets`
- **Position**: [740, 800]
- **Configuration**:
  ```json
  {
    "operation": "read",
    "documentId": "={{ $env.GOOGLE_SHEETS_DOCUMENT_ID }}",
    "sheetName": "Content Calendar",
    "range": "A:E",
    "options": { "valueRenderMode": "UNFORMATTED_VALUE" }
  }
  ```
- **Purpose**: Fetches all posts to analyze weekly performance data
- **Data Range**: Complete post history for analysis

#### 4. FILTER - Last 7 Days Posts
- **Type**: `n8n-nodes-base.if`
- **Position**: [960, 800]
- **Configuration**:
  ```json
  {
    "conditions": {
      "conditions": [
        {
          "id": "date-filter",
          "leftValue": "={{ new Date($json['Date & Time'] || 0).getTime() }}",
          "rightValue": "={{ new Date(Date.now() - 7 * 24 * 60 * 60 * 1000).getTime() }}",
          "operator": { "type": "number", "operation": "gte" }
        },
        {
          "id": "status-filter",
          "leftValue": "={{ $json.Status || '' }}",
          "rightValue": "Posted",
          "operator": { "type": "string", "operation": "equals" }
        }
      ],
      "combinator": "and"
    }
  }
  ```
- **Purpose**: Filters posts published in the past 7 days with "Posted" status
- **Logic**: Ensures accurate weekly reporting data

#### 5. REPORT - Generate Analytics
- **Type**: `n8n-nodes-base.googleSheets`
- **Position**: [1180, 800]
- **Configuration**:
  ```json
  {
    "operation": "create",
    "documentId": "={{ $env.GOOGLE_SHEETS_DOCUMENT_ID }}",
    "sheetName": "Weekly Report",
    "range": "A1",
    "values": [
      ["Weekly Social Media Report", "Generated: {{ new Date().toLocaleDateString() }}"],
      [""],
      [
        "Page Performance:",
        "Impressions: {{ $('ANALYTICS - Facebook Insights').first()?.json?.data?.[0]?.values?.[0]?.value || 0 }}",
        "Engaged Users: {{ $('ANALYTICS - Facebook Insights').first()?.json?.data?.[1]?.values?.[0]?.value || 0 }}",
        "New Followers: {{ $('ANALYTICS - Facebook Insights').first()?.json?.data?.[2]?.values?.[0]?.value || 0 }}",
        "Lost Followers: {{ $('ANALYTICS - Facebook Insights').first()?.json?.data?.[3]?.values?.[0]?.value || 0 }}"
      ],
      [""],
      ["Posts Published This Week:", "{{ $('FILTER - Last 7 Days Posts').length || 0 }}"],
      [""],
      ["Post Details:"]
    ]
  }
  ```
- **Purpose**: Creates comprehensive weekly report header with performance metrics
- **Content**: Report title, generation date, and key performance indicators

#### 6. APPEND - Post Details
- **Type**: `n8n-nodes-base.googleSheets`
- **Position**: [1400, 800]
- **Configuration**:
  ```json
  {
    "operation": "append",
    "documentId": "={{ $env.GOOGLE_SHEETS_DOCUMENT_ID }}",
    "sheetName": "Weekly Report",
    "range": "A10",
    "values": "={{ $('FILTER - Last 7 Days Posts').map(item => [item['Date & Time'] || '', item['Post Text'] || '', item['Facebook Post ID'] || '']) }}"
  }
  ```
- **Purpose**: Adds detailed post information to the weekly report
- **Data**: Date, post text, and Facebook post IDs for all weekly posts

#### 7. EMAIL - Send Report to Manager
- **Type**: `n8n-nodes-base.emailSend`
- **Position**: [1620, 800]
- **Configuration**:
  ```json
  {
    "toEmail": "={{ $env.MANAGER_EMAIL }}",
    "subject": "Weekly Social Media Report - {{ new Date().toLocaleDateString() }}",
    "emailType": "html",
    "message": "=<!DOCTYPE html><html><body style='font-family: Arial, sans-serif; max-width: 600px; margin: 0 auto; padding: 20px;'><div style='background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); color: white; padding: 30px; border-radius: 15px; text-align: center; margin-bottom: 20px;'><h1 style='margin: 0; font-size: 28px;'>Weekly Social Media Report</h1><p style='margin: 10px 0 0; opacity: 0.9;'>Generated: {{ new Date().toLocaleDateString() }}</p></div><div style='background: #f8fafc; padding: 25px; border-radius: 10px; margin-bottom: 20px;'><h2 style='color: #2d3748; margin-top: 0;'>Page Performance</h2><div style='display: grid; grid-template-columns: 1fr 1fr; gap: 15px;'><div style='background: white; padding: 15px; border-radius: 8px; text-align: center;'><div style='font-size: 24px; font-weight: bold; color: #3182ce;'>{{ $('ANALYTICS - Facebook Insights').first()?.json?.data?.[0]?.values?.[0]?.value || 0 }}</div><div style='color: #718096; font-size: 14px;'>Impressions</div></div><div style='background: white; padding: 15px; border-radius: 8px; text-align: center;'><div style='font-size: 24px; font-weight: bold; color: #38a169;'>{{ $('ANALYTICS - Facebook Insights').first()?.json?.data?.[1]?.values?.[0]?.value || 0 }}</div><div style='color: #718096; font-size: 14px;'>Engaged Users</div></div><div style='background: white; padding: 15px; border-radius: 8px; text-align: center;'><div style='font-size: 24px; font-weight: bold; color: #00a86b;'>{{ $('ANALYTICS - Facebook Insights').first()?.json?.data?.[2]?.values?.[0]?.value || 0 }}</div><div style='color: #718096; font-size: 14px;'>New Followers</div></div><div style='background: white; padding: 15px; border-radius: 8px; text-align: center;'><div style='font-size: 24px; font-weight: bold; color: #e53e3e;'>{{ $('ANALYTICS - Facebook Insights').first()?.json?.data?.[3]?.values?.[0]?.value || 0 }}</div><div style='color: #718096; font-size: 14px;'>Lost Followers</div></div></div></div><div style='background: white; padding: 25px; border-radius: 10px; border: 1px solid #e2e8f0;'><h2 style='color: #2d3748; margin-top: 0;'>Posts Published This Week</h2><p style='font-size: 18px; color: #4a5568; margin-bottom: 20px;'><strong>Total Posts:</strong> {{ $('FILTER - Last 7 Days Posts').length || 0 }}</p><table style='width: 100%; border-collapse: collapse; margin-top: 15px;'><thead><tr style='background: #f7fafc;'><th style='border: 1px solid #e2e8f0; padding: 12px; text-align: left; color: #2d3748;'>Date & Time</th><th style='border: 1px solid #e2e8f0; padding: 12px; text-align: left; color: #2d3748;'>Post Text</th><th style='border: 1px solid #e2e8f0; padding: 12px; text-align: left; color: #2d3748;'>Facebook ID</th></tr></thead><tbody>{{ $('FILTER - Last 7 Days Posts').map(item => '<tr><td style=\"border: 1px solid #e2e8f0; padding: 10px;\">' + (item['Date & Time'] || '') + '</td><td style=\"border: 1px solid #e2e8f0; padding: 10px;\">' + (item['Post Text'] || '').substring(0, 50) + (item['Post Text'] && item['Post Text'].length > 50 ? '...' : '') + '</td><td style=\"border: 1px solid #e2e8f0; padding: 10px; font-family: monospace; font-size: 12px;\">' + (item['Facebook Post ID'] || '') + '</td></tr>').join('') }}</tbody></table></div><div style='text-align: center; margin-top: 30px; padding: 20px; background: #f7fafc; border-radius: 10px; color: #718096; font-size: 14px;'>This report was generated automatically by your Social Media Workflow<br>Questions? Contact your system administrator.</div></body></html>"
  }
  ```
- **Purpose**: Sends beautifully formatted HTML email report to manager
- **Features**: Professional styling, performance metrics, and post details
- **Format**: Responsive HTML with tables and visual elements

#### 8. SUCCESS - Analytics Complete
- **Type**: `n8n-nodes-base.code`
- **Position**: [1840, 800]
- **Configuration**:
  ```json
  {
    "jsCode": "console.log('Weekly analytics workflow completed successfully');\nreturn $json;"
  }
  ```
- **Purpose**: Logs successful completion of weekly analytics workflow
- **Output**: Confirmation message and original data

### Error Handling Nodes

#### INFO - No Scheduled Posts
- **Type**: `n8n-nodes-base.code`
- **Position**: [960, 180]
- **Purpose**: Handles gracefully when no posts are scheduled for publishing

#### WAIT - Post Time Pending
- **Type**: `n8n-nodes-base.code`
- **Position**: [1180, 180]
- **Purpose**: Handles gracefully when scheduled post time hasn't arrived yet

#### INFO - No Weekly Posts
- **Type**: `n8n-nodes-base.code`
- **Position**: [1180, 680]
- **Purpose**: Handles gracefully when no posts were published during the reporting week

### Documentation Nodes (Sticky Notes)

#### Content Workflow Note
- **Position**: [60, 200]
- **Content**: Overview of content publishing workflow, environment variables, and Google Sheets structure

#### Analytics Workflow Note
- **Position**: [60, 700]
- **Content**: Overview of weekly analytics workflow, metrics tracked, and additional environment variables

#### Error Handling Note
- **Position**: [60, 1100]
- **Content**: Information about error handling, edge cases, and best practices

#### Setup Instructions Note
- **Position**: [2000, 200]
- **Content**: Step-by-step setup instructions for environment variables, Google Sheets, and Facebook

#### Features Note
- **Position**: [2000, 600]
- **Content**: Overview of workflow features including automation, analytics, reliability, and scalability

#### Maintenance Note
- **Position**: [2000, 1000]
- **Content**: Maintenance and monitoring guidelines, troubleshooting steps, and performance considerations

## Workflow Connections

### Content Publishing Flow
```
TRIGGER - Content Calendar Check
         ↓
   READ - Google Sheets Calendar
         ↓
   FILTER - Scheduled Posts
         ↓
   CHECK - Publishing Time
         ↓
   PUBLISH - Facebook Post
         ↓
   UPDATE - Mark as Posted
         ↓
   STORE - Facebook Post ID
         ↓
   SUCCESS - Content Published
```

### Weekly Analytics Flow
```
TRIGGER - Weekly Analytics
         ↓
   ANALYTICS - Facebook Insights
         ↓
   READ - Weekly Post History
         ↓
   FILTER - Last 7 Days Posts
         ↓
   REPORT - Generate Analytics
         ↓
   APPEND - Post Details
         ↓
   EMAIL - Send Report to Manager
         ↓
   SUCCESS - Analytics Complete
```

### Error Handling Paths
- **No Scheduled Posts**: FILTER → INFO - No Scheduled Posts
- **Post Time Pending**: CHECK → WAIT - Post Time Pending
- **No Weekly Posts**: FILTER → INFO - No Weekly Posts

## Workflow Settings

### Execution Configuration
```json
{
  "executionOrder": "v1",
  "saveExecutionProgress": true,
  "saveManualExecutions": true,
  "callerPolicy": "workflowsFromSameOwner",
  "errorWorkflow": ""
}
```

### Tags
- **Social Media**: Identifies workflow purpose
- **Automation**: Indicates automated nature
- **Facebook**: Specifies platform
- **Analytics**: Indicates reporting capabilities

## Next Steps

- Review the [Setup Guide](setup.md) to configure the workflow
- Check [Configuration](configuration.md) for detailed setup options
- Set up your [Google Sheets Template](google-sheets-template.md)
- Learn about [Customization](customization.md) options
