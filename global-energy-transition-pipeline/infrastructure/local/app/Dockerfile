
FROM python:3.12-slim

WORKDIR /app

ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY pyproject.toml .
COPY src/ ./src/

RUN pip install --no-cache-dir --no-deps .

CMD ["python", "-c", "print('Energy Transition Pipeline application container ready')"]
