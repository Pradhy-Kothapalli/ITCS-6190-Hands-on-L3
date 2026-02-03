# ITCS-6190-Hands-on-L3

Create 4 files Requirements.txt, app.py, Dockerfile, and compose.yaml

```txt
flask
redis
```

```python
import time
import redis
from flask import Flask

app = Flask(__name__)
cache = redis.Redis(host="redis", port=6379)

def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incr("hits")
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)

@app.route("/")
def hello():
    count = get_hit_count()
    return "Hello World! I have been seen {} times.\n".format(count)
```

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

```yaml
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

Then build the application with:
```bash
docker compose up --build
```
Open the browser:
```text
http://localhost:8000
```

I learned how to install Docker Desktop and use the CLI to manage, run, and deploy containers by completing this Hands-on activity.
