
# 🏠 Ev Sunucusu Projesi (Homelab with Docker & Cloudflare Tunnel)

Bu proje, standart bir masaüstü bilgisayarın, modern DevOps araçları kullanılarak, güvenli ve maliyetsiz bir geliştirme/barındırma sunucusuna dönüştürülmesi sürecini belgeler. Statik IP veya port yönlendirme gibi gereksinimler olmadan, ev ağı koşullarında dahi internete açık, güvenli servisler (web sitesi, Git sunucusu vb.) sunmayı amaçlar.

📌 Canlıya alınan örnek: **Portfolio sitesi** (`https://yusufturan.com.tr`)

## 🎯 Projenin Amaçları

- **Hosting Maliyetlerini Sıfırlamak:** Kişisel projeler için ücretsiz ve sürdürülebilir bir yayın ortamı sağlamak.
- **Büyük Veri Limitlerini Aşmak:** GitHub benzeri platformların depolama limitlerini aşmak.
- **Ağ Kısıtlamalarını Aşmak:** Statik IP eksikliği ve CG-NAT gibi ISS kısıtlamalarını aşmak.
- **Güvenliği En Üst Düzeye Çıkarmak:** Gerçek IP adresini gizleyerek ve HTTPS üzerinden tüm trafiği şifreleyerek güvenlik sağlamak.

## 🛠️ Kullanılan Teknoloji Yığını

| Kategori            | Teknoloji                    | Açıklama |
|---------------------|------------------------------|----------|
| İşletim Sistemi     | Ubuntu Server 22.04 LTS      | Kararlı ve güvenli Linux dağıtımı |
| Konteynerleştirme   | Docker & Docker Compose      | Taşınabilir servis yönetimi |
| Reverse Proxy       | Nginx Proxy Manager          | Gelişmiş yönlendirme ve SSL yönetimi |
| Güvenli Erişim      | Cloudflare Tunnel            | Port açmadan dış dünyaya bağlantı |
| DNS & Güvenlik      | Cloudflare                   | DNS, CDN ve DDoS koruma hizmeti |

## 🏗️ Mimari

```mermaid
graph TD
    A[Kullanıcı] -->|https://yusufturan.com.tr| B(Cloudflare Edge);
    B --> C{Cloudflare Tunnel};
    subgraph Ev Ağı (Origin Server - IP Gizli)
        C <--> D[cloudflared Konteyneri];
        D --> E[Nginx Proxy Manager Konteyneri];
        E -->|/| F[Portfolio Sitesi Konteyneri];
        E -->|git.domain.com| G[Gitea Konteyneri];
        E -->|app.domain.com| H[Diğer Projeler...];
    end

    style Ev Ağı fill:#f9f,stroke:#333,stroke-width:2px
```

## 🚀 Kurulum Adımları

### 1. Ortak Docker Ağı
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

### 4. Portfolio Sitesi (Statik)
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

## 💡 Sonuç ve Öğrenimler

- Gerçek bir projede modern sistem yönetimi ve DevOps süreçlerini uygulama fırsatı sunar.
- Cloudflare Tunnel, CG-NAT gibi ağ sınırlamalarına karşı etkili bir çözüm sunar.
- Docker ile mikroservis tabanlı mimari rahatlıkla yönetilebilir.
- Reverse proxy ve SSL yönetimi, tüm servislerin güvenli yayınlanmasını sağlar.
- Genişletilebilir yapı: PostgreSQL, Gitea, Grafana gibi yeni servislerle kolayca genişletilebilir.
