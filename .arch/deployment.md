# Deployment Strategy

## Client-Side Deployment Overview

### Extension Distribution
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Chrome Web    │    │   Firefox       │    │   Edge          │
│   Store         │    │   Add-ons       │    │   Add-ons       │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                │                        │
                                ▼                        ▼
                       ┌─────────────────┐    ┌─────────────────┐
│   Users install   │◄──►│   Extension     │◄──►│   Analytics     │
│   extension       │    │   runs locally │    │   (optional)    │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

## Browser Extension Deployment

### Build Process
```javascript
// webpack.config.js
module.exports = {
  entry: {
    'content/extractor': './src/content/extractor.js',
    'content/overlay': './src/content/overlay.js',
    'background/service-worker': './src/background/service-worker.js',
    'popup/popup': './src/popup/popup.js',
    'options/options': './src/options/options.js'
  },
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].js'
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        use: 'babel-loader',
        exclude: /node_modules/
      },
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader']
      }
    ]
  }
};
```

### Chrome Web Store Deployment
```json
// manifest-chrome.json
{
  "manifest_version": 3,
  "name": "Just the Recipe",
  "version": "1.0.0",
  "description": "Extract clean recipes from any website",
  "permissions": ["activeTab", "storage"],
  "host_permissions": ["https://*/*", "http://*/*"],
  "action": {
    "default_popup": "popup.html"
  },
  "content_scripts": [
    {
      "matches": ["<all_urls>"],
      "js": ["content/extractor.js", "content/overlay.js"],
      "css": ["content/overlay.css"],
      "run_at": "document_idle"
    }
  ],
  "background": {
    "service_worker": "background/service-worker.js"
  }
}
```

### Firefox Add-ons Deployment
```json
// manifest-firefox.json
{
  "manifest_version": 2,
  "name": "Just the Recipe",
  "version": "1.0.0",
  "description": "Extract clean recipes from any website",
  "permissions": ["activeTab", "storage", "<all_urls>"],
  "browser_action": {
    "default_popup": "popup.html"
  },
  "content_scripts": [
    {
      "matches": ["<all_urls>"],
      "js": ["content/extractor.js", "content/overlay.js"],
      "css": ["content/overlay.css"],
      "run_at": "document_idle"
    }
  ],
  "background": {
    "scripts": ["background/service-worker.js"]
  }
}
```

## Build and Release Pipeline

### GitHub Actions Workflow
```yaml
# .github/workflows/build-extension.yml
name: Build Extension

on:
  push:
    paths:
      - 'packages/browser-extension/**'
  pull_request:
    paths:
      - 'packages/browser-extension/**'

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'
        
    - name: Install dependencies
      run: |
        cd packages/browser-extension
        npm ci
        
    - name: Run tests
      run: |
        cd packages/browser-extension
        npm test
        
    - name: Run linting
      run: |
        cd packages/browser-extension
        npm run lint

  build:
    needs: test
    runs-on: ubuntu-latest
    strategy:
      matrix:
        browser: [chrome, firefox]
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'
        
    - name: Install dependencies
      run: |
        cd packages/browser-extension
        npm ci
        
    - name: Build extension
      run: |
        cd packages/browser-extension
        npm run build:${{ matrix.browser }}
        
    - name: Package extension
      run: |
        cd packages/browser-extension/dist
        zip -r ../extension-${{ matrix.browser }}.zip .
        
    - name: Upload artifacts
      uses: actions/upload-artifact@v3
      with:
        name: extension-${{ matrix.browser }}
        path: packages/browser-extension/extension-${{ matrix.browser }}.zip
```

### Extension Store Submission

#### Chrome Web Store
```bash
# Upload to Chrome Web Store
npx chrome-webstore-upload upload \
  --source dist/ \
  --extension-id $CHROME_EXTENSION_ID \
  --client-id $CHROME_CLIENT_ID \
  --client-secret $CHROME_CLIENT_SECRET \
  --refresh-token $CHROME_REFRESH_TOKEN
```

#### Firefox Add-ons
```bash
# Sign and upload to Firefox
web-ext sign \
  --source-dir dist/ \
  --api-key $FIREFOX_API_KEY \
  --api-secret $FIREFOX_API_SECRET
```

## Analytics Service Deployment (Optional)

### Serverless Function
```javascript
// api/analytics.js (Vercel/Netlify)
export default async function handler(req, res) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { event, data } = req.body;

  // Validate event
  if (!event || !data) {
    return res.status(400).json({ error: 'Missing event or data' });
  }

  try {
    // Store in database
    await supabase.from('analytics').insert({
      event,
      data,
      user_agent: req.headers['user-agent'],
      ip: req.ip,
      timestamp: new Date()
    });

    res.status(200).json({ success: true });
  } catch (error) {
    console.error('Analytics error:', error);
    res.status(500).json({ error: 'Internal server error' });
  }
}
```

