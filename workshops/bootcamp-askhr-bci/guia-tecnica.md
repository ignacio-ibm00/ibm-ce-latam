# Bootcamp askHR — BCI — Guía Técnica

<!-- Orientada a AI Engineers que quieren replicar o extender la solución en otro cliente. -->

---

## Stack tecnológico

| Tecnología | Versión | Rol |
|---|---|---|
| IBM watsonx Orchestrate | SaaS | Plataforma del agente AskHR |
| OpenAPI / hr.yaml | 3.0 | Especificación de la API de RRHH mock |
| Node.js / JSON Server | 18.x | Servidor mock de la API HR (para workshop) |

---

## Prerrequisitos

- [ ] Acceso a IBM watsonx Orchestrate (instancia SaaS activa)
- [ ] Node.js 18+ (para correr el mock de la API HR)
- [ ] El archivo `hr.yaml` (incluido en el repo del workshop)
- [ ] El PDF `Employee-Benefits.pdf` (incluido en el repo del workshop)

---

## Estructura de archivos

```
[Workshop] Bootcamp askHR BCI/
├── README.md
├── hr.yaml                        # Especificación OpenAPI de la API de RRHH
├── Employee-Benefits.pdf          # Documento de políticas de beneficios BCI
└── instructions/
    ├── paso-1-crear-agente.md     # Instrucciones paso a paso para el workshop
    ├── paso-2-configurar-kb.md    # Configurar la Knowledge Base
    └── paso-3-conectar-api.md     # Conectar la API hr.yaml al agente
```

---

## Flujo del workshop (para instructores)

El bootcamp sigue este orden:

**Parte 1 — Agente con Knowledge Base (30 min)**
1. Crear agente `askhr` en watsonx Orchestrate
2. Subir `Employee-Benefits.pdf` como Knowledge Base
3. Probar preguntas básicas de políticas (vacaciones, beneficios)

**Parte 2 — Agente con API (30 min)**
4. Registrar `hr.yaml` como herramienta en watsonx Orchestrate
5. Configurar la URL base de la API mock (ver abajo)
6. Probar consultas personalizadas con el ID de empleado

**Parte 3 — Flujo combinado (15 min)**
7. Hacer que el agente combine KB + API en una sola respuesta
8. Validar con casos de prueba end-to-end

---

## Levantar la API mock localmente

```bash
# La API hr.yaml se puede mockear con json-server o con un servidor Express simple.
# Para el workshop, usamos el endpoint público provisto por CE LATAM.
# URL del mock para demos: https://hr-api-mock.ce-latam.ibm.com (pedir acceso al equipo)

# Para correr tu propia instancia local:
npx json-server --watch hr-mock-data.json --port 8080
# → API disponible en http://localhost:8080
```

### Endpoints de la API hr.yaml

| Endpoint | Método | Descripción |
|---|---|---|
| `/employees/{id}` | `GET` | Perfil completo del empleado |
| `/time-off/balance` | `GET` | Balance de días de vacaciones |
| `/time-off/request` | `POST` | Crear solicitud de time-off |
| `/benefits/{id}` | `GET` | Beneficios actuales del empleado |

---

## Configurar el agente en watsonx Orchestrate

### Crear el agente AskHR

1. En watsonx Orchestrate, ir a **AI agent builder** → **New agent**
2. Nombre: `askhr-bci`
3. System prompt sugerido:
   ```
   Eres el asistente de RRHH de BCI. Ayudás a los empleados con consultas sobre 
   beneficios, vacaciones y políticas. Sos amable, preciso y respondés en español.
   Si el empleado pregunta por sus datos personales, usá la herramienta HR API.
   Si pregunta por políticas generales, buscá en la Knowledge Base.
   ```

### Conectar la Knowledge Base

1. **Knowledge bases** → **New knowledge base** → Subir `Employee-Benefits.pdf`
2. Nombre: `kb-beneficios-bci`
3. Vincular la KB al agente `askhr-bci`

### Conectar la API HR

1. **Tools** → **New tool** → **OpenAPI**
2. Subir el archivo `hr.yaml`
3. URL base: URL del mock (local o public endpoint)
4. Vincular la tool al agente `askhr-bci`

---

## Datos de prueba para el workshop

| Campo | Valor | Descripción |
|---|---|---|
| Employee ID | `EMP-001` | ID de empleado de demo |
| Nombre | `María González` | Nombre ficticio para demos |
| Balance vacaciones | `15 días` | Balance del mock |
| Pregunta típica | `"¿Cuántos días de vacaciones me quedan?"` | Combina API + respuesta contextual |

---

## Troubleshooting

??? failure "watsonx Orchestrate no puede importar el hr.yaml"
    Verificá que el archivo es un OpenAPI válido 3.0. Usá [Swagger Editor](https://editor.swagger.io)
    para validar el YAML antes de importarlo.

??? failure "El agente responde solo con la KB y no llama a la API"
    Agregá en el system prompt: "Si el empleado pregunta por sus datos personales (balance, 
    perfil, historial), siempre usá la herramienta HR API, no la knowledge base."

??? failure "La Knowledge Base devuelve respuestas irrelevantes"
    Revisá que el PDF fue indexado correctamente: en la KB, verificá que el status es "Ready".
    Si el PDF tiene mucho texto formateado como tabla, puede necesitar reindexación.
