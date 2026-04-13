FROM lscr.io/linuxserver/qbittorrent:latest

ENV WEBUI_PORT=8000
ENV TORRENTING_PORT=6881
ENV FILEBROWSER_PORT=8080
ENV FILEBROWSER_ROOT=/app/deno/downloads

# Install curl, ca-certificates (Alpine)
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

# Download filebrowser (multi‑arch)
RUN case "${TARGETARCH}" in \
        amd64)  FB_URL="https://github.com/filebrowser/filebrowser/releases/latest/download/linux-amd64-filebrowser.tar.gz" ;; \
        arm64)  FB_URL="https://github.com/filebrowser/filebrowser/releases/latest/download/linux-arm64-filebrowser.tar.gz" ;; \
        *)      echo "Unsupported arch: ${TARGETARCH}"; exit 1 ;; \
    esac && \
    curl -L --retry 5 --fail "$FB_URL" | tar xz -C /usr/local/bin/ filebrowser && \
    chmod +x /usr/local/bin/filebrowser

# Create the downloads directory (if it doesn't exist)
RUN mkdir -p ${FILEBROWSER_ROOT}

# Initialize filebrowser database and set root to /app/deno/downloads
RUN mkdir -p /config/filebrowser && \
    filebrowser config init --database=/config/filebrowser/filebrowser.db && \
    filebrowser config set \
        --address=0.0.0.0 \
        --port=$FILEBROWSER_PORT \
        --root=$FILEBROWSER_ROOT \
        --database=/config/filebrowser/filebrowser.db

# Startup script: start filebrowser, then two cloudflared tunnels
RUN cat > /custom-init.sh << 'EOF'
#!/bin/sh

# Start FileBrowser
echo "Starting FileBrowser on port $FILEBROWSER_PORT (serving $FILEBROWSER_ROOT)"
/usr/local/bin/filebrowser --database=/config/filebrowser/filebrowser.db &

sleep 2

# Start Cloudflare tunnel for qBittorrent (port 8000)
echo "Starting Cloudflare tunnel for qBittorrent (port $WEBUI_PORT)..."
/usr/local/bin/cloudflared tunnel --url http://localhost:$WEBUI_PORT --no-autoupdate &

# Start Cloudflare tunnel for FileBrowser (port 8080)
echo "Starting Cloudflare tunnel for FileBrowser (port $FILEBROWSER_PORT)..."
/usr/local/bin/cloudflared tunnel --url http://localhost:$FILEBROWSER_PORT --no-autoupdate &

exec /init
EOF
RUN chmod +x /custom-init.sh

EXPOSE 8000 6881 6881/udp 8080
ENTRYPOINT ["/custom-init.sh"]
