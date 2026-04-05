# ================================
# 1. Update system
# ================================
sudo apt update && sudo apt upgrade -y

# ================================
# 2. Install basic tools
# ================================
sudo apt install -y curl wget git python3 python3-venv python3-pip

# ================================
# 3. Install Ollama (AI Engine)
# ================================
curl -fsSL https://ollama.com/install.sh | sh

# ================================
# 4. Install AI Models
# ================================
ollama pull llama3
ollama pull phi3
ollama pull deepseek-coder
ollama pull mistral

# ================================
# 5. Setup Open WebUI (Web Interface)
# ================================
mkdir -p ~/openwebui && cd ~/openwebui
python3 -m venv venv
source venv/bin/activate

pip install -U pip
pip install open-webui

# ================================
# 6. Create Open WebUI service
# ================================
sudo bash -c 'cat > /etc/systemd/system/openwebui.service <<EOF
[Unit]
Description=Open WebUI
After=network.target

[Service]
User='$USER'
WorkingDirectory=/home/'$USER'/openwebui
ExecStart=/home/'$USER'/openwebui/venv/bin/open-webui serve --host 0.0.0.0 --port 8080
Restart=always

[Install]
WantedBy=multi-user.target
EOF'

sudo systemctl daemon-reload
sudo systemctl enable openwebui
sudo systemctl start openwebui

# ================================
# 7. Install Cloudflare Tunnel
# ================================
cd ~
wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
sudo dpkg -i cloudflared-linux-amd64.deb

# ================================
# 8. Login Cloudflare (จะเปิด browser)
# ================================
cloudflared tunnel login

# ================================
# 9. Create tunnel
# ================================
cloudflared tunnel create ai-server

# ================================
# 10. Auto create config
# ================================
TUNNEL_ID=$(cloudflared tunnel list | grep ai-server | awk '{print $1}')
TUNNEL_FILE=$(ls ~/.cloudflared/*.json | head -n 1)

mkdir -p ~/.cloudflared

cat > ~/.cloudflared/config.yml <<EOF
tunnel: $TUNNEL_ID
credentials-file: $TUNNEL_FILE

ingress:
  - hostname: ai.yourdomain.com
    service: http://localhost:8080
  - service: http_status:404
EOF

# ================================
# 11. Route DNS
# ================================
cloudflared tunnel route dns ai-server ai.yourdomain.com

# ================================
# 12. Install tunnel service
# ================================
sudo cp ~/.cloudflared/config.yml /etc/cloudflared/config.yml

sudo cloudflared service install
sudo systemctl daemon-reload
sudo systemctl enable cloudflared
sudo systemctl start cloudflared

# ================================
# DONE
# ================================
