---
title : "Kiến Trúc Frontend"
date: 2025-09-09
weight : 4 
chapter : false
pre : " <b> 5.4.1 </b> "
---


# Kiến trúc Frontend – Ứng dụng Web Dự báo Bão

## Tổng quan

Dưới đây là tài liệu chi tiết về quá trình phát triển front-end của nhóm: Một ứng dụng web xây dựng bằng React và TypeScript dùng để theo dõi và dự đoán quỹ đạo bão

## Kiến trúc dịch vụ AWS


```
┌─────────────────────────────────────────────────────────────┐
│                        TRÌNH DUYỆT NGƯỜI DÙNG               │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                         CloudFront CDN                      │
│  - Distribution: d3lj47ilp0fgxy.cloudfront.net              │
│  - SSL/TLS: HTTPS                                           │
│  - Cache: Tài nguyên tĩnh + dữ liệu JSON                    │
│  - Truy cập Origin: OAI/OAC (bảo mật)                       │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                    S3 Bucket (Riêng tư)                     │
│  - Bucket: storm-frontend-hosting-duc-2025                  │
│  - Static Website Hosting: TẮT                              │
│  - Quyền truy cập: Chỉ CloudFront được phép qua REST API    │
│  - Nội dung: HTML, CSS, JS, hình ảnh, recent_storms.json    │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                     Các hàm Lambda                          │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Lambda #1: Dự báo bão                               │    │
│  │ - URL: vill3povlzqxdyxm7ubldizobu0kdgbi...          │    │
│  │ - Phương thức: POST /predict                        │    │
│  │ - Xác thực: KHÔNG (công khai)                       │    │
│  │ - Container: ECR (Docker)                           │    │
│  │ - Mô hình: LSTM + TCN                               │    │
│  └─────────────────────────────────────────────────────┘    │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Lambda #2: Thu thập dữ liệu bão (mới)               │    │
│  │ - Kích hoạt: EventBridge (hàng tuần)                │    │
│  │ - Chức năng: Thu thập dữ liệu IBTrACS               │    │
│  │ - Đầu ra: recent_storms.json → S3                   │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                         EventBridge                         │
│  - Rule: storm-data-crawler-weekly-trigger                  │
│  - Lịch chạy: Mỗi Chủ nhật 00:00 UTC (07:00 giờ Việt Nam)   │
│  - Đích: Lambda #2 (Thu thập dữ liệu bão)                   │
└─────────────────────────────────────────────────────────────┘

```

## Cấu trúc thư mục Frontend

