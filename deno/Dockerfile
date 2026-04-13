FROM lscr.io/linuxserver/qbittorrent:latest

# Set internal web UI port to 8000 (default is 8080)
ENV WEBUI_PORT=8000

# Set torrenting port (optional, but matches your compose)
ENV TORRENTING_PORT=6881

# Expose ports (same as in compose)
EXPOSE 8000
EXPOSE 6881
EXPOSE 6881/udp
