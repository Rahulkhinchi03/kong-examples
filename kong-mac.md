<div align="center">
<img src="https://github.com/user-attachments/assets/54f0c084-65bb-4431-b80d-cceab6c63dc3"/>
</div>

<div align="center">

# Treblle
<a href="http://treblle.com/" target="_blank">Website</a>
<span>&nbsp;&nbsp;•&nbsp;&nbsp;</span>
<a href="https://docs.treblle.com" target="_blank">Documentation</a>
<span>&nbsp;&nbsp;•&nbsp;&nbsp;</span>
<a href="https://docs.treblle.com/en/integrations" target="_blank">Integrations</a>

  <hr />
</div>

## API Intelligence Platform

Treblle is a federated API intelligence platform that helps organization understand their entire API Landscape in less than 60 seconds.

<div align="center">
<a href="https://treblle.com/product/api-observability" target="_blank">API Intelligence</a>
<span>&nbsp;&nbsp;•&nbsp;&nbsp;</span>
<a href="https://treblle.com/product/api-documentation" target="_blank">API Documentation</a>
<span>&nbsp;&nbsp;•&nbsp;&nbsp;</span>
<a href="https://treblle.com/product/api-analytics" target="_blank">API Analytics</a>
<span>&nbsp;&nbsp;•&nbsp;&nbsp;</span>
<a href="https://treblle.com/product/api-governance" target="_blank">API Governance</a>
<span>&nbsp;&nbsp;•&nbsp;&nbsp;</span>
<a href="https://treblle.com/product/api-security" target="_blank">API Security</a>
</div>

<div align="center">
  <br />
  <img src="https://github.com/user-attachments/assets/9b5f40ba-bec9-414b-af88-f1c1cc80781b"/>
  <br />
</div>


# Kong Gateway Plugin for Treblle

The Kong API Gateway plugin for Treblle captures API requests in real time and sends that data to Treblle for monitoring and analysis.

### What Treblle Helps You With:

- Understand who your API consumers are, how they're using the API, and when
- Stay secure and compliant at design and run-time
- Automate API governance checks across security, performance, and design
- Debug APIs in real-time with access to request/response payloads
- Generate and update your API documentation in OpenAPI Spec format
- Build your API developer portal with an AI-powered integration assistant
- Test your APIs in a fast and easy way
- And much more

### Terminology
- `plugin`: a plugin executing actions inside Kong before or after a request has been proxied to the upstream API.
- `Service`: the Kong entity representing an external upstream API or microservice.
- `Route`: The Kong entity represents a way to map downstream requests to upstream services.
- `Consumer`: the Kong entity representing a developer or machine using the API. When using Kong, a Consumer only communicates with Kong, which proxies every call to the upstream API.
- `Credential`: a unique string associated with a Consumer, also called an API key.
- `Upstream service`: refers to your own API/service sitting behind Kong, to which client requests are forwarded.
- `API`: a legacy entity representing your upstream services. Deprecated in favor of Services since CE 0.13.0 and EE 0.32.

# Installation on macOS Using Docker

### Prerequisites
- Docker Desktop installed on your Mac
- Git installed
- A Treblle account with an API key and Project ID

### Step 1: Clone the Repository
```bash
git clone https://github.com/Treblle/treblle-kong.git
cd treblle-kong
```

### Step 2: Build the Docker Image

We'll use the [Dockerfile](https://github.com/Treblle/treblle-kong/blob/main/Dockerfile) in the plugin folder to build a custom image of Kong with the Treblle plugin enabled:

```bash
docker build -t k:v1 .
```

This creates a custom Kong image tagged as "k" with the installed Treblle plugin.

### Step 3: Start the Container

Run Docker Compose to start the Kong container with the plugin:

```bash
docker-compose up -d
```
After this step, your Kong Gateway should be up and running with the Treblle plugin enabled.

## Configuration Using Curl

### 1. Verify Plugin is Enabled

Read the [Plugin Reference](https://docs.konghq.com/gateway-oss/2.4.x/admin-api/#add-plugin) and the [Plugin Precedence](https://docs.konghq.com/gateway-oss/2.4.x/admin-api/#precedence) sections for more information.

```bash
curl -i -X GET http://localhost:8001/plugins/enabled
```

### 2. Create a Service

Configure this plugin on a [Service](https://docs.konghq.com/gateway-oss/2.4.x/admin-api/#service-object) by making the following request on your Kong server:

```bash
curl -i -X POST http://localhost:8001/services \
  --data "name=httpbin-service" \
  --data "url=https://httpbin.org"
```

### 3. Create a Route

Configure this plugin on a [Route](https://docs.konghq.com/gateway-oss/2.4.x/admin-api/#route-object) with:

```bash
curl -i -X POST http://localhost:8001/services/httpbin-service/routes \
  --data "name=httpbin-route-post" \
  --data "paths[]=/httpbin" \
  --data "methods[]=POST"
```

### 4. Create an API on Treblle and Get Credentials

First, you need to create an API on the Treblle platform to get your API key and Project ID, which will be used to configure the plugin in the next step.

- Visit the [Treblle](treblle.com) Dashboard to create a new API
- Note your `TREBLLE_API_KEY` and `TREBLLE_PROJECT_ID`

### 5. Add Treblle Plugin to Service

The `mask_keywords` are the sensitive data fields that will be masked before being sent to Treblle. This important security feature protects sensitive information in your API traffic.

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

At this point, you should be able to go to your Treblle dashboard and see the API request that was just made, complete with all the details captured by the plugin.

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
| config.event_queue_size | 100000 | Maximum number of events to hold in the queue before sending to Treblle |
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

## Community

First and foremost, **Star and watch this repository** to stay up-to-date.

Also, follow our [Blog](https://blog.treblle.com), and on [Twitter](https://twitter.com/treblleapi).

You can chat with the team and other members on [Discord](https://treblle.com/chat) and follow our tutorials and other video material at [YouTube](https://youtube.com/@treblle).

[![Treblle Discord](https://img.shields.io/badge/Treblle%20Discord-Join%20our%20Discord-F3F5FC?labelColor=7289DA&style=for-the-badge&logo=discord&logoColor=F3F5FC&link=https://treblle.com/chat)](https://treblle.com/chat)

[![Treblle YouTube](https://img.shields.io/badge/Treblle%20YouTube-Subscribe%20on%20YouTube-F3F5FC?labelColor=c4302b&style=for-the-badge&logo=YouTube&logoColor=F3F5FC&link=https://youtube.com/@treblle)](https://youtube.com/@treblle)

[![Treblle on Twitter](https://img.shields.io/badge/Treblle%20on%20Twitter-Follow%20Us-F3F5FC?labelColor=1DA1F2&style=for-the-badge&logo=Twitter&logoColor=F3F5FC&link=https://twitter.com/treblleapi)](https://twitter.com/treblleapi)
