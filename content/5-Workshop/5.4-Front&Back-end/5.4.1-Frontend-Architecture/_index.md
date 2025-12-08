---
title : "Frontend Architecture"
date: 2025-09-09
weight : 4 
chapter : false
pre : " <b> 5.4.1 </b> "
---



# Frontend Architecture - Storm Prediction Web Application

## Overview

Below is the detailed documentation of our front-end development:
a React + TypeScript web application for tracking and predicting typhoon trajectories.

## AWS Services Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                         USER BROWSER                        │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                    CloudFront CDN                           │
│  - Distribution: d3lj47ilp0fgxy.cloudfront.net              │
│  - SSL/TLS: HTTPS                                           │
│  - Cache: Static assets + JSON data                         │
│  - Origin Access: OAI/OAC (Secure)                          │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                    S3 Bucket (Private)                      │
│  - Bucket: storm-frontend-hosting-duc-2025                  │
│  - Static Website Hosting: DISABLED                         │
│  - Access: CloudFront only via REST API                     │
│  - Content: HTML, CSS, JS, Images, recent_storms.json       │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                    Lambda Functions                         │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Lambda #1: Storm Prediction                         │    │
│  │ - URL: vill3povlzqxdyxm7ubldizobu0kdgbi...          │    │
│  │ - Method: POST /predict                             │    │
│  │ - Auth: NONE (public)                               │    │
│  │ - Container: ECR (Docker)                           │    │
│  │ - Models: LSTM + TCN                                │    │
│  └─────────────────────────────────────────────────────┘    │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Lambda #2: Storm Data Crawler (New)                 │    │
│  │ - Trigger: EventBridge (Weekly)                     │    │
│  │ - Function: Crawl IBTrACS data                      │    │
│  │ - Output: recent_storms.json → S3                   │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                    EventBridge                              │
│  - Rule: storm-data-crawler-weekly-trigger                  │
│  - Schedule: Every Sunday 00:00 UTC (7AM Vietnam)           │
│  - Target: Lambda #2 (Storm Data Crawler)                   │
└─────────────────────────────────────────────────────────────┘
```


## Frontend Directory Structure

```
frontend/
├── src/
│   ├── components/              # React components
│   │   ├── ui/                 # shadcn/ui components (button, card, input, etc.)
│   │   ├── storm/              # Storm-specific components
│   │   ├── timeline/           # Timeline controls
│   │   ├── wind/               # Wind visualization
│   │   ├── StormPredictionForm.tsx      # Storm coordinate input form
│   │   ├── WeatherMap.tsx               # Main Leaflet map
│   │   ├── StormTracker.tsx             # Storm list
│   │   ├── StormInfo.tsx                # Storm details
│   │   ├── StormAnimation.tsx           # Animated markers
│   │   ├── WeatherOverlay.tsx           # Temperature/Wind overlay
│   │   ├── WeatherLayerControl.tsx      # Satellite/Radar layers
│   │   ├── WeatherLayerControlPanel.tsx # Control panel UI
│   │   ├── WeatherValueTooltip.tsx      # Hover tooltip
│   │   ├── WindyLayer.tsx               # Windy.com integration
│   │   ├── ProvinceLayer.tsx            # Vietnam provinces
│   │   ├── OptimizedTemperatureLayer.tsx
│   │   ├── TemperatureHeatMapLayer.tsx
│   │   ├── ThemeToggle.tsx              # Dark/Light mode
│   │   ├── PreferencesModal.tsx         # User preferences
│   │   ├── RightSidebar.tsx             # Right panel
│   │   └── WeeklyForecast.tsx           # 7-day forecast
│   │
│   ├── pages/
│   │   ├── Index.tsx           # Main page
│   │   └── NotFound.tsx        # 404 page
│   │
│   ├── lib/                    # Business logic & utilities
│   │   ├── api/               # API clients
│   │   ├── __tests__/         # Unit tests
│   │   ├── stormData.ts       # Types & interfaces
│   │   ├── stormAnimations.ts # Animation logic
│   │   ├── stormIntensityChanges.ts
│   │   ├── stormPerformance.ts
│   │   ├── stormValidation.ts
│   │   ├── windData.ts
│   │   ├── windStrengthCalculations.ts
│   │   ├── windyStatePersistence.ts
│   │   ├── windyUrlState.ts
│   │   ├── mapUtils.ts        # Map helpers
│   │   ├── openWeatherMapClient.ts
│   │   ├── dataWorker.ts      # Web Worker
│   │   ├── utils.ts
│   │   └── colorInterpolation.ts
│   │
│   ├── hooks/                  # Custom React hooks
│   │   ├── use-toast.ts
│   │   ├── use-theme.tsx
│   │   ├── use-mobile.tsx
│   │   ├── useTimelineState.ts
│   │   ├── useWindyStateSync.ts
│   │   └── useSimplifiedTooltip.ts
│   │
│   ├── contexts/               # React Context
│   │   └── WindyStateContext.tsx
│   │
│   ├── api/
│   │   └── weatherApi.ts      # API calls
│   │
│   ├── utils/
│   │   └── colorInterpolation.ts
│   │
│   ├── styles/
│   │   └── accessibility.css  # WCAG compliance styles
│   │
│   ├── test/                   # Test suite
│   │   ├── accessibility.test.ts
│   │   ├── accessibility-audit.test.ts
│   │   ├── wcag-compliance.test.ts
│   │   ├── performance.test.ts
│   │   ├── cross-browser.test.ts
│   │   └── setup.ts
│   │
│   ├── assets/                 # Images, icons
│   ├── App.tsx
│   ├── main.tsx
│   └── index.css
│
├── public/                     # Static assets
├── dist/                       # Build output (after npm run build)
├── .env.production             # Production config
├── .env.example
├── package.json
├── vite.config.ts
├── vitest.config.ts            # Test config
├── tailwind.config.ts
├── tsconfig.json
└── components.json             # shadcn/ui config
```

## Environment Variables

### `.env.production`
```bash
# OpenWeather API
VITE_OPENWEATHER_API_KEY=8ff7f009d2bd420c86845c6bcf6de4a9

