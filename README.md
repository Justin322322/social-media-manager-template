# n8n Workflow Collection

A comprehensive collection of n8n workflows for automated business processes including social media management, lead capture, and monitoring.

## Available Workflows

| Workflow Name | Description | File |
|---------------|-------------|------|
| Social Media Content Management | Automated content publishing and analytics reporting | `social-media-workflow.json` |
| Lead Capture & Enrichment | Automated lead processing with Clearbit enrichment | `lead-capture-enrichment-workflow.json` |
| Social Media Monitoring & Alerts | AI-powered sentiment analysis and alerts | `social-media-monitoring-alerts-workflow.json` |

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

1. **Import Workflow**: Import `social-media-workflow.json` into your n8n instance
2. **Set Environment Variables**: Configure required environment variables
3. **Set Up Credentials**: Configure Google Sheets and email credentials
4. **Create Google Sheets**: Set up the required sheet structure
5. **Activate Workflow**: Enable automatic execution

## Documentation

### Getting Started
- **[Setup Guide](docs/setup.md)** - Complete step-by-step setup instructions
- **[Configuration](docs/configuration.md)** - Detailed configuration options
- **[Google Sheets Template](docs/google-sheets-template.md)** - Sheet structure and setup

### 🔧 Technical Details
- **[Workflow Overview](docs/workflow-overview.md)** - How the workflow operates
- **[Workflow Structure](docs/workflow-structure.md)** - Detailed node breakdown
- **[Customization](docs/customization.md)** - How to modify and extend the workflow

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
