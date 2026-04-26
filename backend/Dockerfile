FROM --platform=linux/amd64 python:3.11.5-alpine3.18

RUN apk add --update \
    curl \
    && rm -rf /var/cache/apk/*

RUN adduser -D worker
USER worker
WORKDIR /home/worker

COPY --chown=worker:worker requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

ENV PATH="/home/worker/.local/bin:${PATH}"

COPY --chown=worker:worker app/ app/

LABEL version="1.0.0"

EXPOSE 8000
ENTRYPOINT ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--proxy-headers"]

