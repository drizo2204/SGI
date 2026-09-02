# 📋 SGI LA COMARCA — DOCUMENTO DE CONTEXTO Y ESPECIFICACIÓN DE BRECHAS (LEGACY vs. NUEVO SISTEMA)

> **Propósito de este documento:**  
> Servir como especificación técnica y de contexto exhaustiva para **Claude Code** y **Codex**. Este documento detalla cada campo, flujo operativo y funcionalidad del sistema original (Google Apps Script / Sheets) que fue **omitido o recortado** durante la migración hacia el monorepo (NestJS, Next.js, Prisma, PostgreSQL), junto con las instrucciones precisas para su reintegración inmediata sin romper la arquitectura existente.

---

## 1. 🔍 DIAGNÓSTICO GENERAL Y ORIGEN DE LA DISCREPANCIA

### El hallazgo central
El backend y la base de datos PostgreSQL fueron modelados con alta rigurosidad arquitectónica. Las tablas de PostgreSQL **sí contemplan la mayoría de los campos del negocio**. Sin embargo, durante las Fases 7B, 7C, 8B, 8C y 9, los agentes de desarrollo aplicaron un criterio de **"MVP mínimo para pasar tests de integración"**, provocando que:
1. **Los contratos (`@sgi/contracts`) mutilaran las interfaces:** Se definieron DTOs de request y view que excluyeron deliberadamente los campos secundarios de texto y métricas.
2. **Los servicios del backend ignoraran columnas existentes:** Aunque la tabla `sales` tiene las columnas en PostgreSQL, el servicio `CreateSaleService` nunca las escribe y el mapper `SaleReadMapper` nunca las lee.
3. **El Frontend (`apps/web`) construyera formularios planos y sin contexto:** Se omitieron selectores clave (canal, motorizado, dirección), se eliminó la interactividad en tiempo real (cierre de caja a ciegas) y se degradó la analítica visual (1,528 líneas de gráficos y KPIs reducidas a listas HTML estáticas).

---

## 2. 🔬 AUDITORÍA EXHAUSTIVA DE CAMPOS Y FUNCIONALIDADES FALTANTES

A continuación se presenta la verificación punto por punto contrastando el archivo fuente del sistema Legacy con el estado real en la base de datos, el contrato, el backend y el frontend nuevo.

---

### ÁREA 1: MÓDULO DE VENTAS (MODAL DE VENTA Y REGISTRO)
*Fuentes Legacy:* `c:\Users\drizo\Desktop\SGI_comarca\Comp_ModalVenta.html` y `Service_Ventas.js` (Líneas 1-170).

| Campo / Flujo en Legacy | Código Legacy | Columna en PostgreSQL (`Sale`) | Presente en `@sgi/contracts` | Manejado en NestJS API | Presente en UI (`CreateSaleDialog.tsx`) | Diagnóstico y Acción Requerida |
|---|---|:---:|:---:|:---:|:---:|---|
| **Canal de Venta** | `ventaCanalVenta` (FB Marketplace, WhatsApp, Instagram, TikTok, Presencial) | ✅ `sales_channel_text` | ❌ **NO** | ❌ **NO** | ❌ **NO** | **Grave:** La columna existe en DB. Falta agregar `salesChannelText?: string` al contrato, al DTO de creación, al mapper de lectura y al `<select>` del formulario. |
| **Entregador / Motorizado** | `ventaEntregador` (Texto libre: Jean, Luden, Motorizado Carlos) | ✅ `deliverer_text` | ❌ **NO** | ❌ **NO** | ❌ **NO** | **Grave:** La columna existe en DB. Falta agregar `delivererText?: string` a contratos, DTO, servicio y al `<input>` del formulario. |
| **Lugar de Entrega / Dirección** | `ventaLugarEntrega` (Dirección del cliente, oficina, etc.) | ✅ `delivery_place` | ❌ **NO** | ❌ **NO** | ❌ **NO** | **Grave:** La columna existe en DB. Falta agregar `deliveryPlace?: string` a contratos, DTO, servicio y al `<input>` del formulario. |
| **Método de Pago Inicial** | `ventaMetodoPago` (`Efectivo`, `Digital`) | ✅ `payment_method_text` | ❌ **NO** | ❌ **NO** | ❌ **NO** | **Grave:** La columna existe en DB. Falta agregar `paymentMethodText?: string` a contratos, DTO, servicio y al selector de pago en la UI. |
| **Horas de Entrega** | `ventaHoraSalida`, `ventaHoraFinalizacion` | ✅ `departure_at`, `completed_at` | ⚠️ Solo en `SaleView` | ❌ No en `CreateSaleRequest` | ❌ **NO** | Las columnas existen en DB. Se debe permitir enviar `departureAt?: string` opcional al crear ventas que salgan directamente con motorizado. |
| **Observaciones de la Venta** | `ventaObservaciones` (Notas del pedido) | ✅ `observations` | ❌ **NO** | ❌ **NO** | ❌ **NO** | La columna existe en DB. Falta agregar `observations?: string` a contratos, DTO, servicio y `<textarea>` en la UI. |
| **Autocompletado Rápido de Producto** | `buscarProductoCarrito()` (`Comp_ModalVenta.html` L-70) | N/A | N/A | N/A | ⚠️ **Pobre** | El formulario actual usa un `<select>` nativo que carga todos los productos juntos. En el legacy había un buscador tipo input con dropdown predictivo por nombre o código para ventas ágiles en WhatsApp. |

---

### ÁREA 2: MÓDULO DE FINANZAS Y CIERRE DE CAJA DIARIO
*Fuentes Legacy:* `c:\Users\drizo\Desktop\SGI_comarca\Service_Finanzas.js` (Líneas 204-398) y `views_of_the_system.html` (Líneas 445-486).

| Funcionalidad Legacy | Implementación Legacy | Estado en Nuevo Backend | Estado en Nueva UI (`CreateClosingDialog.tsx`) | Brecha Identificada y Acción Requerida |
|---|---|:---:|:---:|---|
| **Resumen Previo de Ventas del Día** | `obtenerResumenCierreDiario(fecha)` calculaba el total de ventas completadas de la fecha antes de cerrar. | ❌ **Inexistente** | ❌ **Inexistente** | El diálogo actual es una caja "a ciegas". El socio no sabe cuánto vendió el sistema hasta que le da "Guardar". **Acción:** Crear endpoint `GET /api/v1/closings/preview?businessDate=YYYY-MM-DD` que devuelva `systemSales`, `inTransitCount`, `ventasPorVendedor` y `gastosDelDia`. |
| **Desglose de Ventas por Vendedor** | `ventasPorVendedor`: Mapa con `{ efectivo, digital, total, detalles: [] }` por cada socio. | ❌ **Inexistente** | ❌ **Inexistente** | En el cierre diario, los socios necesitan ver: *"Dylan vendió C$ 8,000 (5k efec / 3k dig) | Jean vendió C$ 4,000"*. Sin esto, no saben a quién cobrarle el dinero recolectado. **Acción:** Incluir el desglose por vendedor en el endpoint de preview. |
| **Desglose de Gastos del Día** | `totalGastosDia` y `gastosDetalle` (categoría + monto + descripción). | ❌ No se consulta en el flujo de cierre | ❌ No aparece en la pantalla | Aunque el cuadre no resta gastos (DEC-023), el socio necesita ver los gastos del día listados para comprobar egresos de caja física. **Acción:** El preview debe retornar la lista de gastos registrados en esa fecha. |
| **Cálculo en Tiempo Real del Cuadre (Semáforo)** | `calcularDiferenciasCierre()` en `Global_JS.html`: calculaba `(efectivo + digital - ventas)` al teclear y pintaba verde/rojo. | ❌ Solo se calcula en el POST final | ❌ Sin interactividad | La UI debe reaccionar en vivo mientras el usuario teclea el efectivo y saldo digital, mostrando la diferencia proyectada y si está "Cuadrado" o "Descuadrado" antes de enviar el request. |
| **Alerta Preventiva de Día Ya Cerrado** | `alreadyClosed` en `Service_Finanzas.js` alertaba antes de ingresar datos. | ⚠️ Solo lanza error 400 `CLOSING_ALREADY_EXISTS` al hacer submit | ❌ Sin feedback previo | El diálogo debe verificar inmediatamente si la fecha elegida ya tiene un cierre activo (`status === CLOSED`) y bloquear el formulario con un mensaje explicativo y botón a "Ver Cierre Existente". |
| **Impresión / Exportación del Cierre** | `imprimirReporteCierre()` (`Global_JS.html` L-3735). | ❌ **Inexistente** | ❌ **Inexistente** | Se necesita un botón de "Imprimir Comprobante de Cierre" o vista apta para imprimir (ticket/recibo) para archivar el cuadre del día. |

