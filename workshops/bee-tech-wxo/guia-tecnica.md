# Bee-tech wxO Bootcamp — Guía Técnica

<!-- Orientada a AI Engineers que quieren replicar el deployment del portal del bootcamp. -->

---

## Stack tecnológico

| Tecnología | Versión | Rol |
|---|---|---|
| IBM Code Engine | — | Hosting del portal web del bootcamp |
| IBM Container Registry | — | Registro de la imagen Docker |
| nginx | alpine | Servidor web estático del portal |
| Docker / Podman | 24.x+ | Build y push de la imagen |
| IBM watsonx Orchestrate | SaaS | Agentes del caso de uso Las Marías |

---

## Prerrequisitos

- [ ] Acceso a IBM Cloud con permisos en `Code Engine` y `Container Registry`
- [ ] Podman instalado localmente (o Docker)
- [ ] IBM Cloud CLI con plugins `code-engine` y `container-registry`
- [ ] IBM Cloud API key con acceso al namespace `ce-latam` en ICR

---

## Estructura del portal

```
[Workshop] Bee-tech wxO/
├── index.html                      # Página principal del portal
├── las-marias-guide.html           # Guía del caso de uso Las Marías
├── participant-guide.html          # Guía del participante
├── assets/
│   ├── css/                        # Estilos del portal
│   └── img/                        # Imágenes y logos
└── Dockerfile                      # Build con nginx:alpine
```

---

## Deployment del portal

### Build y push de la imagen

```bash
# 1. Login a IBM Container Registry
ibmcloud login --apikey <IBM_CLOUD_API_KEY> -r us-south
ibmcloud target -g Delitos-Economicos
ibmcloud cr login --client podman

# 2. Build de la imagen (desde el directorio del workshop)
cd "[Workshop] Bee-tech wxO"
podman build --platform linux/amd64 -t us.icr.io/ce-latam/beetech-portal:latest .

# 3. Push a ICR
podman push us.icr.io/ce-latam/beetech-portal:latest
```

### Crear/actualizar la app en Code Engine

```bash
ibmcloud ce project select --name ce-latam-docs

# Crear por primera vez:
ibmcloud ce app create \
  --name beetech-labs-app \
  --image us.icr.io/ce-latam/beetech-portal:latest \
  --registry-secret icr-secret \
  --port 8080 \
  --min-scale 0 --max-scale 2

# Actualizar (si ya existe):
ibmcloud ce app update \
  --name beetech-labs-app \
  --image us.icr.io/ce-latam/beetech-portal:latest
```

### Dockerfile del portal

```dockerfile
FROM nginx:alpine
COPY . /usr/share/nginx/html
EXPOSE 8080
```

> La configuración nginx usa el puerto 8080 (requerido por Code Engine).

---

## Configuración de agentes en watsonx Orchestrate

Los agentes de Las Marías se configuran directamente en la instancia de watsonx Orchestrate del bootcamp.
El instructor CE LATAM es responsable de tener la instancia activa antes del evento.

### Checklist pre-evento

- [ ] Instancia de watsonx Orchestrate activa y accesible
- [ ] Agentes de Las Marías importados y funcionando
- [ ] URL del portal actualizada en el `index.html`
- [ ] Guías del participante actualizadas con las instrucciones correctas
- [ ] Portal deployado en Code Engine y accesible públicamente

---

## URL del portal

🌐 **Portal activo:**  
[beetech-labs-app.2b6jhfm91b2v.us-south.codeengine.appdomain.cloud](https://beetech-labs-app.2b6jhfm91b2v.us-south.codeengine.appdomain.cloud/index.html)

---

## Datos de prueba

No aplica — el portal es un sitio estático. Las guías HTML contienen los datos de prueba
para el workshop embebidos directamente en el HTML.

---

## Troubleshooting

??? failure "El portal devuelve 502 Bad Gateway"
    La app de Code Engine puede estar en estado `scale-to-zero` (dormida por inactividad).
    El primer request puede tardar 10-20 segundos en despertar la app. Es normal.
    Si persiste, revisá el estado con: `ibmcloud ce app get --name beetech-labs-app`

??? failure "Los cambios en el HTML no se reflejan en el portal"
    Hay que rebuildar y repushear la imagen Docker. Code Engine no sirve archivos directamente del repo.
    Seguí los pasos de "Build y push de la imagen" y luego "Actualizar" la app.

??? failure "La imagen no se puede pushear a ICR"
    Verificá que el namespace `ce-latam` existe: `ibmcloud cr namespace-list`
    Si no existe: `ibmcloud cr namespace-add ce-latam`
