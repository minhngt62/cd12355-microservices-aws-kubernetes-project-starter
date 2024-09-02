# Use the official Python slim image from the Docker Hub
FROM python:3.9.19-slim-bullseye

# Set environment variables for PostgreSQL
ENV DB_USERNAME=myuser
ENV DB_PASSWORD=mypassword
ENV DB_HOST=127.0.0.1
ENV DB_PORT=5433
ENV DB_NAME=mydatabase

# Install Python dependencies
COPY requirements.txt /app/requirements.txt
WORKDIR /app
RUN pip install --no-cache-dir --upgrade pip setuptools wheel && \
    pip install --no-cache-dir -r requirements.txt

# Copy the application code
COPY . /app

# Specify the command to run the application
CMD ["python", "app.py"]