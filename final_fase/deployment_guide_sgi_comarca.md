# 🚀 GUÍA DE DESPLIEGUE EN PRODUCCIÓN — SGI LA COMARCA

> **Arquitectura:** Híbrida / Especializada  
> **Dominio:** `lacomarcanic.com` (Administrado en Hostinger)  
> **Subdominio del ERP:** `app.lacomarcanic.com` (Vercel)  
> **Subdominio de la API:** `api.lacomarcanic.com` o URL directa (Render)  
> **Base de Datos:** PostgreSQL en la nube (Neon.tech)  
> **Costo mensual estimado:** **$0.00 USD** (Aprovechando las capas gratuitas profesionales)

---

## 1. 🌐 MAPA GENERAL DE LA ARQUITECTURA

Tu tienda de WordPress y tu nuevo sistema convivirán en armonía sin interferir el uno con el otro:

```mermaid
flowchart TD
    subgraph CLIENTES ["👥 PÚBLICO GENERAL"]
        C1["Navegador Web / Celular"]
    end

    subgraph SOCIOS ["👑 SOCIOS (Dylan, Samantha, Jean, Luden)"]
        S1["Laptop / Celular"]
    end

    subgraph HOSTINGER ["🏢 HOSTINGER (Tu Hosting Actual)"]
        DNS["Gestor de Zona DNS lacomarcanic.com"]
        WP["WordPress + WooCommerce (lacomarcanic.com)"]
    end

    subgraph NUBE_SGI ["☁️ NUBE DE ALTO RENDIMIENTO ($0)"]
        VERCEL["⚡ VERCEL (Frontend Next.js)<br>app.lacomarcanic.com"]
        RENDER["🛠️ RENDER (Backend NestJS API)<br>api.lacomarcanic.com"]
        NEON["🐘 NEON.TECH (PostgreSQL Real)<br>Base de Datos Persistente"]
    end

    C1 -->|"lacomarcanic.com"| DNS -->|"A Record"| WP
    S1 -->|"app.lacomarcanic.com"| DNS -->|"CNAME Record"| VERCEL
    VERCEL <-->|"Peticiones HTTP Seguras (Cookies + CSRF)"| RENDER
    RENDER <-->|"Consultas SQL y Transacciones"| NEON
```

---

## 2. 🐘 PASO 1: BASE DE DATOS POSTGRESQL (Neon.tech)

**Neon** es un servicio de PostgreSQL serverless en la nube diseñado para escalar, con backups automáticos y alta velocidad.