# CloudFront URL - Fetch storm data
VITE_CLOUDFRONT_URL=https://d3lj47ilp0fgxy.cloudfront.net

# Lambda Function URL - Storm prediction API
VITE_PREDICTION_API_URL=https://vill3povlzqxdyxm7ubldizobu0kdgbi.lambda-url.ap-southeast-1.on.aws
```

** Screenshots needed:**
- [ ] AWS CloudFront → Distributions → Distribution domain name
- [ ] AWS Lambda → storm-prediction → Function URL


## Build & Deploy Process

### 1. Build Production
```bash
cd frontend
npm run build
```

**Output**: `dist/` folder contains:
- `index.html`
- `assets/index-[hash].js`
- `assets/index-[hash].css`

### 2. Upload to S3
```bash
aws s3 sync dist/ s3://storm-frontend-hosting-duc-2025/ --delete
```
Important Notes:

S3 bucket is PRIVATE (no public access)
CloudFront uses REST API endpoint, not website endpoint
Origin: storm-frontend-hosting-duc-2025.s3.ap-southeast-1.amazonaws.com

### 3. Invalidate CloudFront Cache
```bash
aws cloudfront create-invalidation \
  --distribution-id E1234567890ABC \
  --paths "/*"
```

## Data Flow

### A. Load Storm Data (Startup)
```
Browser → CloudFront → S3 
  ↓
GET /recent_storms.json
  ↓
Parse JSON → Display on map
```

**File**: `src/pages/Index.tsx` (line ~40)
```typescript
const CLOUDFRONT_URL = import.meta.env.VITE_CLOUDFRONT_URL;
const FETCH_URL = `${CLOUDFRONT_URL}/recent_storms.json?t=${Date.now()}`;
```

### B. Storm Prediction (User Action)
```
User fills form → Click "Run Prediction"
  ↓
POST /predict to Lambda Function URL
  ↓
{
  "history": [{lat, lng}, ...],
  "storm_name": "Test Storm"
}
  ↓
Lambda processes → Returns forecast
  ↓
Display predicted path on map
```

**File**: `src/components/StormPredictionForm.tsx` (line ~80)
```typescript
const API_URL = `${import.meta.env.VITE_PREDICTION_API_URL}/predict`;
const response = await fetch(API_URL, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ history, storm_name })
});
```

## Key Components

### 1. Core Components

#### StormPredictionForm
**File**: `src/components/StormPredictionForm.tsx`

**Features**:
- Form for inputting storm coordinates (min 9 points)
- Input validation (valid lat/lng)
- Calls Lambda API for prediction
- Displays results on map
- Scrollable list with Add/Remove positions

**Props**:
```typescript
interface StormPredictionFormProps {
  onPredictionResult: (result: PredictionResult) => void;
  setIsLoading: (isLoading: boolean) => void;
}
```

#### WeatherMap
**File**: `src/components/WeatherMap.tsx`

**Features**:
- Displays Leaflet map
- Renders storm tracks (historical + forecast)
- Renders prediction path (purple, dashed)
- Weather overlays (temperature, wind, radar)
- Multiple storm rendering
- Auto-zoom to selected storm
- Custom panes for z-index layering

**Props**:
```typescript
interface WeatherMapProps {
  storms: Storm[];
  selectedStorm?: Storm;
  customPrediction?: PredictionResult | null;
  mapFocusBounds?: LatLngBounds | null;
  onMapFocusComplete?: () => void;
}
```

#### Index (Main Page)
**File**: `src/pages/Index.tsx`

**Features**:
- Main layout with header/footer
- State management (storms, selectedStorm, customPrediction)
- Sidebar with tabs (Current Storms / Predict Storm)
- Timeline state synchronization
- Loading & error handling
- Skip links for accessibility

### 2. Storm Components

#### StormTracker
**File**: `src/components/StormTracker.tsx`
- List of current storms
- Filter by status (active/developing/dissipated)
- Click to select storm

#### StormInfo
**File**: `src/components/StormInfo.tsx`
- Detailed storm information
- Wind speed, pressure, category
- Historical data
- Forecast timeline

#### StormAnimation
**File**: `src/components/StormAnimation.tsx`
- Animated markers for storm positions
- Pulsing effect
- Category-based colors

### 3. Weather Layer Components

#### WeatherOverlay
**File**: `src/components/WeatherOverlay.tsx`
- Temperature heatmap overlay
- Wind speed visualization
- Real-time data from OpenWeather API
- Hover to view values

#### WeatherLayerControl
**File**: `src/components/WeatherLayerControl.tsx`
- Satellite imagery layer
- Radar layer
- Temperature layer
- Tile layer management

#### WeatherLayerControlPanel
**File**: `src/components/WeatherLayerControlPanel.tsx`
- UI controls for weather layers
- Opacity slider
- Layer toggle buttons
- Temperature animation toggle

#### OptimizedTemperatureLayer & TemperatureHeatMapLayer
**Files**: `src/components/OptimizedTemperatureLayer.tsx`, `TemperatureHeatMapLayer.tsx`
- Performance-optimized temperature rendering
- Color interpolation
- Grid-based heatmap

### 4. Wind Components

#### WindyLayer
**File**: `src/components/WindyLayer.tsx`
- Windy.com iframe integration
- Wind animation overlay
- Synchronized state with main map

**Context**: `src/contexts/WindyStateContext.tsx`
- Global state for Windy layer
- URL state persistence
- Sync across components

### 5. Map Enhancement Components

#### ProvinceLayer
**File**: `src/components/ProvinceLayer.tsx`
- Vietnam provinces boundaries
- GeoJSON rendering
- Province labels

#### WeatherValueTooltip
**File**: `src/components/WeatherValueTooltip.tsx`
- Tooltip displaying weather values on hover
- Temperature, wind speed, pressure
- Positioned tooltip

### 6. UI Components

#### ThemeToggle
**File**: `src/components/ThemeToggle.tsx`
- Dark/Light mode switch
- Persisted preference
- System theme detection

#### PreferencesModal
**File**: `src/components/PreferencesModal.tsx`
- User preferences settings
- Map options
- Display preferences

#### RightSidebar
**File**: `src/components/RightSidebar.tsx`
- Additional info panel
- Collapsible sidebar

#### WeeklyForecast
**File**: `src/components/WeeklyForecast.tsx`
- 7-day weather forecast
- Temperature trends
- Weather icons

### 7. Timeline Components
**Folder**: `src/components/timeline/`
- Timeline controls for storm animation
- Play/Pause functionality
- Time scrubbing
- Speed controls

## Data Types

### PredictionResult
**File**: `src/lib/stormData.ts`

```typescript
export interface PredictionResult {
  storm_id: string;
  storm_name: string;
  prediction_time: string;
  totalDistance: number;      // km
  actualDistance: number;      // km
  lifespan: number;            // hours
  forecastHours: number;       // hours
  forecast: StormPoint[];      // Predicted positions
  path?: StormPoint[];         // Legacy support
}
```

### StormPoint
```typescript
export interface StormPoint {
  timestamp: number;           // Unix timestamp (ms)
  lat: number;
  lng: number;
  windSpeed: number;           // km/h
  pressure: number;            // hPa
  category: string;            // "Typhoon", "Super Typhoon", etc.
}
```

## AWS Permissions Required

### S3 Bucket Policy
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::storm-frontend-hosting-duc-2025/*"
    }
  ]
}
```

### CloudFront Origin Access
- Origin: S3 bucket
- Origin Access: Public (or OAI if used)


## Testing

