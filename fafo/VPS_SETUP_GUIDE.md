# VPS Setup Guide - Ubuntu Server to Wife's Cloud Home

**Last Updated:** January 21, 2026  
**VPS Provider:** DigitalOcean (Singapore)  
**OS:** Ubuntu 24.04 LTS  

---

## Prerequisites

- Fresh Ubuntu VPS with SSH access
- Your local PC with SSH client
- Domain (optional, for SSL later)

---

## Phase 1: Initial SSH Setup

### 1.1 First Login
```bash
# From your PC
ssh root@YOUR_VPS_IP
```

### 1.2 Update System
```bash
apt update && apt upgrade -y
```

### 1.3 Set Timezone
```bash
timedatectl set-timezone Asia/Kuala_Lumpur
```

### 1.4 Generate SSH Key (if needed)
```bash
ssh-keygen -t ed25519 -C "vps-key"
```

### 1.5 Setup SSH Alias (on your PC)
Add to `~/.ssh/config`:
```
Host vps
    HostName YOUR_VPS_IP
    User root
    IdentityFile ~/.ssh/id_ed25519
```

Now you can just: `ssh vps`

---

## Phase 2: Essential Tools

### 2.1 Python
```bash
apt install python3 python3-pip python3-venv -y

# Verify
python3 --version  # Should be 3.12+
```

### 2.2 Go (Golang)
```bash
# Download latest Go
wget https://go.dev/dl/go1.23.4.linux-amd64.tar.gz

# Extract to /usr/local
rm -rf /usr/local/go && tar -C /usr/local -xzf go1.23.4.linux-amd64.tar.gz

# Add to PATH (add to ~/.bashrc)
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
source ~/.bashrc

# Verify
go version
```

### 2.3 Julia
```bash
# Using juliaup (recommended)
curl -fsSL https://install.julialang.org | sh

# Or manual install
wget https://julialang-s3.julialang.org/bin/linux/x64/1.11/julia-1.11.2-linux-x86_64.tar.gz
tar -xzf julia-1.11.2-linux-x86_64.tar.gz
mv julia-1.11.2 /opt/julia
ln -s /opt/julia/bin/julia /usr/local/bin/julia

# Verify
julia --version
```

### 2.4 Node.js (for some tools)
```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | bash -
apt install nodejs -y

# Verify
node --version
npm --version
```

### 2.5 Essential Utilities
```bash
apt install -y \
    git \
    curl \
    wget \
    htop \
    tmux \
    unzip \
    net-tools \
    build-essential
```

---

## Phase 3: Ubuntu Desktop GUI + RDP

### 3.1 Install XFCE Desktop (Lightweight)
```bash
apt install xfce4 xfce4-goodies -y
```

### 3.2 Install xRDP
```bash
apt install xrdp -y

# Start and enable
systemctl start xrdp
systemctl enable xrdp

# Check status
systemctl status xrdp
```

### 3.3 Configure xRDP to Use XFCE
```bash
echo "xfce4-session" > ~/.xsession
chmod +x ~/.xsession

# For all users
echo "xfce4-session" | tee /etc/skel/.xsession
```

### 3.4 Allow RDP Port
```bash
ufw allow 3389/tcp
```

### 3.5 Connect via RDP
From Windows:
1. Open "Remote Desktop Connection"
2. Enter: `YOUR_VPS_IP:3389`
3. Login with your VPS credentials

---

## Phase 4: WireGuard VPN

### 4.1 Install WireGuard
```bash
apt install wireguard -y
```

### 4.2 Generate Keys
```bash
cd /etc/wireguard

# Generate server keys
wg genkey | tee server_private.key | wg pubkey > server_public.key
chmod 600 server_private.key
```

### 4.3 Create Server Config
```bash
cat > /etc/wireguard/wg0.conf << 'EOF'
[Interface]
Address = 10.0.0.1/24
ListenPort = 51820
PrivateKey = <SERVER_PRIVATE_KEY>

# Enable forwarding
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT

[Peer]
# Home PC
PublicKey = <PC_PUBLIC_KEY>
AllowedIPs = 10.0.0.2/32
EOF
```

Replace `<SERVER_PRIVATE_KEY>` with content of `server_private.key`

### 4.4 Enable IP Forwarding
```bash
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
sysctl -p
```

