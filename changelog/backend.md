# Changelog — Pedidos Backend

## [0.7.0] - 2026-04-12

### Google OAuth login

- **auth/app.py**: `handle_google` — real Google ID token verification via `google.oauth2.id_token.verify_oauth2_token`, replaces mock placeholder
- **auth/app.py**: existing user login + `googleLinked` flag, new user creation without password
- **layers/common/requirements.txt**: added `google-auth>=2.29.0`
- **template.yaml**: `GOOGLE_CLIENT_ID` environment variable via SSM parameter (`/pedidos/google-client-id`)
- **test_auth.py**: 4 new tests — new user, existing user, missing token, invalid token

## [0.6.0] - 2026-04-10

### Zonas de delivery por polígonos

- **geo.py**: refactor completo — eliminado `check_coverage` y zonas por radio (`maxKm`), nuevo `point_in_polygon` (ray-casting algorithm), `calc_delivery_zone` ahora evalúa polígonos dibujados en mapa
- **delivery/app.py**: nuevos endpoints admin `GET/PUT /api/admin/delivery/config` — CRUD de zonas con polígonos, costo, demora estimada, toggle on/off, persistencia de `colorIndex` para estabilidad visual en el frontend
- **delivery/app.py**: `handle_calc` migrado de zonas por radio a polígonos, filtra solo zonas activas (`enabled`)
- **addresses/app.py**: `_check_coverage` migrado de haversine + `maxDeliveryKm` a polígonos vía `calc_delivery_zone`, cobertura evaluada inline en `handle_list`
- **template.yaml**: DeliveryFunction policy `DynamoDBReadPolicy → DynamoDBCrudPolicy`, 2 nuevas rutas HTTP API (`GET/PUT /api/admin/delivery/config`)
- **test_geo.py**: tests actualizados a polígonos (point_in_polygon, calc_delivery_zone sin max_km)
- **test_delivery.py**: tests para admin get/update config, cálculo con polígonos
- **test_addresses.py**: tests actualizados para cobertura por polígonos

## [0.5.3] - 2026-04-07

### Pago en efectivo — monto con el que paga

- **orders/app.py**: campo opcional `cashPaysWith` almacenado en la orden cuando el cliente indica con cuánto paga en efectivo
- **orders/app.py**: `_format_order()` incluye `cashPaysWith` en la respuesta

## [0.5.2] - 2026-04-06

### Enriquecimiento de precios unitarios en catálogo

- **catalog/app.py**: nueva función `_enrich_min_item_prices()` — para productos con `unitPricing`, consulta la flavor source y calcula el precio mínimo entre sabores activos
- **catalog/app.py**: campo `minItemPrice` agregado a la respuesta de `/api/menu` y `/api/counter-menu` para productos con precio por unidad (ej: empanadas)

## [0.5.1] - 2026-04-06

### Venta en mostrador — orderType "mostrador"

- **admin_orders/app.py**: nuevo `COUNTER_FLOW = ["pendiente", "en_preparacion", "entregado"]` — flujo corto para pedidos de mostrador
- **admin_orders/app.py**: helper `_get_flow()` selecciona flujo según `orderType` (mostrador/pickup/delivery)
- **admin_orders/app.py**: `handle_create_counter()` default `orderType` cambiado de `"pickup"` a `"mostrador"`
- **test_admin_orders.py**: 5 tests nuevos — counter flow advance/revert, default mostrador type, explicit delivery type, linked user reference

## [0.5.0] - 2026-04-06

### Imágenes de productos (S3 + CloudFront)

- **admin_products/app.py**: endpoint `GET /api/admin/products/upload-url` genera presigned URL para subir imágenes a S3
- **admin_products/app.py**: al crear/actualizar producto, campo `image` almacena la key S3 (`products/{productId}.{ext}`)
- **admin_products/app.py**: `_get_s3_client()` lee credenciales desde `local_config.json` cuando `AWS_SAM_LOCAL=true` (bypass de env vars no propagadas por SAM)
- **template.yaml**: bucket S3 `PedidosImagesBucket` con política pública de lectura, distribución CloudFront `ImagesCDN` con OAC
- **template.yaml**: outputs `ImagesBucketName` y `CloudFrontDomain` exportados
- **.gitignore**: `local_config.json` excluido (contiene credenciales locales)
- **27 tests** actualizados para mock de `_get_s3_client` y `_get_local_config`

## [0.4.0] - 2026-04-03

### Configuración de tienda

- **store_config/app.py**: nuevo Lambda handler — GET/PUT `/api/admin/config/store` con autenticación admin
- Horarios de apertura multi-rango (ej: 10-14 y 18-23), por día de la semana, con timezone `America/Argentina/Buenos_Aires`
- Apagado de emergencia: flag `emergencyShutdown` + mensaje personalizable
- Mensaje de comanda: campo libre que se muestra al imprimir/ver comanda del pedido
- **25 tests**: cobertura completa de horarios, emergencia, comanda, timezone, bloqueo de pedidos fuera de horario

