# snyk-ecs-xray

A minimal **Flask** web application instrumented with **AWS X-Ray**, packaged as a Docker container. It's intended as a sample workload for deploying to AWS ECS with X-Ray tracing (and for exploring Snyk container/security scanning).

## Features

- Simple Flask app exposing a single `/` endpoint that returns a JSON response.
- AWS X-Ray tracing via `XRayMiddleware`.
- Configuration driven by environment variables.
- Containerized with a Python-based Dockerfile.

## Project structure

```
.
├── app.py             # Flask application with AWS X-Ray middleware
├── requirements.txt   # Python dependencies (flask, aws-xray-sdk)
└── Dockerfile         # Container image definition
```

## Requirements

- Python 3.x (or Docker)
- Dependencies listed in [`requirements.txt`](requirements.txt):
  - `flask`
  - `aws-xray-sdk`

## Environment variables

The app reads the following environment variables at startup:

| Variable           | Description                                  |
| ------------------ | -------------------------------------------- |
| `MY_APP_CONFIG`    | Arbitrary application configuration value.   |
| `MY_DB_PASSWORD`   | Database password (returned in the response).|
| `SERVICE_NAME`     | Service name reported to AWS X-Ray.          |

> **Security note:** The sample `/` endpoint echoes `MY_DB_PASSWORD` in its JSON response. This is intentional for demonstration/scanning purposes — **do not expose secrets like this in real applications.**

## Running locally

### With Python

```bash
pip install -r requirements.txt

export SERVICE_NAME="snyk-ecs-xray"
export MY_APP_CONFIG="example-config"
export MY_DB_PASSWORD="example-password"

python app.py
```

The app will be available at `http://localhost:8080/`.

### With Docker

```bash
# Build the image
docker build -t snyk-ecs-xray .

# Run the container
docker run -p 8080:8080 \
  -e SERVICE_NAME="snyk-ecs-xray" \
  -e MY_APP_CONFIG="example-config" \
  -e MY_DB_PASSWORD="example-password" \
  snyk-ecs-xray
```

## Usage

Once running, send a request to the root endpoint:

```bash
curl http://localhost:8080/
```

Example response:

```json
{
  "message": "Hello from Flask!",
  "app_config": "example-config",
  "db_password": "example-password"
}
```

## AWS X-Ray

The application is instrumented with the AWS X-Ray SDK. When deployed alongside an X-Ray daemon (for example, as a sidecar in AWS ECS), request traces will be sent to AWS X-Ray under the name provided in `SERVICE_NAME`.
