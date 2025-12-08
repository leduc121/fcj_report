---
title : "AI-Model-Integration"
date: 2025-09-09
weight : 4 
chapter : false
pre : " <b> 5.4.2.1 </b> "
---


# Actions to Integrate Hurricane Prediction Model from Lambda

## Overview

Below here we will represent how did we integrate our model from lambda to use step-by-step.

## Files Currently Using Mock Data

### 1. WeatherOverlay.tsx (IMPORTANT)
- Location: `frontend/src/components/WeatherOverlay.tsx`
- Mock data: Temperature and Wind overlay data
- Function: `generateWeatherData()`
- Required change: Replace with an API call to Lambda

### 2. WeeklyForecast.tsx
- Location: `frontend/src/components/WeeklyForecast.tsx`
- Mock data: `mockForecast` array
- Required change: Fetch from the backend API

### 3. windData.ts
- Location: `frontend/src/lib/windData.ts`
- Mock data: `mockWindData`
- Required change: Fetch from OpenWeatherMap or Lambda

### 4. WindFieldManager.ts
- Location: `frontend/src/components/wind/WindFieldManager.ts`
- Mock data: Fallback when there is no API key
- Status: OK — it already fetches from OpenWeatherMap; you just need to configure the API key

## How to Integrate the AI Model from Lambda

### Step 1: Add API endpoints for Storm Prediction

In `frontend/src/api/weatherApi.ts`, add:

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
    confidence?: number; // Confidence score from the AI model
  }>;
}

export const weatherApi = {
  // ... existing methods ...

  // Get storm predictions from the Lambda AI model
  getStormPredictions: async (): Promise<StormPrediction[]> => {
    const response = await api.get<StormPrediction[]>('/storms/predictions');
    return response.data;
  },

  // Get details for a specific storm
  getStormById: async (stormId: string): Promise<StormPrediction> => {
    const response = await api.get<StormPrediction>(`/storms/${stormId}`);
    return response.data;
  },
};
```

### Step 2: Update the Backend to call Lambda

In the C# backend (`backend/Controllers/WeatherController.cs`), add an endpoint:

```csharp
[HttpGet("storms/predictions")]
public async Task<IActionResult> GetStormPredictions()
{
    try
    {
        // Call the Lambda function
        var lambdaClient = new AmazonLambdaClient();
        var request = new InvokeRequest
        {
            FunctionName = "storm-prediction-function",
            InvocationType = InvocationType.RequestResponse,
            Payload = "{}" // Or parameters if needed
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

### Step 3: Update the Frontend to use the real API

In `frontend/src/pages/Index.tsx` (or wherever storm data is fetched):

```typescript
import { weatherApi } from '../api/weatherApi';
import { useQuery } from '@tanstack/react-query';

// Instead of using mock data
const { data: storms, isLoading } = useQuery({
  queryKey: ['storms'],
  queryFn: () => weatherApi.getStormPredictions(),
  refetchInterval: 5 * 60 * 1000, // Refresh every 5 minutes
});
```

### Step 4: Configure Environment Variables

Frontend (`.env.production`):
```env
VITE_API_BASE_URL=https://your-backend-api.com/api/weather
```

Backend (`appsettings.json`):
```json
{
  "AWS": {
    "Region": "ap-southeast-1",
    "LambdaFunctionName": "storm-prediction-function"
  }
}
```

## Deployment Checklist

- [ ] Deploy the AI model to Lambda
- [ ] Test the Lambda function with sample input
- [ ] Add the API endpoint in the C# backend
- [ ] Test the backend endpoint
- [ ] Update `weatherApi.ts` with the new endpoints
- [ ] Replace mock data with real API calls
- [ ] Test the frontend with real data
- [ ] Update `.env.production` with the production URL
- [ ] Build and deploy the frontend
- [ ] Monitor logs and errors

## Notes

1. Caching: You should cache results from Lambda to reduce cost
2. Error handling: Handle Lambda timeouts or errors gracefully
3. Loading states: Show loading indicators while fetching
4. Fallback: You can keep mock data as a fallback when the API fails

## Files You Don’t Need to Change (Examples Only)

These files are demos/examples and do not affect production:
- `*.example.tsx`
- `*/__tests__/*`
- `*/GUIDE.md`
