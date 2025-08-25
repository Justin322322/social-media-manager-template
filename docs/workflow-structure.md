# Workflow Structure

Detailed breakdown of the Social Media Content Management Workflow nodes, connections, and configuration.

## Workflow Overview

The workspace contains **three main workflows** with a total of **40+ nodes**:

### Social Media Content Management (`social-media-workflow.json`)
- **Content Publishing Flow**: 8 nodes for automated content publishing
- **Weekly Analytics Flow**: 8 nodes for comprehensive reporting

### Lead Capture & Enrichment (`lead-capture-enrichment-workflow.json`)
- **Lead Processing Flow**: 6 nodes for automated lead processing

### Social Media Monitoring & Alerts (`social-media-monitoring-alerts-workflow.json`)
- **Sentiment Analysis Flow**: 8 nodes for social media monitoring

## Node Details

### Content Publishing Flow

#### 1. TRIGGER - Content Calendar Check
- **Type**: `n8n-nodes-base.cron`
- **Position**: [-1392, -144]
- **Configuration**:
  ```json
  {
    "rule": {
      "interval": [{ "field": "hour", "expression": "1" }]
    }
  }
  ```
- **Purpose**: Triggers workflow every hour to check for scheduled content
- **Notes**: Hourly checks balance timeliness with API rate limits

#### 2. READ - Google Sheets Calendar
- **Type**: `n8n-nodes-base.googleSheets`
- **Position**: [-1168, -144]
- **Configuration**:
  ```json
  {
    "resource": "sheet",
    "operation": "read",
    "documentId": "YOUR_GOOGLE_SHEETS_DOCUMENT_ID",
    "sheetName": "gid=0",
    "range": "A:H",
    "options": { "valueRenderMode": "UNFORMATTED_VALUE" }
  }
  ```
- **Purpose**: Fetches all scheduled content from the Content Calendar sheet
- **Data Range**: Columns A-H (ID, Date & Time, Post Text, Status, Facebook Post ID, Media URL, Posted Date, Post URL)

