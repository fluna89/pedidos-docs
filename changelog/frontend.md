# Changelog - Pedidos Project

Registro de funcionalidades implementadas y planificadas.

## [0.23.0] - 2026-04-03

### Nuevo — Configuración de tienda (admin)

- **AdminConfigPage**: toggle de apagado de emergencia + editor de mensaje de comanda
- **AdminHorariosPage**: configuración de horarios multi-rango por día con vista visual
- **StoreStatusContext + useStoreStatus**: polling periódico del estado de la tienda (abierta/cerrada/emergencia)
- **StoreClosedBanner**: banner visible cuando la tienda está cerrada o en emergencia
- **CartContext**: bloquea checkout cuando la tienda está cerrada
- **AdminLayout**: links a páginas de configuración y horarios en sidebar

### Nuevo — Teléfono de contacto en pedidos

- **RegisterPage**: campo de teléfono obligatorio en registro
- **CheckoutPage**: toggle "Mi teléfono" / "Usar otro teléfono" para usuarios registrados, input simple para invitados
- **AccountSection**: edición de teléfono en "Mi cuenta" con `updateUser` del contexto
- **AdminPedidosPage**: teléfono del cliente como link `tel:` en detalle de pedido
- **AdminCargarPedidoPage**: campo de teléfono en pedidos de mostrador
- **AuthContext**: `register()` acepta `phone`, expone `updateUser()` para actualizar datos en memoria

### Nuevo — Recuperación de contraseña

- **ResetPasswordPage**: nueva página para resetear contraseña con token
- **LoginPage**: enlace a flujo de recuperación de contraseña

### Otros

- **.env**: `localhost:3000` para SAM local (era IP de LAN obsoleta)
- **.env.production**: URL de API Gateway de AWS para builds de producción
- **vite.config.js**: `host: true` para acceso por red
- **handlers.js + api.js**: endpoints de store config

## [0.22.1] - 2026-03-26

### Mejorado — Visualización de combos en detalle de pedido

- Los pasos del combo ahora se muestran **por separado** con label, sabores y extras
- Borde lateral indigo para distinguir visualmente los pasos del combo
- Los **comentarios por paso** se muestran en caja ámbar debajo de cada step (requiere backend v0.3.1+)

## [0.22.0] - 2026-03-26

### Nuevo — Comentarios de pedido al backend

- **CheckoutPage**: envía `comment` (comentario general) y `customerName` en el payload de creación de pedido
- Los comentarios por item ya se enviaban desde el carrito; ahora el backend los persiste y devuelve

## [0.21.2] - 2026-03-26

### Corregido — Cursor en botón cerrar de dialogs

- Agregado `cursor-pointer` al botón de cerrar (X) del componente `Dialog` para que muestre cursor clickeable en desktop

## [0.21.1] - 2026-03-26

### Mejorado — Legibilidad de items en detalle de pedido

- **Items más claros**: cantidad en badge, nombre en negrita, sabores y extras con tamaño legible para cocina/delivery
- **Comentarios destacados**: los comentarios por item se muestran en fondo ámbar para que no pasen desapercibidos
- **Nota general del pedido**: si existe `order.comment`, se muestra en un bloque destacado debajo de los items

## [0.21.0] - 2026-03-26

### Nuevo — Detalle de pedido en admin

- **Dialog de detalle**: click en una tarjeta kanban o botón ojo en la lista abre un modal con toda la información del pedido
- **Contenido del detalle**: cliente, tipo de entrega, dirección completa, lista de productos con sabores/extras/comentarios, subtotal, envío, cupón, puntos canjeados, total, método y estado de pago, motivo de cancelación con imagen si aplica

## [0.20.3] - 2026-03-26

### Mejorado — Ajustes UI admin pedidos

- **Kanban sin botones avanzar/retroceder**: eliminados de las tarjetas kanban; el cambio de estado se hace con drag & drop. El botón de cancelar permanece
- **Cursor pointer en botones**: agregado `cursor-pointer` al componente `Button` base para que todos los botones muestren el cursor correcto al pasar el mouse
- **Toggle Lista/Kanban más claro**: cambiado de variante `secondary` a `default` para el botón activo, generando mayor contraste visual entre activo e inactivo

## [0.20.2] - 2026-03-26

### Corregido — Bugs en vista kanban de admin

- **Persistencia de vista**: al refrescar la página de pedidos, la vista seleccionada (Lista/Kanban) se mantiene gracias a `localStorage`
- **Drag & drop optimistic update**: corregido bug donde al arrastrar una tarjeta el cambio no se reflejaba visualmente hasta el próximo polling. Causa: `Number(draggableId)` generaba mismatch de tipo con IDs string del backend

## [0.20.1] - 2026-03-25

### Mejorado — Mapeo visual de estados de pedido

- **Vista cliente**: la barra de progreso ya no muestra "Pendiente"; el primer paso es "Conf." (amarillo = esperando confirmación, verde = confirmado). `confirmado` y `en_preparacion` del backend se unifican visualmente como "Prep."
- **Vista cliente historial**: `pendiente` se muestra como "Confirmado", `confirmado` como "En preparación"
- **Admin kanban**: la columna "En preparación" agrupa `confirmado` + `en_preparacion` (diferenciados por badge de color). El admin puede retroceder `confirmado` → `pendiente`
- **Nuevo color de estado** `confirmado` (esmeralda) para diferenciarlo de `pendiente` (amarillo) en las badges del admin
- **StrictMode configurable**: nueva variable `VITE_STRICT_MODE` en `.env` para activar/desactivar React StrictMode sin tocar código

## [0.20.0] - 2026-03-24

### Nuevo — Conexión con backend real

- **API service** (`src/services/api.js`): agregado header `Authorization: Bearer {token}` automático desde localStorage y unwrapping de respuestas `{ data: ... }` del backend
- **Handlers reales** (`src/services/handlers.js`): nuevo módulo que reemplaza `src/mocks/handlers.js` con llamadas HTTP al backend Python (`pedidos-backend`)
- **AuthContext**: login, registro, invitado, Google y recuperación de contraseña ahora llaman a los endpoints reales `/api/auth/*`
- **16 archivos migrados**: todos los imports de `@/mocks/handlers` → `@/services/handlers`; los imports de `orderStatusLabels` desde `@/mocks/data` → `@/services/handlers`
- **Variable de entorno**: `VITE_API_URL` en `.env` (default: `http://localhost:8000/api`)
- Los mocks (`src/mocks/`) se conservan como referencia pero ya no son importados por la aplicación

## [0.19.0] - 2026-03-18

### Nuevo — Cargar pedido (mostrador / WhatsApp)

