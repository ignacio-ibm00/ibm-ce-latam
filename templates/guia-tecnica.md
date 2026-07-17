# [Nombre del Proyecto] — Guía Técnica

<!-- Esta guía está orientada a AI Engineers que quieren replicar o extender la solución.
     Asumí que el lector tiene conocimiento técnico pero no conoce el proyecto. -->

---

## Stack tecnológico

| Tecnología | Versión | Rol |
|---|---|---|
| <!-- IBM watsonx Orchestrate --> | <!-- SaaS / On-prem --> | <!-- Plataforma de agentes --> |
| <!-- Node.js / Python --> | <!-- 18.x / 3.11 --> | <!-- Runtime de la API mock / scripts --> |
| <!-- Docker --> | <!-- 24.x --> | <!-- Containerización para deployment --> |
| <!-- IBM Code Engine --> | <!-- - --> | <!-- Deployment en nube IBM --> |

---

## Prerrequisitos

<!-- Listá todo lo que el engineer necesita tener antes de empezar -->

- [ ] Acceso a IBM Cloud / watsonx con las entitlements necesarias
- [ ] <!-- IBM watsonx Orchestrate — instancia activa con permisos de admin -->
- [ ] <!-- Node.js 18+ / Python 3.11+ instalado localmente -->
- [ ] <!-- Docker instalado (para deployment local) -->
- [ ] <!-- Credenciales de la API del cliente (pedir al equipo CE LATAM) -->

---

## Estructura de archivos

```
nombre-proyecto/
<!-- Copiá aquí la estructura real del proyecto con tree o a mano -->
├── README.md
├── api/
│   ├── server.js          # API mock
│   ├── routes/
│   └── Dockerfile
├── orchestrate/
│   ├── agents/            # Exportaciones de agentes
│   └── knowledge_bases/   # Documentos de la KB
└── assets/
```

---

## Variables de entorno

<!-- Documentá todas las variables necesarias. Nunca incluyas valores reales. -->

Copiá `.env.example` a `.env` y completá los valores:

```bash
cp .env.example .env
```

| Variable | Descripción | Ejemplo |
|---|---|---|
| `WXAI_API_KEY` | API key de IBM watsonx.ai | `sk-...` |
| `WXAI_PROJECT_ID` | ID del proyecto en watsonx.ai | `abc-123-...` |
| `PORT` | Puerto de la API mock | `8080` |
| <!-- VARIABLE --> | <!-- Descripción --> | <!-- ejemplo --> |

---

## Instalación local

```bash
# 1. Clonar el repo
git clone https://github.com/ibm-ce-latam/ce-latam-ai-portfolio.git
cd ce-latam-ai-portfolio/<categoria>/<nombre-proyecto>

# 2. Instalar dependencias
<!-- npm install -->
<!-- pip install -r requirements.txt -->

# 3. Configurar variables de entorno
cp .env.example .env
# Editá .env con tus credenciales

# 4. Levantar el servicio
<!-- npm start -->
<!-- python app.py -->
```

---

## Ejecución

### Desarrollo local

```bash
# Levantá el servidor de desarrollo
<!-- npm run dev -->
<!-- python app.py -->

# La API queda disponible en:
# http://localhost:8080
```

### Con Docker

```bash
cd <directorio-con-Dockerfile>
docker build -t nombre-proyecto .
docker run -p 8080:8080 --env-file .env nombre-proyecto
```

---

## Configuración de agentes en watsonx Orchestrate

<!-- Describí los pasos para importar o configurar los agentes -->

1. Importar los agentes desde `orchestrate/agents/`
2. Subir los documentos de `orchestrate/knowledge_bases/` a la KB
3. Configurar las herramientas con la URL de la API

→ Ver documentación oficial: [IBM watsonx Orchestrate Docs](https://www.ibm.com/docs/en/watson-orchestrate)

---

## Deployment en IBM Code Engine

<!-- Pasos específicos para deployar en producción / demo -->

```bash
# Build y push a IBM Container Registry
docker build -t <region>.icr.io/<namespace>/nombre-proyecto:latest .
docker push <region>.icr.io/<namespace>/nombre-proyecto:latest
```

Luego crear la aplicación en Code Engine apuntando a la imagen.  
El contenedor escucha en el puerto `8080`.

---

## Datos de prueba

<!-- Si el proyecto tiene datos de ejemplo, documentalos aquí (nunca datos reales) -->

| Campo | Valor de ejemplo | Descripción |
|---|---|---|
| <!-- Usuario test --> | <!-- test@example.com --> | <!-- Usuario de demostración --> |

---

## Troubleshooting

<!-- Problemas frecuentes y sus soluciones -->

??? failure "Error de autenticación con IBM Cloud"
    Verificá que `WXAI_API_KEY` sea válida y tenga los permisos necesarios.
    Generá una nueva API key en [IBM Cloud IAM](https://cloud.ibm.com/iam/apikeys).

??? failure "La API no responde en puerto 8080"
    Revisá que el puerto no esté ocupado: `lsof -i :8080`
    Cambiá el puerto con la variable `PORT=8081`.

<!-- Agregá más casos según los problemas reales del proyecto -->
