# Documentación Técnica - Sistema de Matriz de Asignación Electrónica

## Información General

**Archivo:** `mant_agrupador_tarea.html`  
**Versión:** 1.3  
**Fecha:** Noviembre 2025  
**Tipo:** Aplicación web SPA (Single Page Application)  
**Tamaño:** ~1,350 líneas de código
**Última actualización:** Campo Habilitado en exportaciones JSON/CSV

---

## 1. Arquitectura de la Aplicación

### 1.1 Tipo de Aplicación
- **Single Page Application (SPA)** - Todo contenido en un único archivo HTML
- **Aplicación web cliente** - No requiere servidor backend
- **Auto-contenida** - Sin dependencias externas (librerías, frameworks)

### 1.2 Estructura del Archivo
```
mant_agrupador_tarea.html
├── <!DOCTYPE html>
├── <head>
│   ├── Meta tags (charset, viewport)
│   └── <style> (CSS embebido ~550 líneas)
│       ├── Variables CSS
│       ├── Estilos generales
│       ├── Toggle switch (NUEVO)
│       └── Modales
├── <body>
│   ├── <header> - Panel de filtros
│   ├── <main> - Contenido principal
│   │   ├── Tabla de matriz
│   │   │   ├── Toggle por COPE (NUEVO)
│   │   │   ├── Radio buttons
│   │   │   └── Botón limpiar
│   │   └── Botones de acción
│   ├── Modal JSON - Resultado de guardado
│   ├── Modal Ayuda - Manual de usuario
│   └── <script> (JavaScript ~720 líneas)
```

---

## 2. Tecnologías Utilizadas

### 2.1 HTML5
- **Versión:** HTML5 (DOCTYPE html)
- **Codificación:** UTF-8
- **Viewport:** Responsive (width=device-width, initial-scale=1)

**Elementos principales:**
- Formularios: `<select>`, `<input type="text">`, `<input type="radio">`, `<button>`
- Tablas: `<table>`, `<thead>`, `<tbody>`, `<tr>`, `<th>`, `<td>`
- Contenedores: `<header>`, `<main>`, `<div>`
- Modales: `<div class="modal-overlay">`
- Iframe: Para cargar manual de usuario

### 2.2 CSS3
- **Líneas de código:** ~550
- **Metodología:** CSS embebido en `<style>`
- **Pre-procesadores:** Ninguno (CSS vanilla)
- **Nuevas clases:** `.cope-toggle`, `.toggle-slider`, `.cope-row-disabled` (toggle switch)

**Características avanzadas utilizadas:**
```css
/* CSS Variables (Custom Properties) */
:root {
  --color-primary: #0562b1;
  --color-primary-dark: #034a83;
  --color-header-bg: #0a6cbd;
  /* ... 8 variables más */
}

/* CSS Grid */
header.filtros {
  display: grid;
  grid-template-columns: repeat(5, 1fr);
  gap: 12px;
}

/* Flexbox */
.cope-header-content {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

/* Pseudo-elementos */
.cell-selected.drag-enabled::after {
  content: '';
  position: absolute;
  /* Punto negro para arrastre */
}

/* Animaciones CSS */
@keyframes pulseHighlight {
  0%, 100% { 
    box-shadow: 0 0 10px rgba(255, 107, 53, 0.6);
    transform: scale(1);
  }
  50% { 
    box-shadow: 0 0 20px rgba(255, 107, 53, 0.9);
    transform: scale(1.02);
  }
}

/* Media Queries (Responsive) */
@media (max-width: 768px) {
  header.filtros {
    grid-template-columns: repeat(2, 1fr);
  }
}

/* Sticky positioning */
.table-matriz tbody th {
  position: sticky;
  left: 0;
  z-index: 2;
}
```

**Esquema de colores:**
- **Propios (Recursos Internos):** Azul (`#e3f2fd` fondo, `#2196f3` acento)
- **Terceros (Recursos Externos):** Verde (`#e8f5e9` fondo, `#4caf50` acento)
- **Principal:** Azul corporativo (`#0562b1`)
- **Alerta:** Naranja (`#ff6b35`)

### 2.3 JavaScript (ES6+)
- **Versión:** ECMAScript 6+ (ES2015+)
- **Líneas de código:** ~660
- **Modo:** Vanilla JavaScript (sin frameworks)
- **Paradigma:** Programación imperativa y funcional

