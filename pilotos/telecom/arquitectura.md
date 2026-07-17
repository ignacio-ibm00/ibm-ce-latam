# Telecom — Arquitectura de la Solución

> **Estado:** 🔄 En proceso — MVP en definición y construcción

## Diagrama de arquitectura — Visión general del MVP

```mermaid
flowchart TD
    subgraph FUENTES ["📡 Fuentes de datos — Telecom"]
        KAFKA_SRC["⚡ Eventos en tiempo real\nTransacciones de facturación\nAlarmas de red FMC"]
        HIST["🗄️ Datos históricos\nKPIs móviles · SLOs · SLIs\nMétricas de infraestructura VMs"]
    end

    subgraph IBM_CLOUD ["☁️ IBM Cloud"]
        EVENTSTREAMS["📨 IBM Event Streams\nKafka gestionado\nStreaming de eventos"]
        COS["🪣 IBM Cloud Object Storage\nAterrizado de datos\nen formato Parquet"]
    end

    subgraph IBM_WXD ["🧠 IBM watsonx.data intelligence"]
        direction TB
        META["🗂️ Metadata Import\nDetección automática\nde schema y assets"]
        ENRICH["✨ Metadata Enrichment con IA\nDescripciones · Business terms\nClasificación de datos"]
        QUALITY["✅ Data Quality\nProfiling · Checks automáticos\nSLO/SLI monitoring"]
        LINEAGE["🔗 Linaje de datos\nTrazabilidad end-to-end\nVMs e infraestructura"]
    end

    subgraph ANALYTICS ["📊 Consumo analítico"]
        DASH["📈 Dashboards\nKPIs móviles y alarmas FMC"]
        GOVERN["👁️ Gobierno de datos\nCalidad · Linaje · Términos\nde negocio"]
    end

    KAFKA_SRC -->|"Eventos de facturación\ny alarmas en tiempo real"| EVENTSTREAMS
    HIST -->|"Carga batch"| COS
    EVENTSTREAMS -->|"consumer_to_cos.py\nParquet"| COS
    COS --> META
    META --> ENRICH
    ENRICH --> QUALITY
    ENRICH --> LINEAGE
    QUALITY --> DASH
    LINEAGE --> GOVERN
```

---

## Caso 1 — KPI Móvil y Alarmas FMC

```mermaid
flowchart LR
    PROD["⚡ Productor de eventos\nAlarmas FMC\nKPIs móviles"]
    ES["📨 IBM Event Streams\nTopic: transacciones"]
    COS["🪣 IBM COS\nParquet / kafka-landing"]
    WXD["🧠 watsonx.data intelligence\nCatálogo + Calidad"]
    DASH["📈 Análisis de KPIs"]

    PROD --> ES --> COS --> WXD --> DASH
```

## Caso 2 — SLO/SLI y Linaje de VMs

```mermaid
flowchart LR
    VMS["🖥️ Métricas de VMs\ne infraestructura"]
    COS2["🪣 IBM COS\nDatos de SLO/SLI"]
    WXD2["🧠 watsonx.data intelligence\nLinaje end-to-end"]
    GOV["👁️ Gobierno\ny trazabilidad"]

    VMS --> COS2 --> WXD2 --> GOV
```

---

## Componentes clave

| Componente | Tecnología IBM | Rol en la solución |
|---|---|---|
| Ingesta en tiempo real | IBM Event Streams (Kafka) | Recibe eventos de facturación y alarmas de red en tiempo real |
| Almacenamiento | IBM Cloud Object Storage | Aterriza los datos como archivos Parquet para análisis |
| Catálogo y gobierno | IBM watsonx.data intelligence | Importa metadata, enriquece con IA y aplica gobierno de datos |
| Calidad de datos | IBM watsonx.data intelligence (DQ) | Monitoreo automático de SLOs/SLIs sobre los datos de red |
| Linaje | IBM watsonx.data intelligence (Lineage) | Trazabilidad completa del dato desde la VM hasta el dashboard |

---

## Flujo de datos

1. Los **eventos de red** (alarmas FMC, KPIs móviles) son publicados en tiempo real en **IBM Event Streams**
2. El pipeline Python (`consumer_to_cos.py`) consume los eventos y los aterriza como **Parquet en IBM COS**
3. **watsonx.data intelligence** detecta automáticamente los nuevos assets, importa el schema y enriquece la metadata con IA
4. Se aplican **checks de calidad automáticos** sobre los datos para monitorear el cumplimiento de SLOs y SLIs
5. El **linaje end-to-end** permite trazar el origen de cada métrica desde la infraestructura hasta los dashboards analíticos