- **Nueva página admin** "Cargar pedido" (`/admin/cargar-pedido`): permite al operador crear pedidos para clientes de mostrador o WhatsApp
- **Búsqueda de cliente**: busca usuarios registrados por nombre o email con debounce; al vincular un usuario, acumula puntos de fidelidad
- **Tipo de pedido**: toggle retiro en local / delivery con formulario de dirección y costo de envío manual
- **Catálogo completo**: muestra todos los productos (incluyendo `counterOnly`), colapsable por categoría, con badge de cantidad en carrito
- **Selector de producto**: dialog con formato, sabores, extras, comentario y cantidad — misma lógica que el catálogo público
- **Combos**: reutiliza `ComboWizard` dentro de un dialog para la selección paso a paso
- **Cupón de descuento**: el operador puede aplicar cupones con validación
- **Medio de pago**: selector con todos los métodos disponibles
- **Resumen lateral**: sticky sidebar con items editables (cantidad ±, eliminar), subtotal, envío, descuento y total
- **Validaciones inline**: mensajes de ayuda cuando falta cliente, dirección o medio de pago
- **Estado de pago**: efectivo = `pagado` inmediato; otros métodos = `pendiente_pago`
- **Puntos**: se acreditan automáticamente si el usuario está vinculado y el pago es inmediato
- Nuevos handlers: `adminSearchUsers()`, `getCounterMenu()`, `adminCreateOrder()`
- Mock users movidos a `data.js` para compartir entre AuthContext y handlers
- Link "Cargar pedido" agregado al sidebar del admin con ícono PlusCircle

## [0.18.1] - 2026-03-18

### Mejorado — Diálogos de eliminación con protección de uso

- **Listas de sabores**: botón eliminar siempre habilitado; si la lista está en uso por productos, muestra diálogo con los productos que la referencian y un tip de cómo solucionarlo
- **Productos**: al intentar eliminar un producto usado en combos, muestra diálogo con los combos que lo referencian y un tip de cómo solucionarlo
- Nuevo handler `adminGetProductUsage()` que detecta qué combos referencian un producto
- Estilo consistente entre ambos diálogos: ícono centrado, panel amber con items, card de ayuda con 💡

## [0.18.0] - 2026-03-17

### Nuevo — Productos "solo para combos" (comboOnly)

- **Flag `comboOnly`**: productos marcados como "solo para combos" no aparecen en el menú público, pero están disponibles como opciones dentro de un combo
- **Admin**: checkbox "Solo para combos" (purple) en la sección de disponibilidad del formulario de producto
- **Tabla de productos**: badge "Solo combo" en purple, fila con opacidad reducida (como los pausados)
- **Previews**: warning "⚠ Solo para combos — no aparecerá en el menú" en ambas zonas de preview
- **Datos mock**: agregados Cookie chocolate y Cookie vainilla como productos de ejemplo comboOnly
- El filtro en `getMenu()` y `getMenuByCategory()` ya excluía `comboOnly` — ahora hay productos reales que lo usan
- `adminGetBaseProducts()` ahora retorna el campo `comboOnly` para que el picker de combos los incluya

## [0.17.5] - 2026-03-17

### Mejorado — Productos: cantidad persistente y visibilidad del carrito

- **Modo carrito**: al agregar un producto simple (ej. Coca Cola), la card pasa a modo "en carrito" mostrando la cantidad actual con controles ±
- El botón **−** con cantidad 1 elimina el item del carrito
- Botón de **eliminar** (🗑) para quitar el producto del carrito directamente desde la card del menú
- El contador ya no se reinicia — refleja el estado real del carrito en todo momento
- **Productos no simples** (combos, helados, etc.): badge "X en el carrito" en la card cuando ya hay unidades agregadas

## [0.17.4] - 2026-03-17

### Mejorado — Ajustes UX en panel admin

- **Listas**: eliminado el click en fila para editar, consistente con Productos y Combos (se usa el botón de acción)
- **Pedidos kanban**: cursor `grab` al pasar sobre las tarjetas, `grabbing` mientras se arrastra, indicando que son draggables
- **Pedidos lista**: nuevos pedidos resaltados con borde izquierdo verde vía `box-shadow inset` (funciona en `<tr>`), fondo `bg-green-100` en light y `bg-green-900/40` en dark para mayor contraste

## [0.17.3] - 2026-03-17

### Corregido — Preview de combo sin flash, sincronizada con la UX del cliente

- Extraído `ComboOverview` como componente interno reutilizado tanto por la preview del admin como por la fase overview del wizard del cliente — **una sola fuente de verdad** para ambos
- El botón "Armar combo" en la preview inline abre el dialog interactivo completo para probar la experiencia del cliente
- Prop `previewMode` desactiva el botón "Agregar al carrito" en el dialog de preview
- `ComboWizard` acepta prop `preview`: si es `true`, renderiza solo `ComboOverview` sin hooks, sin estado, sin carga async — React lo re-renderiza in-place sin desmontar
- La lógica interactiva del wizard queda en `ComboWizardFull` (componente interno), totalmente aislada
- Eliminado el card estático manual previo y el debounce, ya que el problema de raíz era el `key={JSON.stringify()}` que forzaba remontaje
- Eliminado el debounce anterior que solo reducía la frecuencia del flash sin eliminarlo

## [0.17.2] - 2026-03-17

### Mejorado — Layout desktop-first para AdminCombosPage

- **Lista**: reemplazadas las tarjetas por tabla a ancho completo con columnas de nombre, precio, pasos, estado y acciones (consistente con AdminProductosPage y AdminListasPage)
- **Formulario**: layout de 2 columnas en `xl` — formulario a la izquierda, preview sticky del `ComboWizard` a la derecha; campos de nombre/descripción y precio/tipo en grids de 2 columnas; pasos en grid `lg:grid-cols-2`
- **Preview adaptativo**: en desktop se muestra inline al costado del formulario; en mobile se mantiene el dialog con botón "Vista previa del cliente"
- Removido `max-w-2xl` que restringía el ancho artificialmente

## [0.17.1] - 2026-03-17

### Mejorado — Vista previa de combo en admin

- **Preview dialog**: botón "Vista previa del cliente" en el formulario de combo abre un dialog con el `ComboWizard` completo (overview → pasos → resumen), tal como lo vería el cliente
- Se muestra solo cuando el combo tiene nombre y al menos un paso con producto

## [0.17.0] - 2026-03-17

### Nuevo — Panel de administración de combos