```text
frontend/
├── src/
│   ├── components/                         # Các component React
│   │   ├── ui/                             # Component từ shadcn/ui (button, card, input, ...)
│   │   ├── storm/                          # Component chuyên cho nghiệp vụ bão
│   │   ├── timeline/                       # Điều khiển timeline
│   │   ├── wind/                           # Trực quan hóa gió
│   │   ├── StormPredictionForm.tsx         # Form nhập tọa độ bão
│   │   ├── WeatherMap.tsx                  # Bản đồ Leaflet chính
│   │   ├── StormTracker.tsx                # Danh sách bão
│   │   ├── StormInfo.tsx                   # Thông tin chi tiết bão
│   │   ├── StormAnimation.tsx              # Marker hoạt ảnh
│   │   ├── WeatherOverlay.tsx              # Lớp phủ Nhiệt độ/Gió
│   │   ├── WeatherLayerControl.tsx         # Lớp Satellite/Radar
│   │   ├── WeatherLayerControlPanel.tsx    # UI bảng điều khiển lớp dữ liệu
│   │   ├── WeatherValueTooltip.tsx         # Tooltip khi rê chuột
│   │   ├── WindyLayer.tsx                  # Tích hợp Windy.com
│   │   ├── ProvinceLayer.tsx               # Lớp ranh giới tỉnh/thành Việt Nam
│   │   ├── OptimizedTemperatureLayer.tsx
│   │   ├── TemperatureHeatMapLayer.tsx
│   │   ├── ThemeToggle.tsx                 # Chế độ Sáng/Tối
│   │   ├── PreferencesModal.tsx            # Tùy chọn người dùng
│   │   ├── RightSidebar.tsx                # Panel bên phải
│   │   └── WeeklyForecast.tsx              # Dự báo 7 ngày
│   │
│   ├── pages/
│   │   ├── Index.tsx                       # Trang chính
│   │   └── NotFound.tsx                    # Trang 404
│   │
│   ├── lib/                                # Logic nghiệp vụ & tiện ích
│   │   ├── api/                            # Client gọi API
│   │   ├── __tests__/                      # Unit test
│   │   ├── stormData.ts                    # Kiểu dữ liệu & interface
│   │   ├── stormAnimations.ts              # Logic hoạt ảnh
│   │   ├── stormIntensityChanges.ts
│   │   ├── stormPerformance.ts
│   │   ├── stormValidation.ts
│   │   ├── windData.ts
│   │   ├── windStrengthCalculations.ts
│   │   ├── windyStatePersistence.ts
│   │   ├── windyUrlState.ts
│   │   ├── mapUtils.ts                     # Tiện ích hỗ trợ bản đồ
│   │   ├── openWeatherMapClient.ts
│   │   ├── dataWorker.ts                   # Web Worker
│   │   ├── utils.ts
│   │   └── colorInterpolation.ts
│   │
│   ├── hooks/                              # Custom React hooks
│   │   ├── use-toast.ts
│   │   ├── use-theme.tsx
│   │   ├── use-mobile.tsx
│   │   ├── useTimelineState.ts
│   │   ├── useWindyStateSync.ts
│   │   └── useSimplifiedTooltip.ts
│   │
│   ├── contexts/                           # React Context
│   │   └── WindyStateContext.tsx
│   │
│   ├── api/
│   │   └── weatherApi.ts                   # Các hàm gọi API
│   │
│   ├── utils/
│   │   └── colorInterpolation.ts
│   │
│   ├── styles/
│   │   └── accessibility.css               # Style tuân thủ WCAG
│   │
│   ├── test/                               # Bộ test
│   │   ├── accessibility.test.ts
│   │   ├── accessibility-audit.test.ts
│   │   ├── wcag-compliance.test.ts
│   │   ├── performance.test.ts
│   │   ├── cross-browser.test.ts
│   │   └── setup.ts
│   │
│   ├── assets/                             # Ảnh, icon
│   ├── App.tsx
│   ├── main.tsx
│   └── index.css
│
├── public/                                 # Tài nguyên tĩnh
├── dist/                                   # Output sau khi build (npm run build)
├── .env.production                         # Cấu hình môi trường production
├── .env.example
├── package.json
├── vite.config.ts
├── vitest.config.ts                        # Cấu hình test
├── tailwind.config.ts
├── tsconfig.json
└── components.json                         # Cấu hình shadcn/ui
```

## Biến môi trường

### `.env.production`
```bash
# OpenWeather API
VITE_OPENWEATHER_API_KEY=8ff7f009d2bd420c86845c6bcf6de4a9

# CloudFront URL - Dùng để lấy dữ liệu bão
VITE_CLOUDFRONT_URL=https://d3lj47ilp0fgxy.cloudfront.net

# Lambda Function URL - API dự đoán bão
VITE_PREDICTION_API_URL=https://vill3povlzqxdyxm7ubldizobu0kdgbi.lambda-url.ap-southeast-1.on.aws

```

## Quy trình Build & Deploy

### 1. Build bản Production

```bash
cd frontend
npm run build
```

Đầu ra: thư mục `dist/` bao gồm:
- `index.html`
- `assets/index-[hash].js`
- `assets/index-[hash].css`

### 2. Upload lên S3

```bash
aws s3 sync dist/ s3://storm-frontend-hosting-duc-2025/ --delete
```
Important Notes:

- S3 bucket ở chế độ riêng tư (không public)
- CloudFront dùng REST API endpoint, không dùng website endpoint
- Origin: storm-frontend-hosting-duc-2025.s3.ap-southeast-1.amazonaws.com

### 3. Invalidate cache của CloudFront
```bash
aws cloudfront create-invalidation \
  --distribution-id E1234567890ABC \
  --paths "/*"
```


## Luồng dữ liệu

### A. Tải dữ liệu bão (khi khởi động ứng dụng)
```
Trình duyệt → CloudFront → S3
  ↓
GET /recent_storms.json
  ↓
Parse JSON → Hiển thị lên bản đồ
```


**File**: `src/pages/Index.tsx` (dòng ~40)
```typescript
const CLOUDFRONT_URL = import.meta.env.VITE_CLOUDFRONT_URL;
const FETCH_URL = `${CLOUDFRONT_URL}/recent_storms.json?t=${Date.now()}`;
```

