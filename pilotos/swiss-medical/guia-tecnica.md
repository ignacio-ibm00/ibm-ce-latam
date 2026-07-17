# Swiss Medical — Guía Técnica

<!-- Orientada a AI Engineers que quieren replicar o extender la solución. -->

---

## Stack tecnológico

| Tecnología | Versión | Rol |
|---|---|---|
| IBM watsonx Orchestrate | SaaS | Plataforma del agente conversacional (voz + texto) |
| WhatsApp Business API | — | Canal de acceso para afiliados (voz + texto) |
| REST API Swiss Medical | — | Backend de afiliados, turnos y cartilla médica |
| Audio transcription tool | watsonx Orchestrate built-in | Transcripción de mensajes de voz de WhatsApp |

---

## Prerrequisitos

- [ ] Acceso a IBM watsonx Orchestrate (instancia SaaS activa, permisos de admin)
- [ ] Acceso a la API REST de Swiss Medical (URL + API key — pedir a Swiss Medical)
- [ ] WhatsApp Business Account con un número configurado para el webhook
- [ ] IBM Cloud CLI para deployment (opcional, si se usa Code Engine)

---

## Estructura de archivos

```
[Piloto] Swiss Medical/proyecto/
├── README.md
└── tools/                         # 17 herramientas del agente
    ├── agendarTurno/
    ├── cancelarTurno/
    ├── buscarPrestador/
    ├── buscarCentroPropio/
    ├── consultarCobertura/
    ├── consultarPrestacion/
    ├── cartillaGrupoFamiliar/
    ├── transcribirAudioWhatsapp/
    └── ...                        # 9 herramientas adicionales
```

---

## Variables de entorno

Cada herramienta requiere las credenciales de la API de Swiss Medical:

| Variable | Descripción | Herramientas afectadas |
|---|---|---|
| `SMG_API_URL` | URL base de la API de Swiss Medical | Todas las tools de consulta |
| `SMG_API_KEY` | API key de autorización | Todas las tools de consulta |
| `SMG_API_VERSION` | Versión de la API (`v1`, `v2`) | Todas las tools |
| `WA_WEBHOOK_SECRET` | Secret para validar webhooks de WhatsApp | `transcribirAudioWhatsapp` |

> ⚠️ Las credenciales de Swiss Medical son confidenciales y NDA. Para demos usá un mock local.

---

## Configuración del agente en watsonx Orchestrate

### Crear el agente desde cero

Swiss Medical no se importa desde un JSON exportado, sino que se configura manualmente por la cantidad de herramientas. Seguir estos pasos:

1. En watsonx Orchestrate, crear un nuevo agente: **AI agent builder** → **New agent**
2. Nombre: `swiss-medical-atencion`
3. Idioma: Español (Argentina)
4. Instrucciones del sistema: incluir el contexto de Swiss Medical, el tono de atención médica y las limitaciones (no diagnósticos, no emergencias)

### Crear e importar las 17 herramientas

Para cada herramienta en `tools/`:

1. Ir a **Tools** → **New tool** → **OpenAPI**
2. Subir el archivo `openapi.yaml` o `tool.json` de cada subdirectorio
3. Configurar la URL base y las credenciales de autenticación
4. Vincular la herramienta al agente

### Herramientas prioritarias para demo

Si estás configurando una demo rápida, priorizá estas 5:

| Herramienta | Función | Endpoint |
|---|---|---|
| `agendarTurno` | Agenda un turno médico | `POST /turnos` |
| `cancelarTurno` | Cancela un turno existente | `DELETE /turnos/{id}` |
| `buscarPrestador` | Busca médicos por especialidad y zona | `GET /prestadores` |
| `consultarCobertura` | Consulta qué cubre el plan del afiliado | `GET /cobertura/{afiliado_id}` |
| `transcribirAudioWhatsapp` | Convierte audio de WhatsApp a texto | Tool interna de watsonx |

### Integración con WhatsApp

1. Configurar el webhook de WhatsApp Business API apuntando al endpoint de watsonx Orchestrate
2. Verificar el webhook con el `WA_WEBHOOK_SECRET`
3. Configurar el flujo de entrada: los mensajes de audio deben pasar primero por `transcribirAudioWhatsapp`

→ Documentación: [WhatsApp Business API](https://developers.facebook.com/docs/whatsapp)

---

## Datos de prueba

| Campo | Valor de ejemplo | Descripción |
|---|---|---|
| Afiliado ID | `A-123456` | ID de afiliado de prueba |
| Especialidad | `CARDIOLOGIA`, `TRAUMATOLOGIA` | Especialidades disponibles en el mock |
| Plan | `PLATINUM`, `GOLD` | Planes de ejemplo para validar coberturas |
| Zona | `CABA`, `GBA NORTE` | Zonas de búsqueda de prestadores |

---

## Troubleshooting

??? failure "La herramienta de transcripción de audio falla"
    Verificá que el mensaje de WhatsApp es un audio en formato `.ogg` o `.mp4`.
    La herramienta `transcribirAudioWhatsapp` de watsonx Orchestrate no soporta formatos comprimidos.

??? failure "La API de Swiss Medical devuelve 403"
    El afiliado no tiene permisos para la operación solicitada (plan no cubre esa prestación).
    Para demos, usá el afiliado de prueba `A-123456` que tiene plan PLATINUM con cobertura completa.

??? failure "El agente responde en inglés"
    Verificá que las instrucciones del sistema incluyen explícitamente: "Responder siempre en español (Argentina)".
    En watsonx Orchestrate, esto se configura en el campo **System prompt** del agente.
