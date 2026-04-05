FROM python:3.10-slim
# Set working directory
WORKDIR /app
# Copy dependencies first for caching
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
# Copy app code
COPY . .
EXPOSE 8000
# Default command
CMD ["python", "app.py"]

Interview Talking Points:
 Docker builds images layer by layer, one instruction at a time.
 If you copy requirements.txt and install dependencies before copying your app code, Docker can reuse this layer from cache if your code changes but your dependencies don’t.
 Slim image reduces size
 Simple, clean base for any Python app




# Stage 1: Build dependencies
FROM python:3.10 AS builder
WORKDIR /app   # Set working directory

COPY requirements.txt . # Copy dependencies first for caching
#This tells pip to install all your libraries into a specific folder (/install) instead of the usual system folders.
RUN pip install --no-cache-dir --prefix=/install -r requirements.txt

# Stage 2: Runtime
FROM python:3.10-slim
WORKDIR /app  # Set working directory
#This is the "magic" line. It reaches back into the first container, grabs only the installed libraries from the /install folder, and drops them into the new container’s library folder.
COPY --from=builder /install /usr/local
COPY . .
EXPOSE 8000
CMD ["python", "app.py"]

Interview Talking Points:

Only runtime dependencies are copied → smaller image
Multi-stage builds prevent exposing build tools in production