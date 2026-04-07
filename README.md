# 🏠 Homelab — Self-Hosted Infrastructure

> Personal home server running production-grade services on Ubuntu Server.
> Built and maintained by **Merabet** — network & Linux engineer, Tlemcen 🇩🇿

---

## 📐 Architecture Overview

```
Internet
    │
    ▼
[Tailscale VPN]  ←──────────────────────────────────────┐
    │                                                     │
    ▼                                                     │
[Apache2 Reverse Proxy]  (HTTPS via Let's Encrypt)       │
    │                                                     │
    ▼                                                     │
[Docker]                                           [Remote access]
    │
    ├── n8n  (automation workflows)
    └── ... (expandable)
```

---

## 🧰 Stack

| Component | Role | Notes |
|-----------|------|-------|
| **Ubuntu Server** | Host OS | Base system |
| **Docker** | Container runtime | All services run in containers |
| **n8n** | Workflow automation | Self-hosted, accessible via HTTPS |
| **Apache2** | Reverse proxy | Handles SSL termination + WebSocket |
| **Tailscale** | VPN / secure tunnel | Exposes services without open ports |
| **Let's Encrypt** | TLS certificates | Auto-renewed via `tailscale cert` |

---

## 🔐 Security Highlights

- No open ports on the public internet — all access goes through Tailscale
- HTTPS enforced on all services (Let's Encrypt + `tailscale cert`)
- WebSocket support configured on Apache2 for n8n
- SSH hardened (key-only auth, non-default port)
- Automatic certificate renewal via cron/systemd timer

---

## ⚙️ Key Configurations

### Apache2 reverse proxy for n8n (HTTPS + WebSocket)

```apache
<VirtualHost *:443>
    ServerName n8n.your-tailscale-domain.ts.net

    SSLEngine on
    SSLCertificateFile    /etc/ssl/tailscale/your-domain.crt
    SSLCertificateKeyFile /etc/ssl/tailscale/your-domain.key

    ProxyPass        / http://localhost:5678/
    ProxyPassReverse / http://localhost:5678/

    # WebSocket support
    RewriteEngine On
    RewriteCond %{HTTP:Upgrade} websocket [NC]
    RewriteRule /(.*) ws://localhost:5678/$1 [P,L]

    ProxyPreserveHost On
</VirtualHost>
```

### n8n via Docker Compose

```yaml
version: "3"
services:
  n8n:
    image: n8nio/n8n
    restart: always
    ports:
      - "127.0.0.1:5678:5678"
    environment:
      - N8N_HOST=n8n.your-tailscale-domain.ts.net
      - N8N_PROTOCOL=https
      - WEBHOOK_URL=https://n8n.your-tailscale-domain.ts.net/
      - N8N_ENCRYPTION_KEY=your_encryption_key
    volumes:
      - n8n_data:/home/node/.n8n

volumes:
  n8n_data:
```

### Tailscale certificate renewal (cron)

```bash
# /etc/cron.weekly/tailscale-cert
#!/bin/bash
tailscale cert your-tailscale-domain.ts.net
systemctl reload apache2
```

---

## 📁 Repo Structure

```
homelab/
├── docker/
│   └── n8n/
│       └── docker-compose.yml
├── apache2/
│   └── n8n.conf
├── scripts/
│   └── renew-cert.sh
└── README.md
```

---

## 🚀 What I Learned

- Configuring a reverse proxy with SSL termination and WebSocket support
- Exposing self-hosted services securely without opening firewall ports
- Managing Docker containers and persistent volumes
- Automating TLS certificate renewal
- Troubleshooting network connectivity across VPN layers

---

## 📬 Contact

Open to freelance work — VPS setup, self-hosting, secure networking.

- GitHub: [@mehdi](https://github.com/Alpachino13)
- Email: merabetzakariamehdi@gmail.com
---

*Last updated: 2026*
