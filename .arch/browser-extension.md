# Browser Extension Architecture

## Extension Components

### Manifest V3 Structure
```json
{
  "manifest_version": 3,
  "name": "Just the Recipe",
  "version": "1.0.0",
  "description": "Extract clean recipes from any website",
  "permissions": [
    "activeTab",
    "storage",
    "scripting"
  ],
  "host_permissions": [
    "https://*/*",
    "http://*/*"
  ],
  "content_scripts": [
    {
      "matches": ["<all_urls>"],
      "js": ["content/content.js"],
      "css": ["content/styles.css"],
      "run_at": "document_idle"
    }
  ],
  "background": {
    "service_worker": "background/background.js"
  },
  "action": {
    "default_popup": "popup/popup.html",
    "default_title": "Just the Recipe"
  },
  "options_page": "options/options.html",
  "icons": {
    "16": "icons/icon16.png",
    "48": "icons/icon48.png",
    "128": "icons/icon128.png"
  }
}
```

## Content Script Strategy

### Recipe Detection
```javascript
// content/detector.js
class RecipeDetector {
  constructor() {
    this.selectors = {
      structured: 'script[type="application/ld+json"]',
      ingredients: [
        '.recipe-ingredients',
        '.ingredients',
        '[itemprop="recipeIngredient"]'
      ],
      instructions: [
        '.recipe-instructions',
        '.instructions',
        '[itemprop="recipeInstructions"]'
      ],
      title: [
        'h1',
        '.recipe-title',
        '[itemprop="name"]'
      ]
    };
  }

  async detectRecipe() {
    // Check for structured data first
    const structuredData = this.extractStructuredData();
    if (structuredData) return structuredData;

    // Fall back to semantic HTML parsing
    const semanticData = this.extractSemanticData();
    if (semanticData) return semanticData;

    // Use pattern matching as last resort
    return this.extractPatternData();
  }
}
```

### In-Page Overlay
```javascript
// content/overlay.js
class RecipeOverlay {
  constructor() {
    this.overlay = null;
    this.isShowing = false;
  }

  show(recipeData) {
    if (this.isShowing) return;
    
    this.createOverlay();
    this.renderRecipe(recipeData);
    this.attachEventListeners();
    this.isShowing = true;
  }

  hide() {
    if (this.overlay) {
      this.overlay.remove();
      this.overlay = null;
      this.isShowing = false;
    }
  }

  createOverlay() {
    this.overlay = document.createElement('div');
    this.overlay.id = 'just-the-recipe-overlay';
    this.overlay.innerHTML = `
      <div class="jtr-container">
        <div class="jtr-header">
          <h3>Just the Recipe</h3>
          <button class="jtr-close">&times;</button>
        </div>
        <div class="jtr-content" id="jtr-recipe-content">
          <!-- Recipe content will be rendered here -->
        </div>
        <div class="jtr-actions">
          <button class="jtr-copy">Copy Recipe</button>
          <button class="jtr-print">Print</button>
        </div>
      </div>
    `;
    document.body.appendChild(this.overlay);
  }
}
```

## Background Service Worker

### API Communication
```javascript
// background/api.js
class RecipeAPI {
  constructor(apiKey) {
    this.apiKey = apiKey;
    this.baseUrl = 'https://api.just-the-recipe.com';
  }

  async extractRecipe(url, options = {}) {
    try {
      const response = await fetch(`${this.baseUrl}/api/v1/extract`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${this.apiKey}`
        },
        body: JSON.stringify({ url, options })
      });

      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }

      return await response.json();
    } catch (error) {
      console.error('Recipe extraction failed:', error);
      throw error;
    }
  }
}
```

### Message Passing
```javascript
// background/messages.js
chrome.runtime.onMessage.addListener((request, sender, sendResponse) => {
  switch (request.type) {
    case 'EXTRACT_RECIPE':
      handleRecipeExtraction(request.url, request.options)
        .then(sendResponse)
        .catch(error => sendResponse({ error: error.message }));
      return true; // Keep message channel open

    case 'GET_SETTINGS':
      getSettings().then(sendResponse);
      return true;

    case 'SAVE_SETTINGS':
      saveSettings(request.settings).then(sendResponse);
      return true;
  }
});
```

## Popup UI

### Main Popup
```html
<!-- popup/popup.html -->
<!DOCTYPE html>
<html>
<head>
  <link rel="stylesheet" href="popup.css">
</head>
<body>
  <div class="popup-container">
    <div class="header">
      <img src="../icons/icon48.png" alt="Just the Recipe">
      <h1>Just the Recipe</h1>
    </div>
    
    <div class="main-content">
      <div id="status-message" class="status"></div>
      
      <div id="recipe-preview" class="recipe-preview hidden">
        <h3 id="recipe-title"></h3>
        <div class="recipe-stats">
          <span id="ingredient-count"></span>
          <span id="instruction-count"></span>
        </div>
        <button id="view-full-recipe" class="primary-btn">View Full Recipe</button>
      </div>
      
      <div id="actions" class="actions">
        <button id="extract-recipe" class="primary-btn">Extract Recipe</button>
        <button id="open-settings" class="secondary-btn">Settings</button>
      </div>
    </div>
  </div>
  <script src="popup.js"></script>
