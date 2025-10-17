FROM python:3.10-slim-bookworm

WORKDIR /app

# Install system dependencies for OpenCV and MediaPipe
RUN apt-get update && apt-get install -y \
    libgl1 \
    libglib2.0-0 \
    libsm6 \
    libxext6 \
    libxrender1 \
    libgomp1 \
    && rm -rf /var/lib/apt/lists/*


# Copy requirements first for better caching
COPY requirements.txt .

# Install Python dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Create necessary directories
RUN mkdir -p models data logs

# Set environment variables
ENV TF_CPP_MIN_LOG_LEVEL=2
ENV CUDA_VISIBLE_DEVICES=-1
ENV PYTHONUNBUFFERED=1

# Expose port (Render will use $PORT)
EXPOSE 5000

# Use Gunicorn for production
# --preload: Load app before workers (TensorFlow loads once)
# --workers 1: Single worker (saves memory)
# --threads 4: Handle concurrent requests
# --timeout 300: Allow 5min for training endpoint
# --bind 0.0.0.0:$PORT: Bind to Render's PORT
CMD gunicorn --bind 0.0.0.0:${PORT:-5000} \
    --workers 1 \
    --threads 4 \
    --timeout 300 \
    --preload \
    --access-logfile - \
    --error-logfile - \
    --log-level info \
    app:app