- **AdminCombosPage**: nueva página dedicada para crear, editar, pausar y eliminar combos
- **Formulario completo**: nombre, descripción, tipo de precio (fijo / descuento %), pasos con etiqueta, productos elegibles (multi-select) y cantidad fija opcional
- **Reordenar pasos**: botones arriba/abajo para cambiar el orden de los pasos del combo
- **Navegación admin**: nuevo ítem "Combos" (icono 🎁) en el sidebar, ruta `/admin/combos`
- **Separación de concerns**: los combos ya no aparecen en AdminProductosPage ni en su selector de categorías
- **`adminGetBaseProducts()`**: nuevo handler que devuelve productos no-combo para el picker de pasos

## [0.16.1] - 2026-03-16

### Corregido — Mejoras al wizard de combos

- **Tope de unidades en pasos de combo**: los pasos con `unitCount` (ej. docena de empanadas) ahora limitan correctamente la cantidad máxima de items seleccionables
- **Preservar configuración al volver atrás**: navegar hacia atrás o editar un paso desde el resumen conserva las selecciones previas (formato, sabores, extras, comentario)
- **Extras suman al precio fijo**: en combos de precio fijo, los extras de cada paso se suman al precio base; la tarjeta y el overview muestran "Desde $X"
- **stepSubPhase derivado**: reemplazado `useState` + `useEffect` por estado derivado para eliminar warning de ESLint (`set-state-in-effect`)

## [0.16.0] - 2026-03-16

### Nuevo — Combos rediseñados con wizard por pasos

- **Nuevo modelo de combos**: cada combo define `steps[]` con `productIds` (productos elegibles) y `comboPrice` (precio fijo o descuento %)
- **ComboWizard**: componente paso a paso — overview → selección de producto (si hay múltiples opciones) → configuración (formato, sabores, extras) → resumen con edición por paso
- **Pricing flexible**: combos con precio fijo (`type: 'fixed'`) o descuento porcentual (`type: 'discount'`) sobre el total individual
- **Resumen interactivo**: pantalla final muestra cada paso configurado con botón de edición individual, precio combo y ahorro
- **Carrito actualizado**: nuevo formato `comboSteps[]` reemplaza `comboSelections[]`, muestra detalle de cada producto del combo
- **ProductCard**: combos muestran precio fijo o "X% OFF", botón "Armar combo" en lugar de "Elegir"
- **Catálogo**: productos `comboOnly` se ocultan del menú público (preparado para productos exclusivos de combo)
- **3 combos de ejemplo**: "Combo 2× 1/4 kg" (precio fijo), "Combo Docena de empanadas" (10% dto.) y "Combo Pizza + Helado" (15% dto., multi-producto)

## [0.15.7] - 2026-03-16

### Removido — Modo toggle (chips) de selección de sabores

- **Código muerto eliminado**: ningún producto usaba el modo toggle (todos usan `flavorMode: 'quantity'`), se eliminó el branch completo de UI, el estado `selectedFlavors`, la función `toggleFlavor` y la variable `maxFlavors`
- **Simplificación**: `flavorsComplete` y `handleAdd` ya no contemplan el caso toggle, se evalúan directamente en modo cantidad

## [0.15.6] - 2026-03-16

### Mejorado — Items pausados visibles pero no seleccionables para el cliente

- **Flavors pausados tachados**: en la vista del cliente, los items pausados aparecen tachados, atenuados y con texto "No disponible", sin posibilidad de seleccionarlos
- **Aplica a los 3 modos**: toggle (chips), cantidad sin precio, y cantidad con precio individual
- **`getFlavors` devuelve todos**: los items pausados ya no se ocultan, se muestran como no disponibles

## [0.15.5] - 2026-03-16

### Corregido — Precio dinámico en listado de productos

- **`adminGetAllProducts` recalcula `unitPricing`**: al cambiar una lista de normal a "precio individual" (o viceversa), el listado de productos refleja el cambio inmediatamente sin necesidad de re-guardar cada producto
## [0.15.3] - 2026-03-16

### Corregido — Items pausados visibles en admin, ocultos en catálogo

- **`getFlavors` filtra pausados**: el catálogo público ya no muestra opciones pausadas
- **Nuevo `adminGetFlavors`**: handler admin que devuelve todas las opciones (incluidas pausadas)
- **Admin productos**: la lista de opciones en el formulario muestra items pausados atenuados con badge "Pausado"
- **Admin listas**: usa `adminGetFlavors` para ver todas las opciones

## [0.15.2] - 2026-03-16

### Corregido — Label "Eliminado" sin tachado

- El badge "Eliminado" ya no hereda el tachado del nombre del item

## [0.15.1] - 2026-03-16

### Mejorado — Labels de estado en items de listas

- **Label "Eliminado"**: badge rojo en items marcados para eliminación
- **Label "Pausado"**: badge ámbar en items pausados

## [0.15.0] - 2026-03-16

### Mejorado — Eliminación local y validación de precios en listas

- **Eliminación requiere guardado**: eliminar un item ahora es un cambio local (se muestra tachado con borde rojo y botón "Deshacer"), solo se persiste al guardar
- **Validación de precios**: si la lista tiene "Precio individual por item" activo, no se puede guardar si alguna opción no tiene precio asignado (banner rojo + botón deshabilitado)
- **Contador de opciones actualizado**: refleja la cantidad excluyendo items marcados para eliminación

## [0.14.9] - 2026-03-16

### Mejorado — Ajustes UX en detalle de listas

- **Precio por item reactivo**: al activar/desactivar "Precio individual por item", los campos de precio de cada opción aparecen/desaparecen inmediatamente (sin necesidad de guardar primero)
- **Contador de opciones reubicado**: "N opciones" ahora aparece como encabezado de la lista de opciones, más contextual
- **Botón "Cancelar"**: aparece cuando hay cambios sin guardar, revierte todos los cambios locales (nombre, checkbox, pausas, precios)

## [0.14.8] - 2026-03-16

### Mejorado — Indicadores visuales de cambios sin guardar

- **Banner de aviso**: aparece un banner ámbar "Tenés cambios sin guardar" cuando hay modificaciones pendientes
- **Botón "Guardar" resaltado**: cambia de `outline` a botón primario (relleno) cuando hay cambios sin guardar
- **Borde ámbar en items modificados**: las filas con cambios locales (pausa o precio) muestran un borde izquierdo ámbar

## [0.14.7] - 2026-03-16

### Mejorado — Guardado unificado en detalle de listas

- **Botón "Guardar" siempre visible**: ya no se oculta cuando no hay cambios
- **Cambios locales**: pausar/reactivar items y editar precios ahora son cambios locales que solo se persisten al presionar "Guardar"
- **Guardado unificado**: el botón guarda metadatos de la lista (nombre, precio por item) y todos los cambios de items (pausas, precios) en una sola acción

