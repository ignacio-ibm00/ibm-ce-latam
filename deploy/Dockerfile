# Build de MkDocs + nginx para IBM Code Engine
# ⚠️ Siempre buildear con --platform linux/amd64 desde Mac Apple Silicon:
#    docker build --platform linux/amd64 -f deploy/Dockerfile -t us.icr.io/<namespace>/ce-latam-portfolio:latest .

FROM --platform=linux/amd64 python:3.12-slim AS builder

WORKDIR /build

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copiar TODO el repo para que MkDocs pueda resolver los docs/
COPY . .

# Buildear el sitio estático
RUN mkdocs build --strict

# ── Etapa de runtime: nginx sirve el sitio ─────────────────────────────────
FROM --platform=linux/amd64 nginx:alpine

# Copiar el sitio buildeado
COPY --from=builder /build/site /usr/share/nginx/html

# Config nginx para Code Engine (puerto 8080, index fallback)
COPY deploy/nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 8080

CMD ["nginx", "-g", "daemon off;"]