---

### ÁREA 3: MÓDULO DE ANALÍTICA Y BUSINESS INTELLIGENCE
*Fuentes Legacy:* `c:\Users\drizo\Desktop\SGI_comarca\Service_Analisis.js` (1,528 líneas) y `views_of_the_system.html` (Líneas 505-1527).

| Métrica / Reporte Legacy | Líneas en `Service_Analisis.js` | En Backend Actual (`analytics-read.service.ts`) | En Frontend Actual (`analytics/page.tsx`) | Brecha Técnica y Solución |
|---|:---:|:---:|:---:|---|
| **Ventas y Participación por Canal Publicitario** | L-752 (`calcularVentasPorCanal`) | ❌ **NO** (porque no guardaban el canal) | ❌ **NO** | Una vez reintegrado `salesChannelText`, agregar agregación `byChannel: { channel: string, revenue: string, saleCount: number, percentage: string }[]` en el endpoint de analítica. |
| **Ranking de Mejores Vendedores con Ticket Promedio** | L-702 (`calcularMejoresVendedores`) | ⚠️ Parcial (`bySeller` solo tiene revenue y count) | ❌ **Pésimo** (Solo lista `<li>` de IDs) | Falta calcular el **Ticket Promedio** (`revenue / saleCount`), asociar el nombre real del usuario (no solo el UUID) y renderizarlo en un podio visual de vendedores. |
| **Top Productos con Margen y Participación %** | L-480 (`calcularTopProductos`) | ⚠️ Parcial (`topProducts` solo da unidades y revenue) | ❌ Lista de texto plano | Falta porcentaje de contribución al total de ventas y ganancia bruta por producto. |
| **Top Zonas y Lugares de Entrega** | L-727 (`calcularTopLugares`) | ❌ **NO** | ❌ **NO** | Ranking de barrios y zonas donde más se entrega para enfocar la logística de los repartidores. |
| **Análisis de Tiempos de Entrega (Motorizados)** | L-1109 (`buildTiempo` y `renderTiempo`) | ❌ **NO** | ❌ **NO** | Tiempo promedio entre `departureAt` y `completedAt` por motorizado y por zona. |
| **Top Días y Horas Pico de Venta** | L-1240 y L-1460 | ❌ **NO** | ❌ **NO** | Análisis de qué días de la semana y qué horas generan más tráfico comercial para optimizar la pauta de Facebook Ads. |
| **Visualización Gráfica Interactiva** | Gráficos Chart.js en `Global_JS.html` | N/A | ❌ **CERO GRÁFICOS** | Reemplazar las listas `<li>` por componentes visuales con barras, tarjetas métricas y curvas de tendencia (usando Recharts o Chart.js). |

---

### ÁREA 4: FLUJOS OPERATIVOS ADICIONALES (Catálogo y Búsqueda)
*Fuentes Legacy:* `views_of_the_system.html` (tabs `buscar` y `entrada`).

1. **Búsqueda Rápida de Stock Omnicanal (Kiosko / WhatsApp Assistant):**
   * *Legacy:* En la pestaña `buscar`, un vendedor escribía "Air Max" y en 200 milisegundos veía una tabla con el stock consolidado por casa: Casa Dylan (2), Casa Luden (0), Casa Jean (1).
   * *Actual:* Se debe navegar a `/inventory`, esperar la carga paginada y filtrar. Falta un modal o atajo global (ej. `Ctrl + K` o botón "Buscar Stock") para consulta ultrarrápida mientras se atiende un cliente en chat.
