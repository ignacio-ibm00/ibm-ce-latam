# GCBA Tribunales — Guía Técnica

<!-- Orientada a AI Engineers que quieren replicar o extender la solución. -->

---

## Stack tecnológico

| Tecnología | Versión | Rol |
|---|---|---|
| IBM watsonx.ai | SaaS | Extracción de información y agente conversacional |
| Llama 3 / IBM Granite | Últimas versiones en SaaS | Modelo de lenguaje para comprensión de documentos legales |
| Python | 3.11+ | Scripts de procesamiento de documentos |
| IBM Cloud Object Storage | — | Almacenamiento de expedientes procesados |

---

## Prerrequisitos

- [ ] Acceso a IBM watsonx.ai (instancia SaaS activa con proyecto creado)
- [ ] Python 3.11+ instalado localmente
- [ ] IBM Cloud CLI (`ibmcloud`)
- [ ] API key de IBM Cloud con acceso al proyecto watsonx.ai
- [ ] Project ID del proyecto en watsonx.ai

---

## Estructura de archivos

```
[Piloto] GCBA Tribunales/proyecto/
├── README.md
├── extraction/
│   ├── extract_expediente.py      # Script de extracción de datos de expedientes PDF
│   ├── prompts/
│   │   ├── extract_caratula.txt   # Prompt para extracción de carátula
│   │   ├── extract_partes.txt     # Prompt para extracción de partes intervinientes
│   │   └── extract_fechas.txt     # Prompt para extracción de fechas clave
│   └── requirements.txt
├── agent/
│   ├── agent_config.json          # Configuración del agente conversacional
│   └── prompts/
│       └── system_prompt.txt      # System prompt del agente judicial
└── assets/
    └── expediente_ejemplo.pdf     # Expediente de ejemplo anonimizado
```

---

## Variables de entorno

```bash
cp .env.example .env
```

| Variable | Descripción | Ejemplo |
|---|---|---|
| `WXAI_API_KEY` | API key de IBM watsonx.ai | `sk-...` |
| `WXAI_PROJECT_ID` | ID del proyecto en watsonx.ai | `abc-123-def-456` |
| `WXAI_URL` | URL de la instancia (SaaS = Dallas) | `https://us-south.ml.cloud.ibm.com` |
| `WXAI_MODEL_ID` | ID del modelo a usar | `meta-llama/llama-3-3-70b-instruct` |

---

## Instalación local

```bash
# 1. Ir al directorio del proyecto
cd "[Piloto] GCBA Tribunales/proyecto/extraction"

# 2. Crear entorno virtual e instalar dependencias
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

# 3. Configurar variables de entorno
cp .env.example .env
# Editá .env con tus credenciales de watsonx.ai
```

---

## Uso — Extracción de datos de un expediente

```python
from extract_expediente import ExtractorExpediente

extractor = ExtractorExpediente(
    api_key="<WXAI_API_KEY>",
    project_id="<WXAI_PROJECT_ID>",
    model_id="meta-llama/llama-3-3-70b-instruct"
)

# Procesar un expediente PDF
resultado = extractor.procesar("assets/expediente_ejemplo.pdf")

print(resultado)
# {
#   "caratula": "Pérez, Juan c/ Municipalidad CABA",
#   "partes": ["Juan Pérez (actor)", "GCBA (demandado)"],
#   "fechas": {"inicio": "2023-03-15", "ultima_actuacion": "2024-11-20"},
#   "objeto": "Reclamo por daños y perjuicios"
# }
```

---

## Configuración del agente conversacional

El agente se configura directamente en IBM watsonx.ai:

1. Ir al proyecto en watsonx.ai
2. **Assets** → **New asset** → **AI agent**
3. Seleccionar el modelo (`llama-3-3-70b-instruct` o `granite-3-8b-instruct`)
4. Pegar el contenido de `agent/prompts/system_prompt.txt` como system prompt
5. Conectar el agente a la herramienta de búsqueda en el sistema judicial

---

## Datos de prueba

| Campo | Valor de ejemplo | Descripción |
|---|---|---|
| Número de expediente | `EXP-2024-001234-GCBA` | Formato de expediente de demo |
| Carátula | `"García c/ GCBA s/ amparo"` | Carátula de ejemplo anonimizada |
| Estado | `EN_TRAMITE`, `SENTENCIA`, `ARCHIVADO` | Estados posibles del expediente |

> El archivo `expediente_ejemplo.pdf` contiene un expediente completamente anonimizado y ficticio para pruebas.

---

## Troubleshooting

??? failure "Error 403 al llamar a la API de watsonx.ai"
    Verificá que el `WXAI_API_KEY` tiene permisos sobre el proyecto `WXAI_PROJECT_ID`.
    En IBM Cloud → IAM, revisá que la API key tiene el rol de **Editor** sobre el servicio Watson Machine Learning.

??? failure "El modelo extrae datos incorrectos del expediente"
    Ajustá el prompt en `prompts/extract_caratula.txt`. Los documentos judiciales en español de CABA
    tienen formatos específicos; el prompt incluye ejemplos few-shot que podés extender con más muestras.

??? failure "El PDF no se puede leer (texto no seleccionable)"
    El expediente fue escaneado como imagen. Necesitás agregar un paso de OCR antes de la extracción.
    Usá `pytesseract` con idioma español: `pytesseract.image_to_string(img, lang='spa')`.
