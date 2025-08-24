# API Reference

Technical reference for the Facebook Graph API endpoints and parameters used in the workflow.

## Facebook Graph API

### Base URL
```
https://graph.facebook.com/v18.0/
```

### Authentication
All API calls require a valid access token in the Authorization header:
```
Authorization: Bearer {access-token}
```

## Facebook Posting

### Create Post Endpoint

**URL**: `POST /{page-id}/feed`

**Headers**:
```
Authorization: Bearer {access-token}
Content-Type: application/x-www-form-urlencoded
```

**Body Parameters**:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `message` | string | Yes | The text content of the post |
| `link` | string | No | URL to attach to the post |
| `published` | boolean | No | Whether to publish immediately (default: true) |

**Example Request**:
```bash
curl -X POST \
  "https://graph.facebook.com/v18.0/{page-id}/feed" \
  -H "Authorization: Bearer {access-token}" \
  -d "message=Hello World&published=true"
```

**Response**:
```json
{
  "id": "123456789_987654321"
}
```

**Error Responses**:

| Code | Error | Description |
|------|-------|-------------|
| 400 | Bad Request | Invalid parameters or content policy violation |
| 401 | Unauthorized | Invalid or expired access token |
| 403 | Forbidden | Insufficient permissions or page restrictions |
| 500 | Internal Error | Facebook server error |

## Facebook Insights

### Get Page Insights Endpoint

**URL**: `GET /{page-id}/insights`

**Headers**:
```
Authorization: Bearer {access-token}
```

**Query Parameters**:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `metric` | string | Yes | Comma-separated list of metrics |
| `period` | string | Yes | Time period for data (day, week, month) |
| `since` | string | No | Start date (ISO 8601 format) |
| `until` | string | No | End date (ISO 8601 format) |

**Example Request**:
```bash
curl -X GET \
  "https://graph.facebook.com/v18.0/{page-id}/insights?metric=page_impressions,page_engaged_users&period=week" \
  -H "Authorization: Bearer {access-token}"
```

**Response**:
```json
{
  "data": [
    {
      "name": "page_impressions",
      "period": "week",
      "values": [
        {
          "value": 1500,
          "end_time": "2024-01-15T00:00:00+0000"
        }
      ]
    },
    {
      "name": "page_engaged_users",
      "period": "week",
      "values": [
        {
          "value": 250,
          "end_time": "2024-01-15T00:00:00+0000"
        }
      ]
    }
  ]
}
```

## Available Metrics

### Page Performance Metrics

| Metric | Description | Availability |
|--------|-------------|--------------|
| `page_impressions` | Total page views | All pages |
| `page_engaged_users` | Users who engaged with page | All pages |
| `page_fan_adds` | New page followers | All pages |
| `page_fan_removes` | Lost page followers | All pages |
| `page_posts_impressions` | Post reach | Pages with posts |
| `page_actions_post_reactions_total` | Total reactions on posts | Pages with posts |
| `page_video_views` | Video views | Pages with videos |

### Post Performance Metrics

| Metric | Description | Availability |
|--------|-------------|--------------|
| `post_impressions` | Individual post reach | Per post |
| `post_engaged_users` | Users who engaged with post | Per post |
| `post_reactions_by_type_total` | Reactions by type | Per post |
| `post_video_views` | Video views for post | Video posts only |

### Engagement Metrics

| Metric | Description | Availability |
|--------|-------------|--------------|
| `page_consumptions` | Page content consumption | All pages |
| `page_negative_feedback` | Negative feedback received | All pages |
| `page_positive_feedback_by_type` | Positive feedback by type | All pages |

## Time Periods

### Available Periods

| Period | Description | Data Granularity |
|--------|-------------|------------------|
| `day` | Daily data | 24-hour periods |
| `week` | Weekly data | 7-day periods |
| `days_28` | 28-day data | 28-day periods |
| `month` | Monthly data | Calendar months |
| `quarter` | Quarterly data | 3-month periods |
| `year` | Yearly data | Calendar years |

### Date Format
Use ISO 8601 format for `since` and `until` parameters:
```
YYYY-MM-DDTHH:MM:SS+0000
```

**Examples**:
- `2024-01-01T00:00:00+0000` - January 1, 2024
- `2024-01-15T09:00:00+0000` - January 15, 2024 at 9 AM

## Rate Limits

### API Call Limits

