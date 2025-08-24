# Customization

Learn how to modify and extend the Social Media Content Management Workflow.

## Timing Adjustments

### Change Posting Frequency

Modify the cron expression in "Check Content Calendar" node:

- **Current**: Every 15 minutes (`*/15`)
- **Every hour**: `0 * * * *`
- **Every 2 hours**: `0 */2 * * *`
- **Business hours only**: `0 9-17 * * 1-5`

### Change Weekly Report Timing

Modify the cron expression in "Weekly Analytics Trigger" node:

- **Current**: Every Monday at 9 AM
- **Every Friday at 5 PM**: `0 17 * * 5`
- **Every day at 6 AM**: `0 6 * * *`
- **First of month**: `0 9 1 * *`

## Add More Social Media Platforms

### Twitter Integration

1. **Duplicate Facebook posting nodes**
2. **Replace with Twitter API calls**:
   ```json
   {
     "url": "https://api.twitter.com/2/tweets",
     "method": "POST",
     "headers": {
       "Authorization": "Bearer {{ $env.TWITTER_BEARER_TOKEN }}"
     },
     "body": {
       "text": "={{ $json['Post Text'] }}"
     }
   }
   ```

### LinkedIn Integration

1. **Add LinkedIn API credentials**
2. **Create LinkedIn posting node**:
   ```json
   {
     "url": "https://api.linkedin.com/v2/ugcPosts",
     "method": "POST",
     "headers": {
       "Authorization": "Bearer {{ $env.LINKEDIN_ACCESS_TOKEN }}"
     }
   }
   ```

### Instagram Integration

1. **Use Facebook Graph API for Instagram**
2. **Modify posting endpoint**:
   ```
   https://graph.facebook.com/v18.0/{{ $env.INSTAGRAM_BUSINESS_ACCOUNT_ID }}/media
   ```

## Enhanced Analytics

### Add More Facebook Metrics

Edit the "Get Facebook Page Insights via HTTP" node:

```json
{
  "metric": "page_impressions,page_engaged_users,page_fan_adds,page_fan_removes,page_posts_impressions,page_actions_post_reactions_total,page_video_views"
}
```

### Custom Metrics Calculation

Add a "Code" node after Facebook Insights:

```javascript
// Calculate engagement rate
const impressions = $json.data[0].values[0].value;
const engaged = $json.data[1].values[0].value;
const engagementRate = (engaged / impressions * 100).toFixed(2);

return {
  ...$json,
  engagementRate: engagementRate
};
```

### Multiple Time Periods

Create separate HTTP Request nodes for different periods:

- **Daily**: `period=day`
- **Weekly**: `period=week` (current)
- **Monthly**: `period=month`

## Email Report Customization

### Modify HTML Template

Edit the "Send Email Report to Manager" node message field:

```html
<!DOCTYPE html>
<html>
<head>
  <title>Weekly Social Media Report</title>
  <style>
    body { font-family: Arial, sans-serif; }
    .header { background: #007bff; color: white; padding: 20px; }
    .metric { background: #f8f9fa; padding: 15px; margin: 10px 0; }
    table { border-collapse: collapse; width: 100%; }
    th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }
    th { background-color: #f2f2f2; }
  </style>
</head>
<body>
  <div class="header">
    <h1>Weekly Social Media Report</h1>
    <p>Generated: {{ new Date().toLocaleDateString() }}</p>
  </div>
  
  <div class="metric">
    <h2>Page Performance</h2>
    <ul>
      <li><strong>Impressions:</strong> {{ $('Get Facebook Page Insights via HTTP').first().json.data[0].values[0].value }}</li>
      <li><strong>Engaged Users:</strong> {{ $('Get Facebook Page Insights via HTTP').first().json.data[1].values[0].value }}</li>
      <li><strong>Engagement Rate:</strong> {{ $('Calculate Engagement Rate').first().json.engagementRate }}%</li>
    </ul>
  </div>
  
  <!-- Add more custom sections here -->
</body>
</html>
```

### Add Attachments

Modify the Email Send node to include attachments:

1. **Add "Code" node** to generate CSV report
2. **Use "Move Binary Data" node** to create file
3. **Configure Email Send node** with attachments

## Content Validation

### Add Content Filters

Insert a "Code" node after "Filter Scheduled Posts":

