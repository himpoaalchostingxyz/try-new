FROM lscr.io/linuxserver/qbittorrent:latest

# Set internal web UI port (must match tunnel target)
ENV WEBUI_PORT=8000
ENV TORRENTING_PORT=6881

# Install curl (needed to download cloudflared)
RUN apt-get update && apt-get install -y --no-install-recommends curl && \
    rm -rf /var/lib/apt/lists/*

# Use TARGETARCH (set by buildx) to pick the correct cloudflared binary
# Falls back to amd64 if not set
ARG TARGETARCH
RUN case "${TARGETARCH}" in \
        amd64)  CF_URL="https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64" ;; \
        arm64)  CF_URL="https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-arm64" ;; \
        *)      echo "Unsupported architecture: ${TARGETARCH}"; exit 1 ;; \
    esac && \
    curl -L -o /usr/local/bin/cloudflared "$CF_URL" && \
    chmod +x /usr/local/bin/cloudflared

# Custom startup script: runs cloudflared in background, then starts the original s6 init
RUN printf '#!/bin/bash\n\
echo "Starting Cloudflare Tunnel to http://localhost:8000"\n\
/usr/local/bin/cloudflared tunnel --url http://localhost:8000 --no-autoupdate &\n\
CLOUDFLARED_PID=$!\n\
echo "Cloudflared PID: $CLOUDFLARED_PID"\n\
# Run the original entrypoint (s6 init)\n\
exec /init\n\
' > /custom-init.sh && chmod +x /custom-init.sh

# Expose ports (web UI + torrenting)
EXPOSE 8000 6881 6881/udp

# Override the base image’s entrypoint
ENTRYPOINT ["/custom-init.sh"]