### B. Dự báo bão (khi người dùng thao tác)
```
Người dùng điền form → Nhấn "Run Prediction"
  ↓
POST /predict tới Lambda Function URL
  ↓
{
  "history": [{lat, lng}, ...],
  "storm_name": "Test Storm"
}
  ↓
Lambda xử lý → Trả về dự báo
  ↓
Hiển thị đường đi dự đoán lên bản đồ

```

**File**: `src/components/StormPredictionForm.tsx` (dòng ~80)
```typescript
const API_URL = `${import.meta.env.VITE_PREDICTION_API_URL}/predict`;
const response = await fetch(API_URL, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ history, storm_name })
});
```


## Các thành phần chính

### 1. Thành phần cốt lõi

#### StormPredictionForm
**File**: `src/components/StormPredictionForm.tsx`

**Tính năng**:
- Form nhập tọa độ bão (tối thiểu 9 điểm)
- Kiểm tra dữ liệu đầu vào (lat/lng hợp lệ)
- Gọi Lambda API để dự báo
- Hiển thị kết quả lên bản đồ
- Danh sách vị trí có thể cuộn, hỗ trợ thêm/xóa điểm


**thuộc tính truyền**:
```typescript
interface StormPredictionFormProps {
  onPredictionResult: (result: PredictionResult) => void;
  setIsLoading: (isLoading: boolean) => void;
}
```


#### WeatherMap
**File**: `src/components/WeatherMap.tsx`


**Tính năng**:
- Hiển thị bản đồ Leaflet
- Vẽ quỹ đạo bão (lịch sử + dự báo)
- Vẽ đường dự đoán (màu tím, nét đứt)
- Lớp phủ thời tiết (nhiệt độ, gió, radar)
- Hiển thị nhiều cơn bão cùng lúc
- Tự động zoom tới cơn bão đang được chọn
- Dùng các pane tùy chỉnh để quản lý thứ tự lớp (z-index)


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

**Mục cần chụp màn hình:**
- [ ] Web UI → Bản đồ có quỹ đạo bão (xanh/đỏ)
- [ ] Web UI → Đường dự đoán tùy chỉnh (màu tím, nét đứt)
- [ ] Web UI → Lớp phủ thời tiết (nhiệt độ/gió)

#### Index (Trang chính)
**File**: `src/pages/Index.tsx`

**Tính năng**:
- Bố cục chính với header/footer
- Quản lý state (storms, selectedStorm, customPrediction)
- Sidebar có tab (Current Storms / Predict Storm)
- Đồng bộ trạng thái timeline
- Xử lý loading và lỗi
- Skip link hỗ trợ truy cập (accessibility)

### 2. Các component về bão

#### StormTracker
**File**: `src/components/StormTracker.tsx`
- Danh sách các cơn bão hiện tại
- Lọc theo trạng thái (active/developing/dissipated)
- Bấm để chọn cơn bão

#### StormInfo
**File**: `src/components/StormInfo.tsx`
- Thông tin chi tiết về bão
- Tốc độ gió, áp suất, phân loại
- Dữ liệu lịch sử
- Timeline dự báo

#### StormAnimation
**File**: `src/components/StormAnimation.tsx`
- Marker động cho các vị trí của bão
- Hiệu ứng nhấp nháy/pulsing
- Màu sắc theo cấp độ bão

### 3. Các component lớp thời tiết

#### WeatherOverlay
**File**: `src/components/WeatherOverlay.tsx`
- Lớp phủ heatmap nhiệt độ
- Trực quan hóa tốc độ gió
- Dữ liệu thời gian thực từ OpenWeather API
- Rê chuột để xem giá trị

#### WeatherLayerControl
**File**: `src/components/WeatherLayerControl.tsx`
- Lớp ảnh vệ tinh (satellite)
- Lớp radar
- Lớp nhiệt độ
- Quản lý các tile layer

#### WeatherLayerControlPanel
**File**: `src/components/WeatherLayerControlPanel.tsx`
- Điều khiển UI cho các lớp thời tiết
- Thanh chỉnh độ trong suốt (opacity)
- Nút bật/tắt layer
- Bật/tắt animation cho nhiệt độ