</body>
</html>
```

### Popup Logic
```javascript
// popup/popup.js
class PopupController {
  constructor() {
    this.currentTab = null;
    this.extractedRecipe = null;
    this.init();
  }

  async init() {
    this.currentTab = await this.getCurrentTab();
    this.attachEventListeners();
    this.checkForRecipe();
  }

  async checkForRecipe() {
    try {
      // Check if content script found a recipe
      const response = await chrome.tabs.sendMessage(
        this.currentTab.id, 
        { type: 'CHECK_RECIPE' }
      );

      if (response.hasRecipe) {
        this.showRecipeFound(response.recipePreview);
      } else {
        this.showNoRecipe();
      }
    } catch (error) {
      this.showError('Unable to check page content');
    }
  }

  async extractRecipe() {
    this.showLoading('Extracting recipe...');
    
    try {
      const response = await chrome.runtime.sendMessage({
        type: 'EXTRACT_RECIPE',
        url: this.currentTab.url,
        options: await this.getExtractionOptions()
      });

      if (response.success) {
        this.extractedRecipe = response.data;
        this.showExtractedRecipe();
        
        // Show overlay on the page
        await chrome.tabs.sendMessage(
          this.currentTab.id,
          { 
            type: 'SHOW_RECIPE_OVERLAY',
            recipe: response.data 
          }
        );
      } else {
        this.showError(response.error || 'Extraction failed');
      }
    } catch (error) {
      this.showError('Network error occurred');
    }
  }
}
```

## Options Page

### Settings Management
```javascript
// options/settings.js
class SettingsManager {
  constructor() {
    this.defaultSettings = {
      autoExtract: true,
      showOverlay: true,
      includeImages: false,
      normalizeUnits: true,
      theme: 'light',
      apiEndpoint: 'https://api.just-the-recipe.com'
    };
    this.settings = {};
    this.init();
  }

  async init() {
    await this.loadSettings();
    this.renderSettings();
    this.attachEventListeners();
  }

  async loadSettings() {
    const stored = await chrome.storage.sync.get('settings');
    this.settings = { ...this.defaultSettings, ...stored.settings };
  }

  async saveSettings(newSettings) {
    this.settings = { ...this.settings, ...newSettings };
    await chrome.storage.sync.set({ settings: this.settings });
  }

  renderSettings() {
    // Render settings form based on current settings
    document.getElementById('auto-extract').checked = this.settings.autoExtract;
    document.getElementById('show-overlay').checked = this.settings.showOverlay;
    document.getElementById('include-images').checked = this.settings.includeImages;
    document.getElementById('normalize-units').checked = this.settings.normalizeUnits;
    document.getElementById('theme').value = this.settings.theme;
  }
}
```

## Cross-Browser Compatibility

### Browser Detection
```javascript
// shared/browser.js
export const browserAPI = {
  get runtime() {
    return typeof chrome !== 'undefined' ? chrome : browser;
  },
  get storage() {
    return typeof chrome !== 'undefined' ? chrome.storage : browser.storage;
  },
  get tabs() {
    return typeof chrome !== 'undefined' ? chrome.tabs : browser.tabs;
  }
};
```

### Build Configuration
```javascript
// webpack.config.js
module.exports = {
  entry: {
    background: './src/background/background.js',
    content: './src/content/content.js',
    popup: './src/popup/popup.js',
    options: './src/options/options.js'
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
      }
    ]
  }
};
```

## Security Considerations

### Content Security Policy
```json
{
  "content_security_policy": {
    "extension_pages": "script-src 'self'; object-src 'self'"
  }
}
```

### Input Validation
```javascript
// shared/validation.js
export function validateURL(url) {
  try {
    const urlObj = new URL(url);
    return ['http:', 'https:'].includes(urlObj.protocol);
  } catch {
    return false;
  }
}

export function sanitizeHTML(html) {
  const div = document.createElement('div');
  div.textContent = html;
  return div.innerHTML;
}
```

## Performance Optimization

### Lazy Loading
```javascript
// content/lazy-loader.js
class LazyLoader {
  constructor() {
    this.observer = new IntersectionObserver(this.handleIntersection.bind(this));
  }

  observe(element) {
    this.observer.observe(element);
  }

  handleIntersection(entries) {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        this.loadContent(entry.target);
        this.observer.unobserve(entry.target);
      }
    });
  }
}
```

### Caching Strategy
```javascript
// background/cache.js
class RecipeCache {
  constructor() {
    this.cache = new Map();
    this.maxAge = 24 * 60 * 60 * 1000; // 24 hours
  }

  async get(url) {
    const cached = this.cache.get(url);
    if (cached && Date.now() - cached.timestamp < this.maxAge) {
      return cached.data;
    }
    return null;
  }

  async set(url, data) {
    this.cache.set(url, {
      data,
      timestamp: Date.now()
    });
  }
}
```
