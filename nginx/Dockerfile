# --- 1️⃣ Этап сборки фронтенда ---
FROM node:20-alpine AS frontend-build
WORKDIR /app
COPY client/package*.json ./
RUN npm install
COPY client .
RUN npm run build

# --- 2️⃣ Этап запуска nginx ---
FROM nginx:alpine
COPY nginx/default.conf /etc/nginx/conf.d/default.conf
COPY --from=frontend-build /app/dist /usr/share/nginx/html
