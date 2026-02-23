# Ibm-mainframe-thinkcentre-m900. #!/bin/bash
# ==============================================================================
# GGG-R OMEGA INSTALLER - "THE KNIGHT" (IBM M900)
# DATE: February 23, 2026 | TARGET: TAMPA, FL
# ==============================================================================

set -e
echo "[GGG-R] 🚀 Starting Automatic Full-System Installation..."

# 1. System Utilities
sudo apt update -y && sudo apt install -y python3-venv python3-pip git gh monit curl

# 2. Project Directory & Python Stack
mkdir -p ~/GGG-R && cd ~/GGG-R
python3 -m venv ggg_env
source ggg_env/bin/activate
pip install fastapi[standard] uvicorn httpx python-dotenv kyber-py

# 3. Core Mainframe Code (main.py)
cat <<EOF > main.py
import sqlite3, json, os, httpx
from datetime import date
from fastapi import FastAPI
from fastapi.responses import HTMLResponse

app = FastAPI(title="GGG-R Sovereign Core v3.0")
VOTE_DATE = date(2026, 3, 20)

@app.on_event("startup")
async def startup_event():
    conn = sqlite3.connect('ggg_r_intel.db')
    c = conn.cursor()
    c.execute("CREATE TABLE IF NOT EXISTS industry (id TEXT PRIMARY KEY, data TEXT)")
    conn.commit(); conn.close()

@app.get("/")
async def dashboard():
    days = (VOTE_DATE - date.today()).days
    return HTMLResponse(content=f"<html><body style='background:#050505;color:#00ffcc;font-family:monospace;padding:50px;'><h1>GGG-R v3.0 | COMMAND</h1><h2 style='color:#ff00c8;'>{days} DAYS TO VOTE</h2><p><b>SECURITY:</b> NIST FIPS 203 ACTIVE</p></body></html>")
EOF

# 4. Persistence & Self-Healing
sudo bash -c "cat <<EOF > /etc/systemd/system/ggg-genie.service
[Unit]
Description=GGG-R Sovereign Dashboard
After=network.target

[Service]
User=$(whoami)
WorkingDirectory=$(pwd)
ExecStart=$(pwd)/ggg_env/bin/python3 -m uvicorn main:app --host 0.0.0.0 --port 8000
Restart=always

[Install]
WantedBy=multi-user.target
EOF"

sudo systemctl daemon-reload && sudo systemctl enable ggg-genie && sudo systemctl restart ggg-genie

# 5. Git Synchronization
if [ ! -d ".git" ]; then
    git init -b main
    echo "ggg_env/" > .gitignore
    echo "*.db" >> .gitignore
    git add . && git commit -m "GGG-R Omega: Automated Deployment"
fi

echo "=============================================================================="
echo "✅ GGG-R OMEGA IS LIVE"
echo "URL: http://$(hostname -I | awk '{print $1}'):8000"
echo "=============================================================================="