### 4.5 Start WireGuard
```bash
systemctl start wg-quick@wg0
systemctl enable wg-quick@wg0

# Check status
wg show
```

### 4.6 Allow WireGuard Port
```bash
ufw allow 51820/udp
```

---

## Phase 5: CLIProxyAPI Setup

### 5.1 Upload Files
```bash
# From your PC
scp -r CLIProxyAPI-6.7.8/ vps:/root/
```

### 5.2 Build (if needed)
```bash
cd /root/CLIProxyAPI-6.7.8

# If you have Go source
go build -o server_linux ./cmd/server
```

### 5.3 Create Systemd Service
```bash
cat > /etc/systemd/system/cliproxy.service << 'EOF'
[Unit]
Description=CLI Proxy API
After=network.target

[Service]
Type=simple
User=root
WorkingDirectory=/root/CLIProxyAPI-6.7.8
ExecStart=/root/CLIProxyAPI-6.7.8/server_linux
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable cliproxy
systemctl start cliproxy
```

---

## Phase 6: XML Proxy Setup

### 6.1 Install Python Dependencies
```bash
pip3 install fastapi uvicorn httpx aiohttp
```

### 6.2 Upload xml_proxy_v2.py
```bash
scp xml_proxy_v2.py vps:/root/
```

### 6.3 Create Systemd Service
```bash
cat > /etc/systemd/system/xmlproxy.service << 'EOF'
[Unit]
Description=XML Proxy V2 for Mobile
After=network.target cliproxy.service

[Service]
Type=simple
User=root
WorkingDirectory=/root
ExecStart=/usr/bin/python3 /root/xml_proxy_v2.py
Restart=always
RestartSec=5
Environment=PYTHONUNBUFFERED=1

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable xmlproxy
systemctl start xmlproxy
```

---

## Phase 7: SSHFS Bridge (PC ↔ VPS)

### 7.1 Install SSHFS on VPS
```bash
apt install sshfs -y
```

### 7.2 Create Mount Point
```bash
mkdir -p /mnt/home_pc
```

### 7.3 Mount Home PC (after WireGuard is connected)
```bash
sshfs PC@10.0.0.2:/c/cee_v25_Pre_Final /mnt/home_pc -o allow_other,reconnect
```

### 7.4 Auto-mount Service
```bash
cat > /etc/systemd/system/home-pc-mount.service << 'EOF'
[Unit]
Description=Mount Home PC via SSHFS
After=network-online.target wg-quick@wg0.service
Wants=network-online.target
Requires=wg-quick@wg0.service

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/usr/bin/sshfs PC@10.0.0.2:/c/cee_v25_Pre_Final /mnt/home_pc -o allow_other,reconnect,StrictHostKeyChecking=no
ExecStop=/usr/bin/fusermount -u /mnt/home_pc

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable home-pc-mount
```

---

## Phase 8: Security Hardening

### 8.1 Install fail2ban
```bash
apt install fail2ban -y

# Configure for aggressive banning
cat > /etc/fail2ban/jail.local << 'EOF'
[DEFAULT]
bantime = 3y
findtime = 1d
maxretry = 3

[sshd]
enabled = true
EOF

systemctl restart fail2ban
```

### 8.2 Configure UFW Firewall
```bash
ufw default deny incoming
ufw default allow outgoing

# Allow essentials
ufw allow ssh
ufw allow 3389/tcp    # RDP
ufw allow 51820/udp   # WireGuard
ufw allow 8002/tcp    # xml_proxy
ufw allow 8317/tcp    # CLIProxyAPI

ufw enable
```

---

## Quick Reference Commands

```bash
# Check all services
systemctl status cliproxy xmlproxy wg-quick@wg0 xrdp

# View logs
journalctl -u xmlproxy -f
journalctl -u cliproxy -f

# Restart services
systemctl restart cliproxy xmlproxy

# Check WireGuard
wg show

# Check mounts
df -h | grep mnt
```

---

## Architecture Summary

```
📱 Mobile Phone
    │
    │ HTTP :8002
    ▼
☁️ VPS (Ubuntu + XFCE)
    ├── CLIProxyAPI :8317
    ├── xml_proxy   :8002  
    ├── xRDP        :3389
    └── WireGuard   :51820
            │
            │ VPN Tunnel
            ▼
🖥️ Home PC (10.0.0.2)
```

---

**Tags:** #vps #setup #guide #ubuntu #wireguard #rdp #xfce