## [0.14.6] - 2026-03-16

### Agregado — Edición de precio y pausa de items en listas

- **Edición inline de precio**: en listas con "Precio individual por item", cada opción muestra un input de precio editable (se guarda al perder foco o con Enter)
- **Pausar/reactivar items**: nuevo botón de pausa en cada opción — los items pausados se muestran atenuados y tachados
- **Handler `adminUpdateFlavor`**: nuevo handler mock para actualizar precio y estado `paused` de opciones individuales

## [0.14.5] - 2026-03-16

### Mejorado — Botón Guardar explícito en detalle de listas

- **Botón "Guardar"**: reemplaza el auto-guardado por un botón explícito que aparece solo cuando hay cambios sin guardar en nombre o precio por item

## [0.14.4] - 2026-03-16

### Mejorado — Edición inline en detalle de listas

- **Botón "Editar" en tabla navega al detalle**: en vez de abrir un dialog, el botón lleva a la página de detalle de la lista
- **Edición inline de nombre y precio por item**: en el detalle, el nombre de la lista y el checkbox "Precio individual por item" se editan directamente sin dialog (auto-guardado al perder foco o al cambiar el checkbox)
- **Dialog simplificado**: el dialog ahora se usa solo para crear nuevas listas

## [0.14.3] - 2026-03-16

### Mejorado — Columna productos en listas

- **Pills por producto**: la columna "Productos" ahora muestra cada producto como un badge/pill individual en vez de texto plano separado por comas

## [0.14.2] - 2026-03-16

### Mejorado — Productos vinculados en listas de opciones

- **Columna "Productos" en tabla**: muestra los nombres de los productos que usan cada lista
- **Banner informativo en detalle**: al entrar a una lista usada por productos, se muestra un aviso azul con los nombres y la advertencia de que los cambios impactan a todos
- **Handler `adminGetFlavorSources`**: ahora incluye `usedBy` con id y nombre de productos vinculados (directos y combo items)
- **Protección de eliminación**: no se puede eliminar una lista que esté en uso por algún producto

## [0.14.1] - 2026-03-16

### Mejorado — Listas de opciones

- **Eliminado campo "Descripción"** de las listas de opciones: removido de tabla, dialogs de crear/editar, y vista de detalle

## [0.14.0] - 2026-03-16

### Implementado — Gestión de listas de opciones (admin)

- **AdminListasPage**: nueva página independiente para administrar listas de opciones (sabores, gustos, etc.)
  - Vista de tabla con nombre, descripción, cantidad de opciones y flag de precio por item
  - Crear y editar listas mediante dialog (nombre, descripción, checkbox "Precio individual por item")
  - Eliminar listas vacías con confirmación
  - Vista de detalle: listado completo de opciones con agregar/eliminar inline
  - Soporte de precio opcional por opción cuando la lista tiene `hasItemPrices`
- **Sidebar admin**: nuevo ítem "Listas de opciones" con ícono de lista
- **Ruta**: `/admin/listas`
- **Formulario de producto simplificado**: se removió el CRUD inline de listas; queda solo el selector + link a la página de Listas + gestión inline de opciones
- **CLAUDE.md**: agregada regla de workflow (commit + push con cada cambio)

## [0.13.1] - 2026-03-16

### Mejorado — UX del listado y formulario de productos

- **Indicadores en lista**: badges compactos debajo del nombre muestran cantidad de formatos (violeta) y extras (ámbar) de cada producto
- **Detalle expandible**: botón chevron en cada fila despliega lista inline de formatos con precios y extras con recargos
- **Precio por item en lista**: productos con `unitPricing` muestran "Precio por item" en vez de $0
- **Precio según formato en lista**: productos con múltiples formatos muestran "Según formato" en vez del precio del primer formato
- **Vista previa móvil**: en pantallas menores a `xl`, la tarjeta de preview aparece al final del formulario (antes solo se veía en desktop)
- **Badge "Solo mostrador"**: renombrado de "Mostrador" a "Solo mostrador" para mayor claridad
- **Datos mock enriquecidos**: productos con combinaciones de `paused`, `counterOnly` y múltiples formatos para cubrir todos los estados posibles
  - Banana Split → pausado
  - Sundae clásico → solo mostrador
  - Milkshake grande → pausado + solo mostrador
  - Pizza muzzarella → 2 formatos (Porción/Entera) + 2 extras

## [0.13.0] - 2026-03-16

### Implementado — CRUD de productos (admin) + componente compartido

- **AdminProductosPage**: nueva página de gestión de productos para el panel admin
  - Listado con nombre, categoría, precio, tipo, estado y acciones
  - Formulario de creación/edición con 2 arquetipos: **simple** y **con opciones a elegir** (unifica el antiguo "slots" y "porciones")
  - Productos pueden usar precio único o expandir a variantes con formatos (para cualquier arquetipo)
  - Gestión inline de sabores/opciones (agregar con precio opcional, eliminar) por fuente
  - CRUD de fuentes de opciones (crear, renombrar, eliminar) con flag **"Precio individual por item"**
  - El flag `hasItemPrices` de la lista determina si el precio va en cada opción (ej: empanadas) o en el formato (ej: helado)
  - Badge de estado: `● Activo` (verde) / `● Pausado` (gris) + botón de acción separado (⏸/▶)
  - Checkboxes independientes: **Pausado** (no aparece en menú) y **Solo mostrador** (no aparece online)
  - Vista previa de tarjeta (columna lateral, sticky) con "Desde $X" para multi-formato
  - Vista previa de detalle (dialog) usando el componente compartido
- **ProductDetailView**: nuevo componente compartido (`src/components/catalog/ProductDetailView.jsx`)
  - Unifica vista de detalle entre página del cliente y preview del admin
  - Toda la lógica interactiva (formatos, sabores en 3 modos, combos, extras, comentario, agregar al carrito)
  - Prop `preview` para deshabilitar agregar al carrito
- **ProductDetailPage refactorizado**: reducido de ~660 a ~95 líneas, delega toda la UI a `ProductDetailView`
- **ProductCard mejorado**: productos simples muestran controles inline; multi-formato muestra "Desde $X" con precio mínimo
- **Inputs numéricos**: eliminados spinners nativos (CSS), selección automática al enfocar
- **Mock handlers**: `adminGetAllProducts`, `adminGetCategories`, `adminCreateProduct`, `adminUpdateProduct`, `adminDeleteProduct`, `adminToggleProduct`, `getFlavors` (por fuente), `adminAddFlavor`, `adminDeleteFlavor`, CRUD de fuentes de sabores con `hasItemPrices`