#### OptimizedTemperatureLayer & TemperatureHeatMapLayer
**Files**: `src/components/OptimizedTemperatureLayer.tsx`, `TemperatureHeatMapLayer.tsx`
- Render nhiệt độ tối ưu hiệu năng
- Nội suy màu (color interpolation)
- Heatmap dạng lưới (grid-based)

### 4. Các component về gió

#### WindyLayer
**File**: `src/components/WindyLayer.tsx`
- Tích hợp Windy.com bằng iframe
- Lớp phủ animation gió
- Đồng bộ trạng thái với bản đồ chính

**Context**: `src/contexts/WindyStateContext.tsx`
- State toàn cục cho lớp Windy
- Lưu trạng thái vào URL
- Đồng bộ giữa các component

### 5. Component nâng cấp bản đồ

#### ProvinceLayer
**File**: `src/components/ProvinceLayer.tsx`
- Ranh giới tỉnh/thành Việt Nam
- Render GeoJSON
- Nhãn tên tỉnh/thành

#### WeatherValueTooltip
**File**: `src/components/WeatherValueTooltip.tsx`
- Tooltip hiển thị giá trị thời tiết khi rê chuột
- Nhiệt độ, tốc độ gió, áp suất
- Tooltip định vị theo vị trí con trỏ

### 6. Các component UI

#### ThemeToggle
**File**: `src/components/ThemeToggle.tsx`
- Chuyển chế độ Sáng/Tối
- Lưu lại tùy chọn người dùng
- Tự nhận theme theo hệ thống

#### PreferencesModal
**File**: `src/components/PreferencesModal.tsx`
- Thiết lập tùy chọn người dùng
- Tùy chọn bản đồ
- Tùy chọn hiển thị

#### RightSidebar
**File**: `src/components/RightSidebar.tsx`
- Panel thông tin bổ sung
- Sidebar có thể thu gọn

#### WeeklyForecast
**File**: `src/components/WeeklyForecast.tsx`
- Dự báo thời tiết 7 ngày
- Xu hướng nhiệt độ
- Icon thời tiết

### 7. Các component timeline
**Thư mục**: `src/components/timeline/`
- Điều khiển timeline cho hoạt ảnh bão
- Chức năng Play/Pause
- Kéo để tua thời gian (scrubbing)
- Điều chỉnh tốc độ


## Kiểu dữ liệu

### PredictionResult  
**File**: `src/lib/stormData.ts`
```yaml
export interface PredictionResult {  
  storm_id: string;  
  storm_name: string;  
  prediction_time: string;  
  totalDistance: number;      // km  
  actualDistance: number;     // km  
  lifespan: number;           // giờ  
  forecastHours: number;      // giờ  
  forecast: StormPoint[];     // Các điểm vị trí dự đoán  
  path?: StormPoint[];        // Hỗ trợ tương thích (legacy)  } 

```
### StormPoint
```yaml
export interface StormPoint {  
  timestamp: number;          // Unix timestamp (ms)  
  lat: number;  
  lng: number;  
  windSpeed: number;          // km/h  
  pressure: number;           // hPa  
  category: string;           // "Typhoon", "Super Typhoon", ...  }

```


`md
## Quyền IAM/AWS cần thiết

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
- Origin Access: Public (hoặc OAI nếu dùng)


## Kiểm thử

### Local Development
```bash
npm run dev
# Mở http://localhost:5173
```

### Kiểm thử bản build production
```bash
npm run build
npm run preview
# Mở http://localhost:4173
```

## Các lỗi thường gặp

### 1. Lỗi CORS khi gọi Lambda
Đặc điểm: lỗi `Access-Control-Allow-Origin`

Cách xử lý: Lambda cần trả về header CORS:
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

### 2. CloudFront bị cache cũ
Đặc điểm: Code mới không hiển thị

Cách xử lý: Invalidate cache
```bash
aws cloudfront create-invalidation --distribution-id E... --paths "/*"
```

### 3. Biến môi trường không được load
Triệu chứng: `undefined` khi truy cập `import.meta.env.VITE_*`

Cách xử lý:
- Đảm bảo có file `.env.production`
- Build lại: `npm run build`
- Biến phải bắt đầu bằng `VITE_`


## Checklist triển khai

- [ ] Cập nhật `.env.production` với đúng URL
- [ ] `npm run build` chạy thành công
- [ ] Upload `dist/` lên S3
- [ ] Invalidate CloudFront cache
- [ ] Test trên URL production
- [ ] Kiểm tra Lambda API hoạt động
- [ ] Kiểm tra dữ liệu bão tải được
- [ ] Test form dự đoán với 9+ điểm tọa độ


## API Endpoints

### 1. Lấy dữ liệu bão
```text
GET https://d3lj47ilp0fgxy.cloudfront.net/recent_storms.json
```

Phản hồi: mảng các object Storm

### 2. Dự đoán đường đi bão
```text
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