### Local Development
```bash
npm run dev
# Open http://localhost:5173
```

### Production Build Test
```bash
npm run build
npm run preview
# Open http://localhost:4173
```

**📸 Screenshots needed:**
- [ ] Browser DevTools → Network tab → API calls
- [ ] Browser DevTools → Console → No errors


##  Common Issues

### 1. CORS Error when calling Lambda
**Symptom**: `Access-Control-Allow-Origin` error

**Solution**: Lambda must return CORS headers:
```python
return {
    'statusCode': 200,
    'headers': {
        'Content-Type': 'application/json',
        'Access-Control-Allow-Origin': '*'
    },
    'body': json.dumps(result)
}
```

### 2. CloudFront stale cache
**Symptom**: New code not showing

**Solution**: Invalidate cache
```bash
aws cloudfront create-invalidation --distribution-id E... --paths "/*"
```

### 3. Environment variables not loading
**Symptom**: `undefined` when accessing `import.meta.env.VITE_*`

**Solution**: 
- Ensure `.env.production` file exists
- Rebuild: `npm run build`
- Variables must start with `VITE_`

##  Deployment Checklist

- [ ] Update `.env.production` with correct URLs
- [ ] `npm run build` succeeds
- [ ] Upload `dist/` to S3
- [ ] Invalidate CloudFront cache
- [ ] Test on production URL
- [ ] Verify Lambda API works
- [ ] Verify storm data loads
- [ ] Test prediction form with 9+ positions

## API Endpoints

### 1. Get Storm Data
```
GET https://d3lj47ilp0fgxy.cloudfront.net/recent_storms.json
```

**Response**: Array of Storm objects

### 2. Predict Storm Path
```
POST https://vill3povlzqxdyxm7ubldizobu0kdgbi.lambda-url.ap-southeast-1.on.aws/predict

Body:
{
  "history": [
    {"lat": 15.0, "lng": 120.0},
    {"lat": 15.1, "lng": 120.1},
    ...
  ],
  "storm_name": "Test Storm"
}

Response:
{
  "storm_id": "unknown",
  "storm_name": "Test Storm",
  "totalDistance": 500.5,
  "lifespan": 72,
  "forecast": [...]
}
```

## Performance Optimization

### 1. Code Optimization
- **Code Splitting**: Vite automatically splits chunks by routes
- **Tree Shaking**: Remove unused code
- **Minification**: Production build auto-minifies JS/CSS
- **Lazy Loading**: Components load on demand

### 2. Data Optimization
- **Web Workers**: Heavy computations run in worker (`dataWorker.ts`)
- **Memoization**: React.memo for expensive components
- **Debouncing**: Input handlers are debounced
- **Caching**: LocalStorage cache for preferences

### 3. Rendering Optimization
- **Virtual Scrolling**: Large lists use virtual scrolling
- **Optimized Layers**: `OptimizedTemperatureLayer` for performance
- **Canvas Rendering**: Heatmap uses canvas instead of DOM
- **Pane Management**: Custom Leaflet panes for z-index optimization

### 4. Network Optimization
- **CDN Caching**: CloudFront caches static assets
- **Image Optimization**: WebP format, lazy loading
- **API Caching**: Cache storm data with timestamp
- **Compression**: Gzip/Brotli compression

### 5. Accessibility Performance
- **Skip Links**: Keyboard navigation shortcuts
- **ARIA Labels**: Proper semantic HTML
- **Focus Management**: Logical tab order
- **Screen Reader**: Optimized for screen readers

## Libraries & Utilities

### Business Logic (lib/)

#### Storm Management
- **stormData.ts**: Types, interfaces, Storm/StormPoint definitions
- **stormAnimations.ts**: Animation logic for storm markers
- **stormIntensityChanges.ts**: Storm intensity change calculations
- **stormPerformance.ts**: Performance optimization for rendering
- **stormValidation.ts**: Storm data validation

#### Wind System
- **windData.ts**: Wind data structures
- **windStrengthCalculations.ts**: Wind strength calculations
- **windyStatePersistence.ts**: Windy layer state persistence
- **windyUrlState.ts**: URL state management for Windy

#### Map & Weather
- **mapUtils.ts**: Map helpers (center, zoom, bounds calculations)
- **openWeatherMapClient.ts**: OpenWeather API client
- **colorInterpolation.ts**: Color gradient calculations

#### Performance
- **dataWorker.ts**: Web Worker for heavy computations
- **utils.ts**: General utilities

### Custom Hooks (hooks/)