## [0.12.4] - 2026-03-14

### Implementado — Drag & drop en kanban

- **Drag & drop**: las tarjetas del kanban se pueden arrastrar entre columnas para cambiar el estado del pedido
- **Librería**: `@hello-pangea/dnd` (fork mantenido de react-beautiful-dnd)
- **Feedback visual**: columna destino resaltada en azul al arrastrar, tarjeta con rotación y sombra
- **Actualización optimista**: el estado cambia inmediatamente con rollback automático si falla
- **Handler `adminSetOrderStatus`**: nuevo mock handler para establecer estado arbitrario

## [0.12.3] - 2026-03-14

### Implementado — Cancelación con motivo e imagen

- **Diálogo de cancelación**: al cancelar un pedido se abre un modal que solicita motivo obligatorio y permite adjuntar una imagen (opcional)
- **Componente Dialog (shadcn/ui)**: nuevo componente `dialog.jsx` basado en Radix UI Dialog
- **Datos de cancelación**: el pedido almacena `cancelReason` e `cancelImageUrl`
- **Visualización**: las cards de pedidos cancelados muestran el motivo y la imagen adjunta

## [0.12.2] - 2026-03-10

### Mejorado — Kanban y alertas de pedidos

- **Kanban 1 col/estado**: 5 columnas individuales (Pendiente, En preparación, Listo, En camino, Entregado). Cancelados en sección colapsable aparte
- **Cards compactas**: menor ancho mínimo (150px), texto reducido, layout más denso
- **Ver más/menos**: las columnas muestran máx. 4 cards, con botón expandir/colapsar
- **Revertir cancelación y entrega**: botón Undo en pedidos cancelados y entregados (tabla, kanban y sección cancelados)
- **Alertas de nuevo pedido**: polling cada 10s, sonido audible (Web Audio API) y destaque visual (pulse verde) en cards/filas nuevas durante 8s
- **Mute/Sonido**: toggle para silenciar alertas sonoras
- **Simular pedido**: botón "Simular" para testing que genera pedidos aleatorios

## [0.12.1] - 2026-03-09

### Corregido

- **Redirección admin al login**: GuestRoute ahora redirige a `/admin` cuando el usuario es admin, evitando que el redirect a `/` gane la carrera con `navigate('/admin')`

## [0.12.0] - 2026-03-09

### Implementado — Panel Admin (Fase 1a)

- **Layout admin** con sidebar de navegación, tema claro/oscuro, botón de logout
- **Autenticación admin**: nuevo rol `admin` en el sistema de auth. Mock user: `admin@ainara.com` / `admin`
- **AdminRoute** guard: protege rutas `/admin/*`, redirige a login si no es admin
- **Pedidos — Vista listado**: tabla con columnas ID, fecha, cliente/dirección, tipo de entrega, medio de pago, importe, estado, estado de pago, y acciones
- **Pedidos — Vista Kanban**: 3 columnas (Entrantes / En proceso / Finalizados) con tarjetas de pedido
- **Acciones por pedido**: avanzar estado, retroceder estado, cancelar
- **Badges de estado** con colores diferenciados: pendiente (amarillo), en preparación (azul), listo (verde), en camino (violeta), entregado (gris), cancelado (rojo)
- **Highlight visual** en filas de pedidos pendientes (fondo amarillo sutil)
- **Mock data**: 4 pedidos activos adicionales para el admin (distintos estados y clientes)
- **Redirect automático**: login con admin redirige a `/admin`, login normal a la ruta anterior
- Rutas: `/admin` → redirige a `/admin/pedidos`

## [0.11.5] - 2026-03-09

### Mejorado

- **Botones de volver** más destacados: cambiados de `ghost` a `outline` en ProductDetailPage, CartPage y CheckoutPage

## [0.11.4] - 2026-03-09

### Implementado

- **Barra flotante de carrito**: barra fija en la parte inferior que aparece cuando hay ítems en el carrito, muestra cantidad de ítems + subtotal + botón "Ver carrito"
- Se oculta automáticamente en `/cart`, `/checkout` y `/order-confirmation`
- Padding inferior en el `<main>` para evitar que la barra tape contenido
- Diseño responsive: funciona en mobile y desktop

## [0.11.3] - 2026-03-08

### Cambiado

- **Pizzas individuales**: cada pizza es un producto propio sin selección de gustos — Muzzarella ($5.500), Napolitana ($5.800), Fugazzeta ($6.000)
- **Combo Pizza + 2 Gaseosas** ($6.500): pizza muzzarella + 2 latas de gaseosa a elección
- Eliminado `mockPizzaFlavors` (ya no se necesita)

## [0.11.2] - 2026-03-08

### Implementado

- **Categoría Pizzas** (🍕) con producto Pizza casera (id 22) — selección de gusto (Muzzarella, Napolitana, Fugazzeta, Jamón y morrones, Calabresa, Roquefort)
- **Combo Pizza + Gaseosa** ($5.800): pizza a elección + gaseosa a elección (Coca-Cola, Sprite, Fanta)
- Nuevos flavor sources: `pizzas` y `gaseosas` en handlers

## [0.11.1] - 2026-03-08

### Mejorado

- **Combos primero**: los combos siempre aparecen en las primeras posiciones del catálogo
- **Destaque visual**: las cards de combos tienen borde ámbar, ring y badge "COMBO"

## [0.11.0] - 2026-03-08

### Implementado

- **Combos**: nuevo tipo de producto que combina ítems existentes a precio promocional
- **Combo 2× 1/4 kg** ($6.000 vs $7.000 por separado): 2 helados de 1/4 kg, cada uno con su propia selección de sabores
- **Combo Docena de empanadas** ($8.400): 12 empanadas con selección de gustos y emojis
- **Categoría 🎁 Combos** en el filtro del catálogo
- **Selección de sabores por ítem del combo**: cada componente tiene su propio selector con contador +/- (X de Y) e indicador verde al completar
- **Carrito muestra detalle del combo**: cada ítem con sus sabores seleccionados
- **Normalización de pedidos**: los combos se serializan como "Ítem1: sabores | Ítem2: sabores" en el historial

---

## [0.10.3] - 2026-03-08

### Mejorado

- **Helados con selector de cantidad por sabor**: los helados, postres y milkshakes ahora usan el mismo selector +/- por sabor que las empanadas. Ej: Helado 1 kg → Dulce de leche ×3, Chocolate ×1, Frutilla ×1
- **`flavorMode: 'quantity'` en todos los productos con sabor**: helados (1kg/1/2kg/1/4kg), sundae, banana split, milkshakes mediano y grande
- **Label diferenciado**: "Sabores" para helados/postres/bebidas, "Gustos" para empanadas

