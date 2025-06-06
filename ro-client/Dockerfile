# Stage 1: Clone roBrowserLegacy
FROM node:18-alpine as builder

RUN apk add --no-cache git

# Clona o roBrowserLegacy
RUN git clone https://github.com/MrAntares/roBrowserLegacy.git /tmp/roBrowserLegacy

# Também clona o wsProxy
RUN git clone https://github.com/herenow/wsProxy.git /tmp/wsProxy && \
    sed -i 's/127\.0\.0\.1/rathena/' /tmp/wsProxy/index.js && \
    cd /tmp/wsProxy && npm install

# Stage 2: Final container com nginx + Node
FROM nginx:alpine
RUN apk add --no-cache nodejs npm bash

LABEL maintainer="Davi A. Wasserberg <davi.abreu.w@gmail.com>"
LABEL description="roBrowserLegacy + nginx + websocket proxy (wsProxy)"

# Copy only need input
COPY --from=builder /tmp/roBrowserLegacy/applications/window.html /usr/share/nginx/html/index.html

# Copia o roBrowserLegacy
COPY --from=builder /tmp/roBrowserLegacy /usr/share/nginx/html

# Copia o wsProxy
COPY --from=builder /tmp/wsProxy /app/wsProxy

# Override settings.js AFTER
COPY ./settings.js /usr/share/nginx/html/applications/settings.js

# Copia config do nginx padrão (você pode customizar depois)
RUN mkdir -p /run/nginx

# Exponha HTTP (80) e WebSocket Proxy (5999)
EXPOSE 80 5999

# Script de startup para rodar nginx + wsProxy juntos
COPY <<EOF /app/start.sh
#!/bin/bash

echo "Redirecionando logs do nginx para stdout/stderr..."
ln -sf /dev/stdout /var/log/nginx/access.log
ln -sf /dev/stderr /var/log/nginx/error.log
echo "Iniciando nginx..."
nginx &
echo "Iniciando wsProxy..."
node /app/wsProxy/index.js -p 5999
EOF

RUN chmod +x /app/start.sh

CMD ["/app/start.sh"]