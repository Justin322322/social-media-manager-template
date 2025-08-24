# Troubleshooting

Common issues and solutions for the Social Media Content Management Workflow.

## Common Issues

### Facebook Posting Not Working

#### Error 401: Unauthorized
**Symptoms**: Facebook API returns 401 error
**Causes**:
- Access token has expired
- Access token is invalid
- App permissions have changed

**Solutions**:
1. **Regenerate access token**:
   - Go to Facebook Graph API Explorer
   - Generate new token with required permissions
   - Update environment variable

2. **Check app permissions**:
   - Verify app has `pages_manage_posts` permission
   - Check if app is in development mode
   - Ensure app has been reviewed by Facebook

3. **Verify page access**:
   - Confirm your app has access to the page
   - Check if page is published and active

#### Error 403: Forbidden
**Symptoms**: Facebook API returns 403 error
**Causes**:
- Insufficient permissions
- Page restrictions
- App not approved

**Solutions**:
1. **Request additional permissions**:
   - `pages_manage_posts`
   - `pages_read_engagement`
   - `pages_show_list`

2. **Check page settings**:
   - Ensure page allows third-party posting
   - Check page content restrictions
   - Verify page is not restricted

3. **App review process**:
   - Submit app for review if in development mode
   - Provide detailed use case description

#### Error 400: Bad Request
**Symptoms**: Facebook API returns 400 error
**Causes**:
- Invalid page ID
- Malformed request data
- Content policy violations

**Solutions**:
1. **Verify page ID**:
   - Check page ID in Facebook page settings
   - Ensure no extra characters or spaces
   - Test with Graph API Explorer

2. **Check content**:
   - Ensure post text meets Facebook guidelines
   - Verify media URLs are accessible
   - Check for prohibited content

3. **Validate request format**:
   - Review HTTP Request node configuration
   - Check all required parameters
   - Verify authentication headers

### Facebook Insights Not Working

#### No Data Returned
**Symptoms**: Insights API returns empty data
**Causes**:
- Page has insufficient activity
- Metrics not available for page type
- Time period too short

**Solutions**:
1. **Check page activity**:
   - Ensure page has recent posts
   - Verify page has followers
   - Check if page is active

2. **Adjust metrics**:
   - Try different metric combinations
   - Use longer time periods
   - Check metric availability

3. **Verify permissions**:
   - Ensure `pages_read_engagement` permission
   - Check page access level
   - Verify app status

#### Rate Limiting
**Symptoms**: API calls fail with rate limit errors
**Causes**:
- Too many API calls
- Exceeding Facebook limits
- Concurrent requests

**Solutions**:
1. **Implement delays**:
   - Add delays between API calls
   - Use exponential backoff
   - Limit concurrent executions

2. **Monitor usage**:
   - Track API call frequency
   - Implement call counting
   - Set up alerts for limits

3. **Optimize requests**:
   - Batch multiple metrics
   - Use appropriate time periods
   - Cache results when possible

### Google Sheets Errors

#### Permission Denied
**Symptoms**: Cannot read/write to Google Sheets
**Causes**:
- Service account lacks permissions
- Sheet not shared properly
- API not enabled

**Solutions**:
1. **Check service account**:
   - Verify JSON key is correct
   - Ensure service account exists
   - Check account permissions

2. **Share sheet properly**:
   - Add service account email
   - Grant "Editor" permissions
   - Uncheck notification option

3. **Enable APIs**:
   - Enable Google Sheets API
   - Check API quotas
   - Verify project settings

#### Sheet Not Found
**Symptoms**: Workflow cannot find specified sheet
**Causes**:
- Sheet name mismatch
- Sheet doesn't exist
- Case sensitivity issues

**Solutions**:
1. **Verify sheet names**:
   - Must be exactly "Content Calendar"
   - Must be exactly "Weekly Report"
   - Check for extra spaces

2. **Check sheet existence**:
   - Verify sheets are created
   - Check tab names
   - Ensure no typos

3. **Test access**:
   - Try manual API call
   - Check service account access
   - Verify document ID

#### Document ID Error
**Symptoms**: Invalid document ID errors
**Causes**:
- Incorrect document ID
- Malformed URL
- Access restrictions

**Solutions**:
1. **Get correct ID**:
   - Extract from Google Sheets URL
   - Format: `https://docs.google.com/spreadsheets/d/[ID]/edit`
   - Copy only the ID portion

2. **Verify access**:
   - Check if sheet is accessible
   - Ensure no sharing restrictions
   - Test with service account

3. **Update environment variable**:
   - Set `GOOGLE_SHEETS_DOCUMENT_ID`
   - Restart n8n after changes
   - Test workflow execution

