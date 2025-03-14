# Ollama with Open WebUI and OpenAI-Compatible API

This repository provides a comprehensive guide for setting up Ollama with Open WebUI and configuring it with an OpenAI-compatible API interface.

## Overview

[Ollama](https://ollama.ai/) is a framework for running large language models (LLMs) locally on your own hardware. This guide will help you set up Ollama along with [Open WebUI](https://github.com/open-webui/open-webui) (formerly Ollama WebUI) to provide a user-friendly interface, and configure it to expose an OpenAI-compatible API endpoint for seamless integration with existing tools and applications.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Installation Guide](#installation-guide)
  - [Step 1: Install Ollama](#step-1-install-ollama)
  - [Step 2: Set Up Open WebUI](#step-2-set-up-open-webui)
  - [Step 3: Configure OpenAI-Compatible API](#step-3-configure-openai-compatible-api)
- [Usage](#usage)
- [Advanced Configuration](#advanced-configuration)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)

## Prerequisites

- A computer with at least 8GB RAM (16GB+ recommended for larger models)
- 20GB+ free disk space
- Linux, macOS, or Windows with WSL2
- Docker and Docker Compose (for containerized setup)
- Basic familiarity with command line and terminal operations

## Installation Guide

### Step 1: Install Ollama

#### Linux

```bash
curl -fsSL https://ollama.ai/install.sh | sh
```

#### macOS

Download and install from [Ollama's website](https://ollama.ai/download).

#### Windows

Install WSL2 first, then follow the Linux installation instructions.

After installation, verify Ollama is working:

```bash
ollama run llama2
```

This will download the Llama 2 model and start a chat session. Type 'exit' to quit.

### Step 2: Set Up Open WebUI

We'll use Docker Compose for an easy setup:

1. Create a new directory for your project:

```bash
mkdir ollama-setup
cd ollama-setup
```

2. Create a `docker-compose.yml` file:

```yaml
version: '3.8'

services:
  ollama:
    image: ollama/ollama:latest
    container_name: ollama
    volumes:
      - ollama_data:/root/.ollama
    ports:
      - "11434:11434"
    restart: unless-stopped

  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: open-webui
    volumes:
      - open_webui_data:/app/backend/data
    ports:
      - "3000:8080"
    environment:
      - OLLAMA_API_BASE_URL=http://ollama:11434
      - OPENAI_API_KEY=sk-111111111111111111111111111111111111111111111111
    depends_on:
      - ollama
    restart: unless-stopped

volumes:
  ollama_data:
  open_webui_data:
```

3. Start the services:

```bash
docker-compose up -d
```

4. Access the Open WebUI at `http://localhost:3000`

### Step 3: Configure OpenAI-Compatible API

Open WebUI already provides an OpenAI-compatible API endpoint. To use it:

1. Log in to Open WebUI and go to Settings > API Keys
2. Generate a new API key
3. Use this API key with the following endpoint format:

```
http://localhost:3000/v1/chat/completions
```

For applications that expect an OpenAI-compatible API, configure them to use this endpoint with your API key.

Example with curl:

```bash
curl http://localhost:3000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{
    "model": "llama2",
    "messages": [
      {
        "role": "user",
        "content": "Hello, how are you?"
      }
    ]
  }'
```

## Usage

### Managing Models

To download and manage models through Ollama:

```bash
# List available models
ollama list

# Pull a specific model
ollama pull mistral

# Remove a model
ollama rm llama2
```

### Using the Web Interface

Open WebUI provides:
- Chat interface for conversations
- Model management
- Conversation history
- Prompt templates
- API key management

### Using the API

The OpenAI-compatible API supports these endpoints:
- `/v1/chat/completions` - For chat completions
- `/v1/embeddings` - For generating embeddings
- `/v1/models` - For listing available models

## Advanced Configuration

### Custom Model Configuration

Create a `Modelfile` to customize model parameters:

```
FROM llama2
PARAMETER temperature 0.7
PARAMETER top_p 0.9
SYSTEM You are a helpful AI assistant named Ollama.
```

Build the custom model:

```bash
ollama create mycustommodel -f Modelfile
```

### GPU Acceleration

For NVIDIA GPUs, ensure the NVIDIA Container Toolkit is installed and add these to your docker-compose.yml:

```yaml
services:
  ollama:
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]
```

## Troubleshooting

### Common Issues

1. **Ollama not starting**: Check system resources and logs with `docker logs ollama`
2. **Model download failures**: Verify internet connection and disk space
3. **API connection issues**: Ensure correct API key and endpoint URL

### Getting Help

- [Ollama GitHub Issues](https://github.com/ollama/ollama/issues)
- [Open WebUI GitHub Issues](https://github.com/open-webui/open-webui/issues)

## Contributing

Contributions to this guide are welcome! Please feel free to submit a Pull Request with improvements or corrections.

## License

This project is licensed under the MIT License - see the LICENSE file for details.