---
title : "Platform API"
date: 2025-09-09
weight : 5
chapter : false
pre : " <b> 5.5. </b> "
---

# BACK-END API DETAIL DESCRIPTION

## Table of Contents

1. [Introduction](#1-introduction)
2. [System Architecture](#2-system-architecture)
3. [Core Features](#3-core-features)
4. [Technology Stack](#4-technology-stack)
5. [Project Structure](#5-project-structure)
6. [API Endpoints](#6-api-endpoints)


## 1. Introduction <a id="1-introduction"></a>

**Weather Backend API** is a RESTful service that provides weather information by integrating with OpenWeatherMap API. The backend serves as a middleware layer between frontend applications and external weather data sources.

<p align="center">
  <img src="/images/5-Workshop/5.5-API/photo1.png" alt="Picture 1" />
  <br/>
  <strong style="font-size: 18px;">Figure 1</strong>
</p>

## 2. System Architecture <a id="2-system-architecture"></a>

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      Frontend Applications                  │
│                 (React, Mobile, Web Clients)                │
└─────────────────────────────────────────────────────────────┘
                           │
                           │  HTTPS / REST API
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                     Weather Backend API                     │
│                  (.NET 9.0 - ASP.NET Core)                  │
├─────────────────────────────────────────────────────────────┤
│  ┌────────────────┐   ┌────────────────┐   ┌────────────────┐
│  │  Controllers   │   │    Services    │   │   Program.cs   │
│  │                │   │                │   │ - App Startup  │
│  │ - WeatherCtrl  │   │ - WeatherSvc   │   │ - Logging      │
│  │ - ForecastCtrl │   │ - Cache Layer  │   │ - DI Setup     │
│  └────────────────┘   └────────────────┘   └────────────────┘
└─────────────────────────────────────────────────────────────┘
                           │
                           │  HTTPS / REST API (External)
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                   External Weather Services                 │
├─────────────────────────────────────────────────────────────┤
│ • OpenWeatherMap API                                        │
│ • Redis Caching Layer                                       │
│ • Rate Limiting & Monitoring                                │
└─────────────────────────────────────────────────────────────┘
```
## 3. Core Features <a id="3-core-features"></a>

**Description**: Retrieve current weather conditions for any city worldwide.

**Features**:
- Search by city name (e.g., "Hanoi", "Ho Chi Minh City")
- Optional country code for precise location
- Multiple unit systems support (metric, imperial, standard)
- Multi-language weather descriptions
- Cached responses for performance

**API Parameters**:
- `cityName` (required): Name of the city
- `countryCode` (optional): ISO 3166 country code
- `units` (optional): metric, imperial, or standard
- `language` (optional): en, vi, fr, etc.

**Response Example**:
```json

	
Response body
Download
{
  "localDate": "2025-12-06 19:57:02",
  "city": "Hà Nội",
  "coord": {
    "lon": 105.8412,
    "lat": 21.0245
  },
  "weather": [
    {
      "id": 804,
      "main": "Clouds",
      "description": "mây đen u ám",
      "icon": "04n"
    }
  ],
  "main": {
    "temp": 22,
    "feels_like": 22.11,
    "temp_min": 22,
    "temp_max": 22,
    "pressure": 1018,
    "humidity": 71,
    "sea_level": 1018,
    "grnd_level": 1017
  },
  "wind": {
    "speed": 4.14,
    "deg": 136,
    "gust": 6.84
  },
  "sys": {
    "type": 1,
    "id": 9308,
    "country": "VN",
    "sunrise": 1764976827,
    "sunset": 1765016103
  }
}
```

<p align="center">
  <img src="/images/5-Workshop/5.5-API/photo2.png" alt="Picture 2" />
  <br/>
  <strong style="font-size: 18px;">Figure 2</strong>
</p>

## 4. Technology Stack <a id="4-technology-stack"></a>

### Backend Framework

* **.NET 9.0** – Latest .NET runtime
* **ASP.NET Core** – Web API framework
* **C# 12** – Primary programming language

### API Integration

* **HttpClientFactory** – Managed HTTP client usage
* **Polly** – Retry policies & transient fault handling
* **Newtonsoft.Json / System.Text.Json** – JSON serialization

### Caching & Performance

* **MemoryCache** – In-memory caching
* **Redis (optional)** – Distributed caching
* **ResponseCompression** – Gzip / Brotli compression

### Development Tools

* **Visual Studio 2022 / VS Code** – IDE / Code editor
* **Swagger / OpenAPI** – API documentation
* **Git** – Version control
* **Docker** – Containerization

## 5. Project Structure <a id="5-project-structure"></a>

```text
WeatherBackend/
│
├── WeatherBackend.csproj            # Project file
├── Program.cs                        # Application entry point
├── WeatherBackend.http               # HTTP request testing file
│
├── appsettings.json                 # Configuration settings
│
├── Controllers/                     # API Controllers
│   └── WeatherController.cs         # Main weather endpoints
│
├── Services/                        # Business logic services
│   └── WeatherService/              # Service contracts & implementation
```

## 6. API Endpoints <a id="6-api-endpoints"></a>

### Base URL

```
https://localhost:7042/swagger/index.html
```

### 6.1 GET /api/weather/current

**Description:**
Retrieve the current weather by city name.

**CURL Example:**

```bash
curl -X GET \
  "https://localhost:7042/api/Weather?city=hanoi" \
  -H "accept: */*"
```

**Request URL:**

```
https://localhost:7042/api/Weather?city=hanoi
```

### 6.2 GET /api/weather/forecast

**Description:**
Get the 5-day weather forecast for a selected city.

**CURL Example:**

```bash
curl -X GET \
  "https://localhost:7042/api/Weather/forecast?city=hochiminh" \
  -H "accept: */*"
```

**Request URL:**

```
https://localhost:7042/api/Weather/forecast?city=hochiminh
```

### 6.3 GET /api/weather/coordinates

**Description:**
Retrieve weather data using latitude and longitude.

**CURL Example:**

```bash
curl -X GET \
  "https://localhost:7042/api/Weather/by-coord?lat=21.0245&lon=105.8412" \
  -H "accept: */*"
```

**Request URL:**

```
https://localhost:7042/api/Weather/by-coord?lat=21.0245&lon=105.8412
```

### 6.4 GET /api/weather/location

**Description:**
Retrieve weather data for the user's current location (requires coordinates from client device).

**CURL Example:**
```bash
curl -X GET \
  "https://localhost:7042/api/Weather/global" \
  -H "accept: */*"
```

**Request URL:**
```
https://localhost:7042/api/Weather/global
```

<p align="center">
  <img src="/images/5-Workshop/5.5-API/photo3.png" alt="Picture 3" />
  <br/>
  <strong style="font-size: 18px;">Figure 3</strong>
</p>

---

**Last Updated**: 2025-12-09  
**Version**: 1.0.0  
**Maintained by**: SKYNET
