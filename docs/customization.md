# Customization

Learn how to modify and extend the Social Media Content Management Workflow.

## Timing Adjustments

### Change Posting Frequency

Modify the cron expression in "TRIGGER - Content Calendar Check" node:

- **Current**: Every 15 minutes (`*/15`)
- **Every hour**: `0 * * * *`
- **Every 2 hours**: `0 */2 * * *`
- **Business hours only**: `0 9-17 * * 1-5`
- **Custom intervals**: `*/30` for every 30 minutes

### Change Weekly Report Timing

Modify the cron expression in "TRIGGER - Weekly Analytics" node:

- **Current**: Every Monday at 9:00 AM
- **Every Friday at 5 PM**: `0 17 * * 5`
- **Every day at 6 AM**: `0 6 * * *`
- **First of month**: `0 9 1 * *`
- **Bi-weekly**: `0 9 * * 1/2` (every other Monday)

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

Edit the "ANALYTICS - Facebook Insights" node parameters:

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

## Workflow Modifications

### Add Content Approval Workflow

1. **Add approval status column** to Google Sheets
2. **Create approval filter node**:
   ```javascript
   // Filter for approved posts only
   return $json.filter(item => item.Status === 'Approved');
   ```

3. **Add approval notification**:
   ```javascript
   // Send approval request email
   const approvalEmail = {
     to: item['Approver Email'],
     subject: 'Content Approval Required',
     body: `Please review: ${item['Post Text']}`
   };
   ```

### Enhanced Error Handling

1. **Add retry logic** for failed posts:
   ```javascript
   // Retry failed posts with exponential backoff
   if (failedAttempts < maxRetries) {
     const delay = Math.pow(2, failedAttempts) * 1000;
     setTimeout(() => retryPost(item), delay);
   }
   ```

2. **Add error notification**:
   ```javascript
   // Send error alerts
   const errorAlert = {
     to: 'admin@company.com',
     subject: 'Workflow Error Alert',
     body: `Error in ${workflowName}: ${errorMessage}`
   };
   ```

### Content Validation

1. **Add content length check**:
   ```javascript
   // Validate post length
   if (postText.length > 2200) {
     return { error: 'Post too long for Facebook' };
   }
   ```

2. **Add URL validation**:
   ```javascript
   // Validate media URLs
   const urlRegex = /^https?:\/\/.+/;
   if (mediaUrl && !urlRegex.test(mediaUrl)) {
     return { error: 'Invalid media URL' };
   }
   ```

## Google Sheets Enhancements

### Add Content Categories

1. **Add Category column** (Column F):
   - Product
   - Culture
   - News
   - Promotional
   - User Generated

2. **Add category-based filtering**:
   ```javascript
   // Filter by category
   const categoryFilter = $json.filter(item => 
     item.Category === selectedCategory
   );
   ```

### Add Performance Tracking

1. **Add engagement columns**:
   - Likes
   - Comments
   - Shares
   - Reach

2. **Update metrics after posting**:
   ```javascript
   // Update engagement metrics
   const updateData = [
     [likes, comments, shares, reach]
   ];
   ```

### Add Content Calendar Views

1. **Create filtered views**:
   - Upcoming posts
   - Published posts
   - Draft posts
   - Category-specific views

2. **Add conditional formatting**:
   - Color-code by category
   - Highlight overdue posts
   - Mark high-priority content

## Email Template Customization

### Modify Weekly Report Email

1. **Customize HTML template** in "EMAIL - Send Report to Manager":
   ```html
   <div style="background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);">
     <h1>Custom Weekly Report</h1>
   </div>
   ```

2. **Add company branding**:
   ```html
   <img src="{{ $env.COMPANY_LOGO_URL }}" alt="Company Logo">
   <div style="color: {{ $env.BRAND_COLOR }};">
   ```

### Add Multiple Recipients

1. **Modify email node** to send to multiple people:
   ```javascript
   const recipients = [
     'manager@company.com',
     'marketing@company.com',
     'analytics@company.com'
   ].join(',');
   ```

2. **Add role-based reports**:
   ```javascript
   // Send different reports based on role
   if (role === 'manager') {
     return fullReport;
   } else if (role === 'analyst') {
     return detailedMetrics;
   }
   ```

## Advanced Features

### Add Content Scheduling Rules

1. **Business hours only**:
   ```javascript
   const hour = new Date().getHours();
   const isBusinessHour = hour >= 9 && hour <= 17;
   const isWeekday = new Date().getDay() >= 1 && new Date().getDay() <= 5;
   
   if (!isBusinessHour || !isWeekday) {
     return { skip: 'Outside business hours' };
   }
   ```

2. **Optimal posting times**:
   ```javascript
   const optimalHours = [9, 12, 15, 18];
   const currentHour = new Date().getHours();
   
   if (!optimalHours.includes(currentHour)) {
     return { skip: 'Not optimal posting time' };
   }
   ```

### Add A/B Testing

1. **Create test groups**:
   ```javascript
   const testGroups = ['A', 'B'];
   const selectedGroup = testGroups[Math.floor(Math.random() * 2)];
   
   // Modify content based on group
   if (selectedGroup === 'A') {
     postText += ' #TestA';
   } else {
     postText += ' #TestB';
   }
   ```

2. **Track performance** by group:
   ```javascript
   // Store test group with post
   const testData = [postId, selectedGroup, performance];
   ```

### Add Content Recycling

1. **Identify evergreen content**:
   ```javascript
   const evergreenKeywords = ['tips', 'how-to', 'guide', 'best practices'];
   const isEvergreen = evergreenKeywords.some(keyword => 
     postText.toLowerCase().includes(keyword)
   );
   ```

2. **Schedule reposts**:
   ```javascript
   if (isEvergreen && daysSinceLastPost > 30) {
     // Schedule repost
     return { action: 'repost', content: postText };
   }
   ```

## Performance Optimization

### Reduce API Calls

1. **Batch operations**:
   ```javascript
   // Process multiple posts in one API call
   const batchPosts = posts.map(post => ({
     message: post.text,
     published: true
   }));
   ```

2. **Cache frequently used data**:
   ```javascript
   // Cache Facebook page info
   const pageInfo = await getCachedPageInfo(pageId);
   ```

### Add Monitoring

1. **Track execution time**:
   ```javascript
   const startTime = Date.now();
   // ... workflow execution
   const executionTime = Date.now() - startTime;
   console.log(`Workflow completed in ${executionTime}ms`);
   ```

2. **Monitor resource usage**:
   ```javascript
   // Log memory usage
   const memoryUsage = process.memoryUsage();
   console.log('Memory usage:', memoryUsage);
   ```

## Next Steps

- Review the [Workflow Overview](workflow-overview.md) to understand the current structure
- Check [Configuration](configuration.md) for setup options
- Set up your [Google Sheets Template](google-sheets-template.md)
- Test modifications in a development environment first
