*compose.yaml*

```yaml
services:
  frontend:
    build: ./frontend
    container_name: frontend
    restart: always
    ports:
      - "82:80"

```

*Dockerfile*

```
FROM node:20-alpine as build

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

RUN npm run build


FROM nginx:alpine

COPY --from=build /app/build /usr/share/nginx/html

COPY nginx/default.conf /etc/nginx/conf.d/default.conf

```

*nginx/default.conf*

```
server {
    listen 80;
    server_name _;

    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri /index.html;
    }
}

```