**Características ES6+ utilizadas:**
```javascript
// Arrow Functions
document.getElementById('btnGuardar').addEventListener('click', () => {
  const data = recopilarDatos();
  // ...
});

// Template Literals
alert(`División: ${divisionTexto}\n\nÁrea: ${areaTexto}`);

// Spread Operator
const subcategorias = [
  ...columnasAltas.map(([sub, tipo]) => ({ categoria: 'Altas', sub, tipo })),
  ...columnasCambioDomicilio.map(([sub, tipo]) => ({ categoria: 'CambioDomicilio', sub, tipo })),
  ...columnasMigraciones.map(([sub, tipo]) => ({ categoria: 'Migraciones', sub, tipo }))
];

// Destructuring
columnasAltas.forEach(([sub, tipo]) => {
  // ...
});

// Const/Let (Block scoping)
const copes = ['COPE 1', 'COPE 2', /* ... */];
let selectedCell = null;

// Array Methods (map, filter, forEach)
const rows = data.map(row => 
  `${row.Division},${row.Area},${row.COPE},${row.Categoria},${row.Valor},${row.TipoDeRecurso}`
).join('\n');

// querySelector / querySelectorAll
document.querySelectorAll('#matriz tbody tr').forEach(tr => {
  // ...
});
```

---

## 3. APIs del Navegador Utilizadas

### 3.1 DOM API
```javascript
// Manipulación del DOM
document.getElementById('btnGuardar')
document.querySelector('.cell-selected')
document.querySelectorAll('#matriz tbody td')
document.createElement('tr')
element.appendChild(child)
element.classList.add('tipo-propios')
element.classList.remove('cell-selected')
element.classList.toggle('fullscreen')
```

### 3.2 Event API
```javascript
// Eventos del navegador
element.addEventListener('click', handler)
element.addEventListener('keydown', handler)
element.addEventListener('keyup', handler)
element.addEventListener('mousedown', handler)
element.addEventListener('mousemove', handler)
element.addEventListener('mouseup', handler)
element.addEventListener('change', handler)
element.addEventListener('input', handler)

// Event object
e.preventDefault()
e.stopPropagation()
e.target
e.clientX / e.clientY
e.key (teclado)
```

### 3.3 Blob API (Exportación CSV)
```javascript
// Crear archivo CSV en memoria
const blob = new Blob([csv], { type: 'text/csv;charset=utf-8;' });
const url = URL.createObjectURL(blob);
const link = document.createElement('a');
link.href = url;
link.download = 'matriz_asignacion_2025-11-17.csv';
link.click();
URL.revokeObjectURL(url); // Liberar memoria
```

### 3.4 Console API
```javascript
// Logging y debugging
console.log('🎬 INICIO DEL SCRIPT');
console.log('🔍 Total celdas:', todasLasCeldas.length);
console.log('✅ Archivo CSV exportado:', data.length, 'registros');
```

### 3.5 JSON API
```javascript
// Serialización de datos
const salida = JSON.stringify({
  division: document.getElementById('division').value,
  area: document.getElementById('area').value,
  selecciones: data,
  totalSelecciones: data.length
}, null, 2);
```

### 3.6 Date API
```javascript
// Generación de timestamp para nombres de archivo
const fecha = new Date().toISOString().split('T')[0]; // 2025-11-17
```

### 3.7 Window API
```javascript
// Alertas y confirmaciones
alert('⚠️ Debe seleccionar un Tipo de Recurso');
confirm('¿Está seguro que desea eliminar TODAS las selecciones?');
```

### 3.8 Element API
```javascript
// Manipulación de elementos
element.getBoundingClientRect() // Posición y dimensiones
element.closest('td') // Búsqueda hacia arriba en el DOM
element.textContent // Contenido de texto
element.innerHTML // Contenido HTML
element.focus() // Dar foco al elemento
element.style.display = 'none' // Estilos inline
```

---

## 4. Estructura de Datos

### 4.1 Configuración de COPEs
```javascript
const copes = [
  'COPE 1', 'COPE 2', 'COPE 3', 'COPE 4', 
  'COPE 5', 'COPE 6', 'COPE 7', 'COPE 8',
  'COPE 11', 'COPE 12', 'COPE 13', 'COPE 14', 
  'COPE 15', 'COPE 16', 'COPE 17', 'COPE 18'
];
// Total: 16 COPEs
```

