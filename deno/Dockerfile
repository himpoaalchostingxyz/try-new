FROM lscr.io/linuxserver/qbittorrent:latest

# Set web UI port (must match the tunnel target)
ENV WEBUI_PORT=8000
ENV TORRENTING_PORT=6881

# Install dependencies (curl for downloading cloudflared)
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*

# Detect architecture and download cloudflared
RUN ARCH=$(uname -m) && \
    if [ "$ARCH" = "x86_64" ]; then \
        CF_URL="https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64"; \
    elif [ "$ARCH" = "aarch64" ]; then \
        CF_URL="https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-arm64"; \
    else \
        echo "Unsupported architecture: $ARCH"; exit 1; \
    fi && \
    curl -L -o /usr/local/bin/cloudflared $CF_URL && \
    chmod +x /usr/local/bin/cloudflared

# Create a custom startup script that runs cloudflared in background,
# then starts the original s6 init (the base image's entrypoint)
RUN echo '#!/bin/bash\n\
echo "--- Starting Cloudflare Tunnel to http://localhost:8000 ---"\n\
/usr/local/bin/cloudflared tunnel --url http://localhost:8000 &\n\
CLOUDFLARED_PID=$!\n\
echo "Cloudflared PID: $CLOUDFLARED_PID"\n\
echo "--- Starting qBittorrent (s6 init) ---"\n\
exec /init\n\
' > /custom-init.sh && chmod +x /custom-init.sh

# Expose ports (web UI + torrenting)
EXPOSE 8000 6881 6881/udp

# Override the entrypoint to use our custom script
# The base image uses ENTRYPOINT ["/init"]; we replace it.
ENTRYPOINT ["/custom-init.sh"]
