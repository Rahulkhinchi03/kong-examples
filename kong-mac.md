# Installation on macOS Using Docker

Request Example of [Treblle](https://platform.treblle.com/r?expires=1745677903&signature=dd2cb072a79a97f8e9d97905f8dbb209e5995e44735037e9adcf91c582d654a2&workspaceId=01jq942yxvb13hxme8ymd4k0f2&apiId=01jq947q4ge80jy3fqzg4yfpbh&requestId=01jq94j5y54xtckema3ek6zej6)

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
