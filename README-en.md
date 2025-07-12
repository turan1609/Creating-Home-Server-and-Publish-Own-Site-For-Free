
# 🏠 Home Server Project (Homelab with Docker & Cloudflare Tunnel)

This project documents how to turn a standard desktop computer into a secure and cost-free development/hosting server using modern DevOps tools. The system is designed to run publicly available, secure services (such as websites, Git servers, etc.) without requiring static IP or port forwarding — even in home network environments.

📌 Live Example: **Portfolio Website** (`https://yusufturan.com.tr`)

## 🎯 Project Goals

- **Eliminate Hosting Costs:** Host personal projects and web apps without paying for hosting services.
- **Bypass Storage Limits:** Host large Git repositories beyond GitHub's storage limits.
- **Overcome Network Barriers:** Bypass ISP restrictions such as dynamic IP and CG-NAT.
- **Enhance Security:** Hide origin IP and encrypt all traffic via HTTPS.

## 🛠️ Technology Stack

| Category           | Technology                   | Description |
|--------------------|------------------------------|-------------|
| OS                 | Ubuntu Server 22.04 LTS      | Stable and secure Linux distro |
| Containerization   | Docker & Docker Compose      | Portable and isolated services |
| Reverse Proxy      | Nginx Proxy Manager          | Routing and auto SSL management |
| Secure Access      | Cloudflare Tunnel            | World-access without open ports |
| DNS & Security     | Cloudflare                   | DNS, CDN, and DDoS protection |

## 🏗️ Architecture

```mermaid
graph TD
    A[User] -->|https://yusufturan.com.tr| B(Cloudflare Edge);
    B --> C{Cloudflare Tunnel};
    subgraph Home Network (Origin Server - IP Hidden)
        C <--> D[cloudflared Container];
        D --> E[Nginx Proxy Manager Container];
        E -->|/| F[Portfolio Website Container];
        E -->|git.domain.com| G[Gitea Container];
        E -->|app.domain.com| H[Other Projects...];
    end

    style Home Network fill:#f9f,stroke:#333,stroke-width:2px
```

## 🚀 Setup Instructions

### 1. Shared Docker Network
```bash
docker network create proxy
```

### 2. Nginx Proxy Manager
```yaml
version: '3.8'
services:
  app:
    image: 'jc21/nginx-proxy-manager:latest'
    restart: unless-stopped
    ports:
      - '80:80'
      - '443:443'
      - '81:81'
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
    networks:
      - proxy
networks:
  proxy:
    external: true
```

### 3. Cloudflare Tunnel
```yaml
version: '3.8'
services:
  cloudflared:
    image: cloudflare/cloudflared:latest
    restart: unless-stopped
    command: tunnel --no-autoupdate run --token <CLOUDFLARE_TUNNEL_TOKEN>
    networks:
      - proxy
networks:
  proxy:
    external: true
```

### 4. Portfolio Website (Static)
```yaml
version: '3.8'
services:
  portfolio-web:
    image: nginx:alpine
    restart: unless-stopped
    volumes:
      - ./index.html:/usr/share/nginx/html/index.html
      - ./style.css:/usr/share/nginx/html/style.css
      - ./dag-resmi.jpg:/usr/share/nginx/html/dag-resmi.jpg
    networks:
      - proxy
networks:
  proxy:
    external: true
```

## 💡 Key Learnings & Takeaways

- Hands-on application of system administration and DevOps practices.
- Cloudflare Tunnel proves highly effective against common ISP limitations.
- Docker enables modular and manageable microservice architecture.
- Reverse proxy and SSL setup simplify secure multi-service deployment.
- The infrastructure is scalable for services like Gitea, PostgreSQL, Grafana, and more.