2. **Entrada de Mercadería con Actualización de Costo (Recepción de Importación):**
   * *Legacy:* Había un formulario directo para registrar compras de lote con su costo unitario y asignarlo a una casa.
   * *Actual:* Solo existe el diálogo de "Ajuste de inventario", que se siente como corrección de mermas/errores y no como abastecimiento de mercadería nueva.

---

## 3. 🛠️ PLAN DE ACCIÓN Y ESPECIFICACIÓN TÉCNICA PARA CLAUDE CODE

Para implementar estas correcciones sin romper los 244 tests de integración ni las invariantes de arquitectura (ADR-007, ADR-009, ADR-010), Claude Code debe seguir este orden estricto de archivos:

### BLOQUE 1: Contratos Compartidos (`packages/contracts/src/`)

#### 1.1 Modificar `packages/contracts/src/sales.ts`:
Extender las interfaces para incluir los campos faltantes de forma opcional:

```typescript
// En CreateSaleRequest:
export interface CreateSaleRequest {
  businessDate: string;
  items: CreateSaleItemRequest[];
  sellerUserId?: string;
  shippingAmount?: string;
  status: SaleCreationStatus;
  
  // CAMPOS RESTAURADOS DEL LEGACY:
  delivererText?: string;
  salesChannelText?: string;
  deliveryPlace?: string;
  paymentMethodText?: string;
  departureAt?: string;
  observations?: string;
}

// En SaleView:
export interface SaleView {
  id: string;
  saleNumber: string;
  origin: SaleOrigin;
  businessDate: string;
  status: SaleStatus;
  paymentStatus: SalePaymentStatus;
  sellerUserId: string | null;
  currencyCode: string;
  shippingAmount: string;
  subtotal: string;
  total: string;
  items: SaleItemView[];
  departureAt: string | null;
  completedAt: string | null;
  createdAt: string;

  // CAMPOS RESTAURADOS DEL LEGACY:
  delivererText: string | null;
  salesChannelText: string | null;
  deliveryPlace: string | null;
  paymentMethodText: string | null;
  observations: string | null;
}
```

#### 1.2 Modificar `packages/contracts/src/finances.ts`:
Agregar el contrato del preview de cierre:

```typescript
export interface SellerClosingContribution {
  sellerUserId: string | null;
  sellerName: string;
  cashAmount: string;
  digitalAmount: string;
  totalAmount: string;
  saleCount: number;
}

export interface DayExpenseSummary {
  id: string;
  categoryName: string;
  amount: string;
  description: string | null;
}

export interface DailyClosingPreviewView {
  businessDate: string;
  alreadyClosed: boolean;
  existingClosingId: string | null;
  systemSales: string;
  inTransitCount: number;
  bySeller: SellerClosingContribution[];
  dayExpenses: DayExpenseSummary[];
  totalExpenses: string;
}
```

---

### BLOQUE 2: Backend NestJS (`apps/api/src/`)

#### 2.1 Módulo Ventas:
* **`apps/api/src/sales/dto/create-sale.dto.ts`:**
  Agregar validadores `@IsOptional()`, `@IsString()`, `@MaxLength(...)` para `delivererText`, `salesChannelText`, `deliveryPlace`, `paymentMethodText`, `departureAt`, `observations`.
* **`apps/api/src/sales/create-sale-request.canonical.ts`:**
  Asegurar que la canonicalización incluya estos campos opcionales en el cálculo del hash de idempotencia cuando vengan definidos.
* **`apps/api/src/sales/create-sale.service.ts`:**
  En el método `createInTransaction`, incluir los campos en el `transaction.sale.create({ data: { ... } })`:
  ```typescript
  delivererText: input.delivererText?.trim() || null,
  salesChannelText: input.salesChannelText?.trim() || null,
  deliveryPlace: input.deliveryPlace?.trim() || null,
  paymentMethodText: input.paymentMethodText?.trim() || null,
  departureAt: input.departureAt ? new Date(input.departureAt) : (input.status === 'IN_TRANSIT' ? this.clock.now() : null),
  observations: input.observations?.trim() || null,
  ```