```javascript
// Validate post content
const postText = $json['Post Text'];
const mediaUrl = $json['Media URL'];

// Check content length
if (postText.length > 2000) {
  throw new Error('Post text too long (max 2000 characters)');
}

// Validate media URL
if (mediaUrl && !mediaUrl.startsWith('http')) {
  throw new Error('Invalid media URL');
}

// Check for required hashtags
if (!postText.includes('#')) {
  throw new Error('Post must include at least one hashtag');
}

return $json;
```

### Approval Workflow

Add approval nodes before posting:

1. **"If" node** to check approval status
2. **"Email Send" node** to request approval
3. **"Wait" node** to pause for approval
4. **"Google Sheets" node** to update approval status

## Error Handling

### Custom Error Messages

Add "Error Trigger" nodes to handle failures:

```javascript
// Custom error handling
const error = $json.error;
const postData = $json.postData;

// Send error notification
return {
  error: error,
  postData: postData,
  timestamp: new Date().toISOString(),
  retryCount: $json.retryCount || 0
};
```

### Retry Logic

Implement exponential backoff:

```javascript
// Calculate retry delay
const retryCount = $json.retryCount || 0;
const baseDelay = 1000; // 1 second
const maxDelay = 300000; // 5 minutes

const delay = Math.min(baseDelay * Math.pow(2, retryCount), maxDelay);

return {
  ...$json,
  retryDelay: delay,
  shouldRetry: retryCount < 3
};
```

## Performance Optimization

### Batch Processing

Modify Google Sheets operations to batch updates:

```javascript
// Batch multiple updates
const updates = $('Filter Scheduled Posts').map(post => ({
  range: `D${post.rowIndex}`,
  values: [['Posted']]
}));

return {
  batchUpdates: updates
};
```

### Caching

Add caching for frequently accessed data:

```javascript
// Cache Facebook insights for 1 hour
const cacheKey = 'facebook_insights';
const cached = $cache.get(cacheKey);

if (cached && Date.now() - cached.timestamp < 3600000) {
  return cached.data;
}

// Fetch fresh data and cache
const freshData = $json;
$cache.set(cacheKey, {
  data: freshData,
  timestamp: Date.now()
});

return freshData;
```

## Monitoring and Alerts

### Custom Notifications

Add notification nodes for different events:

1. **Post Success**: Slack/Teams notification
2. **Post Failure**: Email alert to admin
3. **High Engagement**: Celebrate successful posts
4. **API Rate Limits**: Warning before hitting limits

### Dashboard Integration

Send metrics to external dashboards:

```javascript
// Send to Google Analytics
const analyticsData = {
  event: 'social_media_post',
  properties: {
    platform: 'facebook',
    post_id: $json.id,
    engagement: $json.engagement
  }
};

// Send to external API
$http.post('https://your-dashboard.com/api/metrics', analyticsData);

return $json;
```

## Advanced Features

### A/B Testing

Implement content testing:

1. **"Code" node** to select test variant
2. **"If" node** to route to different content
3. **"Wait" node** to delay variant B
4. **"Code" node** to analyze results

### Content Scheduling

Add intelligent scheduling:

```javascript
// Avoid posting during low-engagement hours
const hour = new Date().getHours();
const lowEngagementHours = [0, 1, 2, 3, 4, 5, 6, 23];

if (lowEngagementHours.includes(hour)) {
  // Reschedule for better time
  const betterTime = new Date();
  betterTime.setHours(9); // 9 AM
  betterTime.setMinutes(0);
  
  return {
    ...$json,
    rescheduled: true,
    newTime: betterTime.toISOString()
  };
}

return $json;
```

### Multi-Language Support

Handle multiple languages:

```javascript
// Detect language and translate if needed
const postText = $json['Post Text'];
const targetLanguage = $env.TARGET_LANGUAGE || 'en';

if (targetLanguage !== 'en') {
  // Call translation API
  const translated = await $http.post('https://translation-api.com/translate', {
    text: postText,
    target: targetLanguage
  });
  
  return {
    ...$json,
    'Post Text': translated.text,
    originalText: postText
  };
}

return $json;
```

## Next Steps

- Review the [Workflow Overview](workflow-overview.md) to understand the structure
- Check [Troubleshooting](troubleshooting.md) for common issues
- Explore [API Reference](api-reference.md) for integration options