### Database Setup (Supabase)
```sql
-- Analytics table
CREATE TABLE analytics (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  event TEXT NOT NULL,
  data JSONB NOT NULL,
  user_agent TEXT,
  ip TEXT,
  timestamp TIMESTAMP DEFAULT NOW()
);

-- Indexes for performance
CREATE INDEX idx_analytics_event ON analytics(event);
CREATE INDEX idx_analytics_timestamp ON analytics(timestamp);
CREATE INDEX idx_analytics_data_site ON analytics USING GIN ((data->>'site'));
```

## Version Management

### Semantic Versioning
```json
// package.json
{
  "version": "1.2.3",
  "scripts": {
    "version:patch": "npm version patch && npm run build",
    "version:minor": "npm version minor && npm run build",
    "version:major": "npm version major && npm run build"
  }
}
```

### Auto-Update Strategy
```javascript
// background/update-checker.js
class UpdateChecker {
  constructor() {
    this.currentVersion = chrome.runtime.getManifest().version;
    this.checkInterval = 24 * 60 * 60 * 1000; // 24 hours
  }

  async checkForUpdates() {
    try {
      const response = await fetch('https://api.github.com/repos/your-repo/releases/latest');
      const release = await response.json();
      
      if (this.isNewerVersion(release.tag_name)) {
        this.notifyUpdate(release);
      }
    } catch (error) {
      console.error('Update check failed:', error);
    }
  }

  isNewerVersion(latestVersion) {
    // Compare semantic versions
    const current = this.currentVersion.split('.').map(Number);
    const latest = latestVersion.replace('v', '').split('.').map(Number);
    
    for (let i = 0; i < 3; i++) {
      if (latest[i] > current[i]) return true;
      if (latest[i] < current[i]) return false;
    }
    return false;
  }
}
```

## Security Considerations

### Content Security Policy
```json
// manifest.json
{
  "content_security_policy": {
    "extension_pages": "script-src 'self'; object-src 'self'"
  }
}
```

### Input Validation
```javascript
// shared/validation.js
export function validateAnalyticsData(data) {
  const allowedEvents = [
    'extract_recipe',
    'overlay_open',
    'copy_recipe',
    'error'
  ];

  if (!allowedEvents.includes(data.event)) {
    throw new Error('Invalid event type');
  }

  // Sanitize URLs
  if (data.url) {
    data.url = new URL(data.url).origin; // Only store origin
  }

  return data;
}
```

## Monitoring and Maintenance

### Error Reporting
```javascript
// content/error-reporter.js
class ErrorReporter {
  constructor() {
    this.errorQueue = [];
    this.maxErrors = 10;
  }

  report(error, context = {}) {
    const errorData = {
      message: error.message,
      stack: error.stack,
      context: {
        url: window.location.href,
        userAgent: navigator.userAgent,
        extensionVersion: chrome.runtime.getManifest().version,
        ...context
      },
      timestamp: new Date().toISOString()
    };

    this.errorQueue.push(errorData);
    this.sendErrors();
  }

  async sendErrors() {
    if (this.errorQueue.length === 0) return;

    try {
      await fetch('https://your-analytics-endpoint.com/errors', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ errors: this.errorQueue })
      });
      
      this.errorQueue = [];
    } catch (error) {
      // Keep errors for retry
      if (this.errorQueue.length > this.maxErrors) {
        this.errorQueue.shift();
      }
    }
  }
}
```

## Cost Analysis

### 100% Free Stack
- **Chrome Web Store**: $5 (one-time developer fee)
- **Firefox Add-ons**: Free
- **Edge Add-ons**: Free
- **Google Analytics**: Free (unlimited events)
- **GitHub**: Free (public repository)
- **GitHub Actions**: Free (CI/CD)
- **GitHub Pages**: Free (documentation)
- **Development Tools**: Free (VS Code, Node.js)

**Total Cost**: $5 one-time (Chrome Store fee)

### No Recurring Costs
- **No server hosting** (client-side only)
- **No database fees** (local storage)
- **No API costs** (no backend)
- **No CDN fees** (extension stores handle distribution)
- **No analytics fees** (Google Analytics free tier)

## Deployment Checklist

### Pre-Release
- [ ] All tests passing
- [ ] Code review completed
- [ ] Documentation updated
- [ ] Version number incremented
- [ ] Manifest files validated
- [ ] Extension tested on target browsers

### Release
- [ ] Build artifacts created
- [ ] Extension packages uploaded to stores
- [ ] Release notes published
- [ ] Analytics tracking deployed
- [ ] Documentation updated

### Post-Release
- [ ] Monitor error reports
- [ ] Track usage metrics
- [ ] Gather user feedback
- [ ] Plan next iteration