### 4.2 Estructura de Categorías
```javascript
// Formato: [['código', 'TIPO'], ...]
const columnasAltas = [
  ['A0', 'ORDENES'], 
  ['AT', 'ORDENES_SF']
]; // 2 subcategorías

const columnasCambioDomicilio = [
  ['D1', 'ORDENES'], 
  ['D2', 'ORDENES_SF'], 
  ['D3', 'ORDENES']
]; // 3 subcategorías

const columnasMigraciones = [
  ['TS', 'ORDENES'], 
  ['TV', 'ORDENES_SF']
]; // 2 subcategorías

// Total: 7 subcategorías × 16 COPEs = 112 combinaciones posibles
```

### 4.3 Estructura de Datos de Exportación
```javascript
// Objeto de datos de exportación (JSON y CSV)
{
  Habilitado: "SI" | "NO",      // ⭐ Estado del toggle del COPE
  Division: "(Todas)" | string,
  Area: "(Todas)" | string,
  COPE: "COPE 1" | "COPE 2" | ...,
  Categoria: "Altas" | "CambioDomicilio" | "Migraciones",
  Valor: "A0" | "AT" | "D1" | "D2" | "D3" | "TS" | "TV",
  TipoDeRecurso: "Propios" | "Terceros"
}

// Total: 7 subcategorías × 16 COPEs = 112 celdas de datos
```

### 4.3 Modelo de Datos de Selección
```javascript
{
  Division: "División Norte",      // string
  Area: "Red",                      // string
  COPE: "COPE 5",                   // string
  Categoria: "Altas",               // string
  Valor: "AT",                      // string
  TipoDeRecurso: "Propios"          // string: "Propios" | "Terceros"
}
```

### 4.4 Datos de Estado (Variables Globales)
```javascript
// Variables de estado para selección de celdas
let selectedCell = null;          // HTMLElement | null
let isCtrlPressed = false;        // boolean

// Variables de estado para drag & drop
let dragSourceCell = null;        // HTMLElement | null
let draggedCells = [];            // HTMLElement[]
let isDragging = false;           // boolean
```

---

## 5. Funcionalidades Implementadas

### 5.1 Generación Dinámica de Tabla
**Función:** `generarEncabezado()`, `generarFilas()`

**Proceso:**
1. Lee configuración de arrays (copes, columnasAltas, etc.)
2. Genera encabezado con colspan/rowspan para categorías
3. Crea 16 filas de COPEs con 7 columnas de subcategorías
4. Inyecta radio buttons y event listeners
5. Aplica selecciones aleatorias iniciales

**Complejidad:** O(n × m) donde n = COPEs, m = subcategorías

### 5.2 Sistema de Selección con Validación
**Event listener:** `radio.addEventListener('click')`

**Lógica:**
```javascript
1. Click en radio
   ├─ ¿Ya estaba seleccionado? → Deseleccionar y quitar color
   │
   ├─ ¿Tipo de recurso vacío?
   │  └─ Mostrar alerta + highlight selector + preventDefault
   │
   └─ Tipo válido
      ├─ Aplicar clase CSS (tipo-propios o tipo-terceros)
      ├─ Marcar dataset.wasChecked = 'true'
      └─ Mostrar alert con División, Área, COPE, Categoría, Valor, Tipo
```

### 5.3 Selección de Celdas Tipo Excel
**Función:** `aplicarSeleccionCeldas()`

**Características:**
- Click en celda → Borde negro 3px (`outline`)
- Solo una celda seleccionada a la vez
- Visual feedback con clase `.cell-selected`

**CSS aplicado:**
```css
.cell-selected {
  outline: 3px solid #000 !important;
  outline-offset: -3px !important;
}
```

### 5.4 Drag & Drop con Control
**Eventos:** `keydown`, `keyup`, `mousedown`, `mousemove`, `mouseup`

