FROM lscr.io/linuxserver/qbittorrent:latest

ENV WEBUI_PORT=8000
ENV TORRENTING_PORT=6881
ENV FILEBROWSER_PORT=8080

# Install curl, ca-certificates, and filebrowser (static binary)
RUN apk add --no-cache curl ca-certificates

# Download cloudflared (multi‑arch)
ARG TARGETARCH
RUN case "${TARGETARCH}" in \
        amd64)  CF_URL="https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64" ;; \
        arm64)  CF_URL="https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-arm64" ;; \
        *)      echo "Unsupported arch: ${TARGETARCH}"; exit 1 ;; \
    esac && \
    curl -L --retry 5 --fail -o /usr/local/bin/cloudflared "$CF_URL" && \
    chmod +x /usr/local/bin/cloudflared

# Download filebrowser (static binary, multi‑arch)
RUN case "${TARGETARCH}" in \
        amd64)  FB_URL="https://github.com/filebrowser/filebrowser/releases/latest/download/linux-amd64-filebrowser.tar.gz" ;; \
        arm64)  FB_URL="https://github.com/filebrowser/filebrowser/releases/latest/download/linux-arm64-filebrowser.tar.gz" ;; \
        *)      echo "Unsupported arch: ${TARGETARCH}"; exit 1 ;; \
    esac && \
    curl -L --retry 5 --fail "$FB_URL" | tar xz -C /usr/local/bin/ filebrowser && \
    chmod +x /usr/local/bin/filebrowser

# Create a database directory for filebrowser
RUN mkdir -p /config/filebrowser && \
    filebrowser config init --database=/config/filebrowser/filebrowser.db && \
    filebrowser config set --address=0.0.0.0 --port=$FILEBROWSER_PORT --root=/downloads --database=/config/filebrowser/filebrowser.db

# Create Cloudflare tunnel config with ingress rules
RUN mkdir -p /etc/cloudflared && \
    cat > /etc/cloudflared/config.yml << EOF
url: http://localhost:8000
tunnel: $(uuidgen)
credentials-file: /etc/cloudflared/credentials.json
ingress:
  - hostname: ${TUNNEL_HOSTNAME:-any}
    service: http://localhost:8000
  - hostname: ${TUNNEL_HOSTNAME:-any}
    path: /files/*
    service: http://localhost:$FILEBROWSER_PORT
  - service: http_status:404
EOF

# Startup script: run filebrowser, cloudflared with ingress, and the original init
RUN cat > /custom-init.sh << 'EOF'
#!/bin/sh
echo "Starting FileBrowser on port $FILEBROWSER_PORT (serving /downloads)"
/usr/local/bin/filebrowser --database=/config/filebrowser/filebrowser.db &

echo "Starting Cloudflare Tunnel with ingress rules"
/usr/local/bin/cloudflared tunnel --config /etc/cloudflared/config.yml --no-autoupdate &

sleep 3
exec /init
EOF
RUN chmod +x /custom-init.sh

EXPOSE 8000 6881 6881/udp 8080
ENTRYPOINT ["/custom-init.sh"]