## Tối ưu hiệu năng

### 1. Tối ưu mã nguồn
- **Code Splitting**: Vite tự động tách chunk theo routes
- **Tree Shaking**: Loại bỏ code không dùng
- **Minification**: Build production tự nén JS/CSS
- **Lazy Loading**: Component được tải khi cần

### 2. Tối ưu dữ liệu
- **Web Workers**: Tính toán nặng chạy trong worker (`dataWorker.ts`)
- **Memoization**: Dùng React.memo cho component tốn tài nguyên
- **Debouncing**: Debounce cho handler input
- **Caching**: Lưu cache tùy chọn bằng LocalStorage

### 3. Tối ưu render
- **Virtual Scrolling**: Danh sách lớn dùng virtual scrolling
- **Optimized Layers**: `OptimizedTemperatureLayer` tối ưu hiệu năng
- **Canvas Rendering**: Heatmap render bằng canvas thay vì DOM
- **Pane Management**: Tạo Leaflet pane riêng để tối ưu z-index

### 4. Tối ưu mạng
- **CDN Caching**: CloudFront cache tài nguyên tĩnh
- **Image Optimization**: WebP, lazy loading
- **API Caching**: Cache dữ liệu bão kèm timestamp
- **Compression**: Nén Gzip/Brotli

### 5. Tối ưu cho khả năng truy cập (Accessibility)
- **Skip Links**: Phím tắt điều hướng bằng bàn phím
- **ARIA Labels**: Dùng nhãn ARIA và HTML ngữ nghĩa đúng
- **Focus Management**: Quản lý focus, đảm bảo thứ tự tab hợp lý
- **Screen Reader**: Tối ưu để hoạt động tốt với trình đọc màn hình


## Thư viện & tiện ích

### Business Logic (lib/)

#### Quản lý bão
- **stormData.ts**: Kiểu dữ liệu, interface, định nghĩa Storm/StormPoint
- **stormAnimations.ts**: Logic animation cho marker bão
- **stormIntensityChanges.ts**: Tính toán thay đổi cường độ bão
- **stormPerformance.ts**: Tối ưu hiệu năng render
- **stormValidation.ts**: Kiểm tra/validate dữ liệu bão

#### Hệ thống gió
- **windData.ts**: Cấu trúc dữ liệu gió
- **windStrengthCalculations.ts**: Tính toán cường độ gió
- **windyStatePersistence.ts**: Lưu trạng thái lớp Windy
- **windyUrlState.ts**: Quản lý trạng thái theo URL cho Windy

#### Bản đồ & thời tiết
- **mapUtils.ts**: Tiện ích bản đồ (center, zoom, tính bounds)
- **openWeatherMapClient.ts**: Client gọi OpenWeather API
- **colorInterpolation.ts**: Tính toán gradient/nội suy màu

#### Hiệu năng
- **dataWorker.ts**: Web Worker cho tác vụ tính toán nặng
- **utils.ts**: Tiện ích dùng chung

### Custom Hooks (hooks/)
- **use-toast.ts**: Hệ thống thông báo toast
- **use-theme.tsx**: Quản lý theme sáng/tối
- **use-mobile.tsx**: Nhận diện thiết bị mobile
- **useTimelineState.ts**: Đồng bộ trạng thái timeline
- **useWindyStateSync.ts**: Đồng bộ trạng thái lớp Windy
- **useSimplifiedTooltip.ts**: Logic tooltip rút gọn

### Context (contexts/)
- **WindyStateContext.tsx**: State toàn cục cho tích hợp lớp Windy

### Testing (test/)
- **accessibility.test.ts**: Kiểm thử accessibility
- **accessibility-audit.test.ts**: Audit theo WCAG
- **wcag-compliance.test.ts**: Kiểm thử tuân thủ WCAG 2.1
- **performance.test.ts**: Benchmark hiệu năng
- **cross-browser.test.ts**: Kiểm thử tương thích đa trình duyệt
- **setup.ts**: Thiết lập môi trường test

