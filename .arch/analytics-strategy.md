# Analytics Strategy

## Storage Capacity and Options

### Free Analytics Services

#### Google Analytics 4
- **Storage**: Unlimited
- **Retention**: 14 months for free tier
- **Events**: Up to 500 distinct event types
- **Properties**: Up to 50 properties
- **Data points**: Millions of events per month free

#### Plausible Analytics
- **Storage**: 10K pageviews/month (free tier)
- **Retention**: Unlimited
- **Custom events**: Yes
- **Cost**: $9/month for 100K pageviews

#### Simple Analytics
- **Storage**: 100K pageviews/month (free tier)
- **Retention**: Unlimited
- **Privacy-focused**: No cookies
- **Cost**: $19/month for 1M pageviews

### Self-Hosted Options

#### Supabase (PostgreSQL)
- **Storage**: 500MB free tier
- **Rows**: 50,000 free tier
- **Bandwidth**: 2GB/month free
- **Can store**: ~100K+ analytics events

#### PlanetScale (MySQL)
- **Storage**: 5GB free tier
- **Rows**: 500M rows free tier
- **Read/write**: 1B reads/month, 10M writes/month
- **Can store**: Millions of analytics events

## Telemetry Data You Can Collect

### Usage Metrics
```javascript
{
  timestamp: '2026-01-12T20:05:00Z',
  sessionId: 'uuid',
  extensionVersion: '1.2.3',
  browser: 'Chrome',
  action: 'extract_recipe',
  site: 'allrecipes.com',
  success: true,
  extractionMethod: 'structured_data',
  processingTime: 1250,
  confidence: 0.95
}
```

### Performance Metrics
```javascript
{
  timestamp: '2026-01-12T20:05:00Z',
  action: 'performance',
  metrics: {
    pageLoadTime: 3200,
    extractionTime: 1250,
    overlayRenderTime: 150,
    memoryUsage: 45,
    domNodes: 2847
  }
}
```

### Error Tracking
```javascript
{
  timestamp: '2026-01-12T20:05:00Z',
  action: 'error',
  error: {
    type: 'EXTRACTION_FAILED',
    site: 'unknown-blog.com',
    message: 'No recipe data found',
    stack: 'Error: No recipe data found...',
    userAgent: 'Mozilla/5.0...'
  }
}
```

### Feature Usage
```javascript
{
  timestamp: '2026-01-12T20:05:00Z',
  action: 'feature_used',
  feature: 'copy_recipe',
  context: {
    from: 'overlay',
    recipeLength: 850,
    hasImages: true
  }
}
```

## Implementation Examples

### Simple Analytics Endpoint
```javascript
// analytics.js
class Analytics {
  constructor(apiEndpoint) {
    this.apiEndpoint = apiEndpoint;
    this.sessionId = this.generateSessionId();
    this.queue = [];
    this.isOnline = navigator.onLine;
  }

  async track(event, data = {}) {
    const payload = {
      event,
      data: {
        ...data,
        timestamp: new Date().toISOString(),
        sessionId: this.sessionId,
        extensionVersion: chrome.runtime.getManifest().version,
        browser: this.getBrowserInfo(),
        url: window.location.href
      }
    };

    // Queue for offline support
    if (!this.isOnline) {
      this.queue.push(payload);
      return;
    }

    try {
      await this.send(payload);
    } catch (error) {
      this.queue.push(payload);
    }
  }

  async send(payload) {
    await fetch(this.apiEndpoint, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(payload)
    });
  }

  // Retry queued events when online
  async retryQueued() {
    while (this.queue.length > 0 && this.isOnline) {
      const payload = this.queue.shift();
      try {
        await this.send(payload);
      } catch (error) {
        this.queue.unshift(payload);
        break;
      }
    }
  }
}
```

### Serverless Analytics Handler
```javascript
// api/analytics.js (Vercel/Netlify)
export default async function handler(req, res) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { event, data } = req.body;

  // Store in database
  await db.insert('analytics', {
    event,
    data,
    ip: req.ip,
    userAgent: req.headers['user-agent'],
    timestamp: new Date()
  });

  // Process specific events
  switch (event) {
    case 'extract_recipe':
      await processExtractionData(data);
      break;
    case 'error':
      await processErrorData(data);
      break;
  }

  res.status(200).json({ success: true });
}

async function processExtractionData(data) {
  // Update site success rates
  await db.upsert('site_stats', {
    site: data.site,
    totalExtractions: db.raw('total_extractions + 1'),
    successfulExtractions: db.raw('successful_extractions + ?', [data.success ? 1 : 0]),
    lastUpdated: new Date()
  }, ['site']);
}
```

## Privacy Considerations

### GDPR Compliance
- No personal identifiers
- Anonymous session IDs only
- Clear privacy policy
- Opt-out option
- Data retention policies

### Data Minimization
```javascript
// Only store what you need
const sanitizedData = {
  site: new URL(url).hostname, // Not full URL
  success: boolean,
  extractionMethod: string,
  processingTime: number,
  confidence: number
  // NO: user IP, personal data, full URLs
};
```

## Insights You Can Gain

### Product Metrics
- **Daily/Monthly Active Users**: Unique sessions
- **Retention Rate**: Return usage patterns
- **Feature Adoption**: Which features are used
- **Success Rate**: Extraction success by site

### Technical Metrics
- **Performance**: Extraction times by site
- **Error Patterns**: Common failure points
- **Browser Compatibility**: Issues by browser
- **Version Distribution**: Update adoption

### Business Metrics
- **Site Coverage**: Which recipe sites work best
- **Geographic Distribution**: Where users are located
- **Growth Trends**: User acquisition over time
- **Churn Points**: When users stop using

## Cost Analysis

### Free Tier Limits
- **Google Analytics**: Completely free
- **Supabase**: ~100K events/month
- **PlanetScale**: ~1M events/month
- **Vercel Functions**: 100K invocations/month

### Paid Scaling
- **Analytics services**: $9-50/month
- **Database**: $20-100/month
- **Serverless**: $10-30/month

**Total**: $0-180/month depending on scale

## Implementation Priority

### Phase 1: Basic Tracking
- Extension installs/uninstalls
- Recipe extraction attempts
- Success/failure rates
- Site coverage

### Phase 2: Enhanced Analytics
- Performance metrics
- Feature usage
- Error patterns
- User flows

### Phase 3: Advanced Insights
- A/B testing framework
- Predictive analytics
- User segmentation
- Advanced reporting
