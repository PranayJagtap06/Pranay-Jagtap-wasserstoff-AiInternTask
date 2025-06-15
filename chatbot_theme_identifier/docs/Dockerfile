FROM python:3.13-slim-bookworm

# Install system dependencies
RUN apt-get update && apt-get install -y \
    tesseract-ocr \
    libmagic1 \
    wget \
    && rm -rf /var/lib/apt/lists/*

# Set working directory
WORKDIR /app

# Copy streamlit app code, requirements and install dependencies
COPY requirements-api.txt .
# RUN pip install --no-cache-dir -r requirements.txt
RUN pip install --upgrade pip && \
    pip install --no-cache-dir pip-tools && \
    for i in {1..3}; do \
        pip install --no-cache-dir -r requirements-api.txt && break || \
        echo "Retry attempt $i" && sleep 5; \
    done

# Create a non-root user
# RUN useradd -ms /bin/sh -u 1001 app
# OR 
# RUN useradd -ms /bin/bash app && \
#     mkdir -p /home/app && \
#     chown -R app:app /home/app && \
#     chmod -R 755 /home/app
# Alternatively, create a group and user
# RUN groupadd -r app && \
#     useradd -r -g app -s /bin/bash -d /home/app app

# Create necessary directories
RUN mkdir -p /app/data/chroma /app/data/uploads /app/.cache/chroma /app/.cache/chroma/onnx_models/all-MiniLM-L6-v2 && \
    wget -q https://chroma-onnx-models.s3.amazonaws.com/all-MiniLM-L6-v2/onnx.tar.gz \
    -O /app/.cache/chroma/onnx_models/all-MiniLM-L6-v2/onnx.tar.gz && \
    cd /app/.cache/chroma/onnx_models/all-MiniLM-L6-v2 && \
    tar xzf onnx.tar.gz
    # chown -R app:app /app && \
    # chmod -R 755 /app

# Copy application code
COPY . /app
# RUN chown -R app:app /app

# Switch to non-root user
# USER app

# Set environment variable
# ENV PATH="/home/app/.local/bin:${PATH}"
# ENV PYTHONPATH=/app
# ENV PYTHONUNBUFFERED=1
ENV DOCKER_BUILDKIT=1
ENV COMPOSE_HTTP_TIMEOUT=300
ENV COMPOSE_BAKE=true
ENV CHROMA_CACHE_DIR=/app/.cache/chroma

# Mount secret at build time (if needed)
RUN --mount=type=secret,id=GEMINI_API_KEY,mode=0444,required=true \
    if [ -f "/run/secrets/GEMINI_API_KEY" ]; then \
    export GEMINI_API_KEY=$(cat /run/secrets/GEMINI_API_KEY); \
    fi

# Set environment variable to read from container's environment
ENV GEMINI_API_KEY=${GEMINI_API_KEY}

# Expose ports for FastAPI and Streamlit
EXPOSE 8000

# Create startup script
# RUN echo '''#!/bin/bash\n\
# uvicorn app.main:app --host 0.0.0.0 --port 8000 & \
# streamlit run app.py --server.port 8501 --server.address 0.0.0.0 & \n\
# wait''' > /app/start.sh && chmod +x /app/start.sh

# Command to run both services
# CMD ["/app/start.sh"]
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000", "--reload"]

