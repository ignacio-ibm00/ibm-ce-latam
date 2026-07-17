# AGIP — Guía Técnica

<!-- Orientada a AI Engineers que quieren replicar o extender la solución. -->

---

## Stack tecnológico

| Tecnología | Versión | Rol |
|---|---|---|
| IBM watsonx Orchestrate | SaaS | Plataforma de agentes conversacionales |
| Node.js | 18.x | Runtime de la API mock de AGIP |
| IBM Code Engine | — | Hosting de la API mock en IBM Cloud |
| IBM Container Registry | — | Registro de imágenes Docker |

---

## Prerrequisitos

- [ ] Acceso a IBM watsonx Orchestrate (instancia SaaS activa, permisos de admin)
- [ ] Node.js 18+ instalado localmente
- [ ] Docker instalado (para build y deploy de la API)
- [ ] IBM Cloud CLI con plugins `code-engine` y `container-registry`
- [ ] Credenciales de IBM Cloud con acceso al namespace `ce-latam` en ICR

---

## Estructura de archivos

```
[Piloto] AGIP/agip/
├── README.md                        # Documentación técnica del proyecto
├── orchestrate/
│   └── agents/
│       ├── agip_orquestador.json    # Agente orquestador (punto de entrada)
│       ├── agip_operaciones.json    # Agente de operaciones transaccionales
│       └── agip_consultas_frecuentes.json  # Agente de FAQ
└── api/
    ├── server.js                    # Servidor Express — endpoints de AGIP mock
    ├── routes/
    │   ├── auth.js                  # POST /auth/login (CUIL)
    │   ├── deuda.js                 # GET /deuda/:cuil (ABL, patentes)
    │   └── pagos.js                 # POST /pagos/link, POST /pagos/boleta
    ├── package.json
    └── Dockerfile
```

---

## Variables de entorno

```bash
cp .env.example .env
```

| Variable | Descripción | Ejemplo |
|---|---|---|
| `PORT` | Puerto de la API mock | `8080` |
| `WXO_API_KEY` | API key de watsonx Orchestrate (para callbacks) | `sk-...` |
| `AGIP_MOCK_DELAY_MS` | Delay artificial en responses mock | `200` |

> ⚠️ Nunca commiteés credenciales reales. Pedí las credenciales de demo al equipo CE LATAM.

---

## Instalación local

```bash
# 1. Ir al directorio de la API
cd "[Piloto] AGIP/agip/api"

# 2. Instalar dependencias
npm install

# 3. Configurar variables de entorno
cp .env.example .env
# Editá .env con tus valores

# 4. Levantar la API mock
npm start
# → API disponible en http://localhost:8080
```

---

## Configuración de agentes en watsonx Orchestrate

### Importar los agentes

1. Accedé a tu instancia de IBM watsonx Orchestrate
2. Ir a **AI agent builder** → **Import agent**
3. Importar los tres archivos `.json` en este orden:
   - `agip_consultas_frecuentes.json` (sin dependencias)
   - `agip_operaciones.json` (requiere la URL de la API)
   - `agip_orquestador.json` (referencia a los otros dos agentes)

### Configurar la URL de la API

En el agente `agip_operaciones`, actualizá la URL base de las herramientas para apuntar a tu instancia de la API:

- **Local:** `http://localhost:8080`
- **Code Engine:** `https://<tu-app>.us-south.codeengine.appdomain.cloud`

### Configurar la Knowledge Base

1. En watsonx Orchestrate, ir a **Knowledge bases** → **New knowledge base**
2. Subir los documentos de referencia: ABL/Inmobiliario, Patentes Automotores
3. Vincular la KB al agente `agip_consultas_frecuentes`

→ Documentación oficial: [IBM watsonx Orchestrate Docs](https://www.ibm.com/docs/en/watson-orchestrate)

---

## Deployment de la API en IBM Code Engine

```bash
# 1. Build de la imagen (desde el directorio de la API)
cd "[Piloto] AGIP/agip/api"
podman build --platform linux/amd64 -t us.icr.io/ce-latam/agip-api:latest .

# 2. Login a IBM Container Registry
ibmcloud cr login --client podman

# 3. Push de la imagen
podman push us.icr.io/ce-latam/agip-api:latest

# 4. Crear/actualizar la app en Code Engine
ibmcloud ce project select --name ce-latam-docs
ibmcloud ce app create \
  --name agip-api \
  --image us.icr.io/ce-latam/agip-api:latest \
  --registry-secret icr-secret \
  --port 8080 \
  --min-scale 0 --max-scale 2
```

---

## Datos de prueba

| Campo | Valor de ejemplo | Descripción |
|---|---|---|
| CUIL | `20-12345678-9` | CUIL de usuario de demo (no real) |
| Tipo de impuesto | `ABL`, `PATENTES`, `INMOBILIARIO` | Tipos disponibles en el mock |
| Estado de deuda | `AL_DIA`, `VENCIDA`, `EN_GESTION` | Estados posibles en la API mock |

---

## Troubleshooting

??? failure "Error 401 al llamar a los endpoints de la API"
    Verificá que el header `Authorization: Bearer <token>` esté presente.
    En el mock, el token se genera con `POST /auth/login` usando el CUIL de prueba.

??? failure "El agente orquestador no delega al agente correcto"
    Revisá las instrucciones del agente orquestador. Las palabras clave para delegación son:
    - Consultas informativas → `agip_consultas_frecuentes`
    - Operaciones (login, deuda, pago) → `agip_operaciones`

??? failure "La KB no responde con información de ABL"
    Verificá que los documentos fueron indexados correctamente. En watsonx Orchestrate,
    ir a la KB y revisar el estado de indexación de cada documento.
