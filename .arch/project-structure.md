# Project Structure

## Directory Layout

```
just-the-recipe/
├── .arch/                          # Architecture documentation
│   ├── README.md                   # Project overview
│   ├── system-architecture.md      # System design
│   ├── extraction-strategies.md    # Extraction methods
│   ├── data-models.md             # Data schemas
│   ├── project-structure.md       # This file
│   ├── analytics-strategy.md      # Analytics and telemetry
│   └── implementation-roadmap.md   # Development roadmap
├── packages/                       # Monorepo packages
│   ├── browser-extension/         # Chrome/Firefox extension
│   │   ├── src/
│   │   │   ├── content/           # Content scripts
│   │   │   │   ├── detector.js    # Recipe detection
│   │   │   │   ├── extractor.js   # Recipe extraction
│   │   │   │   ├── overlay.js     # In-page overlay
│   │   │   │   └── analytics.js   # Usage tracking
│   │   │   ├── background/        # Background scripts
│   │   │   │   ├── service-worker.js
│   │   │   │   ├── storage.js     # Local storage management
│   │   │   │   └── analytics.js   # Analytics coordinator
│   │   │   ├── popup/             # Popup UI
│   │   │   │   ├── popup.html
│   │   │   │   ├── popup.js
│   │   │   │   └── popup.css
│   │   │   ├── options/           # Options page
│   │   │   │   ├── options.html
│   │   │   │   ├── options.js
│   │   │   │   └── options.css
│   │   │   └── shared/            # Shared utilities
│   │   │       ├── constants.js   # Site selectors, constants
│   │   │       ├── utils.js       # Helper functions
│   │   │       └── types.js       # TypeScript definitions
│   │   ├── dist/                  # Build output
│   │   ├── manifest.json          # Extension manifest
│   │   └── package.json
│   └── analytics-service/          # Optional analytics backend
│       ├── src/
│       │   ├── api/               # Analytics endpoints
│       │   ├── db/                # Database models
│       │   └── processing/        # Event processing
│       ├── package.json
│       └── vercel.json            # Serverless deployment
├── docs/                          # Documentation
│   ├── user-guide/                # User documentation
│   └── development/               # Development setup
├── scripts/                       # Build and utility scripts
├── package.json                   # Root package.json
├── lerna.json                     # Monorepo config
└── README.md                      # Project README
```

## Package Responsibilities

### Browser Extension Package
- **Purpose**: Complete client-side recipe extraction
- **Key Features**:
  - Recipe detection and extraction
  - In-page overlay display
  - Local storage and caching
  - Anonymous usage analytics
  - Cross-browser compatibility

### Analytics Service Package (Optional)
- **Purpose**: Lightweight usage tracking
- **Key Features**:
  - Event collection API
  - Simple data storage
  - Basic analytics dashboard
  - Privacy-compliant tracking

## Development Workflow

### Local Development
```bash
# Install dependencies
npm install

# Start extension development
npm run dev:extension

# Start analytics service (optional)
npm run dev:analytics

# Run tests
npm test

# Build extension
npm run build:extension
```

### Package Scripts
```json
{
  "scripts": {
    "dev:extension": "cd packages/browser-extension && npm run watch",
    "dev:analytics": "cd packages/analytics-service && npm run dev",
    "build": "npm run build:extension && npm run build:analytics",
    "build:extension": "cd packages/browser-extension && npm run build",
    "build:analytics": "cd packages/analytics-service && npm run build",
    "test": "npm run test:extension && npm run test:analytics",
    "test:extension": "cd packages/browser-extension && npm test",
    "test:analytics": "cd packages/analytics-service && npm test",
    "lint": "lerna run lint",
    "clean": "lerna run clean"
  }
}
```

## Git Workflow

### Branch Strategy
- `main`: Production-ready code
- `develop`: Integration branch
- `feature/*`: Feature development
- `hotfix/*`: Critical fixes

### Commit Convention
```
feat: Add new feature
fix: Bug fix
docs: Documentation update
style: Code formatting
refactor: Code refactoring
test: Test addition/modification
chore: Build process or dependency update
```

## Environment Configuration

### Development Environment
- Local PostgreSQL database
- Redis for caching
- Mock data for testing
- Hot reload for all packages

### Production Environment
- Managed database service
- Redis cluster
- CDN for static assets
- Containerized deployment
