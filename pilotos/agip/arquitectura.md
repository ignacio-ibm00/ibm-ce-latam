# AGIP — Arquitectura de la Solución

## Diagrama de arquitectura

```mermaid
flowchart TD
    USER([👤 Contribuyente CABA])

    subgraph CANAL ["Canal de acceso"]
        CHAT["💬 Chat Embebido\n/ Web"]
    end

    subgraph IBM_WXO ["🧠 IBM watsonx Orchestrate"]
        direction TB
        ORCH["🤖 Agente Orquestador\nagip_orquestador\nPunto de entrada principal"]
        OPS["⚙️ Agente de Operaciones\nagip_operaciones\nLogin · Consulta · Pagos"]
        FAQ["📚 Agente de Consultas\nagip_consultas_frecuentes\nPreguntas informativas"]
        KB["📄 Knowledge Base\nABL · Inmobiliario · Patentes"]
    end

    subgraph MOCK_API ["🔌 API de AGIP\n(Node.js — IBM Code Engine)"]
        AUTH["🔐 Autenticación\nPOST /auth/login por CUIL"]
        DEUDA["🔍 Consulta de deuda\nABL · Patentes · Contribuyente"]
        PAGOS["💳 Pagos\nGenerar link · Enviar boleta"]
    end

    USER -->|"Consulta sobre impuestos"| CHAT
    CHAT --> ORCH
    ORCH -->|"Consulta informativa"| FAQ
    ORCH -->|"Operación transaccional"| OPS
    FAQ -->|"Busca en documentos"| KB
    OPS -->|"Login con CUIL"| AUTH
    OPS -->|"Consulta deuda"| DEUDA
    OPS -->|"Genera pago"| PAGOS
    PAGOS -->|"Link de pago / boleta"| USER
    DEUDA -->|"Detalle de deuda"| USER
```

---

## Componentes clave

| Componente | Tecnología IBM | Rol en la solución |
|---|---|---|
| Agente Orquestador | IBM watsonx Orchestrate | Punto de entrada único, detecta la intención y delega al agente correcto |
| Agente de Operaciones | IBM watsonx Orchestrate | Ejecuta el flujo transaccional: login, consulta de deuda y generación de pagos |
| Agente de Consultas Frecuentes | IBM watsonx Orchestrate | Responde preguntas informativas sobre impuestos, vencimientos y trámites |
| Knowledge Base | IBM watsonx Orchestrate (KB) | Documentos de referencia: Inmobiliario/ABL, Patentes Automotores |
| API AGIP | Node.js en IBM Code Engine | Mock de la API de AGIP con endpoints de autenticación, consulta y pagos |

---

## Flujo de datos

1. El **contribuyente** inicia una conversación desde el canal web o chat embebido
2. El **agente orquestador** recibe el mensaje, identifica la intención y delega: consultas informativas al agente de FAQ, operaciones al agente de operaciones
3. Para operaciones: el contribuyente se autentica con su **CUIL**, el agente consulta la deuda (ABL, patentes) llamando a la API de AGIP
4. El agente genera un **link de pago o boleta** y lo envía al contribuyente
5. Todas las preguntas informativas son respondidas usando los documentos de la **knowledge base**
