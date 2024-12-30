# API gateway với Ocelot


API gateway là một dịch vụ trung gian giữa client và các dịch vụ backend. API gateway có thể thực hiện các chức năng như:
- Load balancing
- Caching
- Authentication
- Authorization
- Request shaping
- Rate limiting
- Logging
- Monitoring
- Security
- Static response handling
- Request and response modification


## Cấu hình Ocelot
- cần dựng sẵn 2 api cần public ra ngoài. Ví dụ
    - Gateway 192.168.1.18:8081
    - Core-api 192.168.1.18:8082
```
{
  "GlobalConfiguration": {
    "RateLimitOptions": {
      "DisableRateLimitHeaders": false,
      "QuotaExceededMessage": "Bạn đã vượt quá số lượng request, vui lòng thử lại sau!",
      "HttpStatusCode": 418,
      "ClientIdHeader": "MyRateLimiting"
    }
  },
  "Routes": [
    {
      "SwaggerKey": "gateway",
      "UpstreamPathTemplate": "/gateway/{everything}",
      "UpstreamHttpMethod": [ "Get", "Post", "Delete", "Put" ],
      "DownstreamPathTemplate": "/{everything}/",
      "DownstreamScheme": "http",
      "DownstreamHostAndPorts": [
        {
          "Host": "192.168.1.18",
          "Port": 8081
        }
      ],
      "RateLimitOptions": {
        "DisableRateLimitHeaders": false, // Hiển thị thông tin giới hạn trong header (RateLimit-Remaining)
        "ClientWhitelist": [], // array of strings
        "EnableRateLimiting": true,
        "Period": "30s", // seconds, minutes, hours, days. Trong 30s giới hạn 5 request. và sau PeriodTimespan 10s sẽ reset số này
        "PeriodTimespan": 10, // only seconds
        "Limit": 5
      }
    },
    {
      "SwaggerKey": "core-api",
      "UpstreamPathTemplate": "/core-api/{everything}",
      "UpstreamHttpMethod": [ "Get", "Post", "Delete", "Put" ],
      "DownstreamPathTemplate": "/{everything}/",
      "DownstreamScheme": "http",
      "DownstreamHostAndPorts": [
        {
          "Host": "192.168.1.18",
          "Port": 8082
        }
      ],
      "RateLimitOptions": {
        "DisableRateLimitHeaders": false, // Hiển thị thông tin giới hạn trong header (RateLimit-Remaining)
        "ClientWhitelist": [], // array of strings
        "EnableRateLimiting": true,
        "Period": "30s", // seconds, minutes, hours, days. Trong 30s giới hạn 5 request. và sau PeriodTimespan 10s sẽ reset số này
        "PeriodTimespan": 10, // only seconds
        "Limit": 5
      }
    }
  ],
  "SwaggerEndPoints": [
    {
      "Key": "gateway",
      "TransformByOcelotConfig": "false",
      "Config": [
        {
          "Name": "Gateway-swagger",
          "Version": "v1",
          "Url": "http://192.168.1.18:8081/swagger/v1/swagger.json"
        }
      ]
    },
    {
      "Key": "core-api",
      "TransformByOcelotConfig": "false",
      "Config": [
        {
          "Name": "Core-api-swagger",
          "Version": "v1",
          "Url": "http://192.168.1.18:8082/../swagger/v1/swagger.json"
        }
      ]
    }
  ]
}
```

## Cấu hình Swagger
```
{
  "SwaggerEndPoints": [
    {
      "Key": "gateway",
      "TransformByOcelotConfig": "false",
      "Config": [
        {
          "Name": "Gateway-swagger",
          "Version": "v1",
          "Url": "http://
          }
        ]
    }
    ]
}
```

## Cấu hình Program.cs
```
phần builder:
builder.Services.AddOcelot(builder.Configuration);
builder.Services.AddSwaggerForOcelot(builder.Configuration);
builder.Services.AddSwaggerGen();

phần app:
    app.UseSwaggerForOcelotUI();
    await app.UseOcelot();
```
## Test OcelotApiGateway với PowerShell
``
for ($i=1; $i -le 10; $i++) {
    curl -Uri http://localhost:5266/gateway/api/ServiceLIS/GetGroup
}
``
![alt text](https://github.com/haph762it/Ocelot-apigateway/blob/main/wwwroot/test-api-gateway.png?raw=true)
