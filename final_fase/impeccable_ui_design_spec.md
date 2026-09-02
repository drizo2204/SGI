# 🎨 SGI LA COMARCA — ESPECIFICACIÓN DE DISEÑO UI/UX CON "IMPECCABLE"

> **Propósito de este documento:**  
> Servir como guía canónica de diseño para que **Claude Code**, **Codex** o cualquier desarrollador transforme la interfaz del SGI desde su estado actual ("dashboard plano de IA") hacia una experiencia **POS / Fintech de clase mundial**, utilizando la metodología, los comandos y el vocabulario de la skill **Impeccable**.

---

## 1. 🎯 FILOSOFÍA Y VOCABULARIO IMPECCABLE

Impeccable prohíbe el "aspecto genérico generado por IA": tarjetas grises infinitas, fuentes sin personalidad, botones sin contraste y falta de ritmo visual. 

Para La Comarca, aplicamos estos principios:

| Comando Impeccable | Directriz para SGI La Comarca |
|---|---|
| `audit` | **Eliminar la ceguera operativa:** Si un botón o formulario oculta datos que el usuario necesita para decidir (como las ventas previas en el cierre), la UI se considera rota. |
| `bolder` | **Alto contraste y jerarquía de dinero:** Los totales facturados, el estado de caja y el stock disponible deben verse a 2 metros de distancia en una pantalla de laptop o teléfono. |
| `colorize` | **Paleta con propósito:** El verde es dinero y cuadre; el ámbar es pedidos en moto y stock bajo; el rojo es descuadre y quiebre de stock; el azul oscuro es la solidez del ERP. |
| `typeset` | **Tipografía con números tabulares:** Todas las cifras monetarias y cantidades de inventario usan `font-mono` o `tabular-nums` para que las columnas alineen perfectamente en decimales. |
| `polish` | **Micro-interacciones y estados:** Bordes sutiles con gradientes, sombras suaves para elevación de modales, badges redondeados y transiciones de 150ms. |
| `quieter` | **Eliminar ruido en formularios:** Separar datos secundarios (observaciones, notas) de los datos de alta frecuencia (producto, cantidad, precio). |

---

## 2. 🏛️ SISTEMA DE DISEÑO Y TOKENS VISUALES

### 2.1 Identidad de Marca: "Modern Retail POS & Fintech"
Inspirado en interfaces de alto rendimiento como **Linear, Stripe Dashboard y Toast POS**. El sistema debe sentirse rápido, confiable y extremadamente ágil para vendedores que atienden en WhatsApp mientras registran ventas.

### 2.2 Paleta de Colores (Tokens CSS)

```css
:root {
  /* Marca Primaria - Deep Slate & Electric Indigo */
  --brand-primary: #0f172a;        /* Azul Noche Profundo */
  --brand-accent: #2563eb;         /* Azul Eléctrico para foco y enlaces */
  --brand-warm: #d97706;           /* Ámbar La Comarca (energía comercial) */

  /* Semáforos de Negocio */
  --status-success-bg: #ecfdf5;    /* Verde fondo */
  --status-success-text: #065f46;  /* Verde texto */
  --status-success-border: #10b981;/* Verde borde / acento */

  --status-warning-bg: #fffbeb;    /* Ámbar fondo (En Tránsito / Stock Bajo) */
  --status-warning-text: #92400e;  /* Ámbar texto */
  --status-warning-border: #f59e0b;/* Ámbar borde */

  --status-danger-bg: #fef2f2;     /* Rojo fondo (Descuadre / Agotado) */
  --status-danger-text: #991b1b;   /* Rojo texto */
  --status-danger-border: #ef4444; /* Rojo borde */

  /* Superficies Modo Claro */
  --surface-bg: #f8fafc;
  --surface-card: #ffffff;
  --surface-border: #e2e8f0;
  --text-primary: #0f172a;
  --text-secondary: #475569;
  --text-muted: #94a3b8;
}

/* Modo Oscuro Nativo para Operación Nocturna */
.dark, [data-theme="dark"] {
  --surface-bg: #0b0f17;
  --surface-card: #111827;
  --surface-border: #1f2937;
  --text-primary: #f8fafc;
  --text-secondary: #94a3b8;
  --text-muted: #64748b;

  --status-success-bg: #064e3b33;
  --status-success-text: #34d399;
  --status-warning-bg: #78350f33;
  --status-warning-text: #fbbf24;
  --status-danger-bg: #7f1d1d33;
  --status-danger-text: #f87171;
}
```

