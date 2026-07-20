# Tech Summit Labs 2026

<div class="asset-header">
<div class="asset-meta">
  <span class="badge badge-completed">✔️ Completado</span>
  <span>🏆 Tech Summit Argentina 2026</span>
  <span>🤖 IBM Bob · watsonx</span>
  <span>🇦🇷 Argentina</span>
</div>
</div>

## Descripción del caso

El **Tech Summit Labs 2026** es el conjunto de laboratorios hands-on del Tech Summit Argentina 2026, el evento técnico más importante de IBM en la región. Los labs cubren tres workshops:

- **IBM Bob** — tres tracks: modernización de RPG (IBM i), modernización de Java a Liberty, y el track estrella: **Agentic Retail con Confluent y watsonx Orchestrate** (inventario en tiempo real + sistema multiagente + tienda React)
- **watsonx Orchestrate** — agentes conversacionales
- **watsonx.ai** — modelos y prompting

La infraestructura se basa en un **Workshop Hub** — un portal MkDocs Material con dos sitios diferenciados: uno público para los participantes y uno interno para los instructores — deployados de forma independiente en IBM Code Engine.

---

## One-Pager

<a href="#" class="download-btn" style="opacity:0.5;cursor:not-allowed;" title="Próximamente">
  📎 One-Pager — próximamente disponible
</a>

| Campo | Detalle |
|---|---|
| **Evento** | Tech Summit Argentina 2026 |
| **Audiencia** | Clientes, partners técnicos e ingenieros IBM |
| **Estado** | ✔️ Completado |
| **Productos IBM** | IBM Bob · IBM watsonx Orchestrate · IBM watsonx.ai · Confluent Kafka · IBM Code Engine |
| **Contacto CE** | Ignacio Ayerbe · Martina Pérez |

### El caso de uso
Proveer una experiencia de laboratorio hands-on de alta calidad para el Tech Summit 2026, con navegación unificada, identidad visual IBM, y separación clara entre contenido para participantes y pre-work para instructores.

El track más destacado es **Real-Time Agentic Retail**: tres labs secuenciales donde los participantes construyen un sistema completo de punta a punta — pipeline de inventario en Confluent Kafka (Lab 1), sistema multiagente en watsonx Orchestrate con MCP + RAG (Lab 2), y tienda web React con el asistente embebido construida con IBM Bob (Lab 3).

### Valor del workshop

- ✅ **Agentic Retail end-to-end** — Confluent Kafka → agentes wxO (MCP + RAG) → tienda React con asistente embebido
- ✅ **3 workshops, 3 tecnologías** — IBM Bob (RPG · Java · Agentic Retail), watsonx Orchestrate, watsonx.ai
- ✅ **Un repo, dos sitios** — Hub público e Instructors Guide interno con deployment independiente en Code Engine
- ✅ **Reusable** — taxonomía de labs (Solución → Grupo → Lab N) lista para futuros eventos

---

## Arquitectura de la solución

```mermaid
flowchart TD
    INSTRUCTOR([👨‍🏫 Instructor CE LATAM])
    PARTICIPANT([👩‍💻 Participante\nTech Summit 2026])

    subgraph REPO ["📁 Repositorio GitHub — Workshop-Hub"]
        HUB_CONTENT["📝 Labs para participantes\nhub/docs/"]
        INST_CONTENT["🔒 Pre-work instructores\ninstructors/docs/"]
        SHARED["🎨 Estilos IBM compartidos"]
    end

    subgraph BUILD ["⚙️ MkDocs Material Build"]
        HUB_BUILD["🌐 Workshop Hub\n(público)"]
        INST_BUILD["🔒 Instructors Guide\n(interno)"]
    end

    subgraph IBM_CLOUD ["☁️ IBM Cloud"]
        ICR["📦 IBM Container Registry"]
        HUB_APP["🚀 Code Engine\nHub — público"]
        INST_APP["🚀 Code Engine\nInstructors — interno"]
    end

    INSTRUCTOR --> INST_CONTENT
    INSTRUCTOR --> HUB_CONTENT
    SHARED --> HUB_BUILD & INST_BUILD
    HUB_CONTENT --> HUB_BUILD
    INST_CONTENT --> INST_BUILD
    HUB_BUILD & INST_BUILD --> ICR
    ICR --> HUB_APP & INST_APP
    PARTICIPANT --> HUB_APP
    INSTRUCTOR --> INST_APP
```

| Componente | Tecnología | Rol |
|---|---|---|
| Workshop Hub | MkDocs Material + IBM Code Engine | Portal público con los labs para participantes |
| Instructors Guide | MkDocs Material + IBM Code Engine | Portal interno con pre-work para instructores |
| Confluent Kafka + ksqlDB | Confluent Cloud | Pipeline de inventario en tiempo real (Lab 1 — Agentic Retail) |
| watsonx Orchestrate (4 agentes) | IBM watsonx Orchestrate | Sistema multiagente: SKU, sustitutos (RAG), supervisor, asistente cliente (Lab 2) |
| Voltia — tienda React | IBM Bob + React | Frontend construido con Bob con asistente wxO embebido (Lab 3) |
| IBM Container Registry | IBM Cloud (ICR) | Imágenes Docker de ambos sitios |

---

??? note "🔧 Guía técnica para engineers"

    **Stack:** MkDocs 1.6 · Material for MkDocs 9.5 · Python · Docker · nginx · IBM Code Engine

    **Levantar localmente:**
    ```bash
    pip install -r requirements.txt

    # Hub (participantes)
    mkdocs serve -f hub/mkdocs.yml -a localhost:8001

    # Instructors Guide (instructor)
    mkdocs serve -f instructors/mkdocs.yml -a localhost:8002
    ```

    **Agregar un nuevo lab:**

    1. Crear `hub/docs/<solución>/<grupo>/lab-NN-nombre/index.md`
    2. Linkear desde `hub/docs/<solución>/<grupo>/index.md`
    3. Crear el pre-work en `instructors/docs/<mismo-path>/index.md`

    → Ver `README.md` completo del Workshop-Hub para convenciones de contenido y deployment
