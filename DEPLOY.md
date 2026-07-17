# Deploy del portal en IBM Code Engine

> Basado en el deploy real de AGIP (`DEPLOY-CODE-ENGINE.md`).
> El portal usa MkDocs + nginx + IBM Code Engine — mismo stack que el Workshop-Hub y Bee-tech.

---

## Primera vez — setup manual

Hacé estos pasos una sola vez. Después, cada push a `main` lo hace el workflow automáticamente.

### 1. Prerrequisitos

```bash
# IBM Cloud CLI
brew install ibm-cloud-cli

# Plugins
ibmcloud plugin install code-engine
ibmcloud plugin install container-registry

# jq (para el registry secret)
brew install jq
```

### 2. Login a IBM Cloud

```bash
ibmcloud login --sso
ibmcloud target -g Default
ibmcloud cr login
```

Seleccioná la cuenta IBM y la región `us-south`.

### 3. Crear el namespace en Container Registry

```bash
ibmcloud cr namespace-add ce-latam
```

La imagen va a vivir en: `us.icr.io/ce-latam/ce-latam-portfolio:latest`

### 4. Build de la imagen Docker

⚠️ **Crítico si usás Mac con Apple Silicon (M1/M2/M3):** siempre usá `--platform linux/amd64`.
Sin esto, Code Engine falla con `Initial scale was never achieved`.

```bash
# Desde la raíz del repo
docker build --platform linux/amd64 \
  -f deploy/Dockerfile \
  -t us.icr.io/ce-latam/ce-latam-portfolio:latest .
```

### 5. Push al Container Registry

```bash
docker push us.icr.io/ce-latam/ce-latam-portfolio:latest
```

### 6. Crear el proyecto en Code Engine

```bash
ibmcloud ce project create --name ce-latam-docs
ibmcloud ce project select --name ce-latam-docs
```

### 7. Crear el registry secret

```bash
ibmcloud ce registry create \
  --name icr-secret \
  --server us.icr.io \
  --username iamapikey \
  --password $(ibmcloud iam api-key-create ce-latam-key --output json | jq -r .apikey)
```

### 8. Deploy de la app

```bash
ibmcloud ce app create \
  --name ce-latam-portfolio \
  --image us.icr.io/ce-latam/ce-latam-portfolio:latest \
  --registry-secret icr-secret \
  --port 8080 \
  --min-scale 0 \
  --max-scale 2
```

### 9. Verificar el deploy

```bash
ibmcloud ce app get -n ce-latam-portfolio
```

Buscás:
```
Status Summary:  Application deployed successfully
URL:             https://ce-latam-portfolio.XXXXX.us-south.codeengine.appdomain.cloud
```

Si falla, revisá los logs:
```bash
ibmcloud ce app logs -f -n ce-latam-portfolio
```

---

## Re-deploy manual (cuando no usás el workflow)

```bash
# 1. Rebuild
docker build --platform linux/amd64 \
  -f deploy/Dockerfile \
  -t us.icr.io/ce-latam/ce-latam-portfolio:latest .

# 2. Push
docker push us.icr.io/ce-latam/ce-latam-portfolio:latest

# 3. Update
ibmcloud ce app update \
  --name ce-latam-portfolio \
  --image us.icr.io/ce-latam/ce-latam-portfolio:latest
```

---

## Deploy automático (GitHub Actions)

Después del setup manual, configurá estos 5 secrets en el repo de GitHub:

**Settings → Secrets and variables → Actions → New repository secret**

| Secret | Valor |
|---|---|
| `IBMCLOUD_APIKEY` | API key de IBM Cloud con permisos de Code Engine e ICR |
| `ICR_REGION` | `us-south` |
| `ICR_NAMESPACE` | `ce-latam` |
| `CE_PROJECT` | `ce-latam-docs` |
| `CE_APP_NAME` | `ce-latam-portfolio` |

A partir de ahí, cada push a `main` dispara el workflow `.github/workflows/deploy-docs.yml`
que buildea el sitio MkDocs, construye la imagen Docker, la pushea al ICR y actualiza la app
en Code Engine.

---

## Comandos útiles

```bash
# Estado de la app
ibmcloud ce app get -n ce-latam-portfolio

# Logs en tiempo real
ibmcloud ce app logs -f -n ce-latam-portfolio

# Eventos del sistema
ibmcloud ce app events -n ce-latam-portfolio

# Todas las apps del proyecto
ibmcloud ce app list

# Todos los proyectos
ibmcloud ce project list
```

---

## Errores comunes

| Error | Causa | Solución |
|---|---|---|
| `Initial scale was never achieved` | Imagen buildeada para ARM (Apple Silicon) | Rebuild con `--platform linux/amd64` |
| `Authorization required` al hacer push | `ibmcloud cr login` no hecho | Correr `ibmcloud cr login` y volver a pushear |
| `no resource group is targeted` | Falta apuntar al resource group | `ibmcloud target -g Default` |
| Sitio carga pero rutas dan 404 | Configuración de nginx | Verificar `deploy/nginx.conf` con `try_files` |