### 2.3 Tipografía
* **Familia Principal:** `Plus Jakarta Sans`, `Geist`, o `system-ui, -apple-system, sans-serif`.
* **Familia Numérica / Monetaria:** `JetBrains Mono`, `Fira Code`, o `ui-monospace` con la propiedad CSS `font-variant-numeric: tabular-nums`.

---

## 3. 📱 REDISEÑO DETALLADO DE COMPONENTES CRÍTICOS

---

### COMPONENTE 1: MODAL DE VENTA TIPO POS (`CreateSaleDialog.tsx`)

#### Concepto Visual:
Debe lucir como una caja registradora moderna de dos columnas en escritorio y un flujo por pestañas/acordeón en móvil:
* **Columna Izquierda (2/3):** Selección ágil de productos con autocompletado y tabla de carrito interactiva.
* **Columna Derecha (1/3):** Ficha de logística (Canal, Repartidor, Dirección) y resumen de cobro con botón de acción gigante.

#### Estructura y Código JSX Objetivo:

```tsx
<div className="fixed inset-0 z-50 flex items-center justify-center bg-black/60 backdrop-blur-sm p-4">
  <div className="bg-surface-card border border-surface-border rounded-2xl shadow-2xl w-full max-w-5xl max-h-[90vh] flex flex-col overflow-hidden">
    
    {/* Header del POS */}
    <div className="flex items-center justify-between px-6 py-4 border-b border-surface-border bg-slate-50 dark:bg-slate-900/50">
      <div className="flex items-center gap-3">
        <div className="w-10 h-10 rounded-xl bg-blue-600 text-white flex items-center justify-center font-bold text-lg shadow-md shadow-blue-500/20">
          💰
        </div>
        <div>
          <h2 className="text-lg font-bold text-text-primary">Nueva Venta</h2>
          <p className="text-xs text-text-secondary">Registro comercial con descuento de inventario en vivo</p>
        </div>
      </div>
      <button onClick={onCancel} className="text-text-muted hover:text-text-primary p-2 rounded-lg hover:bg-slate-100 dark:hover:bg-slate-800 transition">
        ✕
      </button>
    </div>

    {/* Cuerpo de 2 Columnas */}
    <div className="grid grid-cols-1 lg:grid-cols-3 flex-1 overflow-y-auto">
      
      {/* COLUMNA IZQUIERDA: CARRITO Y PRODUCTOS (2/3) */}
      <div className="lg:col-span-2 p-6 border-b lg:border-b-0 lg:border-r border-surface-border flex flex-col gap-6">
        
        {/* Selector Rápido de Producto */}
        <div className="bg-slate-50 dark:bg-slate-800/40 p-4 rounded-xl border border-surface-border">
          <label className="block text-xs font-semibold text-text-secondary uppercase tracking-wider mb-2">
            Agregar Producto al Carrito
          </label>
          <div className="grid grid-cols-12 gap-2">
            <div className="col-span-12 sm:col-span-6">
              <input 
                type="text" 
                placeholder="🔍 Buscar por código o nombre..." 
                className="w-full px-3 py-2 bg-white dark:bg-slate-900 border border-surface-border rounded-lg text-sm focus:ring-2 focus:ring-blue-500"
              />
            </div>
            <div className="col-span-6 sm:col-span-3">
              <select className="w-full px-3 py-2 bg-white dark:bg-slate-900 border border-surface-border rounded-lg text-sm">
                <option>Casa Dylan (Stock: 4)</option>
                <option>Casa Luden (Stock: 1)</option>
                <option>Casa Jean (Stock: 2)</option>
              </select>
            </div>
            <div className="col-span-3 sm:col-span-2">
              <input 
                type="number" 
                defaultValue="1" 
                className="w-full px-3 py-2 bg-white dark:bg-slate-900 border border-surface-border rounded-lg text-sm text-center font-bold"
              />
            </div>
            <div className="col-span-3 sm:col-span-1 flex items-end">
              <button className="w-full h-9 bg-blue-600 hover:bg-blue-700 text-white rounded-lg font-bold flex items-center justify-center shadow-sm">
                +
              </button>
            </div>
          </div>
        </div>

        {/* Tabla del Carrito */}
        <div className="flex-1 border border-surface-border rounded-xl overflow-hidden flex flex-col">
          <table className="w-full text-left text-sm">
            <thead className="bg-slate-100 dark:bg-slate-800/80 text-xs font-semibold text-text-secondary uppercase">
              <tr>
                <th className="px-4 py-3">Producto</th>
                <th className="px-4 py-3">Almacén</th>
                <th className="px-4 py-3 text-center">Cant.</th>
                <th className="px-4 py-3 text-right">Precio Unit.</th>
                <th className="px-4 py-3 text-right">Subtotal</th>
                <th className="px-4 py-3 text-center"></th>
              </tr>
            </thead>
            <tbody className="divide-y divide-surface-border">
              {/* Filas del carrito con cantidades editables y botón eliminar */}
            </tbody>
          </table>
        </div>
      </div>

      {/* COLUMNA DERECHA: LOGÍSTICA, CANAL Y TOTAL (1/3) */}
      <div className="p-6 bg-slate-50/50 dark:bg-slate-900/20 flex flex-col justify-between gap-6">
        
        <div className="space-y-4">
          <h3 className="text-xs font-bold text-text-secondary uppercase tracking-wider">
            Detalles de Logística & Canal
          </h3>

          {/* Selector de Canal */}
          <div>
            <label className="block text-xs font-medium text-text-secondary mb-1">Canal de Contacto</label>
            <div className="grid grid-cols-2 gap-2">
              {['WhatsApp', 'Marketplace', 'Instagram', 'TikTok'].map(ch => (
                <button 
                  key={ch} 
                  type="button"
                  className="px-3 py-2 border border-surface-border rounded-lg text-xs font-medium text-text-secondary hover:border-blue-500 hover:text-blue-600 focus:bg-blue-50 focus:border-blue-600 transition"
                >
                  {ch}
                </button>
              ))}
            </div>
          </div>

          {/* Repartidor / Motorizado */}
          <div>
            <label className="block text-xs font-medium text-text-secondary mb-1">Motorizado / Entregador</label>
            <input 
              type="text" 
              placeholder="ej: Motorizado Carlos" 
              className="w-full px-3 py-2 bg-white dark:bg-slate-900 border border-surface-border rounded-lg text-sm"
            />
          </div>

          {/* Lugar de Entrega */}
          <div>
            <label className="block text-xs font-medium text-text-secondary mb-1">Lugar de Entrega</label>
            <input 
              type="text" 
              placeholder="ej: Metrocentro, portón principal" 
              className="w-full px-3 py-2 bg-white dark:bg-slate-900 border border-surface-border rounded-lg text-sm"
            />
          </div>

          {/* Estado Inicial & Flete */}
          <div className="grid grid-cols-2 gap-3 pt-2">
            <div>
              <label className="block text-xs font-medium text-text-secondary mb-1">Estado Entrega</label>
              <select className="w-full px-3 py-2 bg-white dark:bg-slate-900 border border-surface-border rounded-lg text-xs font-semibold">
                <option value="IN_TRANSIT">🛵 En Ruta</option>
                <option value="COMPLETED">✅ Inmediato</option>
              </select>
            </div>
            <div>
              <label className="block text-xs font-medium text-text-secondary mb-1">Envío (C$)</label>
              <input 
                type="number" 
                placeholder="0.00" 
                className="w-full px-3 py-2 bg-white dark:bg-slate-900 border border-surface-border rounded-lg text-xs font-mono"
              />
            </div>
          </div>
        </div>

        {/* Tarjeta de Cobro Total */}
        <div className="bg-blue-50 dark:bg-blue-950/40 p-4 rounded-xl border border-blue-200 dark:border-blue-900/60">
          <div className="flex justify-between text-xs text-blue-800 dark:text-blue-300 mb-1">
            <span>Subtotal Productos:</span>
            <span className="font-mono">C$ 1,200.00</span>
          </div>
          <div className="flex justify-between text-xs text-blue-800 dark:text-blue-300 mb-2">
            <span>Costo de Envío:</span>
            <span className="font-mono">C$ 100.00</span>
          </div>
          <div className="flex justify-between text-lg font-black text-blue-950 dark:text-white border-t border-blue-200/60 dark:border-blue-900 pt-2">
            <span>TOTAL:</span>
            <span className="font-mono text-xl text-blue-600 dark:text-blue-400">C$ 1,300.00</span>
          </div>
        </div>

        {/* Botón de Confirmación POS */}
        <button className="w-full py-3.5 bg-emerald-600 hover:bg-emerald-700 text-white font-bold rounded-xl shadow-lg shadow-emerald-600/20 text-sm transition transform active:scale-[0.99]">
          💾 Registrar Venta (F2)
        </button>
      </div>

    </div>
  </div>
</div>
```

