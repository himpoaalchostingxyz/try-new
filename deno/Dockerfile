FROM lscr.io/linuxserver/qbittorrent:latest

ENV WEBUI_PORT=8000
ENV TORRENTING_PORT=6881

# Install curl and ca-certificates using Alpine's apk
RUN apk add --no-cache curl ca-certificates

# Use TARGETARCH (set by buildx) to pick the correct cloudflared binary
ARG TARGETARCH
RUN case "${TARGETARCH}" in \
        amd64)  CF_URL="https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64" ;; \
        arm64)  CF_URL="https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-arm64" ;; \
        *)      echo "Unsupported arch: ${TARGETARCH}"; exit 1 ;; \
    esac && \
    echo "Downloading cloudflared from $CF_URL" && \
    curl --retry 5 --retry-delay 2 --fail -L -o /usr/local/bin/cloudflared "$CF_URL" && \
    chmod +x /usr/local/bin/cloudflared

# Custom startup script (Alpine uses /bin/sh)
RUN printf '#!/bin/sh\n\
echo "Starting Cloudflare Tunnel to http://localhost:8000"\n\
/usr/local/bin/cloudflared tunnel --url http://localhost:8000 --no-autoupdate --log-level info &\n\
sleep 2\n\
exec /init\n\
' > /custom-init.sh && chmod +x /custom-init.sh

EXPOSE 8000 6881 6881/udp

ENTRYPOINT ["/custom-init.sh"]