### Teléfono de contacto obligatorio en pedidos

- **orders/app.py**: campo `contactPhone` requerido en creación de pedido, almacenado y devuelto en `_format_order`
- **admin_orders/app.py**: `contactPhone` en pedidos de mostrador y listado admin
- **auth/app.py**: teléfono obligatorio en registro (`if not phone: return bad_request`), actualizable en perfil, incluido en `_sanitize_user`

### Recuperación de contraseña

- **auth/app.py**: `handle_recover` genera token temporal y `handle_reset` valida token + actualiza contraseña (bcrypt)

### Infraestructura

- `JWT_EXPIRATION_HOURS` configurable via env var (default 24h) en `auth.py`
- `StoreConfigFunction` agregada a `template.yaml`
- `scripts/tail-logs.ps1`: script para tail de logs de CloudWatch

## [0.3.1] - 2026-03-26

### Comentarios de pasos en combos

- **orders/app.py**: incluye `comment` a nivel de paso en el string de sabores del combo (formato: `— [comment]`)
- **admin_orders/app.py**: mismo cambio para la normalización de pedidos de mostrador

## [0.3.0] - 2026-03-25

### Campos de comentario en pedidos

- **orders/app.py**: soporte de campo `comment` a nivel de item y de pedido en create, normalize y format
- **admin_orders/app.py**: mismo soporte de comentarios para pedidos de mostrador y listado admin
- **seeds/seed.py**: agregados ejemplos de comentarios por item y por pedido en los seeds

## [0.2.0] - 2026-03-24

### Suite de tests y setup local

- **125 tests**: cobertura completa para los 12 handlers — auth, catalog, addresses, orders, payments, loyalty, coupons, delivery, admin_orders, admin_products, admin_flavors, admin_users
- **pyproject.toml**: config de pytest con `pythonpath` para resolución del common layer
- **conftest.py**: agregado `AWS_DEFAULT_REGION` para compatibilidad con moto, removido `@mock_aws` redundante de las funciones de test
- **Dev local**: `env.json` para variables de entorno de SAM local, `DYNAMODB_ENDPOINT` en globals del template
- **db.py**: `region_name` explícito al usar endpoint custom (DynamoDB Local)
- **.gitignore**: agregado `env.json`

## [0.1.0] - 2026-03-24

### Setup inicial del proyecto

- **Proyecto AWS SAM**: `template.yaml` con API Gateway HTTP API, DynamoDB single-table, Lambda Layer y 12 funciones Lambda
- **Diseño single-table de DynamoDB**: `PedidosTable` con PK/SK + GSI1 soportando 18 patrones de acceso (usuarios, productos, pedidos, direcciones, puntos, cupones, configuración)
- **Lambda Layer compartido**: utilidades comunes — `db.py` (helpers de DynamoDB), `auth.py` (JWT + bcrypt), `responses.py` (respuestas HTTP estandarizadas), `exceptions.py`, `geo.py` (Haversine + zonas de delivery)
- **Handler de auth**: login, registro, sesión de invitado, placeholder OAuth Google, placeholder recuperación de contraseña, actualización de perfil
- **Handler de catálogo**: obtener menú (todo/por categoría/por ID/por IDs), categorías, sabores por fuente, menú de mostrador (admin)
- **Handler de direcciones**: CRUD con validación de cobertura por Haversine
- **Handler de pedidos**: crear pedido (con normalización de items), listar pedidos del usuario, obtener pedidos activos
- **Handler de pagos**: obtener métodos de pago disponibles, procesar pago (estado según tipo de pago)
- **Handler de fidelización**: obtener saldo de puntos (con chequeo de vencimiento), historial, canjear puntos, acumular puntos
- **Handler de cupones**: validar código de cupón (tipo, expiración, pedido mínimo, descuento máximo)
- **Handler de delivery**: calcular costo de envío por coordenadas con matching de zonas
- **Handler admin de pedidos**: listar todos los pedidos, avanzar/retroceder/cancelar/establecer estado
- **Handler admin de productos**: CRUD completo, toggle pausa, uso de producto en combos, obtener productos base para picker de combos
- **Handler admin de sabores**: CRUD de fuentes de sabores y sabores individuales
- **Handler admin de usuarios**: búsqueda de usuarios por nombre/email
- **Script de seed**: pobla DynamoDB con todos los datos mock del frontend (usuarios, productos, categorías, sabores, direcciones, pedidos, cupones, métodos de pago, config de delivery, historial de puntos)
- **Infraestructura de tests**: pytest + fixtures de moto con setup de tabla DynamoDB, helpers JWT
- **Docker Compose**: DynamoDB Local en puerto 8100
- **CLAUDE.md**: contexto completo para agente IA con convenciones, patrones de acceso y comandos