### Email Not Sending

#### SMTP Authentication Failed
**Symptoms**: Email credentials rejected
**Causes**:
- Incorrect username/password
- 2FA not configured properly
- App password required

**Solutions**:
1. **Gmail setup**:
   - Enable 2-factor authentication
   - Generate app password
   - Use app password in SMTP

2. **Check credentials**:
   - Verify username format
   - Ensure password is correct
   - Check for special characters

3. **Test connection**:
   - Use email client to test
   - Check SMTP settings
   - Verify port and security

#### Network/Firewall Issues
**Symptoms**: Connection timeout or refused
**Causes**:
- Firewall blocking SMTP
- Network restrictions
- Port blocking

**Solutions**:
1. **Check ports**:
   - Port 587 (STARTTLS)
   - Port 465 (SSL)
   - Port 25 (unencrypted)

2. **Network configuration**:
   - Check firewall rules
   - Verify outbound SMTP allowed
   - Test with different providers

3. **Alternative solutions**:
   - Use cloud email services
   - Try different SMTP providers
   - Use webhook alternatives

## Debug Steps

### 1. Check Execution Logs

1. **Open workflow execution**:
   - Go to n8n dashboard
   - Click on failed execution
   - Review detailed logs

2. **Identify failure point**:
   - Look for error messages
   - Check node execution status
   - Review input/output data

3. **Common log patterns**:
   - `401 Unauthorized`: Authentication issues
   - `403 Forbidden`: Permission problems
   - `400 Bad Request`: Invalid data
   - `500 Internal Error`: Service issues

### 2. Test API Calls Manually

1. **Facebook Graph API**:
   - Use Graph API Explorer
   - Test with your access token
   - Verify endpoint responses

2. **Google Sheets API**:
   - Use Google APIs Explorer
   - Test read/write operations
   - Check authentication

3. **Email SMTP**:
   - Use email client
   - Test SMTP settings
   - Verify credentials

### 3. Verify Credentials

1. **Check environment variables**:
   - Verify all variables are set
   - Check for typos
   - Ensure no extra spaces

2. **Test credentials**:
   - Verify Facebook token works
   - Check Google service account
   - Test email authentication

3. **Permission verification**:
   - Confirm app permissions
   - Check service account access
   - Verify page access

### 4. Check Data Format

1. **Google Sheets structure**:
   - Verify column headers
   - Check data formats
   - Ensure required columns exist

2. **Facebook post data**:
   - Validate post text length
   - Check media URL format
   - Verify date format

3. **Environment variable format**:
   - Check variable names
   - Verify value formats
   - Ensure no quotes or spaces

## Prevention Strategies

### 1. Regular Monitoring

- **Daily checks**: Review execution logs
- **Weekly reviews**: Check for patterns
- **Monthly audits**: Review permissions and tokens

### 2. Automated Alerts

- **Error notifications**: Email alerts for failures
- **Rate limit warnings**: Monitor API usage
- **Performance tracking**: Track execution times

### 3. Backup Procedures

- **Workflow backups**: Export workflow regularly
- **Credential rotation**: Update tokens periodically
- **Documentation updates**: Keep setup guides current

### 4. Testing Procedures

- **Pre-deployment testing**: Test in staging environment
- **Regular validation**: Test all components monthly
- **Change testing**: Test after any modifications

## Getting Help

### 1. Self-Service Resources

- **n8n Documentation**: [docs.n8n.io](https://docs.n8n.io/)
- **Facebook Graph API**: [developers.facebook.com](https://developers.facebook.com/)
- **Google Sheets API**: [developers.google.com/sheets](https://developers.google.com/sheets)

### 2. Community Support

- **n8n Community Forum**: [community.n8n.io](https://community.n8n.io/)
- **Stack Overflow**: Tag with `n8n`
- **GitHub Issues**: Check existing issues

### 3. Professional Support

- **n8n Cloud Support**: For cloud users
- **Facebook Business Support**: For business accounts
- **Google Cloud Support**: For enterprise users

## Success Criteria

Your workflow is working correctly when:

- ✅ **Posts publish automatically** at scheduled times
- ✅ **Facebook post IDs** are captured and stored
- ✅ **Weekly reports** generate automatically
- ✅ **Email reports** are sent to managers
- ✅ **No failed executions** in the logs
- ✅ **All nodes** show green (successful) status
- ✅ **API calls** complete without errors
- ✅ **Data flows** through all nodes correctly

## Next Steps

- Review the [Setup Guide](setup.md) if you're just starting
- Check [Configuration](configuration.md) for setup options
- Learn about [Customization](customization.md) possibilities
- Explore [API Reference](api-reference.md) for technical details
