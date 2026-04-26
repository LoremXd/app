FROM --platform=linux/amd64 python:3.11.5-slim

RUN apt-get update && apt-get install --no-install-recommends -y \
    curl \
 && rm -rf /var/lib/apt/lists/*

RUN adduser --disabled-password worker
USER worker
WORKDIR /home/worker

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app/ app/

LABEL version="1.0.0"
EXPOSE 8000
ENTRYPOINT ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--proxy-headers"]

