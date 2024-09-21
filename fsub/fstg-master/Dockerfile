# Stage 1: Build image for installing dependencies
FROM python:3.11.10-alpine3.20 AS builder

# Set environment variables for Poetry and Python
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PIP_NO_CACHE_DIR=1 \
    PIP_DISABLE_PIP_VERSION_CHECK=1 \
    POETRY_VIRTUALENVS_CREATE=false \
    POETRY_NO_INTERACTION=1 \
    # Adding poetry installation directory to PATH
    PATH="/root/.local/bin:$PATH"

# Install essential system packages and build dependencies
RUN apk add --no-cache \
    tzdata \
    bash \
    curl \
    && apk add --no-cache --virtual .build-deps \
    gcc \
    musl-dev \
    libffi-dev \
    openssl-dev \
    zlib-dev \
    sqlite-dev

# Set the system timezone to Asia/Jakarta
RUN cp /usr/share/zoneinfo/Asia/Jakarta /etc/localtime \
    && echo "Asia/Jakarta" > /etc/timezone

# Install Poetry package manager
RUN curl -sSL https://install.python-poetry.org | python3 -

# Define the working directory
WORKDIR /app

# Copy Poetry configuration files
COPY pyproject.toml poetry.lock ./

# Install project dependencies without creating a virtual environment
RUN poetry install --no-root

# Stage 2: Final lightweight image
FROM python:3.11.10-alpine3.20 AS final

# Set environment variables for Python runtime
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    # Adding poetry installation directory to PATH
    PATH="/root/.local/bin:$PATH"

# Add a non-root user for security
RUN adduser -D -g '' appuser

# Set the system timezone in the final image
RUN apk add --no-cache tzdata \
    && cp /usr/share/zoneinfo/Asia/Jakarta /etc/localtime \
    && echo "Asia/Jakarta" > /etc/timezone \
    && apk del --no-cache tzdata

# Define the working directory
WORKDIR /app

# Copy installed dependencies from the builder stage
COPY --from=builder /usr/local/lib/python3.11/site-packages /usr/local/lib/python3.11/site-packages

# Copy poetry binary from the builder stage
COPY --from=builder /root/.local /root/.local

# Copy the application source code
COPY . .

# Adjust ownership of application files to the non-root user
RUN chown -R appuser:appuser /app

# Switch to the non-root user for executing the application
USER appuser

# Default command to run the application
CMD ["poetry", "run", "python", "main.py"]
