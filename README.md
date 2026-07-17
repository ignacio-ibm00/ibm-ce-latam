# CE LATAM AI Portfolio

> Repositorio privado del equipo de **Client Engineering LATAM** — IBM.  
> Centraliza los assets de IA (pilotos, MVPs y workshops) desarrollados para clientes de la región.

**Equipo:** Ignacio Ayerbe · Martina Pérez  
**Portal de documentación:** [ver el portal](https://ce-latam-portfolio.2cf01a0x2smr.us-south.codeengine.appdomain.cloud) *(disponible una vez deployado)*

---

## 📁 Estructura del repositorio

```
ce-latam-ai-portfolio/
├── pilotos/                  # Proyectos con cliente — PoC, MVP, Piloto
│   ├── agip/
│   ├── galicia/
│   ├── swiss-medical/
│   ├── gcba-tribunales/
│   └── telecom/              # En proceso
├── workshops/                # Bootcamps, labs y eventos
│   ├── bootcamp-askhr-bci/
│   ├── bee-tech-wxo/
│   └── tech-summit-labs-2026/
├── templates/                # Plantillas de documentación
├── docs/                     # Fuente del portal MkDocs
└── .github/                  # Workflows y PR template
```

---

## 🗂 Assets por categoría

### Pilotos / MVPs

| Proyecto | Cliente / Contexto | Estado | Productos IBM |
|---|---|---|---|
| [AGIP](pilotos/agip/) | Gobierno CABA — Gestión tributaria | ✅ Activo | watsonx Orchestrate |
| [Galicia](pilotos/galicia/) | Banco Galicia — Soporte IT interno | ✅ Activo | watsonx Orchestrate |
| [Swiss Medical](pilotos/swiss-medical/) | Swiss Medical — Atención al afiliado | ✅ Activo | watsonx Orchestrate |
| [GCBA Tribunales](pilotos/gcba-tribunales/) | Gobierno CABA — Justicia digital | ✅ Activo | watsonx.ai |
| [Telecom](pilotos/telecom/) | Telecom Argentina — Calidad de datos | 🔄 En proceso | watsonx.data intelligence |

### Workshops

| Proyecto | Evento / Contexto | Productos IBM |
|---|---|---|
| [Bootcamp askHR — BCI](workshops/bootcamp-askhr-bci/) | Bootcamp Agentic BCI | watsonx Orchestrate |
| [Bee-tech wxO Bootcamp](workshops/bee-tech-wxo/) | Bootcamp Bee-tech 2026 | watsonx Orchestrate |
| [Tech Summit Labs 2026](workshops/tech-summit-labs-2026/) | Tech Summit Argentina 2026 | IBM Bob, watsonx |

---

## 🚀 Cómo contribuir

Leé [`CONTRIBUTING.md`](CONTRIBUTING.md) antes de agregar un nuevo asset.

**Documentos obligatorios por proyecto:**
- `README.md` — índice del proyecto
- `one-pager.md` — resumen ejecutivo
- `arquitectura.md` — diagrama de arquitectura con Mermaid

---

## 🛠 Portal de documentación local

```bash
# Instalar dependencias
pip install -r requirements.txt

# Levantar el portal
mkdocs serve
```

Accedé en `http://localhost:8000`

## 🚀 Deploy en IBM Code Engine

El portal se deploya automáticamente en IBM Code Engine en cada push a `main`.

Para el primer deploy manual, seguí [`DEPLOY.md`](DEPLOY.md).

---

## 📋 Convenciones

- **Nombres de carpetas**: `kebab-case` sin espacios ni caracteres especiales
- **Idioma**: español en todo el contenido del portal
- **Videos**: nunca en el repo → links a Box en `one-pager.md`
- **Datos sensibles**: nunca en el repo → usar datos de ejemplo anonimizados
- **Credenciales**: siempre en `.env` (ignorado por `.gitignore`)