### Dependencies

#### Core
- React 18
- TypeScript
- Vite (công cụ build)
- Vitest (framework test)

#### UI Framework
- Tailwind CSS
- shadcn/ui (thư viện component)
- Lucide Icons
- Radix UI (primitives)

#### Map & Visualization
- Leaflet
- React-Leaflet
- Hỗ trợ GeoJSON

#### API & Data
- Fetch API (native)
- OpenWeather API
- AWS Lambda Function URL

#### Quản lý state
- React Context API
- State theo URL (query params)
- Lưu trạng thái bằng LocalStorage

#### Hiệu năng
- Web Workers
- Code splitting (Vite)
- Lazy loading

## Ảnh chụp tài liệu

### CloudFront

#### Phân phối (Distribution)

<p align="center">
  <img src="/images/5-Workshop/5.4-FB_END/5.4.1-FB/image.png" alt="Picture 1" />
  <br/>
  <strong style="font-size: 18px;">Hình 1</strong>
</p>

#### Cài đặt Origin (Origin Settings)

<p align="center">
  <img src="/images/5-Workshop/5.4-FB_END/5.4.1-FB/image-1.png" alt="Picture 1" />
  <br/>
  <strong style="font-size: 18px;">Hình 1</strong>
</p>

#### Invalidations

<p align="center">
  <img src="/images/5-Workshop/5.4-FB_END/5.4.1-FB/image-2.png" alt="Picture 2" />
  <br/>
  <strong style="font-size: 18px;">Hình 2</strong>
</p>

#### storm-frontend-hosting-duc-2025

<p align="center">
  <img src="/images/5-Workshop/5.4-FB_END/5.4.1-FB/image-3.png" alt="Picture 3" />
  <br/>
  <strong style="font-size: 18px;">Hình 3</strong>
</p>

#### Phân quyền (Permissions)

<p align="center">
  <img src="/images/5-Workshop/5.4-FB_END/5.4.1-FB/image-4.png" alt="Picture 4" />
  <br/>
  <strong style="font-size: 18px;">Hình 4</strong>
</p>

#### storm-ai-models-2025

<p align="center">
  <img src="/images/5-Workshop/5.4-FB_END/5.4.1-FB/image-5.png" alt="Picture 5" />
  <br/>
  <strong style="font-size: 18px;">Hình 5</strong>
</p>

#### storm-data-store-2025

<p align="center">
  <img src="/images/5-Workshop/5.4-FB_END/5.4.1-FB/image-6.png" alt="Picture 6" />
  <br/>
  <strong style="font-size: 18px;">Hình 6</strong>
</p>

#### Trang chính (Main Page)

<p align="center">
  <img src="/images/5-Workshop/5.4-FB_END/5.4.1-FB/image-7.png" alt="Picture 7" />
  <br/>
  <strong style="font-size: 18px;">Hình 7</strong>
</p>

#### Tính năng theo dõi bão (Storm Tracking Features)

<p align="center">
  <img src="/images/5-Workshop/5.4-FB_END/5.4.1-FB/image-8.png" alt="Picture 8" />
  <br/>
  <strong style="font-size: 18px;">Hình 8</strong>
</p>

#### Chi tiết cơn bão (Storm Details)

<p align="center">
  <img src="/images/5-Workshop/5.4-FB_END/5.4.1-FB/image-9.png" alt="Picture 9" />
  <br/>
  <strong style="font-size: 18px;">Hình 9</strong>
</p>

#### Tính năng dự đoán (Predict Feature)

<p align="center">
  <img src="/images/5-Workshop/5.4-FB_END/5.4.1-FB/image-10.png" alt="Picture 10" />
  <br/>
  <strong style="font-size: 18px;">Hình 10</strong>
</p>

<p align="center">
  <img src="/images/5-Workshop/5.4-FB_END/5.4.1-FB/image-11.png" alt="Picture 11" />
  <br/>
  <strong style="font-size: 18px;">Hình 11</strong>
</p>

<p align="center">
  <img src="/images/5-Workshop/5.4-FB_END/5.4.1-FB/image-12.png" alt="Picture 12" />
  <br/>
  <strong style="font-size: 18px;">Hình 12</strong>
</p>

