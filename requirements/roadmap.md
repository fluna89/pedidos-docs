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

---

## 1. Módulo de Pedidos

### Vista principal — Tabla

| Columna       | Detalle                                              |
|---------------|------------------------------------------------------|
| ID            | Número de pedido                                     |
| Fecha         | Fecha y hora del pedido                              |
| Cliente       | Nombre / dirección (o "Invitado")                    |
| Tipo entrega  | Delivery / Retiro en local                           |
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
| Imprimir comanda                 | Genera formato para pegar en bolsa                     |
| Editar pedido                    | Modificar sabores, dirección, extras, etc.             |
| Cancelar / eliminar              | Con confirmación                                       |
| Cargar pedido sin costo          | Para reposiciones por error                            |
| Forzar carga fuera de cobertura  | Override manual de la restricción de zona              |

### Datos del pedido (detalle)

- Número de pedido
- Cliente registrado o invitado
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

### Canal por producto

- **Delivery**: aparece en la app de delivery.
- **Mostrador**: solo en ventas de mostrador (ej: cucurucho).
- **Ambos**: visible en todos los canales.

Ejemplo: cucurucho no aparece en delivery, pero sí "cucuruchos sueltos".

### Stock y disponibilidad

- Toggle rápido **ON/OFF** en el admin para productos y sabores.
- Los sabores son una **lista maestra global**: desactivar un sabor lo quita de todos los productos que lo usen.
- El filtro se aplica en el **backend** (query SQL), no en el frontend.

---

## 3. Clientes

### Registro y autenticación

- Registro al iniciar el carrito.
- Login social (Gmail, Facebook) o formulario clásico.
- Recuperación de contraseña.

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

### Dirección del cliente

- Selector de mapa (siempre parte desde CABA como centro por defecto).
- Validación de cobertura.

---

## 6. Configuración

### Horarios de apertura

- Configuración por día de la semana.
- Hora desde / hasta.
- Estado activo (verde) / inactivo (rojo).

### Controles de emergencia

| Control                        | Detalle                                                     |
|--------------------------------|-------------------------------------------------------------|
| Apagado de emergencia          | Un botón para cerrar todo inmediatamente                    |
| Mensaje de ausencia            | Configurable (ej: "Ainara está cerrado, abrimos a las 16:00hs") |
| Mensajes por situación         | Sin repartidores, pedir por WhatsApp, etc.                  |

### Pagos y precios

- Pago en efectivo habilitado/deshabilitado (según disponibilidad de repartidores propios).
- Valor de **compra mínima**.
- **Precios diferenciados por medio de pago** (opcional).

### Otros

- Lista de repartidores (altas, bajas, asignación).
- Leyenda de ticket.
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

- [ ] Layout admin (sidebar + contenido) con autenticación de rol admin
- [ ] **Pedidos**: vista listado con tabla, acciones básicas (avanzar estado, cancelar)
- [ ] **Pedidos**: vista Kanban (3 columnas)
- [ ] **Pedidos**: alerta sonora y visual de nuevo pedido
- [ ] **Productos**: CRUD de productos con los 3 arquetipos
- [ ] **Sabores**: lista maestra global con toggle ON/OFF
- [ ] **Configuración**: horarios de apertura, apagado de emergencia, mensaje de ausencia

### Fase 2 — Operación completa

> Objetivo: cubrir el día a día operativo completo.

- [ ] **Pedidos**: carga manual (WhatsApp, mostrador, delivery)
- [ ] **Pedidos**: imprimir comanda
- [ ] **Pedidos**: edición de pedido, pedido sin costo, forzar fuera de zona
- [ ] **Pedidos**: estados de delivery paralelos
- [ ] **Zonas**: configuración por radio y polígono, demora estimada, apagar zona
- [ ] **Delivery**: lista de repartidores, asignación
- [ ] **Configuración**: pagos (efectivo on/off, compra mínima, precios diferenciados)

### Fase 3 — Clientes y fidelización

> Objetivo: gestionar la relación con los clientes.

- [ ] **Clientes**: listado, clasificación VIP, notas privadas
- [ ] **Puntos**: configuración de acumulación y canje
- [ ] **Puntos Usados**: historial de canjes
- [ ] **Promos**: CRUD de promos, vinculación a productos o al carrito

### Fase 4 — Facturación y analítica

> Objetivo: dar visibilidad financiera y de negocio.

- [ ] **Facturación**: acumulados por fecha/medio de pago, caja chica, exportación
- [ ] **Informe**: origen de clientes (UTM), encuesta post-pedido
- [ ] **Rentabilidad**: costos estimados, análisis TC y comisiones (fase futura)

### Fase 5 — Integraciones

> Objetivo: conectar con el ecosistema externo.

- [ ] Integración Rapiboy (GPS tracking de delivery)
- [ ] Gift cards / vouchers
- [ ] Cash flow y estado de resultados
