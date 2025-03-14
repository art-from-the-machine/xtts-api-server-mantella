# Use an official NVIDIA base image with CUDA support
FROM nvidia/cuda:12.1.1-cudnn8-devel-ubuntu22.04

# Set label for the docker image description
LABEL description="Docker image for xtts-api-server"

# Install required packages (avoid cache to reduce image size)
RUN apt-get update && \
    apt-get install --no-install-recommends -y \
    python3-dev portaudio19-dev libportaudio2 libasound2-dev libportaudiocpp0 \
    git python3 python3-pip make g++ ffmpeg sudo && \
    rm -rf /var/lib/apt/lists/*

# First, upgrade pip and install basic tools
RUN python3 -m pip install --upgrade pip wheel ninja virtualenv

# Downgrade setuptools to a version known to work with numpy/scipy
RUN python3 -m pip install setuptools==59.8.0

# Set the working directory
WORKDIR /app

# Install PyTorch and CUDA dependencies first
RUN pip install torch==2.1.0 torchaudio==2.1.0 --index-url https://download.pytorch.org/whl/cu121

# Install specific versions of problematic packages in the correct order
RUN pip install numpy==1.24.3
RUN pip install scipy==1.10.1
RUN pip install deepspeed==0.16.4

# Create a non-root user with the same UID/GID as the host user
ARG USER_ID=1000
ARG GROUP_ID=1000
RUN groupadd -g ${GROUP_ID} appuser && \
    useradd -u ${USER_ID} -g appuser -s /bin/bash -m appuser

# Create necessary directories and set permissions
RUN mkdir -p /app/xtts-server/speakers /app/xtts-server/output /app/xtts-server/models

# Copy just the requirements file first
COPY requirements.txt /app/
RUN pip install -r requirements.txt

# Now copy the application code (this layer will change when code changes)
COPY . /app

# Set proper ownership for all files
RUN chown -R appuser:appuser /app/xtts-server

# Switch to the non-root user
USER appuser

# Expose the container ports
EXPOSE 8020

# Run xtts_api_server when the container starts
# Using a multi-line format for better readability
CMD ["bash", "-c", "\
python3 -m xtts_api_server \
  --listen \
  -p 8020 \
  -t 'http://localhost:8020' \
  -sf '/app/xtts-server/speakers' \
  -o '/app/xtts-server/output' \
  -mf '/app/xtts-server/models' \
  -lsf '/app/xtts-server/latent_speaker_folder' \
  --deepspeed \
"]
