# Self-Hosted AI Platform

A private AI infrastructure built using local resources.  
This project demonstrates how to run multiple large language models (LLMs) locally with a web interface and secure remote access.

---

## Overview

This system allows you to:

- Run AI models locally without external APIs
- Use a web interface similar to ChatGPT
- Access your AI through a custom domain
- Manage your own AI infrastructure

---

## Architecture

```

User (Browser)
↓
Cloudflare (Domain + Tunnel)
↓
Tunnel Server (cloudflared)
↓
AI Server (Ubuntu)
↓
Open WebUI (Port 8080)
↓
Ollama (LLM Runtime)
↓
Local Models

````

---

## Tech Stack

### Infrastructure
- Proxmox VE
- Ubuntu Server

### AI Runtime
- Ollama

### Models
- llama3
- deepseek-coder
- phi3
- mistral

### Interface
- Open WebUI

### Networking
- Cloudflare Tunnel
- Custom Domain

---

## Requirements

Minimum:

- CPU: 4 cores
- RAM: 10–12 GB
- Storage: 50 GB

Recommended:

- RAM: 16 GB or higher
- GPU (optional)

---

## Installation

### 1. Install Ollama

```bash
curl -fsSL https://ollama.com/install.sh | sh
````

---

### 2. Install Models

```bash
ollama pull llama3
ollama pull phi3
ollama pull deepseek-coder
ollama pull mistral
```

---

### 3. Run AI

```bash
ollama run llama3
```

---

### 4. Install Open WebUI

```bash
pip install open-webui
open-webui serve --host 0.0.0.0 --port 8080
```

Access:

```
http://YOUR_SERVER_IP:8080
```

---

### 5. Setup Cloudflare Tunnel

Install:

```bash
wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
sudo dpkg -i cloudflared-linux-amd64.deb
```

Login:

```bash
cloudflared tunnel login
```

Create tunnel:

```bash
cloudflared tunnel create ai-server
```

Route domain:

```bash
cloudflared tunnel route dns ai-server ai.yourdomain.com
```

Create config:

```bash
mkdir -p ~/.cloudflared

cat > ~/.cloudflared/config.yml <<EOF
tunnel: YOUR_TUNNEL_ID
credentials-file: /home/YOUR_USER/.cloudflared/YOUR_TUNNEL_ID.json

ingress:
  - hostname: ai.yourdomain.com
    service: http://YOUR_SERVER_IP:8080
  - service: http_status:404
EOF
```

Run:

```bash
cloudflared tunnel run ai-server
```

Optional (run as service):

```bash
sudo cloudflared service install
sudo systemctl enable cloudflared
sudo systemctl start cloudflared
```

---

## Access

Local:

```
http://YOUR_SERVER_IP:8080
```

Public:

```
https://ai.yourdomain.com
```

---

## Notes

* This setup uses CPU by default
* Performance depends on hardware
* Run one model at a time for best performance

---

## Future Improvements

* GPU acceleration
* Multi-user system
* Model routing
* Monitoring system
* Containerized deployment

---

## Author

Thiraphat

```
