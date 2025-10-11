# client/Dockerfile
FROM node:18-alpine

WORKDIR /app

# Устанавливаем зависимости
COPY package*.json ./
RUN npm install

# Копируем код
COPY . .

# Открываем порт
EXPOSE 5173

# Запускаем dev-сервер Vue
CMD ["npm", "run", "dev", "--", "--host"]