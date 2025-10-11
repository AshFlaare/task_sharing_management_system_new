# Используем легковесный Python образ
FROM python:3.11-slim

# Устанавливаем рабочую директорию
WORKDIR /app

# Копируем файл с зависимостями и устанавливаем их
COPY ../requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Копируем весь проект в контейнер
COPY ../app ./app
COPY ../manage.py .
COPY ../application ./application

# Указываем команду запуска Django
CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
