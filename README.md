
# Increment Service

Increment Service is a Spring Boot application that provides an increment endpoint. The service is designed to handle 
requests where a key-value pair is sent, and the value is summed in a Redis cache. 
The summed value is held in the cache for 10 seconds, after which it is pushed to a PostgreSQL database for persistence.

## Features

- Increases the value by keeping it in memory (Redis) for 10 seconds, then pushes the sum to PostgreSQL.
- Redis key expiration listener to handle expired keys.
- Logging of incoming requests.
- Rate limiting implemented using Bucket4j.
- Secured using OAuth2 protocol, integrated with Auth0 for authentication.
- Custom metrics exposed via Prometheus to expose application metrics.

## Prerequisites

- Java 21
- Maven
- Docker and Docker Compose

## Setup and Running the Application

### 1. Clone the Repository
```bash
git clone https://github.com/yourusername/increment-service.git
cd increment-service
```

### 2. Build the Application
```bash
./mvn clean install
```

### 3. Running the Application
You can run the application using Docker Compose. Go to the project directory, open terminal, and type the following command:
Once all the 5 containers are up, you can proceed to the next point to accessing the application
```bash
docker-compose up --build
```

### 4. Accessing the Application
You can access the application using the following endpoints:
```bash
Generate the bearer token: curl --location 'https://dev-xokjpwctwfjktg8f.us.auth0.com/oauth/token' --header 'content-type: application/json' --header 'Cookie: did=s%3Av0%3A97b38b1d-c5b9-40df-974e-64a0163f1b37.VVqQihnwVatNZR3OKh%2BTiFF5sFJsITHfwysIhNPz2qQ; did_compat=s%3Av0%3A97b38b1d-c5b9-40df-974e-64a0163f1b37.VVqQihnwVatNZR3OKh%2BTiFF5sFJsITHfwysIhNPz2qQ' --data '{"client_id": "myMGEtGdR8c8PVpaHZNTIVpusQlCdJjp","client_secret": "DySz2DTM8O_XZ5CXvoq1zM2c8V1m_l10cbHk_78Ml0wndSxZu_RYKakWo6lddM8m","audience": "http://increment.api","grant_type": "client_credentials"}'
```
```bash
Invoke increment endpoint with token: curl --location 'localhost:3333/api/increment' --header 'Content-Type: application/json' --header 'Authorization: Bearer {token}' --data '{"key":"test","value":100}' 
```

### 6. Accessing the Database
To view or manage the data stored in PostgreSQL, you can use a database client like DBeaver. Here are the details you need to connect:
```bash
    URL: localhost
    Port: 5432
    Database: increment_db
    Username: user
    Password: pwd
```

### 7. Rate Limiting
Rate limiting is implemented using Bucket4j. The `RateLimitFilter` excludes requests to the `/actuator` endpoints 
and limits other requests based on the client's IP address. f a client sends more than 5 requests within one minute, 
the application will respond with an HTTP 429 Too Many Requests status code.

### 6. Accessing Prometheus to View Metrics
To view the metrics, you can access the following endpoint in your browser:
```bash
Metrics Endpoint: http://localhost:9090
```
To visualize the metrics, you can use the following Prometheus queries and create graphs:
```bash
 - Total Number of HTTP Requests: sum(http_server_requests_seconds_count{uri!~"/actuator.*|.*UNKNOWN.*"})
 - Total Number of Error Responses: sum by (uri) (http_server_requests_seconds_count{uri!~"/actuator.*|.*UNKNOWN.*"})
 - Total number of requests grouped by HTTP status code: sum by (status) (http_server_requests_seconds_count{uri!~"/actuator.*|.*UNKNOWN.*"})
 - Total number of requests grouped by HTTP method: sum by (method) (http_server_requests_seconds_count{uri!~"/actuator.*|.*UNKNOWN.*"})
 - average duration of requests for each endpoint URI: avg by (uri) (http_server_requests_seconds_sum{uri!~"/actuator.*|.*UNKNOWN.*"} / http_server_requests_seconds_count{uri!~"/actuator.*|.*UNKNOWN.*"})
 - Track the rate of error responses: sum by (status) (http_server_requests_seconds_count{status=~"4..|5..", uri!~"/actuator.*|.*UNKNOWN.*"})
```

### 7. Stop the Application
```bash
docker-compose down
...