---

## [0.10.2] - 2026-03-08

### Mejorado

- **Empanadas con precio por gusto**: cada gusto tiene su propio precio (Carne $900, Pollo $850, etc.) y emoji representativo
- **Cantidad libre sin tope**: el usuario agrega +/- de cada gusto directamente, sin necesidad de elegir un total primero
- **Precio dinámico**: el total se calcula en tiempo real como suma de (cantidad × precio unitario) de cada gusto seleccionado
- **Filas con imagen**: cada gusto muestra su emoji, nombre, precio unitario y controles +/-
- **ProductCard**: los productos con `unitPricing` muestran "Armá tu pedido" en lugar de $0
- **Modo quantity sin pricing preservado**: los productos con `unitCount` fijo siguen funcionando igual

---

## [0.10.1] - 2026-03-08

### Mejorado

- **Empanadas con cantidad libre**: reemplazados los 2 productos fijos (docena / media docena) por un único producto "Empanadas" con selector de cantidad libre — el usuario elige cuántas quiere con +/−, ve precio unitario × cantidad, y luego distribuye gustos
- **`unitPricing`**: nuevo flag en producto; el precio se calcula por unidad y la sección de formato se oculta automáticamente

---

## [0.10.0] - 2026-03-08

### Implementado

- **Empanadas en el menú**: docena ($9.600) y media docena ($5.400) con selección de gustos por cantidad
- **Modo de selección por cantidad (`flavorMode: 'quantity'`)**: nuevo modo donde el usuario elige cuántas unidades de cada gusto quiere (ej: 4 Carne + 4 Pollo + 4 J&Q = 12), con controles +/- por gusto y contador de unidades restantes
- **Gustos de empanadas**: Carne, Pollo, Jamón y queso, Humita, Caprese, Verdura
- **Categoría Empanadas 🥟**: nueva categoría en el filtro del catálogo
- **`getFlavors(source)` parametrizado**: el handler acepta source opcional para devolver la lista de gustos correcta según el producto
- **Carrito muestra cantidades por gusto**: ej. "3 Carne, 3 Pollo, 6 Jamón y queso"

---

## [0.9.1] - 2026-03-08

### Corregido

- **Tortas eliminadas**: se removieron las 4 tortas heladas del menú
- **Filtro de categorías restaurado**: botones Todos / Helados / Postres / Bebidas en CatalogPage

---

## [0.9.0] - 2026-03-08

### Implementado

- **Catálogo plano**: cada producto es un SKU individual visible de un vistazo, sin navegar categorías
- **Helados por kilo**: separados en 3 productos (1 kg, 1/2 kg, 1/4 kg) con sabores y extras
- **Bebidas individuales**: Coca-Cola, Sprite, Fanta, Agua sin gas, Agua con gas (en lugar de "Gaseosa")
- **Milkshakes**: mediano y grande como productos separados
- **Tortas**: cada tamaño como producto independiente (clásica 6/12 porc., brownie 6/12 porc.)
- **Cucurucho eliminado**: no apto para delivery
- **CatalogPage simplificado**: sin filtro de categorías, grilla directa de productos
- **ProductCard**: precio directo (sin "Desde")

---

## [0.8.10] - 2026-03-05

### Mejorado

- **Barra de progreso con paso actual en amarillo**: el estado actual se muestra en ámbar (barra y texto), los anteriores en verde, los pendientes en gris

---

## [0.8.9] - 2026-03-05

### Corregido

- **Barra de progreso mobile**: labels ultra cortos para evitar solapamiento (Pend. · Conf. · Prep. · Listo · Envío · Entreg.)

---

## [0.8.8] - 2026-03-05

### Mejorado

- **Enlace al panel con nombre de usuario**: reemplazado "Mis pedidos" por el nombre del usuario con ícono `UserCircle`, enlaza al panel
- **Aspecto tappeable en mobile**: nombre del usuario en MobileUserBar con estilo pill (`rounded-full`, fondo gris, `active:` feedback)
- **Header desktop**: nombre como botón ghost con `font-semibold`

---

## [0.8.7] - 2026-03-05

### Corregido

- **Tabs del panel no scrolleables**: eliminado scroll horizontal, las tabs se ajustan al ancho disponible

---

## [0.8.6] - 2026-03-05

### Implementado

- **Múltiples pedidos activos**: el panel y la home ahora muestran todos los pedidos en curso, no solo uno
- **Aviso de pago pendiente**: los pedidos con efectivo o transferencia muestran cartel "Pendiente de pago — el pedido avanzará cuando se confirme el pago"

### Corregido

- **Progresión detenida para pagos pendientes**: los pedidos con pago en efectivo o transferencia ya no avanzan de estado hasta que se confirma el pago
- `processPayment` arranca la progresión automática cuando el pago se confirma

---

## [0.8.5] - 2026-03-05

### Corregido

- **Crash al volver a Home tras confirmar pedido**: los items del carrito (format como objeto, flavors como array) se normalizan a strings en `createOrder` para que el panel los renderice sin error

---

## [0.8.4] - 2026-03-05

### Implementado

- **Persistencia mock de pedidos**: `createOrder` ahora guarda el pedido en memoria para que aparezca en panel e historial
- **Progresión automática de estado**: cada 15 s el pedido avanza (pendiente → confirmado → preparación → listo → en camino → entregado)
- **Polling en ActiveOrderSection**: refresca cada 10 s para mostrar el avance en tiempo real
- `processPayment` actualiza el `paymentStatus` del pedido almacenado
- `CheckoutPage` pasa `userId` a `createOrder`

### Corregido

- **Layout barra de progreso**: label "En preparación" acortado a "Preparación", agregado `truncate` y `min-w-0` para evitar desborde

---

## [0.8.3] - 2026-03-05

### Corregido

- **Pedido activo mock eliminado**: se quitó el pedido en curso (id 2048) de los datos mock para que no aparezca sin haber hecho un pedido real

---

## [0.8.2] - 2026-03-05

### Mejorado

- **Banner invitado mejorado**: texto actualizado a "Creá una cuenta para ver el estado de tus pedidos, acumular puntos y guardar direcciones"
- **Pedido activo en Home**: usuarios registrados ven su pedido en curso directamente en la página principal
- **"Mi panel" → "Mis pedidos"**: renombrado y resaltado con `variant="outline"` y `font-semibold` en header y barra mobile

---

## [0.8.1] - 2026-03-05

### Mejorado

- **MobileUserBar solo para registrados**: la barra inferior ya no se muestra para invitados
- **Banner invitado en Home**: los invitados ven un aviso para crear cuenta en la página principal
- **Botón explícito al panel**: reemplazado el enlace con nombre por un botón "Mi panel" con ícono

