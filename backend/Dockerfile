FROM --platform=linux/amd64 python:3.11.5-alpine

RUN apk add --no-cache \
    curl

RUN adduser -D worker
USER worker
WORKDIR /home/worker

COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

ENV PATH="/home/worker/.local/bin:${PATH}"

COPY  app/ app/

LABEL version="1.0.0"

EXPOSE 8000
ENTRYPOINT ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--proxy-headers"]
