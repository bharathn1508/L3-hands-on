# Multi-Container Microservice with Docker

This project demonstrates how to use **Docker** and **Docker Compose** to build and run a multi-container microservice consisting of:
- A **Flask web application**
- A **Redis cache**
- A **PostgreSQL database**

It was developed as part of **Cloud Computing for Data Analysis (ITCS 6190/8190, Fall 2025)**.

---

## 📚 Learning Goals
- Understand containerized environments and Docker fundamentals.
- Gain hands-on experience setting up and configuring PostgreSQL, Flask, and Redis.
- Learn how to connect multiple services using Docker Compose.
- Practice testing, monitoring, and documenting containerized applications.
- Report and log issues in GitHub as part of the development workflow.

---

## ⚙️ Setup Instructions

### 1. Install Docker Desktop
- Download [Docker Desktop](https://www.docker.com/products/docker-desktop/) for your OS.
- Follow the installer instructions (enable **WSL2** on Windows if prompted).
- Verify installation:
```bash
docker -v
```

2. Run PostgreSQL with Docker
Pull the PostgreSQL image:
```bash
docker pull postgres
```
Start a PostgreSQL container:
```bash
docker run -d -p 5432:5432 --name postgres1 -e POSTGRES_PASSWORD=pass12345 postgres
```
Access the container:
```bash
docker exec -it postgres1 bash
```
Connect to PostgreSQL:
```bash
psql -d postgres -U postgres
```
3. Build Flask + Redis Web App
requirements.txt
```txt
flask
redis
```
app.py
```python
import time
import redis
from flask import Flask

app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)

def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)

@app.route('/')
def hello():
    count = get_hit_count()
    return 'Hello World! I have been seen {} times.\n'.format(count)
```
Dockerfile
```dockerfile
FROM python:3.7-alpine
WORKDIR /code
ENV FLASK_APP=app.py
ENV FLASK_RUN_HOST=0.0.0.0
RUN apk add --no-cache gcc musl-dev linux-headers
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
EXPOSE 5000
COPY . .
CMD ["flask", "run"]
```
compose.yaml
```yaml
version: "3.9"
services:
  web:
    build: .
    ports:
      - "8000:5000"
    depends_on:
      - redis
  redis:
    image: "redis:alpine"
```
4. Run the Application
Build and start all containers:
```bash
docker compose up --build
```
Visit the app in a browser:
👉 http://localhost:8000

✅ Example Output

When refreshing the page, the counter increases:
```bash
Hello World! I have been seen 1 times.
Hello World! I have been seen 2 times.
Hello World! I have been seen 3 times.
```
📖 What I Learned

How to install and configure Docker Desktop.

How to set up and run a PostgreSQL container.

How to build a Flask web application with Redis caching.

How to orchestrate multiple services with Docker Compose.

How to debug container networking issues and log them properly in GitHub Issues.

The importance of clear documentation for reproducibility.
