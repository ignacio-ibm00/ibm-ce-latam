# Swiss Medical — Arquitectura de la Solución

## Diagrama de arquitectura

```mermaid
flowchart TD
    USER([👤 Afiliado Swiss Medical])

    subgraph CANALES ["Canales de acceso"]
        WA["📱 WhatsApp\n(voz + texto)"]
        WEB["🌐 Web / App"]
    end

    subgraph IBM_WXO ["🧠 IBM watsonx Orchestrate"]
        direction TB
        AGENT_VOZ["🎙️ Agente de Atención\nVoz y texto (español)\nFlucso de turnos y coberturas"]
        TOOLS["🔧 Herramientas (17 tools)\nCartilla · Turnos · Cobertura\nPrestadores · Servicios"]
    end

    subgraph TOOLS_DETAIL ["🔧 Herramientas disponibles"]
        direction LR
        T1["📅 Agenda/Cancela turno"]
        T2["🏥 Busca prestadores\ny centros propios"]
        T3["💊 Coberturas\ny prestaciones"]
        T4["👨‍👩‍👧 Cartilla\ngrupo familiar"]
        T5["🎤 Transcripción\naudio WhatsApp"]
    end

    subgraph SMG_SISTEMAS ["🏢 Sistemas Swiss Medical"]
        API_SMG["🔌 API Swiss Medical\nBackend de afiliados"]
        DB_TURN["🗓️ Sistema de turnos"]
        DB_CART["📋 Cartilla médica"]
    end

    USER -->|"Consulta o solicitud de turno\nvoz / texto"| WA
    USER -->|"Consulta desde web"| WEB
    WA -->|"Audio transcripto\n+ mensaje"| AGENT_VOZ
    WEB --> AGENT_VOZ
    AGENT_VOZ --> TOOLS
    TOOLS --> T1 & T2 & T3 & T4 & T5
    T1 -->|"Agenda / cancela"| DB_TURN
    T2 & T3 & T4 -->|"Consulta"| API_SMG
    API_SMG --> DB_CART
    DB_TURN -->|"Confirmación"| USER
    API_SMG -->|"Resultado"| USER
```

---

## Componentes clave

| Componente | Tecnología IBM | Rol en la solución |
|---|---|---|
| Agente de Atención | IBM watsonx Orchestrate | Agente conversacional de voz y texto para gestión completa del afiliado |
| 17 Herramientas (tools) | IBM watsonx Orchestrate (Tools) | Cobertura funcional completa: turnos, prestadores, cartilla, coberturas, servicios |
| Transcripción de audio | IBM watsonx Orchestrate | Convierte mensajes de voz de WhatsApp a texto para procesamiento |
| API Swiss Medical | Integración REST | Conecta con los sistemas del backend de la aseguradora |

---

## Flujo de datos

1. El **afiliado** inicia una conversación desde **WhatsApp** (voz o texto) o desde el portal web
2. Si es un mensaje de **voz**, la herramienta `transcribirAudioWhatsapp` convierte el audio a texto
3. El **agente de atención** interpreta la solicitud: turno, cobertura, prestador, o consulta de cartilla
4. El agente ejecuta la **herramienta correspondiente** de las 17 disponibles, que llama a la **API de Swiss Medical**
5. El afiliado recibe la confirmación del turno, el detalle de cobertura o la información del prestador directamente en el canal
