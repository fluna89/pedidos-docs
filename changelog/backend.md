# Changelog — Pedidos Backend

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
