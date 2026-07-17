# Guía de contribución

Gracias por querer agregar un asset al portfolio. Esta guía describe los pasos y reglas para mantener la calidad y consistencia del repositorio.

---

## Antes de empezar

1. Cloná el repo:
   ```bash
   git clone https://github.com/ibm-ce-latam/ce-latam-ai-portfolio.git
   cd ce-latam-ai-portfolio
   ```

2. Creá una rama con el nombre del proyecto:
   ```bash
   git checkout -b add/<categoria>/<nombre-proyecto>
   # Ejemplos:
   # git checkout -b add/piloto/nuevo-cliente
   # git checkout -b add/workshop/bootcamp-nombre
   ```

---

## Estructura de un nuevo asset

Copiá la plantilla base desde `templates/`:

```bash
# Para un piloto/MVP
cp -r templates/ pilotos/nombre-cliente/

# Para un workshop
cp -r templates/ workshops/nombre-evento/
```

Cada asset **debe** tener esta estructura mínima:

```
nombre-proyecto/
├── README.md          ✅ obligatorio — índice del proyecto
├── one-pager.md       ✅ obligatorio — resumen ejecutivo
├── arquitectura.md    ✅ obligatorio — diagrama Mermaid + componentes
├── caso-de-uso.md     📋 recomendado — descripción del caso
├── guia-tecnica.md    📋 recomendado — para AI Engineers
└── assets/            📁 imágenes, capturas, diagramas exportados
```

---

## Categorías: ¿cuándo usar cada una?

| Categoría | Carpeta | Cuándo usarla |
|---|---|---|
| **Piloto / MVP** | `pilotos/` | Proyecto con cliente real — PoC, MVP o piloto en curso o finalizado |
| **Workshop** | `workshops/` | Bootcamp, lab de evento, hands-on para partners o clientes |

---

## Reglas de contenido

### ✅ Sí se puede subir
- Código fuente y configuraciones sin credenciales
- Datos de ejemplo anonimizados
- Diagramas de arquitectura (Mermaid o PNG/SVG en `assets/`)
- Screenshots de interfaces (sin datos reales de clientes)
- Documentación técnica y funcional

### ❌ Nunca subir
- Videos (`.mp4`, `.mov`) → linkeá a Box en `one-pager.md`
- Archivos `.env` con credenciales reales
- Datos reales de clientes (emails, CUILs, expedientes, registros médicos)
- Archivos de precios o contratos (`Pricing and Packaging`)
- Documentos marcados como confidenciales

---

## Proceso de PR

1. Completá los archivos obligatorios usando las plantillas de `templates/`
2. Agregá el proyecto al índice en `README.md` raíz
3. Abrí un Pull Request hacia `main`
4. El PR template te va a pedir completar el checklist de documentación
5. Al menos uno de los autores principales debe aprobar antes de hacer merge

### Protección de la rama `main`

- `main` es la rama que alimenta el portal de documentación
- **No hay push directo a `main`** — todo pasa por PR
- El workflow de GitHub Actions buildea y publica el portal automáticamente en cada merge a `main`

---

## Preguntas frecuentes

**¿Puedo subir el código fuente del proyecto?**  
Sí, pero sin credenciales. Usá `.env.example` con variables vacías o de ejemplo.

**¿Puedo referenciar un repo externo en lugar de duplicar el código?**  
Sí. En `guia-tecnica.md` podés linkear al repo original. Lo importante es que la documentación (one-pager, arquitectura, caso de uso) esté en este repo.

**¿Qué hago con un proyecto que está en proceso?**  
Subilo igual, marcá el estado como `🔄 En proceso` en `one-pager.md` y completá lo que tengas. Podés hacer PRs parciales y completar después.

**¿Quién revisa los PRs?**  
Ignacio Ayerbe (`@ignacioayerbe`) y Martina Pérez (`@martinaperez`). Al menos uno debe aprobar.
