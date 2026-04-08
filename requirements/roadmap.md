# Panel de Administración — Roadmap

Documento de planificación para el módulo de administración de Ainara Helados.
Diseñado para **desktop** (notebook / PC de escritorio). No requiere mobile first.

---

## Tabla de contenido

1. [Módulo de Pedidos](#1-módulo-de-pedidos)
2. [Catálogo de Productos](#2-catálogo-de-productos)
3. [Clientes](#3-clientes)
4. [Puntos y Fidelización](#4-puntos-y-fidelización)
5. [Zonas y Delivery](#5-zonas-y-delivery)
6. [Configuración](#6-configuración)
7. [Facturación y Cierre de Caja](#7-facturación-y-cierre-de-caja)
8. [Promos](#8-promos)
9. [Analítica y Reportería](#9-analítica-y-reportería)
10. [Menú del Admin](#10-menú-del-admin)
11. [Fases de implementación](#11-fases-de-implementación)
12. [Feedback de stakeholder (2026-04-02)](#12-feedback-de-stakeholder-2026-04-02)

---

## 1. Módulo de Pedidos

### Vista principal — Tabla

| Columna       | Detalle                                              |
|---------------|------------------------------------------------------|
| ID            | Número de pedido                                     |
| Fecha         | Fecha y hora del pedido                              |
| Cliente       | Nombre / dirección (o "Invitado")                    |
| Tipo entrega  | Delivery / Retiro en local / Venta en mostrador       |
| Medio de pago | MercadoPago, Efectivo, Transferencia, Tarjeta        |
| Importe       | Total del pedido                                     |
| Estado        | Estado actual del pedido                             |
| Repartidor    | Asignado (si aplica)                                 |
| Acciones      | Botones de acción rápida                             |

### Modos de visualización

- **Listado**: tabla con filtros y ordenamiento.
- **Kanban**: 3 columnas — Entrantes / En proceso / Retirados. Drag & drop para cambiar estado.

### Alerta de nuevo pedido

- Sonido audible al recibir un pedido nuevo.
- Destaque visual (color, animación) en la fila/tarjeta del pedido.

### Tipos de pedido

| Tipo | Quién inicia | Productos | Flujo |
|------|-------------|-----------|-------|
| **Delivery** | Cliente (app) | Todos excepto "Solo mostrador" | Nuevo → Elaboración → Despacho → Entregado |
| **Retiro en Local** | Cliente (app) | Todos excepto "Solo mostrador" | Nuevo → Elaboración → Listo → Cliente retira |
| **Venta en Mostrador** | Operador (admin) | Todos (incluye "Solo mostrador") | Venta instantánea, cliente presente |

**Venta en Mostrador**: no requiere dirección ni teléfono. Opcionalmente puede vincular un cliente registrado para sumar puntos de fidelización.

### Estados del pedido

```
Nuevo → En proceso de elaboración → Listo → Retirado → Entregado
```

### Estados del delivery (paralelo al pedido)

```
A la espera de proceso listo → Buscando repartidor → Retirado → Entregado
```

- Integración futura con **Rapiboy** (seguimiento GPS en tiempo real).

### Acciones por pedido

| Acción                           | Detalle                                                |
|----------------------------------|--------------------------------------------------------|
| Avanzar/retroceder estado        | Botones para mover el pedido en el flujo               |
| Imprimir comanda                 | Genera formato para pegar en bolsa. Incluye **mensaje personalizable** (ej: "Felices Pascuas ❤️") |
| Editar pedido                    | Modificar sabores, dirección, extras, etc.             |
| Cancelar / eliminar              | Con confirmación                                       |
| Cargar pedido sin costo          | Para reposiciones por error                            |
| Forzar carga fuera de cobertura  | Override manual de la restricción de zona              |

### Teléfono de contacto obligatorio

- El checkout **requiere** un teléfono celular de contacto antes de confirmar el pedido.
- Si el usuario tiene un teléfono guardado en su perfil, se pre-carga pero **debe confirmar o modificar**.
- El teléfono ingresado se asocia al pedido (puede diferir del teléfono del perfil).
- Permite recibir llamadas del repartidor o del local.

### Datos del pedido (detalle)

- Número de pedido
- Cliente registrado o invitado
- **Teléfono de contacto** (obligatorio, confirmado en cada pedido)
- Detalle de ítems (producto, sabores, extras, cantidad)
- Horario del pedido
- Barrio / dirección
- Tipo y estado de pago (MercadoPago puede entrar sin estar pagado aún)

### Carga manual de pedidos

| Canal                | Detalle                                                          |
|----------------------|------------------------------------------------------------------|
| WhatsApp             | Ventana de escape ante cualquier problema con la app             |
| Venta de mostrador   | Sin sabores obligatorios, solo producto. Puede vincular cliente para puntos |
| Venta de delivery    | Carga completa como si fuera un pedido del cliente               |

---

## 2. Catálogo de Productos

### Arquetipos de producto

| Arquetipo                    | Ejemplo                         | Comportamiento                                                                  |
|------------------------------|---------------------------------|---------------------------------------------------------------------------------|
| **Simple**                   | Pizza, cucurucho, bebida        | Botones +/−, adicionales opcionales                                             |
| **Con Slots Fijos**          | 6 empanadas, docena             | Cliente elige N opciones de una lista, puede repetir. No agrega al carrito hasta completar todos los slots. Contador "X/N elegidos" |
| **Con Porciones y Sabores**  | ¼ kg, ½ kg, 1 kg helado        | Cada instancia en el carrito tiene su propia selección independiente. Puede agregar el mismo producto varias veces con sabores distintos. Repetir sabor configurable. Botón habilitado desde el 1er sabor |

### Jerarquía

```
Macro categorías → Categorías → Productos → Sabores/Opciones → Adicionales
```

### Visibilidad por canal

- Por defecto **todos los productos** están disponibles en los 3 tipos de pedido (Delivery, Retiro en Local, Mostrador).
- Un producto puede marcarse como **"Solo venta en mostrador"** (flag booleano): queda excluido de Delivery y Retiro en Local, no aparece en la app del cliente.
- Ejemplo: cucurucho unitario es "Solo mostrador", pero "cucuruchos sueltos x6" está disponible en todos los canales.

### Imágenes de producto

- Cada producto puede tener una **foto** cargada desde el admin.
- Se almacenan en S3 y se sirven vía CloudFront.
- Visibles en el catálogo del cliente y en el detalle del producto.
- Formatos aceptados: JPG, PNG, WebP. Redimensionado automático para optimizar carga.

### Stock y disponibilidad

- Toggle rápido **ON/OFF** en el admin para productos y sabores.
- Los sabores son una **lista maestra global**: desactivar un sabor lo quita de todos los productos que lo usen.
- El filtro se aplica en el **backend** (query SQL), no en el frontend.

---

## 3. Clientes

### Registro y autenticación

- Registro al iniciar el carrito.
- **Login con Google** (OAuth 2.0 / Google Identity Services). Prioridad alta.
- Login social con Facebook (fase futura).
- Formulario clásico (email + contraseña).
- Recuperación de contraseña. **Requiere dominio propio** registrado en Amazon SES para enviar emails (verificación de identidad, enlace de reseteo). Sin dominio no se pueden enviar correos en producción.
- **Expiración de sesión**: las sesiones del cliente expiran después de un tiempo configurable (ej: 7 días de inactividad). El JWT tiene tiempo de vida limitado. Al expirar se redirige al login.

### Perfil del cliente

- Email y teléfono como datos clave.
- Múltiples direcciones guardadas con etiqueta (Mi casa, De mi pareja, Amigos, etc.).
- Comentarios permanentes en perfil (ej: "No tocar timbre").
- Comentarios por pedido (independientes del perfil).

### Gestión desde admin

- Clasificación interna de clientes: **VIP**, notas privadas (no visibles para el cliente).
- Historial de pedidos por cliente.
- Edición de datos del cliente.

---

## 4. Puntos y Fidelización

### Sistema de puntos

- Acumulación de puntos por compra (1 peso = 1 punto, configurable).
- Canje de puntos con lógica configurable (monto mínimo, ratio de conversión).
- Historial de puntos ganados y usados, visible en:
  - Perfil del cliente (app).
  - Panel del admin.

### Beneficios adicionales

- **Gift cards / vouchers** para regalar.
- **Club de beneficios** (banner ya visible en la app del cliente).
- Cupones de descuento (ya implementados en el front del cliente).

---

## 5. Zonas y Delivery

### Configuración de zonas

- Zonas por **radio en km** desde un punto central.
- Zonas por **perímetros personalizados** (polígonos en mapa), útil para zonas conflictivas.
- Posibilidad de **apagar una zona temporalmente**.

### Demora estimada

- Configurable por zona.
- Editable en tiempo real (alta demanda, pocos repartidores).
- Se muestra al cliente al ingresar su dirección.

### Dirección del cliente — Validación con Google Maps

- **Autocompletado con Google Maps Places API**: al tipear "Mitre 123", el mapa sugiere las opciones reales ("Emilio Mitre 123", "Av. Bartolomé Mitre 123", etc.) para desambiguar.
- Selector de mapa (siempre parte desde CABA como centro por defecto).
- El usuario **debe seleccionar una dirección sugerida** por el mapa (no se aceptan direcciones de texto libre sin geocodificación).
- Se almacenan coordenadas (lat/lng) además del texto, para validar cobertura con precisión.
- Validación de cobertura contra las zonas configuradas (radio o polígono).

### Zona de cobertura — Polígono en mapa (ADMIN)

- El admin puede **dibujar un polígono en Google Maps** para definir el área exacta de cobertura.
- Soporte para múltiples zonas (con distinto costo de envío o demora).
- Alternativa simplificada: radio en km desde un punto central.
- Las zonas se pueden encender/apagar individualmente.
- El cliente ve en tiempo real si su dirección está dentro de la cobertura.

---

## 6. Configuración

### Horarios de apertura y cierre — Bloqueo de pedidos

- Configuración por día de la semana.
- Hora desde / hasta.
- Estado activo (verde) / inactivo (rojo).
- **Fuera del horario de atención, el cliente NO puede realizar pedidos**. Se muestra un mensaje con el próximo horario de apertura.
- El catálogo sigue visible en modo lectura, pero el botón de "Agregar al carrito" y el checkout están deshabilitados.

### Controles de emergencia

| Control                        | Detalle                                                     |
|--------------------------------|-------------------------------------------------------------|
| **Cierre de emergencia**       | Un botón prominente (rojo) para cerrar el comercio inmediatamente. Los pedidos en curso se siguen procesando, pero no se aceptan nuevos. |
| Mensaje de ausencia            | Configurable (ej: "Ainara está cerrado, abrimos a las 16:00hs") |
| Mensajes por situación         | Sin repartidores, pedir por WhatsApp, etc.                  |

### Comanda — Mensaje personalizable

- El admin puede configurar un **mensaje libre** que se imprime en todas las comandas (ej: "Felices Pascuas ❤️", "¡Gracias por tu compra!").
- Se edita desde Configuración.
- Se puede dejar vacío para no imprimir mensaje adicional.

### Pagos y precios

- Pago en efectivo habilitado/deshabilitado (según disponibilidad de repartidores propios).
- Valor de **compra mínima**.
- **Precios diferenciados por medio de pago** (opcional).

### Otros

- Lista de repartidores (altas, bajas, asignación).
- Leyenda de ticket.
- **Mensaje de comanda** (texto libre impreso en cada comanda).
- Link de encuesta post-pedido.
- Texto de transferencia (datos bancarios para el cliente).
- % descuento de alianza.

---

## 7. Facturación y Cierre de Caja

### Acumulados

- Por fecha.
- Por medio de pago.
- Separación: efectivo vs. otros medios.

### Caja chica

- Ajustes por pagos en efectivo.
- Registro de diferencias al cierre.

### Exportación

- Exportable (CSV/Excel) para combinar con pedidos de otras plataformas (Rappi, PedidosYa, etc.).

### Rentabilidad (fase futura)

- Costos estimados por tamaño/sabor.
- Análisis de ticket de cambio (TC) y comisiones.

---

## 8. Promos

- Promos dentro de productos (ej: 2x1 en un producto específico).
- Promos fuera de productos (ej: descuento general en el carrito).
- Configurables de forma **independiente del catálogo**.
- Fechas de vigencia, condiciones, límites de uso.

---

## 9. Analítica y Reportería

### Origen de clientes

- Tracking UTM: Instagram, búsqueda web, link directo, etc.
- Dashboard de adquisición de clientes.

### Encuesta post-pedido

- Integrada en el flujo de confirmación.
- Resultados agregados visibles en el admin.

### Fase futura

- Cash flow.
- Estado de resultados.

---

## 10. Menú del Admin

Secciones confirmadas del sidebar:

| Sección        | Módulo relacionado               |
|----------------|----------------------------------|
| Pedidos        | §1 Módulo de Pedidos             |
| Productos      | §2 Catálogo de Productos         |
| Sabores        | §2 Catálogo (lista maestra)      |
| Horarios       | §6 Configuración                 |
| Configuración  | §6 Configuración                 |
| Zonas          | §5 Zonas y Delivery              |
| Promos         | §8 Promos                        |
| Facturación    | §7 Facturación y Cierre de Caja  |
| Delivery       | §5 Zonas y Delivery              |
| Informe        | §9 Analítica y Reportería        |
| Puntos         | §4 Puntos y Fidelización         |
| Puntos Usados  | §4 Puntos y Fidelización         |

---

## 11. Fases de implementación

### Fase 1 — Fundación (MVP Admin)

> Objetivo: que el dueño pueda gestionar los pedidos del día y el catálogo.

- [x] Layout admin (sidebar + contenido) con autenticación de rol admin
- [x] **Pedidos**: vista listado con tabla, acciones básicas (avanzar estado, cancelar)
- [x] **Pedidos**: vista Kanban (3 columnas)
- [x] **Pedidos**: alerta sonora y visual de nuevo pedido
- [x] **Productos**: CRUD de productos con los 3 arquetipos
- [ ] **Productos**: carga de imágenes de producto (S3 + CloudFront)
- [x] **Productos**: flag "Solo venta en mostrador"
- [x] **Sabores**: lista maestra global con toggle ON/OFF
- [x] **Configuración**: horarios de apertura con bloqueo de pedidos fuera de horario
- [x] **Configuración**: botón de cierre de emergencia
- [x] **Configuración**: mensaje de ausencia

### Fase 2 — Operación completa

> Objetivo: cubrir el día a día operativo completo.

- [ ] **Pedidos**: venta en mostrador (flujo corto, sin dirección/teléfono, vinculación opcional de cliente)
- [x] **Pedidos**: carga manual (WhatsApp, delivery)
- [x] **Pedidos**: imprimir comanda con mensaje personalizable
- [x] **Pedidos**: estados de delivery paralelos
- [x] **Checkout**: teléfono de contacto obligatorio (confirmación o ingreso manual en cada pedido)
- [ ] **Direcciones**: integración Google Maps Places API (autocompletado + desambiguación)
- [ ] **Direcciones**: geocodificación obligatoria (lat/lng almacenado con cada dirección)
- [ ] **Zonas**: dibujo de polígono de cobertura en mapa (admin)
- [ ] **Zonas**: configuración por radio, demora estimada, apagar zona
- [x] **Configuración**: mensaje de comanda personalizable

### Fase 3 — Clientes y fidelización

> Objetivo: gestionar la relación con los clientes.

- [ ] **Auth**: login con Google (OAuth 2.0 / Google Identity Services)
- [ ] **Auth**: recuperar/cambiar contraseña (requiere dominio + Amazon SES)
- [ ] **Auth**: expiración de sesión configurable (JWT con TTL + refresh)
- [ ] **Clientes**: listado, clasificación VIP, notas privadas
- [ ] **Puntos**: configuración de acumulación y canje
- [ ] **Puntos Usados**: historial de canjes
- [ ] **Promos**: CRUD de promos, vinculación a productos o al carrito

---

## 12. Feedback de stakeholder (2026-04-02)

Registro de la revisión con el stakeholder. Cada punto se integró en las secciones y fases correspondientes.

| # | Feedback | Sección | Fase |
|---|----------|---------|------|
| 1 | Teléfono de contacto obligatorio en cada pedido (confirmar o ingresar otro) | §1 Pedidos — Teléfono de contacto obligatorio | Fase 2 |
| 2 | Dirección validada con Google Maps (desambiguación: Emilio Mitre vs Av. Bartolomé Mitre) | §5 Zonas — Dirección del cliente con Google Maps | Fase 2 |
| 3 | ADMIN: Dibujar polígono de cobertura en mapa | §5 Zonas — Polígono en mapa | Fase 2 |
| 4 | Login con Google | §3 Clientes — Registro y autenticación | Fase 3 |
| 5 | Expiración de sesión (JWT con TTL) | §3 Clientes — Registro y autenticación | Fase 3 |
| 6 | ADMIN: Horario de cierre bloquea pedidos | §6 Configuración — Horarios de apertura y cierre | Fase 1 |
| 7 | ADMIN: Botón de cierre de emergencia | §6 Configuración — Controles de emergencia | Fase 1 |
| 8 | ADMIN: Imprimir comandas | §1 Pedidos — Acciones por pedido | Fase 2 |
| 9 | ADMIN: Mensaje personalizable en comandas (ej: "Felices Pascuas ❤️") | §6 Configuración — Comanda | Fase 2 |
| 10 | Fotos de productos (carga desde admin, visibles en catálogo) | §2 Catálogo — Imágenes de producto | Fase 1 |
| 11 | Tipo de pedido "Venta en mostrador" (venta instantánea, cliente presente) | §1 Pedidos — Tipos de pedido | Fase 2 |
| 12 | Flag "Solo venta en mostrador" en productos (reemplaza sistema de canales) | §2 Catálogo — Visibilidad por canal | Fase 1 |