---

## [0.8.0] - 2026-03-04

### Implementado

- **Panel del usuario** — nueva página `/panel` con 4 secciones en tabs
- **Pedido activo**: estado en tiempo real con barra de progreso visual (pendiente → entregado)
- **Historial de pedidos**: lista de pedidos anteriores con detalle expandible (items, descuentos, envío, pago)
- **Saldo de puntos**: balance actual, historial de movimientos con fecha de vencimiento
- **Mi cuenta**: edición de nombre y email, accesos rápidos a direcciones y cambio de contraseña
- **Mock orders**: 4 pedidos de ejemplo para el usuario mock (1 activo en camino, 3 entregados)
- **Handlers**: `getUserOrders`, `getActiveOrder`, `updateUserProfile`
- **Navegación**: nombre de usuario en header (md+) enlaza al panel, saludo en MobileUserBar enlaza al panel
- **Ruta protegida**: solo usuarios registrados acceden al panel

---

## [0.7.4] - 2026-03-04

### Mejorado

- **Puntos pendientes visibles**: la página de confirmación muestra la cantidad de puntos que se acreditarán cuando se confirme el pago

---

## [0.7.3] - 2026-03-04

### Corregido

- **Puntos diferidos para pagos pendientes**: en pagos con efectivo o transferencia, los puntos se acreditan recién cuando se confirma el pago
- Página de confirmación muestra aviso "Los puntos se acreditarán cuando se confirme el pago" para estos métodos

---

## [0.7.2] - 2026-03-04

### Corregido

- **Efectivo disponible para retiro en local**: se eliminó la restricción que limitaba el pago en efectivo solo a delivery

---

## [0.7.1] - 2026-03-04

### Corregido

- **Transferencia bancaria como pendiente de pago**: el pago por transferencia ahora queda con estado "pendiente de pago" igual que efectivo

---

## [0.7.0] - 2026-03-05

### Implementado

- **Medios de pago** — selección de método de pago en el checkout
- **Mercado Pago**: pago online simulado (mock)
- **Transferencia bancaria**: muestra datos bancarios (CBU, alias, titular, CUIT) con botón de copiar
- **Tarjeta crédito/débito**: pago online simulado (mock)
- **Efectivo al delivery**: solo disponible para delivery, pedido queda como "pendiente de pago"
- **PaymentMethodSelector**: selector visual con íconos, info contextual por método
- **OrderConfirmationPage**: página de confirmación post-pago con resumen, estado, puntos ganados
- **Flujo completo**: confirmar pedido → procesar pago → canjear puntos → acumular puntos → limpiar carrito → confirmación
- **Mock handlers**: `getPaymentMethods` (filtra por tipo de pedido) y `processPayment` (simula procesamiento)

---

## [0.6.1] - 2026-03-04

### Mejorado

- **Header mobile optimizado**: solo iconos esenciales en pantallas chicas, textos en sm+
- **Marca responsive**: "Ainara" en mobile, "Ainara Helados" en sm+
- **MobileUserBar**: barra contextual debajo del header (solo mobile) con saludo, puntos y acceso a Direcciones
- **Botón logout** en rojo para distinción visual
- **Botón login** en verde con fondo semitransparente e ícono `CircleUserRound`
- **Nombre de usuario** visible solo en md+ para ahorrar espacio

---

## [0.6.0] - 2026-03-01

### Implementado

- **Programa de Fidelización** — sistema de puntos para usuarios registrados
- **Acumulación de puntos**: 1 peso = 1 punto (excluye costo de envío)
- **Vencimiento**: puntos expiran a los 3 meses desde la acreditación
- **Canje de puntos** como descuento en el checkout (1 punto = $1)
- **Cupones de descuento** con validación automática (porcentaje o monto fijo)
- **CouponInput**: ingreso y validación de códigos de descuento con feedback de error
- **RedeemPoints**: selector de puntos a canjear con botón "Usar máximo"
- **PointsBadge**: indicador de saldo de puntos en el header (estrella dorada)
- **LoyaltyContext**: contexto global con saldo, historial, canje y acumulación
- **Mock data**: historial de puntos, 4 cupones de prueba (HELADOGRATIS, VERANO20, AINARA10, EXPIRADO)
- **Desglose en checkout**: subtotal − puntos − cupón + envío = total
- Descuentos solo para usuarios registrados (invitados ven cupones pero no puntos)

---

## [0.5.1] - 2026-03-01

### Implementado

- **Opción de login para invitados en checkout**: los usuarios guest ven botones "Iniciar sesión" y "Crear cuenta" en el resumen del pedido, con redirect de vuelta al checkout

---

## [0.5.0] - 2026-03-01

### Implementado

- **Toggle delivery / retiro en local** en la página de checkout
- **Selector de dirección** para usuarios registrados, con indicador de cobertura
- **Formulario de dirección inline** para invitados (calle + ciudad)
- **Cálculo automático de costo de envío** por distancia (Haversine), con zonas configurables (Cercana, Media, Lejana)
- **Zonas de delivery mock** con 3 tramos: hasta 1.5 km ($500), hasta 3 km ($800), hasta 5 km ($1200)
- **Bloqueo de pedido** si la dirección está fuera de cobertura (alerta visual + botón deshabilitado)
- **Desglose de costos** en resumen: subtotal + envío + total
- **Handler `calcDeliveryCost`** que retorna zona, distancia y costo
- **Estado derivado** para loading de costo (sin setState sincrónico en effects)

---

## [0.4.1] - 2026-03-01

### Implementado

- **Catálogo de heladería**: datos mock adaptados con 16 sabores, helados por kilo/cucurucho/vasito, postres, tortas heladas, milkshakes y bebidas
- **Selección de sabores por formato**: cada formato define `maxFlavors`, el usuario elige sabores con límite visual
- **Sabores flexibles**: se permite elegir entre 1 y `maxFlavors` sabores (no obliga a completar el máximo)
- **Sabores en carrito y checkout**: se muestran los sabores elegidos en cada línea del pedido
- **Chips de sabores**: layout tipo chip (`flex-wrap`, `rounded-full`) que se adapta al ancho sin truncar nombres largos
- **Branding**: nombre del local "Ainara Helados" en header y homepage
- **ngrok**: `server.allowedHosts` configurado para dominios `*.ngrok-free.app`

---

## [0.4.0] - 2026-03-01

### Implementado

