# =========================================================
# Stage 1: Base image with Python
# =========================================================
FROM nvidia/cuda:12.4.0-runtime-ubuntu22.04

# Prevent interactive prompts
ENV DEBIAN_FRONTEND=noninteractive

# =========================================================
# Install system dependencies
# =========================================================
RUN apt-get update && apt-get install -y --no-install-recommends \
    python3.10 \
    python3.10-venv \
    python3-pip \
    git \
    sox \
    libsox-fmt-all \
    && rm -rf /var/lib/apt/lists/*

# =========================================================
# Set working directory
# =========================================================
WORKDIR /app

# =========================================================
# Clone WeKWS repository
# =========================================================
RUN git clone https://github.com/wenet-e2e/wekws.git . 

# =========================================================
# Install Python dependencies
# =========================================================
RUN python3 -m pip install --upgrade pip \
    && pip install torch==2.5.1+cu124 torchaudio==2.5.1 --index-url https://download.pytorch.org/whl/cu124 \
    && pip install -r requirements.txt \
    && pip install pre-commit \
    && pre-commit install

# =========================================================
# Default command
# =========================================================
CMD ["/bin/bash"]