**Flujo de trabajo:**
```
1. Presionar Ctrl
   └─ Mostrar punto negro (8×8px) en esquina inferior derecha

2. Click en punto negro + arrastre
   ├─ Iniciar drag (isDragging = true)
   ├─ Marcar celda origen (.drag-source)
   └─ Acumular celdas en draggedCells[]

3. Soltar mouse
   ├─ Mostrar confirmación (confirm dialog)
   ├─ Si acepta:
   │  ├─ Aplicar tipo de recurso a todas las celdas
   │  ├─ Marcar radio buttons
   │  └─ Aplicar colores
   └─ Limpiar estados visuales
```

### 5.5 Toggle Switch por COPE (NUEVO - v1.2)
**Función:** Toggle activar/desactivar COPE individualmente

**Componente HTML:**
```html
<label class="cope-toggle">
  <input type="checkbox" checked data-cope="COPE 1">
  <span class="toggle-slider"></span>
</label>
```

**Características:**
- Interruptor visual estilo iOS/Material Design
- Estado por defecto: Activado (checked=true)
- Animación suave de transición (0.3s)
- Color verde cuando está activado (#4caf50)
- Color gris cuando está desactivado (#ccc)

**Lógica de activación/desactivación:**
```javascript
inputToggle.addEventListener('change', function() {
  if (this.checked) {
    tr.classList.remove('cope-row-disabled');  // Activar
  } else {
    tr.classList.add('cope-row-disabled');     // Desactivar
  }
});
```

**Efectos visuales al desactivar:**
- Fila completa con opacidad 0.4
- Celdas con fondo gris (#f0f0f0)
- pointer-events: none (deshabilita interacciones)
- ⭐ **IMPORTANTE:** Los COPEs deshabilitados **SÍ se incluyen** en exportaciones JSON/CSV con el campo `"Habilitado": "NO"`

**Integración con exportaciones:**
```javascript
// Tanto JSON como CSV incluyen TODOS los COPEs (habilitados y deshabilitados)
document.querySelectorAll('#matriz tbody tr').forEach(tr => {
  const toggle = tr.querySelector('.cope-toggle input[type="checkbox"]');
  const copeHabilitado = toggle && toggle.checked ? 'SI' : 'NO';
  
  // ... recopilar datos con campo Habilitado
  data.push({
    Habilitado: copeHabilitado,  // ⭐ Campo que indica estado del toggle
    Division: divisionTexto,
    Area: areaTexto,
    COPE: cope,
    Categoria: item.categoria,
    Valor: item.sub,
    TipoDeRecurso: tipoRecurso
  });
});
```

**Comportamiento de exportaciones:**
- **JSON (botón Guardar):** Incluye TODOS los COPEs con campo `"Habilitado": "SI"` o `"Habilitado": "NO"`
- **CSV (botón Exportar CSV):** Incluye columna "Habilitado" con valores SI/NO para TODOS los COPEs
- **Ventaja:** Permite mantener un registro histórico completo, incluyendo COPEs temporalmente desactivados

### 5.6 Filtrado y Búsqueda
**Input event:** `buscar.addEventListener('input', filtrarCopes)`

**Algoritmo:**
```javascript
function filtrarCopes() {
  const filtro = buscarInput.value.trim().toLowerCase();
  
  document.querySelectorAll('#matriz tbody tr').forEach(tr => {
    const cope = tr.querySelector('th').textContent.toLowerCase();
    tr.style.display = cope.includes(filtro) ? '' : 'none';
  });
  
  clearSearchBtn.classList.toggle('visible', filtro.length > 0);
}
```

**Complejidad:** O(n) donde n = número de filas

### 5.6 Exportación a CSV
**Función:** `recopilarDatos()` + Event listener en `btnExportarCSV`

**Proceso:**
```javascript
1. Recopilar datos
   └─ Iterar filas → Buscar radios checked → Extraer datos

2. Validar datos
   └─ Si array vacío → Alert y return

3. Generar CSV
   ├─ Headers: "Division,Area,COPE,Categoria,Subcategoria,TipoRecurso"
   └─ Rows: data.map(...).join('\n')

4. Crear Blob
   └─ type: 'text/csv;charset=utf-8;'

5. Descargar archivo
   ├─ URL.createObjectURL(blob)
   ├─ link.download = "matriz_asignacion_2025-11-17_Division_Area.csv"
   ├─ link.click()
   └─ URL.revokeObjectURL(url)
```

**Formato CSV:**
```csv
Habilitado,Division,Area,COPE,Categoria,Subcategoria,TipoRecurso
SI,División Norte,Red,COPE 5,Altas,AT,Propios
NO,División Sur,Operaciones,COPE 3,CambioDomicilio,D2,Terceros
```

**⭐ IMPORTANTE:** El CSV incluye TODOS los COPEs (habilitados y deshabilitados) con la columna "Habilitado" indicando SI/NO según el estado del toggle.

### 5.7 Guardado en JSON (Modal)
**Función:** Event listener en `btnGuardar`

**Estructura JSON:**
```json
{
  "division": "División Norte",
  "area": "Red",
  "cope": "",
  "selecciones": [
    {
      "Habilitado": "SI",
      "Division": "División Norte",
      "Area": "Red",
      "COPE": "COPE 5",
      "Categoria": "Altas",
      "Valor": "AT",
      "TipoDeRecurso": "Propios"
    },
    {
      "Habilitado": "NO",
      "Division": "División Sur",
      "Area": "Operaciones",
      "COPE": "COPE 3",
      "Categoria": "CambioDomicilio",
      "Valor": "D2",
      "TipoDeRecurso": "Terceros"
    }
  ],
  "totalSelecciones": 15
}
```

### 5.8 Sistema de Modales
**Modales implementados:**
1. **Modal JSON** - Resultado de guardado
2. **Modal Ayuda** - Manual de usuario con iframe

**Características:**
- Overlay con `rgba(0, 0, 0, 0.6)`
- Cierre con botón × o click fuera del contenido
- Modal ayuda con botón de pantalla completa (⛶)
- Tecla Escape para salir de fullscreen
- Iframe para cargar `manual_usuario_matriz.html`

**Modo pantalla completa:**
```javascript
// Toggle fullscreen
modalContent.classList.toggle('fullscreen');

// CSS
.modal-content.fullscreen {
  width: 100vw !important;
  height: 100vh !important;
  border-radius: 0 !important;
}
```

### 5.9 Limpiar Selecciones
**Dos niveles de limpieza:**

1. **Limpiar Todo** (`btnLimpiar`)
   - Confirmación con dialog
   - Deselecciona todos los radios
   - Quita todas las clases de color
   - Alert de confirmación

2. **Limpiar por COPE** (botón 🗑️ en cada fila)
   - Confirmación con dialog
   - Solo afecta la fila específica
   - Mantiene selecciones de otros COPEs

---

## 6. Patrones de Diseño Utilizados

### 6.1 Module Pattern (Encapsulación)
```javascript
// Todo el código está encapsulado en <script>
// Variables globales limitadas al scope del script
const copes = [...];
let selectedCell = null;
```

### 6.2 Observer Pattern (Event Listeners)
```javascript
// Múltiples componentes escuchan eventos
document.getElementById('btnGuardar').addEventListener('click', handler);
buscarInput.addEventListener('input', filtrarCopes);
```

### 6.3 Factory Pattern (Generación Dinámica)
```javascript
// Función factory para crear elementos DOM
function generarFilas() {
  copes.forEach((cope, ci) => {
    const tr = document.createElement('tr');
    const th = document.createElement('th');
    // ... crear estructura completa
    tbody.appendChild(tr);
  });
}
```

### 6.4 State Pattern (Gestión de Estado)
```javascript
// Estado de la aplicación en variables
let isCtrlPressed = false;
let isDragging = false;
let selectedCell = null;
```

---

## 7. Optimizaciones y Buenas Prácticas

### 7.1 Rendimiento
- **Event delegation** para celdas (evento en celda, no en radio)
- **Query Selectors optimizados** (`#id` antes que `.class`)
- **Caching de elementos** cuando es posible
- **Evitar reflows** (modificar CSS en lote)

### 7.2 Accesibilidad
- **Labels para inputs** (for + id)
- **Title attributes** en botones
- **Focus management** (focus() después de alerta)
- **Keyboard support** (Ctrl para drag, Escape para fullscreen)

### 7.3 UX
- **Feedback visual** inmediato (colores, animaciones)
- **Confirmaciones** para acciones destructivas
- **Mensajes informativos** (alerts con iconos emoji)
- **Loading states** implícitos (CSV descarga inmediata)

### 7.4 Mantenibilidad
- **Configuración centralizada** (arrays en top del script)
- **Funciones reutilizables** (`recopilarDatos()`)
- **Comentarios descriptivos** en secciones clave
- **Naming conventions** claros (btnGuardar, modalJSON)

---

## 8. Dependencias y Compatibilidad

### 8.1 Dependencias Externas
**Ninguna** - La aplicación es completamente standalone

### 8.2 Archivos Relacionados
- `manual_usuario_matriz.html` - Manual de usuario (cargado en iframe)
- `PROPUESTA_DB2_MATRIZ_ASIGNACION.md` - Documentación de base de datos (referencia)

### 8.3 Compatibilidad de Navegadores

**Completamente compatible:**
- Chrome 90+ ✅
- Edge 90+ ✅
- Firefox 88+ ✅
- Safari 14+ ✅
- Opera 76+ ✅

**Características que requieren navegadores modernos:**
- CSS Grid (2017+)
- CSS Variables (2016+)
- ES6+ Arrow Functions (2015+)
- Template Literals (2015+)
- Spread Operator (2015+)
- Blob API (2012+)

**No compatible con:**
- Internet Explorer 11 ❌ (no soporta ES6)
- Navegadores anteriores a 2017 ❌

---

## 9. Seguridad

### 9.1 Consideraciones de Seguridad
- **No hay backend** - Todo es cliente, no hay riesgo de inyección SQL
- **No hay autenticación** - Aplicación de uso local/interno
- **Sin localStorage** - No se almacenan datos sensibles
- **Sin cookies** - No se rastrea al usuario
- **Sin AJAX** - No hay comunicación con servidores externos

### 9.2 Validaciones Implementadas
```javascript
// Validación de tipo de recurso antes de marcar
if (!tipoRecurso) {
  e.preventDefault();
  alert('Debe seleccionar un Tipo de Recurso');
  return false;
}

// Validación de datos antes de exportar
if (data.length === 0) {
  alert('No hay datos para exportar');
  return;
}

// Confirmación de acciones destructivas
const confirmacion = confirm('¿Está seguro que desea eliminar?');
if (!confirmacion) return;
```

---

## 10. Métricas del Código

### 10.1 Estadísticas Generales
```
Total líneas:              ~1,300
├─ HTML:                   ~110 líneas
├─ CSS:                    ~550 líneas (incluye toggle switch)
└─ JavaScript:             ~720 líneas (incluye toggle logic)

Funciones definidas:       8
Nuevas funcionalidades:    Toggle switch por COPE (v1.2)
├─ generarEncabezado()
├─ generarFilas()
├─ aplicarSeleccionCeldas()
├─ filtrarCopes()
├─ recopilarDatos()
├─ cerrarModal()
├─ cerrarModalAyuda()
└─ Event handlers anónimos: ~20

Event Listeners:           ~30
Variables globales:        ~10
Constantes:                ~4
```

### 10.2 Complejidad Ciclomática
- **generarEncabezado():** Baja (2)
- **generarFilas():** Media (5)
- **recopilarDatos():** Media (4)
- **Event handlers:** Baja-Media (2-6)

### 10.3 Tamaño del Archivo
- **Sin comprimir:** ~50 KB
- **Gzipped:** ~12 KB (estimado)

---

## 11. Flujo de Ejecución

### 11.1 Carga Inicial
```
1. Navegador carga HTML
   ├─ Parsea <head>
   ├─ Carga y aplica CSS embebido (~480 líneas)
   └─ Renderiza <body> inicial (vacío)

2. Ejecuta <script>
   ├─ Define constantes (copes, subcategorias)
   ├─ Define variables de estado (selectedCell, etc.)
   ├─ Ejecuta generarEncabezado()
   │  └─ Crea 2 filas de encabezados con colspan/rowspan
   ├─ Ejecuta generarFilas()
   │  └─ Crea 16 filas × 7 columnas = 112 celdas
   │     └─ Cada celda con radio + event listeners
   ├─ Ejecuta aplicarSeleccionCeldas()
   │  └─ Aplica listeners de selección a todas las celdas
   └─ Registra ~30 event listeners globales

3. Página lista para interacción
```

### 11.2 Interacción del Usuario
```
Usuario selecciona Tipo de Recurso
  ↓
Hace clic en radio button
  ↓
Event listener valida tipo de recurso
  ↓
Si válido: Aplica color + marca radio + muestra alert
Si inválido: Previene selección + muestra alerta + resalta selector
```

### 11.3 Exportación CSV
```
Usuario hace clic en "Exportar CSV"
  ↓
Llama a recopilarDatos()
  ↓
Itera todas las filas y extrae selecciones
  ↓
Valida que haya datos
  ↓
Genera string CSV
  ↓
Crea Blob en memoria
  ↓
Genera URL temporal
  ↓
Crea link <a> dinámico + click()
  ↓
Navegador descarga archivo
  ↓
Limpia URL temporal (revokeObjectURL)
```

---

## 12. Mejoras Futuras Posibles

### 12.1 Funcionalidad
- [ ] Guardar configuración en localStorage
- [ ] Importar CSV existente
- [ ] Exportar a Excel (XLSX) con estilos
- [ ] Deshacer/Rehacer (Undo/Redo)
- [ ] Historial de cambios
- [ ] Validación de reglas de negocio
- [ ] Integración con API REST

### 12.2 UX/UI
- [ ] Tema oscuro (Dark mode)
- [ ] Tooltips informativos
- [ ] Drag & drop de archivos
- [ ] Preview de cambios antes de guardar
- [ ] Indicador de progreso para operaciones largas
- [ ] Notificaciones toast en lugar de alerts

### 12.3 Técnico
- [ ] Migrar a TypeScript
- [ ] Usar framework (React/Vue/Angular)
- [ ] Separar CSS en archivo externo
- [ ] Separar JS en módulos
- [ ] Build system (Webpack/Vite)
- [ ] Testing unitario (Jest)
- [ ] Testing E2E (Playwright)
- [ ] CI/CD pipeline

### 12.4 Performance
- [ ] Virtual scrolling para muchos COPEs
- [ ] Lazy loading de datos
- [ ] Web Workers para procesamiento pesado
- [ ] Service Worker para offline support

---

## 13. Glosario Técnico

| Término | Definición |
|---------|-----------|
| **COPE** | Centro de Operaciones - Unidad organizativa |
| **SPA** | Single Page Application - Aplicación de una sola página |
| **Blob** | Binary Large Object - Objeto binario para archivos |
| **CSV** | Comma-Separated Values - Formato de datos tabulares |
| **DOM** | Document Object Model - Árbol de elementos HTML |
| **Event Listener** | Función que escucha eventos del usuario |
| **Arrow Function** | Sintaxis ES6 para funciones: `() => {}` |
| **Template Literal** | Strings con interpolación: `` `Hola ${nombre}` `` |
| **Destructuring** | Extraer valores de arrays/objetos: `[a, b] = arr` |
| **Spread Operator** | Expandir arrays: `[...arr1, ...arr2]` |
| **querySelector** | API para buscar elementos en el DOM |
| **Flexbox** | Sistema de layout CSS flexible |
| **Grid** | Sistema de layout CSS en cuadrícula |
| **Media Query** | Regla CSS para responsive design |
| **Viewport** | Área visible del navegador |
| **Sticky** | Posicionamiento CSS que se mantiene visible |
| **z-index** | Orden de apilamiento de elementos CSS |

---

## 14. Conclusión

`mant_agrupador_tarea.html` es una aplicación web **moderna, eficiente y auto-contenida** que demuestra el poder de las tecnologías web estándar (HTML5, CSS3, ES6+) sin necesidad de frameworks externos.

### Fortalezas
✅ **Simplicidad:** Un solo archivo, fácil de distribuir  
✅ **Moderno:** Usa características ES6+ y CSS3 avanzado  
✅ **Funcional:** Cubre todos los requisitos del negocio  
✅ **Mantenible:** Código bien estructurado y comentado  
✅ **Responsive:** Se adapta a diferentes tamaños de pantalla  

### Áreas de Mejora
⚠️ **Escalabilidad:** ~1,200 líneas en un archivo puede ser difícil de mantener  
⚠️ **Testing:** No hay tests automatizados  
⚠️ **Persistencia:** Los datos no se guardan automáticamente  
⚠️ **Compatibilidad:** No funciona en IE11 o navegadores muy antiguos  

---

**Documento generado:** Noviembre 2025  
**Autor:** Documentación técnica del sistema  
**Versión:** 1.0
