---
title : "Nền tảng API"
date: 2025-09-09
weight : 5
chapter : false
pre : " <b> 5.5. </b> "
---

# MÔ TẢ CHI TIẾT BACK-END API

## Mục lục

1. [Giới thiệu](#1-introduction)
2. [Kiến trúc hệ thống](#2-system-architecture)
3. [Tính năng cốt lõi](#3-core-features)
4. [Công nghệ sử dụng](#4-technology-stack)
5. [Cấu trúc dự án](#5-project-structure)
6. [API Endpoints](#6-api-endpoints)


## 1. Giới thiệu <a id="1-introduction"></a>

**Weather Backend API** là một dịch vụ RESTful cung cấp thông tin thời tiết bằng cách tích hợp với OpenWeatherMap API. Backend hoạt động như lớp trung gian giữa ứng dụng frontend và các nguồn dữ liệu thời tiết bên ngoài.

<p align="center">
  <img src="/images/5-Workshop/5.5-API/photo1.png" alt="Picture 1" />
  <br/>
  <strong style="font-size: 18px;">Hình 1</strong>
</p>

## 2. Kiến trúc hệ thống <a id="2-system-architecture"></a>

### Kiến trúc 

```
┌─────────────────────────────────────────────────────────────┐
│                     Ứng dụng Frontend                       │
│                (React, Mobile, Web Clients)                 │
└─────────────────────────────────────────────────────────────┘
                           │
                           │  HTTPS / REST API
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                    Weather Backend API                      │
│                  (.NET 9.0 - ASP.NET Core)                  │
├─────────────────────────────────────────────────────────────┤
│  ┌────────────────┐   ┌────────────────┐   ┌────────────────┐
│  │  Bộ điều khiển │   │    Dịch vụ     │   │   Program.cs   │
│  │                │   │                │   │ - Khởi động app│
│  │ - WeatherCtrl  │   │ - WeatherSvc   │   │ - Ghi log      │
│  │ - ForecastCtrl │   │ - Lớp cache    │   │ - Thiết lập DI │
│  └────────────────┘   └────────────────┘   └────────────────┘
└─────────────────────────────────────────────────────────────┘
                           │
                           │  HTTPS / REST API (Bên ngoài)
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                Dịch vụ thời tiết bên ngoài                  │
├─────────────────────────────────────────────────────────────┤
│ • OpenWeatherMap API                                        │
│ • Redis Caching Layer                                       │
│ • Giới hạn tốc độ & Giám sát                                │
└─────────────────────────────────────────────────────────────┘
```

## 3. Tính năng cốt lõi <a id="3-core-features"></a>

**Mô tả**: Lấy dữ liệu thời tiết hiện tại của bất kỳ thành phố nào trên thế giới.

**Tính năng**:
- Tìm kiếm theo tên thành phố (ví dụ: "Hà Nội", "TP Hồ Chí Minh")
- Tùy chọn mã quốc gia để xác định vị trí chính xác
- Hỗ trợ nhiều hệ thống đơn vị (metric, imperial, standard)
- Hỗ trợ đa ngôn ngữ cho phần mô tả thời tiết
- Phản hồi được lưu cache để tăng hiệu suất

**Tham số API**:
- `cityName` (bắt buộc): Tên thành phố
- `countryCode` (tùy chọn): Mã quốc gia ISO 3166
- `units` (tùy chọn): metric, imperial, standard
- `language` (tùy chọn): en, vi, fr, ...

**Ví dụ phản hồi**:
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
  <strong style="font-size: 18px;">Hình 2</strong>
</p>

## 4. Công nghệ sử dụng <a id="4-technology-stack"></a>

### Backend Framework

* **.NET 9.0** – Runtime .NET mới nhất  
* **ASP.NET Core** – Framework xây dựng Web API  
* **C# 12** – Ngôn ngữ lập trình chính  

### Tích hợp API

* **HttpClientFactory** – Quản lý sử dụng HTTP client  
* **Polly** – Chính sách thử lại & xử lý lỗi tạm thời
* **Newtonsoft.Json / System.Text.Json** – Tuần tự hóa JSON  

### Cache & Hiệu năng

* **MemoryCache** – Cache trong bộ nhớ  
* **Redis (tùy chọn)** – Cache phân tán  
* **ResponseCompression** – Nén Gzip / Brotli  

### Công cụ phát triển

* **Visual Studio 2022 / VS Code** - IDE / Trình soạn thảo mã 
* **Swagger / OpenAPI** – Tài liệu API  
* **Git** – Kiểm soát phiên bản  
* **Docker** – Container hóa  

## 5. Cấu trúc dự án <a id="5-project-structure"></a>

```text
WeatherBackend/
│
├── WeatherBackend.csproj            # Tập tin dự án
├── Program.cs                        # Điểm vào ứng dụng
├── WeatherBackend.http               # Tập tin kiểm tra yêu cầu HTTP
│
├── appsettings.json                 # Cài đặt cấu hình
│
├── Controllers/                     # Bộ điều khiển API
│   └── WeatherController.cs         # Điểm cuối thời tiết chính
│
├── Services/                        # Dịch vụ logic nghiệp vụ
│   └── WeatherService/              # Hợp đồng & triển khai dịch vụ
```

## 6. API Endpoints <a id="6-api-endpoints"></a>

### Base URL

```
https://localhost:7042/swagger/index.html
```

### 6.1 GET /api/weather/current

**Mô tả:**  
Lấy dữ liệu thời tiết hiện tại theo tên thành phố.

**Ví dụ CURL:**

```bash
curl -X GET \
  "https://localhost:7042/api/Weather?city=hanoi" \
  -H "accept: */*"
```

**URL yêu cầu:**

```
https://localhost:7042/api/Weather?city=hanoi
```

### 6.2 GET /api/weather/forecast

**Mô tả:**
Lấy dự báo thời tiết 5 ngày cho một thành phố được chọn.

**Ví dụ CURL:**

```bash
curl -X GET \
  "https://localhost:7042/api/Weather/forecast?city=hochiminh" \
  -H "accept: */*"
```

**URL yêu cầu:**

```
https://localhost:7042/api/Weather/forecast?city=hochiminh
```

### 6.3 GET /api/weather/coordinates

**Mô tả:**
Lấy thông tin thời tiết bằng vĩ độ và kinh độ.

**Ví dụ CURL:**

```bash
curl -X GET \
  "https://localhost:7042/api/Weather/by-coord?lat=21.0245&lon=105.8412" \
  -H "accept: */*"
```

**URL yêu cầu:**

```
https://localhost:7042/api/Weather/by-coord?lat=21.0245&lon=105.8412
```

### 6.4 GET /api/weather/location

**Mô tả:**
Lấy thông tin thời tiết theo vị trí của người dùng (yêu cầu thiết bị người dùng gửi tọa độ).

**Ví dụ CURL:**
```bash
curl -X GET \
  "https://localhost:7042/api/Weather/global" \
  -H "accept: */*"
```

**URL yêu cầu:**
```
https://localhost:7042/api/Weather/global
```

<p align="center">
  <img src="/images/5-Workshop/5.5-API/photo3.png" alt="Picture 3" />
  <br/>
  <strong style="font-size: 18px;">Hình 3</strong>
</p>

---

**Cập nhập lần cuối**: 2025-12-09  
**Phiên bản**: 1.0.0  
**Bảo trì bởi**: SKYNET
