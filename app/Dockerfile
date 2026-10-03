# app/Dockerfile
FROM python:3.12-slim

# No .pyc files, and unbuffered logs so `docker compose logs api` is live
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

# Non-root user, plus a /data directory it owns. The named volume mounted
# at /data copies this ownership the first time the volume is created.
RUN useradd --system --create-home --uid 1000 appuser \
    && mkdir -p /data \
    && chown appuser:appuser /data

WORKDIR /app

# Install dependencies first so this layer is cached when only code changes
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy the application code, owned by the non-root user
COPY --chown=appuser:appuser . .

USER appuser

EXPOSE 8000

# Slim images have no curl, so use Python for the health probe
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD python -c "import urllib.request; urllib.request.urlopen('http://127.0.0.1:8000/api/health')" || exit 1

# 0.0.0.0 so Nginx can reach it over the Docker network; no --reload
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
