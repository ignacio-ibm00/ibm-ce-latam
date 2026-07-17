# Telecom — Guía Técnica

<!-- Orientada a AI Engineers que quieren replicar o extender la solución. -->
<!-- Estado: MVP en construcción — esta guía documenta el estado actual del pipeline. -->

---

## Stack tecnológico

| Tecnología | Versión | Rol |
|---|---|---|
| IBM Event Streams | SaaS (Kafka) | Ingesta de eventos en tiempo real |
| IBM Cloud Object Storage | SaaS | Aterrizaje de datos en formato Parquet |
| IBM watsonx.data intelligence | SaaS | Gobierno de datos, calidad y linaje |
| Python | 3.11+ | Pipeline `consumer_to_cos.py` (Kafka → COS) |
| Apache Kafka Python client | `confluent-kafka` | Cliente Kafka para consumir eventos |
| PyArrow | 14.x | Escritura de archivos Parquet |

---

## Prerrequisitos

- [ ] Acceso a IBM Event Streams (instancia activa con topic `transacciones` creado)
- [ ] Acceso a IBM Cloud Object Storage (bucket `kafka-landing` creado)
- [ ] Acceso a IBM watsonx.data intelligence (instancia activa)
- [ ] Python 3.11+ instalado localmente
- [ ] IBM Cloud CLI con plugin `cos` instalado

---

## Estructura de archivos

```
[Piloto] Telecom/Technical_Docs.../
├── Kafka-demo/
│   ├── README.md                  # Documentación del pipeline Kafka → COS
│   ├── consumer_to_cos.py         # Script principal: consume Kafka y escribe Parquet en COS
│   ├── requirements.txt           # confluent-kafka, pyarrow, ibm-cos-sdk
│   └── .env.example               # Template de variables de entorno
├── Demo-APIs/
│   ├── README.md                  # Documentación de la demo con Streamlit
│   ├── app.py                     # App Streamlit para visualizar datos de watsonx.data
│   └── requirements.txt           # streamlit, ibm-watsonx-data
└── Material MVP/
    ├── PoC_Telecom.pptx           # Presentación del PoC
    ├── MVP_KPI_Movil_Alarmas.docx # Descripción del caso 1
    └── MVP_SLO_SLI_Lineage.docx   # Descripción del caso 2
```

---

## Variables de entorno

```bash
cd "[Piloto] Telecom/Technical_Docs.../Kafka-demo"
cp .env.example .env
```

### Pipeline Kafka → COS

| Variable | Descripción | Ejemplo |
|---|---|---|
| `KAFKA_BROKERS` | Bootstrap servers de IBM Event Streams | `broker-0.example.eventstreams.cloud.ibm.com:9093` |
| `KAFKA_API_KEY` | API key de IBM Event Streams | `abcdef123456...` |
| `KAFKA_TOPIC` | Topic a consumir | `transacciones` |
| `COS_ENDPOINT` | Endpoint regional de IBM COS | `https://s3.us-south.cloud-object-storage.appdomain.cloud` |
| `COS_API_KEY` | API key de IBM COS (con acceso al bucket) | `sk-...` |
| `COS_INSTANCE_CRN` | CRN de la instancia de COS | `crn:v1:bluemix:...` |
| `COS_BUCKET` | Nombre del bucket de destino | `kafka-landing` |
| `PARQUET_BATCH_SIZE` | Cantidad de mensajes por archivo Parquet | `1000` |

---

## Caso 1 — KPI Móvil y Alarmas FMC

### Levantar el pipeline localmente

```bash
# 1. Instalar dependencias
cd "[Piloto] Telecom/Technical_Docs.../Kafka-demo"
pip install -r requirements.txt

# 2. Configurar .env con credenciales de Event Streams y COS
cp .env.example .env

# 3. Ejecutar el consumer
python consumer_to_cos.py

# El script consume eventos del topic configurado y escribe archivos
# Parquet en el bucket de COS cada PARQUET_BATCH_SIZE mensajes.
```

### Schema de eventos esperado

```json
{
  "timestamp": "2024-11-20T10:30:00Z",
  "tipo": "ALARMA_FMC | KPI_MOVIL",
  "region": "CABA | GBA | NOA | NEA",
  "valor": 98.5,
  "unidad": "%",
  "servicio": "LTE | 5G | FMC",
  "alarma_id": "ALM-2024-001234"
}
```

---

## Caso 2 — SLO/SLI y Linaje de VMs

### Importar datos en watsonx.data intelligence

1. En watsonx.data intelligence, ir a **Metadata import** → **New import**
2. Seleccionar la conexión a IBM COS (bucket `kafka-landing`)
3. Seleccionar los schemas/paths con los datos de SLO/SLI
4. Ejecutar la importación → los assets quedan disponibles en el catálogo

### Configurar linaje

1. En watsonx.data intelligence, ir a **Lineage** → **New lineage connection**
2. Registrar las fuentes: VMs de infraestructura → COS → watsonx.data
3. El linaje end-to-end queda trazado automáticamente una vez configuradas las conexiones

### Ejecutar enrichment con IA

```bash
# Desde la UI de watsonx.data intelligence:
# 1. Metadata enrichment → seleccionar los assets importados
# 2. Activar: "Assign business terms" + "Classify data" + "Analyze quality"
# 3. Ejecutar → los assets quedan enriquecidos con términos, clasificaciones y score de calidad
```

---

## Datos de prueba

| Campo | Valor de ejemplo | Descripción |
|---|---|---|
| KPI | `disponibilidad_red_lte` | Nombre de KPI de demo |
| SLO target | `99.5%` | Objetivo de nivel de servicio |
| Período | `2024-Q4` | Trimestre de datos históricos |
| VM ID | `vm-telecom-prod-001` | ID de VM de infraestructura (anonimizado) |

---

## Troubleshooting

??? failure "consumer_to_cos.py falla con SSL error al conectar a Kafka"
    IBM Event Streams requiere SSL. Verificá que `KAFKA_SECURITY_PROTOCOL=SASL_SSL` y
    que `KAFKA_SASL_MECHANISM=PLAIN` están configurados correctamente en el `.env`.

??? failure "Los archivos Parquet no aparecen en el bucket de COS"
    Verificá que la API key de COS tiene el rol **Writer** sobre el bucket `kafka-landing`.
    En IBM Cloud → COS → Bucket → Permissions, revisá las políticas de acceso.

??? failure "watsonx.data intelligence no detecta los assets nuevos"
    Ejecutá un **Metadata import** manual apuntando al path del bucket donde se escribieron los Parquet.
    El import automático puede tener un delay de hasta 15 minutos.
