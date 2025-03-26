# Kong Gateway Plugin for Treblle - macOS Installation Guide

The Kong API Gateway plugin for Treblle captures API requests in real-time and sends that data to Treblle for monitoring and analysis.

### What Treblle Helps You With:

- Understand who your API consumers are, how they're using the API, and when
- Stay secure and compliant at design and run-time
- Automate API governance checks across security, performance, and design
- Debug APIs in real-time with access to request/response payloads
- Generate and update your API documentation in OpenAPI Spec format
- Build your API developer portal with an AI-powered integration assistant
- Test your APIs in a fast and easy way
- And much more

## Installation on macOS Using Docker

### Prerequisites
- Docker Desktop installed on your Mac
- Git installed
- A Treblle account with API key and Project ID

### Step 1: Clone the Repository
```bash
git clone https://github.com/Treblle/treblle-kong.git
cd treblle-kong
```

### Step 2: Build the Docker Image
```bash
docker build -t k:v1 .
```

### Step 3: Start the Container
```bash
docker-compose up -d
```

## Configuration Using Curl

### 1. Verify Plugin is Enabled
```bash
curl -i -X GET http://localhost:8001/plugins/enabled
```

### 2. Create a Service
```bash
curl -i -X POST http://localhost:8001/services \
  --data "name=httpbin-service" \
  --data "url=https://httpbin.org"
```

### 3. Create a Route
```bash
curl -i -X POST http://localhost:8001/services/httpbin-service/routes \
  --data "name=httpbin-route-post" \
  --data "paths[]=/httpbin" \
  --data "methods[]=POST"
```

### 4. Create an API on Treblle and Get Credentials
- Visit the [Treblle](treblle.com) Dashboard to create a new API
- Note your `TREBLLE_API_KEY` and `TREBLLE_PROJECT_ID`

### 5. Add Treblle Plugin to Service
```bash
curl -i -X POST http://localhost:8001/services/httpbin-service/plugins \
  --data "name=treblle" \
  --data "config.api_key=YOUR_TREBLLE_API_KEY" \
  --data "config.project_id=YOUR_TREBLLE_PROJECT_ID" \
  --data "config.mask_keywords[]=Authorization" \
  --data "config.mask_keywords[]=User-Agent" \
  --data "config.mask_keywords[]=Cookie"
```

### 6. Test Publishing to Treblle
```bash
curl -i -X POST http://localhost:8000/httpbin/post \
  -H "Content-Type: application/json" \
  -d '{"test": "v1final3d", "foo": "bar"}'
```

## Configuration Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| name | | The name of the plugin, in this case `treblle` |
| config.api_key | | The Treblle SDK token provided by Treblle |
| config.project_id | | The Treblle API ID provided by Treblle |
| config.connect_timeout | 1000 | Timeout in milliseconds when connecting to Treblle |
| config.send_timeout | 5000 | Timeout in milliseconds when sending data to Treblle |
| config.keepalive | 5000 | Value in milliseconds that defines how long an idle connection will live before being closed |
| config.max_callback_time_spent | 750 | Limiter on how much time to send events to Treblle per worker cycle |
| config.request_max_body_size_limit | 100000 | Maximum request body size in bytes to log |
| config.response_max_body_size_limit | 100000 | Maximum response body size in bytes to log |
| config.event_queue_size | 100000 | Maximum number of events to hold in queue before sending to Treblle |
| config.debug | false | If set to true, prints internal log messages for debugging integration issues |
| config.enable_compression | false | If set to true, requests are compressed before sending to Treblle |
| config.max_retry_count | 1 | Retry count to send the payload to the Treblle API |
| config.retry_interval | 5 | Retry interval between retries in seconds |
| config.mask_keywords | | Masking keywords to be used for the entire payload |

## Troubleshooting

### Enable Debug Logs
If you want to print Treblle debug logs, set `config.debug=true` when enabling the plugin:

```bash
curl -i -X POST http://localhost:8001/services/httpbin-service/plugins \
  --data "name=treblle" \
  --data "config.api_key=YOUR_TREBLLE_API_KEY" \
  --data "config.project_id=YOUR_TREBLLE_PROJECT_ID" \
  --data "config.debug=true"
```

**Note**: If you already have Treblle installed, you must update the configuration of the existing instance rather than installing Treblle twice.