#### 3. FILTER - Scheduled Posts
- **Type**: `n8n-nodes-base.if`
- **Position**: [-944, -144]
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
- **Position**: [-720, -240]
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
- **Position**: [-496, -336]
- **Configuration**:
  ```json
  {
    "method": "POST",
    "url": "https://graph.facebook.com/v18.0/YOUR_FACEBOOK_PAGE_ID/feed",
    "authentication": "predefinedCredentialType",
    "nodeCredentialType": "facebookGraphApi",
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
- **Authentication**: Facebook Graph API credential

#### 6. UPDATE - Mark as Posted
- **Type**: `n8n-nodes-base.googleSheets`
- **Position**: [-272, -336]
- **Configuration**:
  ```json
  {
    "resource": "sheet",
    "operation": "update",
    "documentId": "YOUR_GOOGLE_SHEETS_DOCUMENT_ID",
    "sheetName": "gid=0",
    "columns": {
      "mappingMode": "defineBelow",
      "value": {
        "Status": "Posted",
        "Posted Date": "={{ new Date().toISOString() }}"
      }
    },
    "options": {
      "whereClause": "={{ 'Post Text=\"' + $json['Post Text'] + '\"' }}"
    }
  }
  ```
- **Purpose**: Updates post status and tracks posting date
- **Target**: Status column (D) and Posted Date column (G)

#### 7. STORE - Facebook Post ID and URL
- **Type**: `n8n-nodes-base.googleSheets`
- **Position**: [-48, -336]
- **Configuration**:
  ```json
  {
    "resource": "sheet",
    "operation": "update",
    "documentId": "YOUR_GOOGLE_SHEETS_DOCUMENT_ID",
    "sheetName": "gid=0",
    "columns": {
      "mappingMode": "defineBelow",
      "value": {
        "Facebook Post ID": "={{ $('PUBLISH - Facebook Post').item.json.id }}",
        "Post URL": "={{ 'https://facebook.com/' + $('PUBLISH - Facebook Post').item.json.id }}"
      }
    },
    "options": {
      "whereClause": "={{ 'Post Text=\"' + $('CHECK - Publishing Time').item.json['Post Text'] + '\"' }}"
    }
  }
  ```
- **Purpose**: Stores Facebook post ID and generates direct post URL
- **Target**: Facebook Post ID column (E) and Post URL column (H)

#### 8. SUCCESS - Content Published
- **Type**: `n8n-nodes-base.code`
- **Position**: [176, -336]
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
- **Position**: [-1392, 272]
- **Configuration**:
  ```json
  {
    "rule": {
      "interval": [
        { "field": "weekDay", "expression": "1" }
      ]
    }
  }
  ```
- **Purpose**: Triggers weekly report generation every Monday
- **Frequency**: Weekly on Mondays

#### 2. ANALYTICS - Facebook Insights
- **Type**: `n8n-nodes-base.httpRequest`
- **Position**: [-1168, 272]
- **Configuration**:
  ```json
  {
    "method": "GET",
    "url": "https://graph.facebook.com/v18.0/YOUR_FACEBOOK_PAGE_ID/insights",
    "authentication": "predefinedCredentialType",
    "nodeCredentialType": "facebookGraphApi",
    "sendQuery": true,
    "queryParameters": {
      "parameters": [
        { "name": "metric", "value": "page_impressions,page_engaged_users,page_fan_adds,page_fan_removes" },
        { "name": "period", "value": "week" },
        { "name": "since", "value": "={{ new Date(Date.now() - 7 * 24 * 60 * 60 * 1000).toISOString().split('T')[0] }}" },
        { "name": "until", "value": "={{ new Date().toISOString().split('T')[0] }}" }
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
- **Type**: `n8n-nodes-base.code`
- **Position**: [-496, 176]
- **Configuration**:
  ```json
  {
    "jsCode": "// Combine Facebook Analytics with Post Data\nconst analytics = $('ANALYTICS - Facebook Insights').item.json.data;\nconst posts = $input.all();\n\n// Calculate summary metrics\nconst summary = {\n  totalPosts: posts.length,\n  weekStartDate: new Date(Date.now() - 7 * 24 * 60 * 60 * 1000).toLocaleDateString(),\n  weekEndDate: new Date().toLocaleDateString(),\n  impressions: 0,\n  engagedUsers: 0,\n  newFollowers: 0,\n  unfollowers: 0\n};\n\n// Process analytics data\nif (analytics && Array.isArray(analytics)) {\n  analytics.forEach(metric => {\n    if (metric.name === 'page_impressions' && metric.values?.[0]?.value) {\n      summary.impressions = metric.values[0].value;\n    }\n    // ... [additional metric processing]\n  });\n}\n\n// Return combined data\nreturn {\n  summary,\n  posts: posts.map(post => ({\n    postText: post.json['Post Text'] || 'N/A',\n    dateTime: post.json['Date & Time'] || 'N/A',\n    status: post.json['Status'] || 'N/A',\n    postId: post.json['Facebook Post ID'] || 'N/A'\n  })),\n  generatedAt: new Date().toISOString()\n};"
  }
  ```
- **Purpose**: Processes Facebook insights and post data into structured format
- **Data Processing**: Combines analytics with post history for comprehensive reporting

#### 6. STORE - Weekly Summary
- **Type**: `n8n-nodes-base.googleSheets`
- **Position**: [-272, 176]
- **Configuration**:
  ```json
  {
    "resource": "sheet",
    "operation": "appendOrUpdate",
    "documentId": "YOUR_GOOGLE_SHEETS_DOCUMENT_ID",
    "sheetName": "gid=1",
    "columns": {
      "mappingMode": "defineBelow",
      "value": {
        "Week Start": "={{ $json.summary.weekStartDate }}",
        "Week End": "={{ $json.summary.weekEndDate }}",
        "Total Posts": "={{ $json.summary.totalPosts }}",
        "Impressions": "={{ $json.summary.impressions }}",
        "Engaged Users": "={{ $json.summary.engagedUsers }}",
        "New Followers": "={{ $json.summary.newFollowers }}",
        "Unfollowers": "={{ $json.summary.unfollowers }}",
        "Generated At": "={{ $json.generatedAt }}"
      }
    }
  }
  ```
- **Purpose**: Saves comprehensive weekly analytics to Google Sheets
- **Data**: Week date range, total posts, impressions, engaged users, followers

#### 7. EMAIL - Send Report to Manager
- **Type**: `n8n-nodes-base.emailSend`
- **Position**: [-48, 176]
- **Configuration**:
  ```json
  {
    "fromEmail": "noreply@yourcompany.com",
    "toEmail": "manager@yourcompany.com",
    "subject": "Weekly Social Media Report - {{ $json.summary.weekStartDate }} to {{ $json.summary.weekEndDate }}",
    "message": "Weekly Social Media Performance Report\\n\\n📊 SUMMARY:\\n• Total Posts: {{ $json.summary.totalPosts }}\\n• Page Impressions: {{ $json.summary.impressions }}\\n• Engaged Users: {{ $json.summary.engagedUsers }}\\n• New Followers: {{ $json.summary.newFollowers }}\\n• Unfollowers: {{ $json.summary.unfollowers }}\\n\\n📝 POSTS THIS WEEK:\\n{% for post in $json.posts %}\\n• {{ post.postText }} ({{ post.dateTime }})\\n{% endfor %}\\n\\nReport generated: {{ $json.generatedAt }}"
  }
  ```
- **Purpose**: Sends simple email notification about report generation
- **Features**: Summary statistics and key metrics
- **Format**: Plain text with performance data

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

## Lead Capture & Enrichment (`lead-capture-enrichment-workflow.json`)

### Node Details

#### 1. TRIGGER - New Form Response
- **Type**: `n8n-nodes-base.googleSheetsTrigger`
- **Position**: [300, 300]
- **Configuration**:
  ```json
  {
    "event": "rowAdded",
    "documentId": "YOUR_SHEET_ID",
    "sheetName": "gid=0",
    "scope": "thisDocument"
  }
  ```
- **Purpose**: Detects new form submissions in Google Sheets
- **Trigger**: Automatic when new row is added

#### 2. IF Company Email?
- **Type**: `n8n-nodes-base.if`
- **Position**: [600, 300]
- **Configuration**:
  ```json
  {
    "conditions": {
      "conditions": [
        {
          "leftValue": "={{ $json.email }}",
          "rightValue": "@company.com",
          "operator": { "type": "string", "operation": "contains" }
        }
      ]
    }
  }
  ```
- **Purpose**: Determines enrichment path based on email domain

#### 3. ENRICH via Clearbit
- **Type**: `n8n-nodes-base.clearbit`
- **Position**: [900, 200]
- **Configuration**:
  ```json
  {
    "resource": "person",
    "operation": "enrich",
    "email": "={{ $json.email }}"
  }
  ```
- **Purpose**: Fetches company and person data from Clearbit API

#### 4. SAVE Enriched Lead
- **Type**: `n8n-nodes-base.googleSheets`
- **Position**: [1200, 200]
- **Configuration**:
  ```json
  {
    "resource": "sheet",
    "operation": "appendOrUpdate",
    "documentId": "YOUR_LEADS_SHEET_ID",
    "sheetName": "gid=0",
    "columns": {
      "mappingMode": "defineBelow",
      "value": {
        "Name": "={{ $('Trigger - New Form Response').item.json.name || $json.person?.name?.fullName || 'Unknown' }}",
        "Email": "={{ $('Trigger - New Form Response').item.json.email }}",
        "Company": "={{ $json.person?.employment?.name || $('Trigger - New Form Response').item.json.company || 'N/A' }}",
        "Title": "={{ $json.person?.employment?.title || 'N/A' }}",
        "Location": "={{ $json.person?.location || 'N/A' }}",
        "Enriched": "Yes",
        "Date Added": "={{ new Date().toISOString().split('T')[0] }}"
      }
    }
  }
  ```
- **Purpose**: Stores enriched lead data with additional company information

#### 5. SAVE Basic Lead
- **Type**: `n8n-nodes-base.googleSheets`
- **Position**: [1200, 400]
- **Configuration**:
  ```json
  {
    "resource": "sheet",
    "operation": "appendOrUpdate",
    "documentId": "YOUR_LEADS_SHEET_ID",
    "sheetName": "gid=0",
    "columns": {
      "mappingMode": "defineBelow",
      "value": {
        "Name": "={{ $json.name || 'Unknown' }}",
        "Email": "={{ $json.email }}",
        "Company": "={{ $json.company || 'Personal Email' }}",
        "Title": "N/A",
        "Location": "N/A",
        "Enriched": "No",
        "Date Added": "={{ new Date().toISOString().split('T')[0] }}"
      }
    }
  }
  ```
- **Purpose**: Stores basic lead data without enrichment

#### 6. NOTIFY Team via Slack
- **Type**: `n8n-nodes-base.slack`
- **Position**: [1500, 300]
- **Configuration**:
  ```json
  {
    "resource": "message",
    "operation": "post",
    "channel": "#leads",
    "text": "🎯 New Lead Captured!\n\n**Name:** {{ $('Trigger - New Form Response').item.json.name || 'Unknown' }}\n**Email:** {{ $('Trigger - New Form Response').item.json.email }}\n**Company:** {{ $json.person?.employment?.name || $('Trigger - New Form Response').item.json.company || 'Personal' }}\n**Title:** {{ $json.person?.employment?.title || 'N/A' }}\n**Enriched:** {{ $json.person ? '✅ Yes' : '❌ No' }}"
  }
  ```
- **Purpose**: Sends formatted notification to team channel

## Social Media Monitoring & Alerts (`social-media-monitoring-alerts-workflow.json`)

### Node Details

#### 1. TRIGGER - Every 1 Hour
- **Type**: `n8n-nodes-base.cron`
- **Position**: [-592, 128]
- **Configuration**:
  ```json
  {
    "rule": {
      "interval": [{ "field": "hour", "expression": "1" }]
    }
  }
  ```
- **Purpose**: Triggers hourly social media monitoring cycle

#### 2. FETCH Twitter Mentions
- **Type**: `n8n-nodes-base.twitter`
- **Position**: [-352, 128]
- **Configuration**:
  ```json
  {
    "operation": "search",
    "searchText": "YourBrandName",
    "returnAll": false,
    "limit": 5
  }
  ```
- **Purpose**: Searches for recent brand mentions on Twitter

#### 3. SPLIT Tweets
- **Type**: `n8n-nodes-base.splitInBatches`
- **Position**: [-96, 128]
- **Configuration**:
  ```json
  {
    "batchSize": 1,
    "options": {}
  }
  ```
- **Purpose**: Processes individual tweets for sentiment analysis

#### 4. SENTIMENT Analysis
- **Type**: `n8n-nodes-base.openAi`
- **Position**: [160, 128]
- **Configuration**:
  ```json
  {
    "resource": "text",
    "operation": "message",
    "model": "gpt-3.5-turbo",
    "messages": {
      "messageValues": [
        {
          "role": "user",
          "content": "Analyze the sentiment of this text and return only 'positive', 'negative', or 'neutral' as a single word response: {{ $json.text }}"
        }
      ]
    }
  }
  ```
- **Purpose**: Analyzes tweet sentiment using OpenAI GPT-3.5

#### 5. IF Negative Sentiment?
- **Type**: `n8n-nodes-base.if`
- **Position**: [416, 128]
- **Configuration**:
  ```json
  {
    "conditions": {
      "conditions": [
        {
          "leftValue": "={{ $json.choices[0].message.content.trim().toLowerCase() }}",
          "rightValue": "negative",
          "operator": { "type": "string", "operation": "equals" }
        }
      ]
    }
  }
  ```
- **Purpose**: Identifies tweets requiring immediate attention

#### 6. SEND Slack Alert
- **Type**: `n8n-nodes-base.slack`
- **Position**: [656, 32]
- **Configuration**:
  ```json
  {
    "resource": "message",
    "operation": "post",
    "channel": "#alerts",
    "text": "⚠️ Negative Tweet Found!\n\n**Tweet:** {{ $('Split Tweets').item.json.text }}\n**Sentiment:** {{ $('Sentiment Analysis').item.json.choices[0].message.content }}\n**Author:** @{{ $('Split Tweets').item.json.user.screen_name }}\n**Time:** {{ $('Split Tweets').item.json.created_at }}"
  }
  ```
- **Purpose**: Sends immediate alert about negative sentiment

#### 7. LOG to Notion
- **Type**: `n8n-nodes-base.notion`
- **Position**: [656, 224]
- **Configuration**:
  ```json
  {
    "resource": "databasePage",
    "operation": "create",
    "databaseId": "YOUR_NOTION_DATABASE_ID",
    "title": "Negative Tweet Alert - {{ new Date().toISOString().split('T')[0] }}",
    "propertiesUi": {
      "propertyValues": [
        {
          "key": "Tweet Text",
          "textValue": "={{ $('Split Tweets').item.json.text }}"
        },
        {
          "key": "Sentiment",
          "selectValue": "Negative"
        },
        {
          "key": "Date",
          "dateValue": "={{ $('Split Tweets').item.json.created_at }}"
        },
        {
          "key": "Status",
          "selectValue": "New"
        },
        {
          "key": "Author",
          "textValue": "={{ $('Split Tweets').item.json.user.screen_name }}"
        }
      ]
    }
  }
  ```
- **Purpose**: Creates tracking entry in Notion database

#### 8. AGGREGATE Results
- **Type**: `n8n-nodes-base.aggregate`
- **Position**: [912, 128]
- **Configuration**:
  ```json
  {
    "aggregate": "aggregateAllItemData",
    "options": {}
  }
  ```
- **Purpose**: Combines all processed results for summary