---

### COMPONENTE 2: CIERRE DE CAJA GUIADO (`CreateClosingDialog.tsx`)

#### Concepto Visual: "La Balanza Interactiva"
El usuario nunca debe calcular a ciegas. El modal debe mostrar una balanza en tiempo real:
* **Lado Izquierdo (Ventas Esperadas por el Sistema):** Total acumulado del día con lista de vendedores.
* **Lado Derecho (Dinero Real en Mano):** Inputs numéricos gigantes para Efectivo y Saldo Digital.
* **Centro Inferior (El Semáforo de Cuadre):** Un indicador dinámico en tiempo real que dice:
  * 🟢 **"CAJA CUADRADA (Diferencia: C$ 0.00)"** si coincide.
  * 🔴 **"DESCUADRE DE -C$ 150.00 (Falta dinero)"** en color rojo de advertencia.

#### Estructura y Código JSX Objetivo:

```tsx
<div className="bg-surface-card border border-surface-border rounded-2xl p-6 max-w-2xl w-full">
  <div className="flex justify-between items-center mb-6">
    <div>
      <h3 className="text-xl font-bold text-text-primary">Cierre Diario de Operaciones</h3>
      <p className="text-xs text-text-secondary">Arqueo y conciliación de caja para la fecha seleccionada</p>
    </div>
    <input 
      type="date" 
      className="px-3 py-1.5 border border-surface-border rounded-lg text-sm font-semibold"
      defaultValue={today} 
    />
  </div>

  {/* Resumen del Sistema (Lo que el ERP espera) */}
  <div className="grid grid-cols-2 gap-4 mb-6">
    <div className="p-4 rounded-xl bg-slate-50 dark:bg-slate-900 border border-surface-border">
      <span className="text-xs font-semibold text-text-secondary uppercase">Ventas Completadas Hoy</span>
      <p className="text-2xl font-black font-mono text-text-primary mt-1">C$ 14,850.00</p>
      <p className="text-xs text-text-muted mt-1">12 pedidos entregados</p>
    </div>
    <div className="p-4 rounded-xl bg-amber-50 dark:bg-amber-950/30 border border-amber-200 dark:border-amber-900/40">
      <span className="text-xs font-semibold text-amber-800 dark:text-amber-300 uppercase">En Tránsito (Pendientes)</span>
      <p className="text-2xl font-black font-mono text-amber-900 dark:text-amber-200 mt-1">3 Pedidos</p>
      <p className="text-xs text-amber-700 dark:text-amber-400 mt-1">No afectan el cuadre de hoy</p>
    </div>
  </div>

  {/* Desglose de Vendedores (Acordeón Rápido) */}
  <div className="mb-6 border border-surface-border rounded-xl p-3 bg-slate-50/50">
    <span className="text-xs font-bold text-text-secondary block mb-2">Recaudación por Vendedor:</span>
    <div className="grid grid-cols-2 gap-2 text-xs">
      <div className="flex justify-between p-2 bg-white rounded border">
        <span>Dylan Rizo:</span>
        <span className="font-mono font-bold">C$ 8,200.00</span>
      </div>
      <div className="flex justify-between p-2 bg-white rounded border">
        <span>Jean:</span>
        <span className="font-mono font-bold">C$ 6,650.00</span>
      </div>
    </div>
  </div>

  {/* Captura de Dinero Real */}
  <div className="space-y-4 mb-6">
    <span className="text-xs font-bold text-text-secondary uppercase tracking-wider block">
      Conteo de Dinero Real en Mano
    </span>
    <div className="grid grid-cols-2 gap-4">
      <div>
        <label className="block text-xs font-medium mb-1">💵 Efectivo Físico (C$)</label>
        <input 
          type="number" 
          placeholder="0.00"
          className="w-full text-lg font-mono font-bold p-3 border rounded-xl bg-white dark:bg-slate-900"
        />
      </div>
      <div>
        <label className="block text-xs font-medium mb-1">📱 Saldo Digital / Bancos (C$)</label>
        <input 
          type="number" 
          placeholder="0.00"
          className="w-full text-lg font-mono font-bold p-3 border rounded-xl bg-white dark:bg-slate-900"
        />
      </div>
    </div>
  </div>

  {/* EL SEMÁFORO EN TIEMPO REAL (Banner Reactivo) */}
  <div className="p-4 rounded-xl mb-6 bg-emerald-50 border border-emerald-300 text-emerald-900 flex items-center justify-between">
    <div className="flex items-center gap-3">
      <span className="text-2xl">⚖️</span>
      <div>
        <p className="font-bold text-sm">BALANCE DE CAJA: CUADRADO</p>
        <p className="text-xs text-emerald-700">La diferencia está dentro de la tolerancia permitida (±0.50)</p>
      </div>
    </div>
    <span className="text-xl font-mono font-black">C$ 0.00</span>
  </div>

  {/* Botones de Acción */}
  <div className="flex justify-between items-center pt-2">
    <button className="px-4 py-2 text-sm text-text-secondary hover:text-text-primary border rounded-xl">
      🖨️ Imprimir Borrador
    </button>
    <div className="flex gap-3">
      <button className="px-5 py-2.5 text-sm text-text-secondary hover:bg-slate-100 rounded-xl">
        Cancelar
      </button>
      <button className="px-6 py-2.5 text-sm font-bold bg-blue-600 hover:bg-blue-700 text-white rounded-xl shadow-md shadow-blue-500/20">
        Confirmar y Cerrar Caja
      </button>
    </div>
  </div>
</div>
```

