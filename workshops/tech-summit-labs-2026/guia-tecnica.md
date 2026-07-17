# Tech Summit Labs 2026 — Guía Técnica

<!-- Orientada a AI Engineers que quieren replicar el setup del Workshop-Hub para nuevos eventos. -->

---

## Stack tecnológico

| Tecnología | Versión | Rol |
|---|---|---|
| MkDocs Material | 9.5+ | Build de los sitios estáticos (Hub + Instructors) |
| IBM Code Engine | — | Hosting de los dos portales del evento |
| IBM Container Registry | — | Registro de las imágenes Docker |
| nginx | alpine | Servidor web estático de los sitios |
| Docker / Podman | 24.x+ | Build y push de las imágenes |
| GitHub Actions | — | CI/CD automático al pushear a main |

---

## Prerrequisitos

- [ ] Python 3.11+ con MkDocs Material instalado
- [ ] Acceso a IBM Cloud con permisos en Code Engine y Container Registry
- [ ] Podman instalado localmente
- [ ] IBM Cloud CLI con plugins `code-engine` y `container-registry`
- [ ] Acceso al repositorio de Workshop-Hub en GitHub

---

## Estructura del repositorio

```
Workshop-Hub/
├── mkdocs.yml                   # Config del Hub público (participantes)
├── mkdocs-instructors.yml       # Config del sitio de instructores (interno)
├── requirements.txt             # mkdocs, mkdocs-material, pymdown-extensions
├── hub/
│   └── docs/
│       ├── index.md             # Home del Workshop Hub
│       ├── bob/                 # Labs de IBM Bob
│       ├── orchestrate/         # Labs de watsonx Orchestrate
│       └── watsonx-ai/          # Labs de watsonx.ai
├── instructors/
│   └── docs/
│       ├── index.md             # Home de la guía de instructores
│       ├── pre-work.md          # Pre-work antes del evento
│       └── gotchas.md           # Problemas frecuentes y soluciones
├── assets/
│   └── stylesheets/
│       ├── ibm-brand.css        # Estilos IBM Brand compartidos
│       └── solutions.css        # Badges y estilos de labs
└── deploy/
    ├── Dockerfile.hub           # Build del Hub público
    └── Dockerfile.instructors   # Build del sitio de instructores
```

---

## Build y desarrollo local

```bash
# Instalar dependencias
pip install -r requirements.txt

# Levantar el Hub en modo desarrollo (con hot reload)
mkdocs serve -f mkdocs.yml
# → Disponible en http://localhost:8000

# Levantar el sitio de instructores en modo desarrollo
mkdocs serve -f mkdocs-instructors.yml -a localhost:8001
# → Disponible en http://localhost:8001

# Build para producción (valida que no haya links rotos)
mkdocs build -f mkdocs.yml --strict
mkdocs build -f mkdocs-instructors.yml --strict
```

---

## Añadir un nuevo lab

1. Crear el archivo Markdown en el directorio correspondiente:
   ```
   hub/docs/<solución>/lab-N-titulo.md
   ```

2. El archivo debe seguir la estructura:
   ```markdown
   # Lab N — Título del lab

   **Duración estimada:** X minutos  
   **Dificultad:** ⭐⭐☆☆☆

   ## Objetivo
   ...

   ## Paso 1 — ...
   ...
   ```

3. Agregar la entrada en `mkdocs.yml` bajo la sección correspondiente.

4. Hacer push a `main` → el CI/CD rebuilda y redeploya automáticamente.

---

## Deployment manual en IBM Code Engine

### Hub público

```bash
# Build de la imagen
podman build --platform linux/amd64 \
  -f deploy/Dockerfile.hub \
  -t us.icr.io/ce-latam/tech-summit-hub:latest .

# Push a ICR
ibmcloud cr login --client podman
podman push us.icr.io/ce-latam/tech-summit-hub:latest

# Deploy en Code Engine
ibmcloud ce project select --name ce-latam-docs
ibmcloud ce app update \
  --name tech-summit-hub \
  --image us.icr.io/ce-latam/tech-summit-hub:latest
```

### Sitio de instructores

```bash
podman build --platform linux/amd64 \
  -f deploy/Dockerfile.instructors \
  -t us.icr.io/ce-latam/tech-summit-instructors:latest .

podman push us.icr.io/ce-latam/tech-summit-instructors:latest

ibmcloud ce app update \
  --name tech-summit-instructors \
  --image us.icr.io/ce-latam/tech-summit-instructors:latest
```

---

## Checklist pre-evento

- [ ] Todos los labs tienen el contenido final (sin `TODO` ni placeholders)
- [ ] Las URLs de credenciales/accesos en `instructors/pre-work.md` están actualizadas
- [ ] El build con `--strict` pasa sin errores (no hay links rotos)
- [ ] Ambos portales deployados y accesibles en Code Engine
- [ ] Probado desde un browser en modo incógnito (sin caché)

---

## Troubleshooting

??? failure "mkdocs build --strict falla con 'link not found'"
    Un archivo Markdown referencia un link relativo que no existe.
    El error indica el archivo y línea exacta. Revisá los `[texto](ruta.md)` en ese archivo.

??? failure "Los estilos IBM Brand no se aplican"
    Verificá que `extra_css` en `mkdocs.yml` incluye `assets/stylesheets/ibm-brand.css`.
    Si usás el config de instructores, también revisá `mkdocs-instructors.yml`.

??? failure "Code Engine sirve el contenido viejo después del deploy"
    Code Engine puede cachear la imagen. Forzá un nuevo deploy con un tag nuevo:
    ```bash
    IMAGE_TAG=$(date +%Y%m%d%H%M%S)
    podman tag us.icr.io/ce-latam/tech-summit-hub:latest \
               us.icr.io/ce-latam/tech-summit-hub:$IMAGE_TAG
    podman push us.icr.io/ce-latam/tech-summit-hub:$IMAGE_TAG
    ibmcloud ce app update --name tech-summit-hub \
      --image us.icr.io/ce-latam/tech-summit-hub:$IMAGE_TAG
    ```
