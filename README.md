# n8n Workflow Collection

A comprehensive collection of n8n workflows for automated business processes including social media management, lead capture, and monitoring.

## Available Workflows

| Workflow Name | Description | File | Documentation |
|---------------|-------------|------|---------------|
| Social Media Content Management | Automated content publishing and analytics reporting | [`social-media-workflow.json`](#social-media-content-management) | [Overview](docs/workflow-overview.md#social-media-content-management-workflow) • [Structure](docs/workflow-structure.md#node-details) |
| Lead Capture & Enrichment | Automated lead processing with Clearbit enrichment | [`lead-capture-enrichment-workflow.json`](#lead-capture--enrichment) | [Overview](docs/workflow-overview.md#lead-capture--enrichment-workflow) • [Structure](docs/workflow-structure.md#lead-capture--enrichment-workflow) |
| Social Media Monitoring & Alerts | AI-powered sentiment analysis and alerts | [`social-media-monitoring-alerts-workflow.json`](#social-media-monitoring--alerts) | [Overview](docs/workflow-overview.md#social-media-monitoring--alerts-workflow) • [Structure](docs/workflow-structure.md#social-media-monitoring--alerts-workflow) |

## Social Media Content Management
<img width="1280" height="623" alt="image" src="https://github.com/user-attachments/assets/9eea65fc-176b-487a-a02a-f7ba8578bca8" />

## Lead Capture & Enrichment
<img width="1524" height="450" alt="image" src="https://github.com/user-attachments/assets/37092dd4-bc9a-45d6-adb6-ba2091df4c75" />

## Social Media Monitoring & Alerts
<img width="1573" height="414" alt="image" src="https://github.com/user-attachments/assets/3cb7c3aa-e48d-4b85-bf88-0b5d147bb974" />





## Overview

This workflow automates the entire social media content management process, from scheduling posts to generating comprehensive weekly analytics reports. It integrates with Facebook Graph API and Google Sheets to provide a seamless content management experience.

## Features

### Content Publishing
- **Automated Scheduling**: Checks every 15 minutes for scheduled content
- **Time Validation**: Only publishes when scheduled time arrives
- **Status Tracking**: Updates post status from "Scheduled" to "Posted"
- **Post ID Storage**: Captures Facebook post IDs for analytics

### Weekly Analytics
- **Comprehensive Reporting**: Generates detailed weekly reports every Monday at 9:00 AM
- **Performance Metrics**: Tracks impressions, engagement, and follower changes
- **Post History**: Includes all posts from the past week
- **Professional Email**: Sends formatted HTML reports to managers

### Error Handling
- **Graceful Failures**: Handles edge cases without workflow interruption
- **Comprehensive Logging**: Provides detailed execution information
- **Status Preservation**: Failed posts remain in "Scheduled" status
- **Retry Logic**: Automatic retries for failed operations

## Quick Start

### For All Workflows
1. **Import Workflows**: Import any of the workflow JSON files into your n8n instance
2. **Set Environment Variables**: Configure required environment variables for your chosen workflow(s)
3. **Set Up Credentials**: Configure necessary API credentials (Google, Facebook, Slack, etc.)
4. **Configure Services**: Set up required external services (Google Sheets, Facebook Page, etc.)
5. **Activate Workflows**: Enable automatic execution for your chosen workflow(s)

### Individual Workflow Setup
- **[Social Media Content Management](docs/workflow-overview.md#social-media-content-management)**: Requires Facebook API and Google Sheets
- **[Lead Capture & Enrichment](docs/workflow-overview.md#lead-capture--enrichment)**: Requires Clearbit API and Slack
- **[Social Media Monitoring & Alerts](docs/workflow-overview.md#social-media-monitoring--alerts)**: Requires Twitter API, OpenAI, and Notion

## Documentation

### Getting Started
- **[Setup Guide](docs/setup.md)** - Complete step-by-step setup instructions
- **[Configuration](docs/configuration.md)** - Detailed configuration options
- **[Google Sheets Template](docs/google-sheets-template.md)** - Sheet structure and setup

### 🔧 Technical Details
- **[Complete Workflow Overview](docs/workflow-overview.md)** - Detailed overview of all workflows
  - [Social Media Content Management](docs/workflow-overview.md#social-media-content-management)
  - [Lead Capture & Enrichment](docs/workflow-overview.md#lead-capture--enrichment)
  - [Social Media Monitoring & Alerts](docs/workflow-overview.md#social-media-monitoring--alerts)
- **[Workflow Structure](docs/workflow-structure.md)** - Detailed node breakdown and configurations
  - [Social Media Nodes](docs/workflow-structure.md#node-details)
  - [Lead Capture Nodes](docs/workflow-structure.md#lead-capture--enrichment)
  - [Monitoring Nodes](docs/workflow-structure.md#social-media-monitoring--alerts)
- **[Customization](docs/customization.md)** - How to modify and extend the workflows

### Troubleshooting
- **[Troubleshooting](docs/troubleshooting.md)** - Common issues and solutions
- **[API Reference](docs/api-reference.md)** - Integration details

## Requirements

### Environment Variables
```bash
GOOGLE_SHEETS_DOCUMENT_ID=your_document_id
FACEBOOK_PAGE_ID=your_page_id
FACEBOOK_ACCESS_TOKEN=your_access_token
MANAGER_EMAIL=manager@company.com
```

### Prerequisites
- n8n instance (self-hosted or cloud)
- Google account with Google Sheets access
- Facebook Business account with page access
- Email service (SMTP or cloud provider)

## Workflow Structure

The workflow consists of **24 nodes** organized into two main flows:

### Content Publishing Flow
1. **TRIGGER** - Content Calendar Check (every 15 minutes)
2. **READ** - Google Sheets Calendar
3. **FILTER** - Scheduled Posts
4. **CHECK** - Publishing Time
5. **PUBLISH** - Facebook Post
6. **UPDATE** - Mark as Posted
7. **STORE** - Facebook Post ID
8. **SUCCESS** - Content Published

### Weekly Analytics Flow
1. **TRIGGER** - Weekly Analytics (every Monday at 9:00 AM)
2. **ANALYTICS** - Facebook Insights
3. **READ** - Weekly Post History
4. **FILTER** - Last 7 Days Posts
5. **REPORT** - Generate Analytics
6. **APPEND** - Post Details
7. **EMAIL** - Send Report to Manager
8. **SUCCESS** - Analytics Complete

## Google Sheets Structure

### Content Calendar Sheet
| Column | Header | Description |
|--------|--------|-------------|
| A | ID | Unique identifier |
| B | Date & Time | Scheduled posting time |
| C | Post Text | Content to post |
| D | Status | Post status (Scheduled/Posted) |
| E | Facebook Post ID | Auto-populated |

### Weekly Report Sheet
Automatically generated with performance metrics and post details.

## Security Features

- **Credential Management**: Uses n8n's built-in credential system
- **Environment Variables**: Secure storage for sensitive data
- **Minimal Permissions**: Only required API access
- **Audit Logging**: Comprehensive execution tracking

## Customization

The workflow is designed to be easily customizable:
- **Timing**: Adjust posting frequency and report schedules
- **Platforms**: Extend to Twitter, LinkedIn, Instagram
- **Analytics**: Add custom metrics and calculations
- **Reporting**: Customize email templates and recipients

## Support

For issues and questions:
1. Check the [Troubleshooting](docs/troubleshooting.md) guide
2. Review workflow execution logs in n8n
3. Verify environment variables and credentials
4. Test API endpoints manually

## Contributing

This workflow is designed to be a starting point for social media automation. Feel free to:
- Customize for your specific needs
- Add new platforms and features
- Improve error handling and monitoring
- Share enhancements with the community

## License

This project is open source and available under the [LICENSE](LICENSE) file.

---

**Note**: This workflow requires proper Facebook API permissions and Google Sheets access. Ensure compliance with platform terms of service and data protection regulations.
