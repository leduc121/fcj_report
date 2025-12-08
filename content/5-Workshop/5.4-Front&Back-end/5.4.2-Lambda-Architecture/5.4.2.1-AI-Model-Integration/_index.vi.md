---
title : "Tích Hợp AI"
date: 2025-09-09
weight : 4 
chapter : false
pre : " <b> 5.4.2.1 </b> "
---


# Tiến hành tích hợp AI Model từ Lambda

## Tổng quan

Dưới đây xin trình bày về cách mà nhóm đã tích hợp mô hình AI dự đoán bão từ lambda để dùng theo từng bước.

## Các file đang dùng Mock Data

### 1. **WeatherOverlay.tsx** (QUAN TRỌNG)
- **Vị trí**: `frontend/src/components/WeatherOverlay.tsx`
- **Mock data**: Temperature và Wind overlay data
- **Function**: `generateWeatherData()`
- **Cần sửa**: Thay thế bằng API call đến Lambda

### 2. **WeeklyForecast.tsx**
- **Vị trí**: `frontend/src/components/WeeklyForecast.tsx`
- **Mock data**: `mockForecast` array
- **Cần sửa**: Fetch từ backend API

### 3. **windData.ts**
- **Vị trí**: `frontend/src/lib/windData.ts`
- **Mock data**: `mockWindData`
- **Cần sửa**: Fetch từ OpenWeatherMap hoặc Lambda

### 4. **WindFieldManager.ts**
- **Vị trí**: `frontend/src/components/wind/WindFieldManager.ts`
- **Mock data**: Fallback khi không có API key
- **Đã OK**: Có logic fetch từ OpenWeatherMap, chỉ cần config API key

## Cách tích hợp AI Model từ Lambda

### Bước 1: Thêm API endpoint cho Storm Prediction

Trong file `frontend/src/api/weatherApi.ts`, thêm:

```typescript
export interface StormPrediction {
  stormId: string;
  name: string;
  nameVi: string;
  currentPosition: {
    lat: number;
    lng: number;
    timestamp: number;
    windSpeed: number;
    pressure: number;
    category: string;
  };
  historicalTrack: Array<{
    lat: number;
    lng: number;
    timestamp: number;
    windSpeed: number;
    pressure: number;
    category: string;
  }>;
  forecastTrack: Array<{
    lat: number;
    lng: number;
    timestamp: number;
    windSpeed: number;
    pressure: number;
    category: string;
    confidence?: number; // Độ tin cậy từ AI model
  }>;
}

export const weatherApi = {
  // ... existing methods ...

  // Lấy dự đoán bão từ Lambda AI model
  getStormPredictions: async (): Promise<StormPrediction[]> => {
    const response = await api.get<StormPrediction[]>('/storms/predictions');
    return response.data;
  },

  // Lấy chi tiết một cơn bão cụ thể
  getStormById: async (stormId: string): Promise<StormPrediction> => {
    const response = await api.get<StormPrediction>(`/storms/${stormId}`);
    return response.data;
  },
};
```

### Bước 2: Cập nhật Backend để gọi Lambda

Trong backend C# (`backend/Controllers/WeatherController.cs`), thêm endpoint:

```csharp
[HttpGet("storms/predictions")]
public async Task<IActionResult> GetStormPredictions()
{
    try
    {
        // Gọi Lambda function
        var lambdaClient = new AmazonLambdaClient();
        var request = new InvokeRequest
        {
            FunctionName = "storm-prediction-function",
            InvocationType = InvocationType.RequestResponse,
            Payload = "{}" // Hoặc parameters nếu cần
        };

        var response = await lambdaClient.InvokeAsync(request);
        
        using var reader = new StreamReader(response.Payload);
        var result = await reader.ReadToEndAsync();
        
        return Ok(JsonSerializer.Deserialize<List<StormPrediction>>(result));
    }
    catch (Exception ex)
    {
        return StatusCode(500, new { error = ex.Message });
    }
}
```

### Bước 3: Cập nhật Frontend để dùng API thật

Trong `frontend/src/pages/Index.tsx` hoặc nơi fetch storm data:

```typescript
import { weatherApi } from '../api/weatherApi';
import { useQuery } from '@tanstack/react-query';

// Thay vì dùng mock data
const { data: storms, isLoading } = useQuery({
  queryKey: ['storms'],
  queryFn: () => weatherApi.getStormPredictions(),
  refetchInterval: 5 * 60 * 1000, // Refresh mỗi 5 phút
});
```

### Bước 4: Cấu hình Environment Variables

**Frontend** (`.env.production`):
```env
VITE_API_BASE_URL=https://your-backend-api.com/api/weather
```

**Backend** (`appsettings.json`):
```json
{
  "AWS": {
    "Region": "ap-southeast-1",
    "LambdaFunctionName": "storm-prediction-function"
  }
}
```

## Checklist Deploy

- [ ] Deploy AI model lên Lambda
- [ ] Test Lambda function với sample input
- [ ] Thêm API endpoint trong backend C#
- [ ] Test backend endpoint
- [ ] Cập nhật `weatherApi.ts` với endpoints mới
- [ ] Thay thế mock data bằng API calls
- [ ] Test frontend với data thật
- [ ] Cập nhật `.env.production` với URL production
- [ ] Build và deploy frontend
- [ ] Monitor logs và errors

## Files không cần sửa (chỉ là examples)

Các file này chỉ là demo/example, không ảnh hưởng production:
- `*.example.tsx`
- `*/__tests__/*`
- `*/GUIDE.md`
