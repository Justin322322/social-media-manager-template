# Social Media Content Management Workflow for n8n

This n8n workflow automates the entire social media content management process, from scheduling posts to generating weekly analytics reports.

## Quick Start

1. **Import the workflow**: Use `social-media-workflow.json`
2. **Set environment variables** (see [Configuration](docs/configuration.md))
3. **Configure credentials** for Google Sheets and Email
4. **Test the workflow** manually
5. **Activate** for automatic execution

## Documentation

📚 **[Setup Guide](docs/setup.md)** - Complete step-by-step setup instructions  
⚙️ **[Configuration](docs/configuration.md)** - Environment variables and credentials setup  
📊 **[Google Sheets Template](docs/google-sheets-template.md)** - Spreadsheet structure and setup  
🔧 **[Workflow Overview](docs/workflow-overview.md)** - How the workflow works  
🚀 **[Customization](docs/customization.md)** - Modify timing, add features, and extend functionality  
🐛 **[Troubleshooting](docs/troubleshooting.md)** - Common issues and solutions  
📖 **[API Reference](docs/api-reference.md)** - Facebook Graph API endpoints and parameters  

## Features

- **Automated Post Scheduling**: Checks content calendar every 15 minutes and posts when scheduled time arrives
- **Facebook Integration**: Automatically creates posts on your Facebook page with text and media
- **Status Tracking**: Updates spreadsheet with post status and Facebook post IDs
- **Weekly Analytics**: Generates comprehensive reports with Facebook Insights data
- **Email Reporting**: Sends formatted weekly reports to managers
- **Google Sheets Integration**: Uses Google Sheets as the content calendar and reporting database

## Prerequisites

- n8n instance (self-hosted or cloud)
- Google account with Google Sheets access
- Facebook Business account with page access
- Email service (SMTP or cloud provider)

## What's Special About This Workflow

✅ **No Custom Dependencies**: Uses only standard n8n nodes available in all versions  
✅ **Facebook Integration**: Uses HTTP Request nodes to call Facebook Graph API directly  
✅ **Production Ready**: Includes error handling, monitoring, and security best practices  
✅ **Easy to Customize**: Well-documented structure for adding new features  

## Support

- 📖 Check the [documentation](docs/) first
- 🐛 Review [troubleshooting guide](docs/troubleshooting.md)
- 💬 [n8n Community Forum](https://community.n8n.io/)
- 📚 [n8n Documentation](https://docs.n8n.io/)

---

**Note**: This workflow is designed for production use but should be tested thoroughly in a development environment first.
