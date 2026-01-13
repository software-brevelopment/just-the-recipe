# System Architecture

## Client-Side Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Browser       │    │   Browser       │    │   Browser       │
│   Extension     │    │   Extension     │    │   Extension     │
│   (Content)     │    │   (Background)  │    │   (Popup)       │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                │                        │
                                └──────────┬─────────────┘
                                           ▼
                                ┌─────────────────┐
                                │   Local         │
                                │   Storage       │
                                └─────────────────┘
                                           │
                                           ▼ (Optional)
                                ┌─────────────────┐
                                │   Analytics     │
                                │   Service       │
                                └─────────────────┘
```

## Data Flow

1. **Input**: User visits recipe website
2. **Detect**: Extension identifies recipe content
3. **Extract**: Parse HTML locally using multiple strategies
4. **Clean**: Remove noise, normalize format
5. **Store**: Cache results in browser local storage
6. **Present**: Display clean recipe data via modal overlay
7. **Track**: Optional anonymous usage analytics

## Overlay Display Strategy

### User Experience Flow
```
User visits recipe page → Extension detects recipe → Toolbar icon highlights
User clicks icon → Extracts recipe → Shows modal overlay with clean recipe
User can close overlay → Returns to original page → Recipe cached locally
```

### Overlay Features
- **Modal overlay** with clean recipe display
- **Toggle on/off** to view original page
- **Copy/print** functionality 
- **Responsive design** for mobile/desktop
- **Keyboard shortcuts** (ESC to close, Ctrl+C to copy)
- **Local caching** for instant re-opening

## Technology Stack

### Browser Extension
- **Manifest V3** (Chrome/Firefox compatible)
- **Vanilla JavaScript** or **TypeScript**
- **Content Scripts** for DOM manipulation
- **Background Scripts** for processing
- **Local Storage** for caching

### Analytics (Optional)
- **Google Analytics 4** or **Plausible** for usage tracking
- **Serverless function** (Vercel/Netlify) for custom events
- **Simple database** (Supabase/PlanetScale) for telemetry storage