- **Catálogo de productos** con grilla responsive y filtro por categoría (tabs con íconos)
- **Detalle de producto** con selección de formato (tamaño), extras opcionales con costo, comentario libre
- **Carrito de compras** con agregar, eliminar, modificar cantidad, comentario general del pedido
- **CartContext** con estado global: items, itemCount, subtotal, clearCart
- **Checkout con login gate**: usuarios no autenticados ven opciones de ingreso; al autenticarse se redirigen de vuelta al checkout
- **Ícono de carrito en header** con badge de cantidad de items, visible para todos
- **Link "Menú" en header** accesible para visitantes y logueados
- **HomePage** actualizada con CTA "Ver menú"
- **Datos mock enriquecidos**: productos con array de formatos (nombre, precio) y extras (nombre, precio)
- **GuestRoute mejorado**: preserva `location.state` al redirigir (soporte para flujo checkout → login → checkout)
- **Redirección post-auth**: LoginPage, RegisterPage y GuestPage respetan `state.from` para volver a la ruta de origen

---

## [0.3.1] - 2026-03-01

### Implementado

- **Campos de dirección extendidos**: piso, departamento y barrio como campos separados
- **Alias y ciudad obligatorios** con validación en formulario
- **Etiquetas "(opcional)"** en campos no obligatorios (piso, depto, barrio, comentario)
- **Convención de formularios**: campos opcionales marcados, obligatorios sin asterisco

---

## [0.3.0] - 2026-03-01

### Implementado

- **CRUD de direcciones** con agregar, editar y eliminar
- **Alias y comentarios** por dirección (ej: 'Casa', 'Piso 3 timbre B')
- **Dirección activa** seleccionable para el pedido, persistida en localStorage
- **Validación de cobertura** con fórmula Haversine (zona de 5 km)
- **Indicador visual** de cobertura en cada dirección (verde/rojo + alerta)
- **Ruta protegida** `/addresses` solo para usuarios registrados (no guests)
- **Link en header** con ícono de mapa, visible para usuarios logueados
- **AddressContext** con estado global de direcciones y dirección activa
- **Componente Textarea** UI reutilizable con dark mode
- **ProtectedRoute** para restringir acceso a rutas autenticadas

---

## [0.2.1] - 2026-03-01

### Implementado

- **Login con Google** (OAuth mock) con botón e ícono SVG
- **Modo oscuro** con toggle sol/luna en header, persistido en localStorage
- **Detección automática** del tema del sistema (prefers-color-scheme)
- **Clases `dark:`** en todos los componentes UI y páginas
- **GuestRoute** para redirigir usuarios logueados fuera de login/register

---

## [0.2.0] - 2026-02-28

### Implementado

- **Tailwind CSS v4** + **shadcn/ui** (Button, Input, Label, Card)
- **React Router** con layout y navegación entre páginas
- **Layout mobile first** con header responsive
- **Login** con email y contraseña (mock)
- **Registro** con validación de contraseñas (mock)
- **Recuperación de contraseña** (mock)
- **Modo invitado** con datos mínimos (nombre, teléfono, email opcional)
- **Auth context** con persistencia en localStorage
- **Alias `@/`** para imports absolutos

---

## [0.1.0] - 2026-02-28

### Implementado

- **Setup inicial del proyecto** con Vite + React
- **Gestión de versión de Node** con fnm (`.node-version`)
- **ESLint + Prettier** configurados e integrados
- **Estructura base de carpetas**: components, pages, hooks, context, services, mocks, utils
- **Capa de servicios API** (`src/services/api.js`) preparada para conectar con backend Python
- **Mock de datos** con menú de ejemplo, categorías y estados de pedido
- **Mock handlers** con delay simulado para desarrollo sin backend
- **Documentación SETUP.md** para onboarding del equipo

---

## Roadmap — Panel de Administración

> Detalle completo en [`docs/admin-roadmap.md`](docs/admin-roadmap.md)

### v1.0.0 — Fase 1: MVP Admin

- [x] Layout admin (sidebar + contenido) con autenticación de rol admin *(v0.12.0)*
- [x] **Pedidos — Listado**: tabla con ID, fecha, cliente, tipo entrega, medio de pago, importe, estado, repartidor, acciones *(v0.12.0)*
- [x] **Pedidos — Kanban**: 3 columnas (Entrantes / En proceso / Finalizados) *(v0.12.0)*
- [x] **Pedidos — Alertas**: sonido y destaque visual al recibir pedido nuevo *(v0.12.2 — parcial: polling, no real-time)*
- [x] **Pedidos — Acciones básicas**: avanzar/retroceder estado, cancelar *(v0.12.0)*
- [ ] **Productos**: CRUD con los 3 arquetipos (Simple, Slots Fijos, Porciones y Sabores)
- [ ] **Sabores**: lista maestra global con toggle ON/OFF
- [ ] **Configuración**: horarios de apertura, apagado de emergencia, mensaje de ausencia

### v1.1.0 — Fase 2: Operación completa

- [ ] **Pedidos — Carga manual**: WhatsApp, mostrador, delivery
- [ ] **Pedidos — Comanda**: impresión para pegar en bolsa
- [ ] **Pedidos — Edición**: modificar sabores, dirección, extras
- [ ] **Pedidos — Especiales**: pedido sin costo (reposición), forzar fuera de zona
- [ ] **Pedidos — Delivery**: estados paralelos (buscando repartidor → retirado → entregado)
- [ ] **Zonas**: configuración por radio y polígono, demora estimada, apagar zona
- [ ] **Delivery**: lista de repartidores, asignación
- [ ] **Configuración**: pagos (efectivo on/off, compra mínima, precios diferenciados)

### v1.2.0 — Fase 3: Clientes y fidelización

- [ ] **Clientes**: listado, clasificación VIP, notas privadas, historial de pedidos
- [ ] **Puntos**: configuración de acumulación y canje desde admin
- [ ] **Puntos Usados**: historial de canjes
- [ ] **Promos**: CRUD de promos, vinculación a productos o al carrito

### v1.3.0 — Fase 4: Facturación y analítica

- [ ] **Facturación**: acumulados por fecha/medio de pago, caja chica, exportación CSV/Excel
- [ ] **Informe**: origen de clientes (UTM), encuesta post-pedido
- [ ] **Rentabilidad**: costos estimados, análisis TC y comisiones

### v1.4.0 — Fase 5: Integraciones

- [ ] Integración Rapiboy (GPS tracking de delivery)
- [ ] Gift cards / vouchers
- [ ] Cash flow y estado de resultados

### Futuro (sin versión asignada)

- [ ] Integración con backend real (Python)
- [ ] Notificaciones en tiempo real
- [ ] Testing (Vitest + React Testing Library)
- [ ] CI/CD pipeline (GitHub Actions)
