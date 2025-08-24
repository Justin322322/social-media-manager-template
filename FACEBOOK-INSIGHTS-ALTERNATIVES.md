# Facebook Insights Alternatives for n8n

## Problem
The Facebook Insights node is not available in your current n8n version, showing the error:
> "Install this node to use it. This node is not currently installed. It is either from a newer version of n8n, a custom node, or has an invalid structure."

## Solution: Use HTTP Request Node

I've created an updated workflow (`social-media-workflow-updated.json`) that replaces the Facebook Insights node with an HTTP Request node to call Facebook's Graph API directly.

### How It Works

The HTTP Request node makes a direct call to Facebook's Graph API endpoint:
```
https://graph.facebook.com/v18.0/{page-id}/insights
```

### Required Parameters

1. **URL**: `https://graph.facebook.com/v18.0/{{ $env.FACEBOOK_PAGE_ID }}/insights`
2. **Method**: GET
3. **Headers**: 
   - `Authorization: Bearer {{ $env.FACEBOOK_ACCESS_TOKEN }}`
4. **Query Parameters**:
   - `metric`: `page_impressions,page_engaged_users,page_fan_adds,page_fan_removes`
   - `period`: `week`
   - `since`: `{{ new Date(Date.now() - 7 * 24 * 60 * 60 * 1000).toISOString() }}`
   - `until`: `{{ new Date().toISOString() }}`

### Setup Steps

1. **Import the updated workflow**: Use `social-media-workflow-updated.json`
2. **Set environment variables**:
   - `FACEBOOK_PAGE_ID`: Your Facebook page ID
   - `FACEBOOK_ACCESS_TOKEN`: Your Facebook access token
3. **Configure the HTTP Request node**:
   - Set the URL with your page ID
   - Add your access token in the Authorization header
   - Verify the query parameters match your needs

## Alternative Option 2: Install Facebook Node Package

If you prefer to use the official Facebook node, you can install it:

### For n8n Cloud
1. Go to Settings → Community Nodes
2. Search for "Facebook"
3. Install the official Facebook node package

### For Self-Hosted n8n
1. Navigate to your n8n installation directory
2. Install the Facebook node package:
   ```bash
   npm install n8n-nodes-facebook
   ```
3. Restart n8n

## Alternative Option 3: Use Webhook Node

If you have access to Facebook Webhooks, you can:
1. Set up a Facebook App with webhook subscriptions
2. Use the Webhook node to receive real-time updates
3. Process the data as it comes in

## Alternative Option 4: Manual Data Import

For simple use cases, you can:
1. Export Facebook Insights data manually from Facebook Business Manager
2. Import the CSV/Excel file into Google Sheets
3. Use the Google Sheets node to read the data
4. Process it in your workflow

## Recommended Approach

**Use the HTTP Request node approach** (Option 1) because:
- ✅ Works with any n8n version
- ✅ No additional installations required
- ✅ Direct access to Facebook Graph API
- ✅ Full control over the request parameters
- ✅ Easy to customize and extend

## Facebook Graph API Reference

For more metrics and parameters, refer to the [Facebook Graph API Insights Documentation](https://developers.facebook.com/docs/graph-api/reference/v18.0/page/insights).

### Common Metrics
- `page_impressions`: Total page views
- `page_engaged_users`: Users who engaged with your page
- `page_fan_adds`: New page followers
- `page_fan_removes`: Lost page followers
- `page_posts_impressions`: Post reach
- `page_actions_post_reactions_total`: Total reactions on posts

### Period Options
- `day`: Daily data
- `week`: Weekly data
- `days_28`: 28-day data
- `month`: Monthly data
- `quarter`: Quarterly data
- `year`: Yearly data

## Troubleshooting

### Common Issues

1. **401 Unauthorized**: Check your access token and permissions
2. **403 Forbidden**: Ensure your app has the required permissions
3. **400 Bad Request**: Verify your query parameters are correct
4. **Rate Limiting**: Facebook has API call limits; implement delays if needed

### Debug Steps

1. Test the API call in a tool like Postman first
2. Check the n8n execution logs for detailed error messages
3. Verify your Facebook app permissions include "pages_read_engagement"
4. Ensure your access token hasn't expired

## Security Notes

- Store your Facebook access token securely as an environment variable
- Never commit tokens to version control
- Use the minimum required permissions for your app
- Regularly rotate your access tokens

## Next Steps

1. Import the updated workflow file
2. Configure your Facebook credentials
3. Test the HTTP Request node with a simple API call
4. Verify the data structure matches your expectations
5. Customize the metrics and parameters as needed