| Limit Type | Limit | Description |
|------------|-------|-------------|
| **App-level** | 200 calls/hour | Per app per user |
| **Page-level** | 4800 calls/day | Per page per app |
| **User-level** | 100 calls/hour | Per user per app |

### Best Practices

1. **Implement delays** between API calls
2. **Cache results** when possible
3. **Monitor usage** to stay within limits
4. **Use batch requests** for multiple metrics

### Handling Rate Limits

When rate limited, implement exponential backoff:

```javascript
const delay = Math.min(1000 * Math.pow(2, retryCount), 300000);
setTimeout(() => retry(), delay);
```

## Error Handling

### Common Error Codes

| HTTP Code | Error Type | Description | Action |
|-----------|------------|-------------|---------|
| 1 | API Unknown | Unknown API error | Retry with delay |
| 2 | API Service | Service temporarily unavailable | Retry with delay |
| 4 | API Too Many Calls | Rate limit exceeded | Implement backoff |
| 17 | API User Too Many Calls | User rate limit exceeded | Reduce frequency |
| 100 | API Param | Invalid parameter | Check parameter values |
| 102 | API Session | Session expired | Refresh access token |
| 190 | API Invalid OAuth | Invalid access token | Regenerate token |
| 200 | API Permissions | Insufficient permissions | Request additional permissions |

### Error Response Format

```json
{
  "error": {
    "message": "Error message description",
    "type": "OAuthException",
    "code": 190,
    "error_subcode": 33,
    "fbtrace_id": "ABC123DEF456"
  }
}
```

## Webhooks (Optional)

### Page Webhooks

Subscribe to real-time page events:

**Endpoint**: `POST /{page-id}/subscribed_apps`

**Events**:
- `messages` - Direct messages
- `messaging_postbacks` - Postback events
- `messaging_optins` - Opt-in events
- `message_deliveries` - Message delivery confirmations
- `message_reads` - Message read receipts

### Webhook Verification

Facebook sends a verification request:

```json
{
  "hub.mode": "subscribe",
  "hub.challenge": "CHALLENGE_STRING",
  "hub.verify_token": "YOUR_VERIFY_TOKEN"
}
```

## Testing

### Graph API Explorer

1. Go to [Facebook Graph API Explorer](https://developers.facebook.com/tools/explorer/)
2. Select your app
3. Generate access token with required permissions
4. Test endpoints manually

### Test Data

Use test pages and posts for development:
- Create test Facebook pages
- Use test content that won't violate policies
- Test with minimal permissions first

## Security Considerations

### Access Token Security

1. **Store securely**: Use environment variables
2. **Minimal permissions**: Only request needed permissions
3. **Regular rotation**: Update tokens monthly
4. **Scope limitation**: Use page-specific tokens when possible

### App Security

1. **App review**: Submit for review before production
2. **Privacy policy**: Include clear data usage policy
3. **User consent**: Get explicit user consent
4. **Data handling**: Follow Facebook's data policies

## Integration Examples

### Multiple Metrics Request

```bash
curl -X GET \
  "https://graph.facebook.com/v18.0/{page-id}/insights?metric=page_impressions,page_engaged_users,page_fan_adds,page_fan_removes&period=week&since=2024-01-08T00:00:00+0000&until=2024-01-15T00:00:00+0000" \
  -H "Authorization: Bearer {access-token}"
```

### Batch Request

```bash
curl -X POST \
  "https://graph.facebook.com/v18.0/" \
  -H "Authorization: Bearer {access-token}" \
  -d "batch=[{\"method\":\"GET\",\"relative_url\":\"{page-id}/insights?metric=page_impressions&period=week\"},{\"method\":\"GET\",\"relative_url\":\"{page-id}/insights?metric=page_engaged_users&period=week\"}]"
```

### Error Handling Example

```javascript
try {
  const response = await fetch(url, options);
  const data = await response.json();
  
  if (data.error) {
    switch (data.error.code) {
      case 4:
        // Rate limited - implement backoff
        await delay(5000);
        break;
      case 190:
        // Invalid token - refresh
        await refreshToken();
        break;
      default:
        throw new Error(data.error.message);
    }
  }
  
  return data;
} catch (error) {
  console.error('API call failed:', error);
  throw error;
}
```

## Next Steps

- Review the [Setup Guide](setup.md) to implement these APIs
- Check [Configuration](configuration.md) for authentication setup
- Learn about [Customization](customization.md) for advanced usage
- Use [Troubleshooting](troubleshooting.md) for common issues