---

### COMPONENTE 3: DASHBOARD ANALÍTICO VISUAL (`app/(private)/analytics/page.tsx`)

#### Concepto Visual: "Fintech Command Center"
Reemplazar la lista `<li>` por un centro de inteligencia con 4 bloques:
1. **Fila Superior de KPIs (Hero Cards):**
   * Total Facturado (con comparativa de periodo anterior).
   * Unidades Vendidas.
   * Margen de Ganancia Promedio (con badge de honest margin coverage).
   * Valorización Total del Inventario.
2. **Podio de Vendedores (Gamificación Comercial):**
   * Tarjetas destacadas para 1º, 2º y 3º lugar con medallas (`🥇, 🥈, 🥉`), barra de progreso y ticket promedio.
3. **Distribución por Canales (Donut Chart / Barras Proporcionales):**
   * Visualización del porcentaje de ventas provenientes de Facebook Marketplace vs WhatsApp vs TikTok.
4. **Top Productos con Margen:**
   * Tabla visual con avatares de producto, barras de volumen y porcentaje de participación sobre las ventas.

---

## 4. 📋 CHECKLIST DE IMPLEMENTACIÓN CON CLAUDE CODE E IMPECCABLE

Cuando ejecutes Claude Code o Codex con la skill Impeccable instalada, utiliza este checklist de comandos:

```bash
# 1. Auditoría de accesibilidad y contraste en las páginas privadas
/impeccable audit apps/web/app/(private)/

# 2. Refinamiento estético del Modal de Ventas (look POS)
/impeccable polish apps/web/components/sales/create-sale-dialog.tsx --bolder

# 3. Refinamiento del Diálogo de Cierre Diario (Balanza interactiva)
/impeccable polish apps/web/components/finances/create-closing-dialog.tsx

# 4. Transformación del Dashboard Analítico a centro visual Fintech
/impeccable colorize apps/web/app/(private)/analytics/page.tsx --theme=slate-blue

# 5. Estandarización de tablas y tipografía monetaria (tabular numbers)
/impeccable typeset apps/web/
```

Siguiendo esta especificación, la interfaz de **SGI La Comarca** dejará de parecer una aplicación estéril generada por IA y se convertirá en un **sistema comercial atractivo, fluido y de nivel profesional**.
