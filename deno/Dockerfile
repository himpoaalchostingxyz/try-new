FROM ubuntu:22.04

ENV DEBIAN_FRONTEND=noninteractive

# Install everything
RUN apt-get update && apt-get install -y \
    xfce4 xfce4-goodies \
    dbus-x11 \
    x11vnc \
    xvfb \
    novnc websockify \
    curl wget \
    net-tools \
    libgl1-mesa-dri \
    libgl1-mesa-glx \
    mesa-utils \
    libxcb-xinerama0 \
    libxkbcommon-x11-0 \
    chromium-browser \
    && apt-get clean

# Install RustDesk (auto detect arch)
RUN ARCH=$(uname -m) && \
    if [ "$ARCH" = "aarch64" ]; then \
        curl -LO https://github.com/rustdesk/rustdesk/releases/download/1.4.6/rustdesk-1.4.6-aarch64.deb; \
    else \
        curl -LO https://github.com/rustdesk/rustdesk/releases/download/1.4.6/rustdesk-1.4.6-x86_64.deb; \
    fi && \
    apt-get install -y ./rustdesk-*.deb && \
    rm rustdesk-*.deb

# Create startup script INSIDE Dockerfile
RUN echo '#!/bin/bash\n\
set +e\n\
\n\
echo "--- Cleanup ---"\n\
pkill -f websockify || true\n\
pkill -f x11vnc || true\n\
pkill -f Xvfb || true\n\
rm -rf /tmp/.X11-unix\n\
\n\
echo "--- Start Display ---"\n\
export DISPLAY=:1\n\
export LIBGL_ALWAYS_SOFTWARE=1\n\
export MESA_GL_VERSION_OVERRIDE=3.3\n\
export QT_XCB_GL_INTEGRATION=none\n\
Xvfb :1 -screen 0 1280x800x24 &\n\
sleep 2\n\
\n\
echo "--- Start XFCE ---"\n\
dbus-run-session -- startxfce4 &\n\
sleep 5\n\
\n\
echo "--- Desktop shortcuts ---"\n\
mkdir -p ~/Desktop\n\
\n\
echo "[Desktop Entry]\n\
Name=Chromium\n\
Exec=chromium-browser --no-sandbox --disable-gpu\n\
Type=Application" > ~/Desktop/Chromium.desktop\n\
\n\
echo "[Desktop Entry]\n\
Name=RustDesk\n\
Exec=env DISPLAY=:1 LIBGL_ALWAYS_SOFTWARE=1 MESA_GL_VERSION_OVERRIDE=3.3 QT_XCB_GL_INTEGRATION=none rustdesk\n\
Type=Application" > ~/Desktop/RustDesk.desktop\n\
\n\
chmod +x ~/Desktop/*.desktop\n\
\n\
echo "--- Start VNC ---"\n\
x11vnc -display :1 -forever -shared -rfbport 5900 -nopw -bg -listen 127.0.0.1\n\
ln -sf /usr/share/novnc/vnc.html /usr/share/novnc/index.html\n\
websockify --web=/usr/share/novnc/ 8000 localhost:5900 &\n\
\n\
echo "Ready on port 8000"\n\
\n\
while true; do sleep 1000; done\n\
' > /start.sh && chmod +x /start.sh

EXPOSE 8000

CMD ["/start.sh"]