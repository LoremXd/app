
FROM --platform=linux/amd64 node:20.12.1-bookworm as builder

ENV NODE_ENV=production

ENV REACT_APP_FRONT_VERSION=$REACT_APP_FRONT_VERSION
ENV REACT_APP_BACKEND_URL=$REACT_APP_BACKEND_URL

WORKDIR /usr/src/app

COPY package*.json ./

RUN npm ci --no-audit --maxsockets 1

COPY . .

RUN npm run build

FROM nginx:1.25.2
LABEL version="1.0.0"

COPY --from=builder /usr/src/app/build/ /usr/share/nginx/html
COPY --from=builder /usr/src/app/nginx.conf /etc/nginx/conf.d/default.conf

COPY entrypoint.sh /entrypoint.sh

RUN chmod +x /entrypoint.sh

ENTRYPOINT ["/entrypoint.sh"]

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