- **use-toast.ts**: Toast notification system
- **use-theme.tsx**: Dark/Light theme management
- **use-mobile.tsx**: Mobile device detection
- **useTimelineState.ts**: Timeline state synchronization
- **useWindyStateSync.ts**: Windy layer state sync
- **useSimplifiedTooltip.ts**: Simplified tooltip logic

### Context (contexts/)

- **WindyStateContext.tsx**: Global state for Windy layer integration

### Testing (test/)

- **accessibility.test.ts**: Accessibility testing
- **accessibility-audit.test.ts**: WCAG audit
- **wcag-compliance.test.ts**: WCAG 2.1 compliance
- **performance.test.ts**: Performance benchmarks
- **cross-browser.test.ts**: Cross-browser compatibility
- **setup.ts**: Test environment setup

### Dependencies

#### Core
- React 18
- TypeScript
- Vite (build tool)
- Vitest (testing)

#### UI Framework
- Tailwind CSS
- shadcn/ui (component library)
- Lucide Icons
- Radix UI (primitives)

#### Map & Visualization
- Leaflet
- React-Leaflet
- GeoJSON support

#### API & Data
- Fetch API (native)
- OpenWeather API
- AWS Lambda Function URL

#### State Management
- React Context API
- URL state (query params)
- LocalStorage persistence

#### Performance
- Web Workers
- Code splitting (Vite)
- Lazy loading

## Screenshots

### Cloudfront 

#### Distribution
<p align="center">
  <img src="/images/5-Workshop/5.4-FB_END/5.4.1-FB/image.png" alt="Picture 1" />
  <br/>
  <strong style="font-size: 18px;">Figure 1</strong>
</p>

#### Origin Settings

<p align="center">
  <img src="/images/5-Workshop/5.4-FB_END/5.4.1-FB/image-1.png" alt="Picture 1" />
  <br/>
  <strong style="font-size: 18px;">Figure 1</strong>
</p>


#### Invalidations

<p align="center">
  <img src="/images/5-Workshop/5.4-FB_END/5.4.1-FB/image-2.png" alt="Picture 2" />
  <br/>
  <strong style="font-size: 18px;">Figure 2</strong>
</p>

#### storm-frontend-hosting-duc-2025

<p align="center">
  <img src="/images/5-Workshop/5.4-FB_END/5.4.1-FB/image-3.png" alt="Picture 3" />
  <br/>
  <strong style="font-size: 18px;">Figure 3</strong>
</p>

#### Permissions

<p align="center">
  <img src="/images/5-Workshop/5.4-FB_END/5.4.1-FB/image-4.png" alt="Picture 4" />
  <br/>
  <strong style="font-size: 18px;">Figure 4</strong>
</p>

#### storm-ai-models-2025

<p align="center">
  <img src="/images/5-Workshop/5.4-FB_END/5.4.1-FB/image-5.png" alt="Picture 5" />
  <br/>
  <strong style="font-size: 18px;">Figure 5</strong>
</p>

#### storm-data-store-2025

<p align="center">
  <img src="/images/5-Workshop/5.4-FB_END/5.4.1-FB/image-6.png" alt="Picture 6" />
  <br/>
  <strong style="font-size: 18px;">Figure 6</strong>
</p>

#### Main Page

<p align="center">
  <img src="/images/5-Workshop/5.4-FB_END/5.4.1-FB/image-7.png" alt="Picture 7" />
  <br/>
  <strong style="font-size: 18px;">Figure 7</strong>
</p>

#### Storm Tracking Features

<p align="center">
  <img src="/images/5-Workshop/5.4-FB_END/5.4.1-FB/image-8.png" alt="Picture 8" />
  <br/>
  <strong style="font-size: 18px;">Figure 8</strong>
</p>

#### Storm Details

<p align="center">
  <img src="/images/5-Workshop/5.4-FB_END/5.4.1-FB/image-9.png" alt="Picture 9" />
  <br/>
  <strong style="font-size: 18px;">Figure 9</strong>
</p>

#### Predict Feature

<p align="center">
  <img src="/images/5-Workshop/5.4-FB_END/5.4.1-FB/image-10.png" alt="Picture 10" />
  <br/>
  <strong style="font-size: 18px;">Figure 10</strong>
</p>

<p align="center">
  <img src="/images/5-Workshop/5.4-FB_END/5.4.1-FB/image-11.png" alt="Picture 11" />
  <br/>
  <strong style="font-size: 18px;">Figure 11</strong>
</p>

<p align="center">
  <img src="/images/5-Workshop/5.4-FB_END/5.4.1-FB/image-12.png" alt="Picture 12" />
  <br/>
  <strong style="font-size: 18px;">Figure 12</strong>
</p>
