FROM python:3.11-slim

WORKDIR /app

# Copy the modified exporter script
COPY netgear-exporter.py /app/

# Install Python dependencies
RUN pip install --no-cache-dir requests beautifulsoup4 prometheus_client

EXPOSE 8000

CMD ["python", "netgear-exporter.py"]
