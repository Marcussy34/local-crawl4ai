# Setup Guide: Self-hosted AI Starter Kit with Crawl4AI

This guide provides detailed instructions for setting up the Self-hosted AI Starter Kit with Crawl4AI integration on a new computer.

## System Requirements

- **Operating System**: Windows, macOS, or Linux
- **RAM**: Minimum 8GB, recommended 16GB+
- **Storage**: At least 10GB of free space
- **CPU**: 4+ cores recommended
- **GPU** (optional): NVIDIA GPU with CUDA support for GPU acceleration
- **Internet Connection**: Required for downloading Docker images

## Prerequisites

### 1. Install Docker and Docker Compose

#### Windows

1. Install [Docker Desktop for Windows](https://www.docker.com/products/docker-desktop)
2. During installation, ensure WSL 2 is enabled
3. After installation, open Docker Desktop as administrator and ensure it's running

#### macOS

1. Install [Docker Desktop for Mac](https://www.docker.com/products/docker-desktop)
2. After installation, open Docker Desktop and ensure it's running

#### Linux

1. Install Docker:
   ```bash
   sudo apt-get update
   sudo apt-get install docker.io
   ```
2. Install Docker Compose:
   ```bash
   sudo apt-get install docker-compose
   ```
3. Start Docker service:
   ```bash
   sudo systemctl start docker
   sudo systemctl enable docker
   ```
4. Add your user to the docker group:
   ```bash
   sudo usermod -aG docker $USER
   ```
   (Log out and log back in for this to take effect)

### 2. NVIDIA GPU Setup (Optional)

If you have an NVIDIA GPU and want to use it for acceleration:

#### Windows

1. Install the latest [NVIDIA drivers](https://www.nvidia.com/Download/index.aspx)
2. Install [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html#docker)

#### Linux

1. Install NVIDIA drivers:
   ```bash
   sudo apt-get install nvidia-driver-XXX  # Replace XXX with the latest version
   ```
2. Install NVIDIA Container Toolkit:
   ```bash
   distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
   curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
   curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
   sudo apt-get update
   sudo apt-get install -y nvidia-container-toolkit
   sudo systemctl restart docker
   ```

## Installation Steps

### 1. Clone the Repository

```bash
git clone https://github.com/n8n-io/self-hosted-ai-starter-kit.git
cd self-hosted-ai-starter-kit
```

### 2. Configure Environment Variables

Create or edit the `.env` file in the project root directory:

```bash
# Database credentials - DO NOT CHANGE THESE VALUES
POSTGRES_USER=postgres
POSTGRES_PASSWORD=password
POSTGRES_DB=postgres

# n8n security
N8N_ENCRYPTION_KEY=super-secret-key
N8N_USER_MANAGEMENT_JWT_SECRET=even-more-secret

# Optional: OpenAI API key if you want to use OpenAI models
# OPENAI_API_KEY=your-openai-api-key
```

> [!IMPORTANT]
> The PostgreSQL credentials must remain as shown above for compatibility with the n8n container configuration.

### Additional PostgreSQL Setup Steps

After starting the services, configure the PostgreSQL connection in n8n:

1. Go to http://localhost:5678/
2. Navigate to Credentials
3. Add new Postgres credentials with these exact values:
   - Host: `postgres`
   - Database: `postgres`
   - User: `postgres`
   - Password: `password`
   - Maximum Number of Connections: `100`

### 3. Start the Core Services

Choose the appropriate command based on your hardware:

#### For NVIDIA GPU Users

```bash
docker compose --profile gpu-nvidia up -d
```

#### For AMD GPU Users on Linux

```bash
docker compose --profile gpu-amd up -d
```

#### For CPU-only Users

```bash
docker compose --profile cpu up -d
```

### 4. Set Up Crawl4AI

Choose the appropriate command based on your system architecture and GPU availability:

#### For AMD64/x86_64 Systems (Most Windows/Linux PCs)

##### With NVIDIA GPU Support

```bash
# Run Crawl4AI with GPU support
docker run -d --name crawl4ai \
  --network self-hosted-ai-starter-kit_demo \
  -p 11235:11235 \
  -e CRAWL4AI_API_TOKEN="mysecrettoken" \
  --gpus all \
  unclecode/crawl4ai:gpu-amd64
```

##### Without GPU (AMD64/x86_64)

```bash
# Run Crawl4AI without GPU
docker run -d --name crawl4ai \
  --network self-hosted-ai-starter-kit_demo \
  -p 11235:11235 \
  -e CRAWL4AI_API_TOKEN="mysecrettoken" \
  unclecode/crawl4ai:amd64
```

#### For ARM64 Systems (Apple Silicon, etc.)

```bash
# Run Crawl4AI
docker run -d --name crawl4ai \
  --network self-hosted-ai-starter-kit_demo \
  -p 11235:11235 \
  -e CRAWL4AI_API_TOKEN="mysecrettoken" \
  unclecode/crawl4ai:latest
```

### 5. Verify System Architecture and GPU Support

#### Check System Architecture

```bash
# Verify your system architecture
docker version --format '{{.Server.Arch}}'
```

#### Verify GPU Support (if using GPU version)

```bash
# For NVIDIA GPU systems
docker run --rm --gpus all nvidia/cuda:11.8.0-base-ubuntu22.04 nvidia-smi
```

You should see your GPU information displayed if GPU support is working correctly.

### 6. Verify Services Are Running

```bash
docker ps
```

You should see containers for:

- n8n
- postgres
- qdrant
- ollama (if using GPU profile)
- crawl4ai

### 7. Access n8n

Open your browser and navigate to:
