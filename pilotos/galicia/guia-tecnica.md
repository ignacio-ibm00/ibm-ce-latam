# Galicia — Guía Técnica

<!-- Orientada a AI Engineers que quieren replicar o extender la solución. -->

---

## Stack tecnológico

| Tecnología | Versión | Rol |
|---|---|---|
| IBM watsonx Orchestrate | SaaS | Plataforma de agentes conversacionales |
| Microsoft Teams | — | Canal de acceso para empleados del banco |
| Elastic Search | 8.x | Búsqueda de tickets históricos |
| ServiceNow | — | Gestión de incidentes ITSM |
| IBM watsonx Orchestrate (KB) | SaaS | Knowledge base con documentación interna |

---

## Prerrequisitos

- [ ] Acceso a IBM watsonx Orchestrate (instancia SaaS activa, permisos de admin)
- [ ] Acceso a Microsoft Teams con permisos para integrar apps
- [ ] Instancia de Elastic Search accesible (URL + credenciales)
- [ ] Instancia de ServiceNow accesible (URL + API token)
- [ ] Credenciales de IBM Cloud con acceso al proyecto de watsonx Orchestrate

---

## Estructura de archivos

```
[Piloto] Galicia/proyecto/
├── README.md
└── agents/
    └── native/
        ├── orquestador.json              # Agente clasificador de intención
        ├── agent_elastic_search.json     # Agente de búsqueda de tickets
        └── agent_service_now.json        # Agente de creación de incidentes
```

---

## Variables de entorno

Las herramientas de los agentes requieren las siguientes configuraciones:

| Variable | Descripción | Dónde configurarla |
|---|---|---|
| `ELASTIC_URL` | URL base de la instancia Elastic Search | Tool `buscarTicket` en wxO |
| `ELASTIC_API_KEY` | API key de Elastic Search | Tool `buscarTicket` en wxO |
| `ELASTIC_INDEX` | Nombre del índice de tickets | Tool `buscarTicket` en wxO |
| `SNOW_URL` | URL base de ServiceNow (`https://<instancia>.service-now.com`) | Tool `createIncident` en wxO |
| `SNOW_TOKEN` | Bearer token de ServiceNow | Tool `createIncident` en wxO |

> ⚠️ Las credenciales de Galicia son confidenciales. Para demos usá instancias de prueba propias.

---

## Configuración de agentes en watsonx Orchestrate

### Importar los agentes

1. Accedé a tu instancia de IBM watsonx Orchestrate
2. Ir a **AI agent builder** → **Import agent**
3. Importar en este orden:
   - `agent_elastic_search.json`
   - `agent_service_now.json`
   - `orquestador.json`

### Configurar herramientas con credenciales propias

Cada agente usa herramientas con URLs hardcodeadas en el JSON de exportación. Después de importar:

1. En el agente `agent_elastic_search`, editar la herramienta `buscarTicket` y actualizar la URL y API key de Elastic
2. En el agente `agent_service_now`, editar la herramienta `createIncident` y actualizar la URL y token de ServiceNow

### Configurar la Knowledge Base

1. En watsonx Orchestrate, ir a **Knowledge bases** → **New knowledge base**
2. Subir la documentación interna de soporte IT (procedimientos, escalation paths)
3. Vincular la KB al agente orquestador

### Integración con Microsoft Teams

1. En Teams, ir a **Apps** → **Manage your apps** → **Upload a custom app**
2. En watsonx Orchestrate, obtener el manifest de la integración Teams desde **Settings → Integrations → Microsoft Teams**
3. Configurar el webhook con la URL pública del agente orquestador

→ Documentación oficial: [watsonx Orchestrate + Teams](https://www.ibm.com/docs/en/watson-orchestrate?topic=integrations-microsoft-teams)

---

## Flujo de configuración end-to-end

```bash
# 1. Importar agentes (ver pasos arriba)
# 2. Configurar credenciales en cada herramienta
# 3. Probar el agente de Elastic con una búsqueda de ticket de prueba
# 4. Probar el agente de ServiceNow con la creación de un ticket de prueba
# 5. Activar la integración Teams
# 6. Validar el flujo completo desde Teams
```

---

## Datos de prueba

| Campo | Valor de ejemplo | Descripción |
|---|---|---|
| Ticket ID | `INC0001234` | Número de incidente en ServiceNow (formato demo) |
| Query de búsqueda | `"error VPN acceso remoto"` | Búsqueda de ejemplo en Elastic |
| Descripción de incidente | `"No puedo acceder al sistema de home banking"` | Descripción de prueba |

---

## Troubleshooting

??? failure "El agente de Elastic no encuentra tickets"
    Verificá que el índice configurado en la herramienta existe y tiene datos.
    En Elastic, correr: `GET /<índice>/_count` para validar que hay documentos indexados.

??? failure "ServiceNow devuelve 401"
    El token de ServiceNow puede haber expirado. Generá uno nuevo en:
    ServiceNow → **System OAuth** → **Application Registry**.

??? failure "La integración Teams no aparece en la lista"
    Verificá que el tenant de Teams tiene habilitada la instalación de apps personalizadas
    (requiere permisos de administrador de Teams).