* **`apps/api/src/sales/sale-read.mapper.ts`:**
  Mapear las columnas de la entidad Prisma al objeto `SaleView`.

#### 2.2 Módulo Finanzas:
* **`apps/api/src/finances/daily-closing.service.ts`:**
  Crear el método `preview(businessDate: string): Promise<DailyClosingPreviewView>`.
  - Debe consultar las ventas `COMPLETED` de la fecha y agruparlas por `sellerUserId` y método de pago.
  - Debe verificar si ya existe un `DailyClosing` para esa fecha con `status === 'CLOSED'`.
  - Debe consultar los `financialEntry` de tipo `EXPENSE` de esa fecha.
* **`apps/api/src/finances/finances.controller.ts`:**
  Exponer `GET /api/v1/closings/preview` protegido por `closings.read`.

---

### BLOQUE 3: Frontend Next.js (`apps/web/`)

#### 3.1 Modal de Ventas (`apps/web/components/sales/create-sale-dialog.tsx`):
Reincorporar las secciones del modal legacy con un diseño moderno:
1. **Sección Logística & Canal:**
   * Selector de Canal de Venta con opciones: `Facebook Marketplace`, `WhatsApp`, `Instagram`, `TikTok`, `Presencial/Local`, `Otro`.
   * Input de Entregador / Motorizado.
   * Input de Dirección de Entrega.
2. **Sección Pago & Notas:**
   * Selector de Método de Pago: `Efectivo`, `Digital`.
   * Textarea de Observaciones.

#### 3.2 Diálogo de Cierre Diario (`apps/web/components/finances/create-closing-dialog.tsx`):
Transformar el modal en un asistente guiado interactivo:
1. Al seleccionar la fecha, hacer fetch automático a `/api/v1/closings/preview`.
2. Si `alreadyClosed === true`, mostrar banner de advertencia con badge rojo y botón a ver el cierre existente.
3. Mostrar tarjetas de resumen:
   * **Ventas del Sistema:** C$ X (con desglose de vendedores en acordeón o mini-tabla).
   * **Ventas en Tránsito:** Y pedidos pendientes.
   * **Gastos del Día:** C$ Z.
4. Inputs de Efectivo Real y Digital Real.
5. **Caja de Cuadre en Vivo:** Mientras el usuario digita, calcular:
   $$\text{Diferencia} = \text{Efectivo Real} + \text{Digital Real} - \text{Ventas del Sistema}$$
   Mostrar badge dinámico en verde (**"CUADRADO"**) si $|\text{Diferencia}| \le 0.50$ o en rojo (**"DESCUADRE DE C$ X"**) con el monto exacto.

#### 3.3 Dashboard Analítico (`apps/web/app/(private)/analytics/page.tsx`):
1. Instalar o utilizar componentes visuales (Recharts / SVG Charts).
2. Crear podio de vendedores (1º, 2º, 3º lugar) con total facturado, cantidad de ventas y ticket promedio.
3. Crear gráfico de pastel o barras de ventas por canal.
4. Mostrar KPIs con micro-gráficos de tendencia (Sparklines).

---

## 4. ⚠️ INVARIANTES Y REGLAS QUE NO SE DEBEN ROMPER

Al aplicar estas mejoras, Claude Code debe respetar estrictamente las reglas ya consagradas en los ADRs del proyecto:

1. **Invariante ADR-009 (Costos en Ventas):** `sales.read` nunca debe exponer `unitCostSnapshot`, margen bruto ni ganancias a usuarios que no posean el permiso `finances.read`.
2. **Invariante ADR-010 (Fórmula de Cierre):** Los gastos **no** restan en la fórmula de cuadre del cierre (`efectivo + digital - ventas`). Se muestran como información de contexto para arqueo de caja física, pero no alteran el cálculo del balance.
3. **Invariante de Idempotencia:** Toda mutación (`POST /api/v1/sales`, `POST /api/v1/closings`) debe seguir exigiendo `Idempotency-Key` en el header con advisory locks de PostgreSQL.
4. **Invariante de Enteros Escalados:** Todo cálculo financiero se realiza en enteros escalados (centavos/céntimos), nunca con aritmética de punto flotante en JS.
