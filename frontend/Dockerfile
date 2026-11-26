FROM nginx:1.31.10-alpine-slim

WORKDIR /app

COPY project.Coderr /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