### 2.1 Crear el proyecto en Neon
1. Ve a **[neon.tech](https://neon.tech)** y crea una cuenta gratuita (puedes iniciar sesión con tu GitHub).
2. Haz clic en **"Create Project"**:
   * **Project name:** `sgi-comarca-prod`
   * **Postgres version:** `16` o `17`
   * **Region:** `US East (Ohio)` o `US East (N. Virginia)` (la más cercana a Nicaragua).
3. Haz clic en **Create Project**.

### 2.2 Obtener la cadena de conexión
Neon te mostrará de inmediato una pantalla con los datos de conexión. Asegúrate de seleccionar el modo **"Pooled connection"** (usa PgBouncer para soportar cientos de conexiones concurrentes):

Copiarás algo como esto:
```env
DATABASE_URL="postgresql://neondb_owner:npg_AbCd1234EfGh@ep-cool-fog-123456-pooler.us-east-2.aws.neon.tech/neondb?sslmode=require"
```

### 2.3 Aplicar las migraciones y bootstrap inicial hacia Neon
Desde tu computadora local, no necesitas instalar nada raro. Solo abres tu terminal en la carpeta raíz del proyecto y corres:

```powershell
# 1. Configurar temporalmente la URL de Neon en tu terminal
$env:DATABASE_URL="postgresql://neondb_owner:tu_password@tu_host-pooler.neon.tech/neondb?sslmode=require"

# 2. Desplegar todas las migraciones (Phase 3A hasta Phase 9A)
pnpm db:migrate:deploy

# 3. Correr el bootstrap inicial de los 4 socios y permisos
pnpm db:bootstrap
```

> ✅ **Resultado:** Tu base de datos en la nube ya tiene las 27+ tablas, triggers, constraints y los usuarios de Dylan, Samantha, Jean y Luden creados.

---

## 3. 🛠️ PASO 2: BACKEND NESTJS (Render.com)

**Render** alojará tu servidor de NestJS (`apps/api`), manteniéndolo activo y corriendo de forma segura.

### 3.1 Crear el Web Service
1. Entra a **[render.com](https://render.com)** y crea tu cuenta gratuita vinculada a GitHub.
2. Haz clic en **New +** > **Web Service**.
3. Selecciona tu repositorio de GitHub: `DylanRizo/sgi-comarca`.
4. Configura los siguientes campos:

| Campo | Valor exacto |
|---|---|
| **Name** | `sgi-comarca-api` |
| **Region** | `Ohio (US East)` (la misma que elegiste en Neon) |
| **Branch** | `main` |
| **Root Directory** | *(Dejar en blanco para que tome la raíz del monorepo)* |
| **Runtime** | `Node` |
| **Build Command** | `corepack enable && corepack prepare pnpm@11.18.0 --activate && pnpm install --frozen-lockfile && pnpm --filter @sgi/api build` |
| **Start Command** | `node apps/api/dist/main.js` |
| **Instance Type** | `Free` |

### 3.2 Variables de Entorno en Render
En la pestaña **"Environment"** de Render, agrega estas variables:

```env
NODE_ENV=production
PORT=10000
DATABASE_URL=postgresql://neondb_owner:tu_password@tu_host-pooler.neon.tech/neondb?sslmode=require
CORS_ORIGIN=https://app.lacomarcanic.com
SESSION_COOKIE_DOMAIN=.lacomarcanic.com
SESSION_SECRET=generar_un_string_aleatorio_de_64_caracteres_seguro
COOKIE_SECURE=true
```

5. Haz clic en **"Deploy Web Service"**.
6. Una vez terminado, Render te dará una URL como: `https://sgi-comarca-api.onrender.com`.

---

## 4. ⚡ PASO 3: FRONTEND NEXT.JS (Vercel.com)

**Vercel** es la plataforma oficial de los creadores de Next.js. Ofrece la mayor velocidad de carga en teléfonos y laptops.

### 4.1 Importar el proyecto
1. Entra a **[vercel.com](https://vercel.com)** e inicia sesión con tu GitHub.
2. Haz clic en **"Add New..."** > **"Project"**.
3. Selecciona el repositorio `sgi-comarca`.
4. En la configuración del proyecto:
   * **Framework Preset:** `Next.js`
   * **Root Directory:** Haz clic en *Edit* y selecciona `apps/web`.
5. En la sección **Environment Variables**, agrega:

```env
NEXT_PUBLIC_API_URL=https://sgi-comarca-api.onrender.com
```
*(Si luego le pones subdominio a la API como `api.lacomarcanic.com`, pones esa URL).*

6. Haz clic en **"Deploy"**. Vercel compilará la aplicación en menos de 2 minutos.

---

## 5. 🏢 PASO 4: CONECTAR TU DOMINIO EN HOSTINGER (DNS)

Este es el paso que une todo bajo tu marca `lacomarcanic.com`.

### 5.1 Vincular el subdominio en Vercel
1. En tu proyecto de Vercel, ve a **Settings** > **Domains**.
2. Escribe: `app.lacomarcanic.com` y dale a **Add**.
3. Vercel te dirá: *"Esperando registro DNS: CNAME apuntando a cname.vercel-dns.com"*.

### 5.2 Configurar el registro en Hostinger
1. Inicia sesión en **Hostinger** (hPanel).
2. Ve a la sección **Dominios** > Haz clic en `lacomarcanic.com`.
3. En el menú lateral izquierdo, haz clic en **DNS / Servidores de nombres**.
4. En la sección **"Administrar registros DNS"**, agrega un nuevo registro con estos datos:

| Tipo | Nombre | Contenido / Valor | TTL |
|---|---|---|---|
| **CNAME** | `app` | `cname.vercel-dns.com` | `300` o predeterminado |

5. Haz clic en **"Agregar registro"**.

---

*(Opcional pero recomendado para la API):*
Si también quieres que tu backend use tu dominio en vez de la URL de Render:
1. En Hostinger agregas otro registro:
   * **Tipo:** `CNAME`
   * **Nombre:** `api`
   * **Contenido:** Tu host de Render (ej. `sgi-comarca-api.onrender.com`)
2. En Render vas a **Settings** > **Custom Domains** y agregas `api.lacomarcanic.com`.

---

## 6. 🔐 PASO 5: ACTIVACIÓN DE CUENTAS DE LOS SOCIOS

Una vez que `https://app.lacomarcanic.com` esté en línea:

1. Entras a la consola local y generas la invitación para ti (Dylan):
   ```powershell
   pnpm auth:invite-admin
   ```
2. La terminal te imprimirá un link de activación único con token de 24 horas.
3. Abres ese link en tu navegador: `https://app.lacomarcanic.com/activate?token=...`
4. Creas tu contraseña segura.
5. Inicias sesión en **`https://app.lacomarcanic.com/login`**.
6. Desde la consola o desde la API generas las invitaciones para Samantha, Jean y Luden para que cada uno active su propia contraseña desde su teléfono.

---

## 🔄 RESUMEN DEL FLUJO DE TRABAJO DIARIO DESPUÉS DEL DESPLIEGUE

A partir de este momento, cualquier mejora o arreglo que hagas en el código:

```text
1. Modificas el código en tu máquina
2. Haces commit y push a GitHub:
   git add .
   git commit -m "feat: mejoras en modal de ventas"
   git push origin main
3. Vercel y Render detectan el cambio y actualizan la nube automáticamente en 90 segundos.
4. Cero comandos manuales en servidores.
```

¡Todo queda automatizado, profesional y con costo de infraestructura de **$0 USD**!
