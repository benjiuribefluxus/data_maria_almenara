# INFORME SISTEMA DE VENTAS - INFOREST

## RESUMEN EJECUTIVO

Este informe analiza la estructura de datos del sistema de ventas INFOREST, compuesto por 16 tablas que manejan el proceso completo de pedidos, facturación, pagos y productos. El sistema está diseñado para restaurantes con múltiples canales de venta (local, delivery, aggregadores) y maneja tanto pedidos simples como combos.

## ANÁLISIS DE TABLAS

### 1. TABLAS MAESTRAS (Configuración)

#### TCAJA - Configuración de Cajas
- **Propósito**: Define las cajas registradoras y sus configuraciones
- **Campos principales**: 
  - `tCaja`: Código de caja (PK)
  - `tDescripcion`: Nombre descriptivo de la caja
  - `lActivo`: Estado activo/inactivo
  - Múltiples configuraciones de operación (passwords, impresión, etc.)
- **Registros**: 15 cajas configuradas

#### TCANALVENTA - Canales de Venta
- **Propósito**: Define los diferentes canales de venta disponibles
- **Campos principales**:
  - `tCodigoCanalVenta`: Código del canal (PK)
  - `tDetallado`: Descripción del canal
  - `lCanalDelivery`: Indica si es canal delivery
- **Canales identificados**:
  - EN EL LOCAL
  - ECOMMERCE  
  - TO GO
  - AGREGADORES
  - CANAL CORP

#### TORIGENVENTA - Orígenes de Venta
- **Propósito**: Especifica el origen específico dentro de cada canal
- **Relación**: Relacionado con TCANALVENTA
- **Ejemplos**: Rappi, Pedidos Ya, Uber, Didi

#### TPRODUCTO - Catálogo de Productos
- **Propósito**: Maestro de productos disponibles
- **Campos principales**:
  - `tCodigoProducto`: Código único del producto (PK)
  - `tGrupo`, `tSubGrupo`: Categorización jerárquica
  - `tDetallado`: Nombre completo del producto
  - `nPrecioVenta`: Precio base
  - `lActivo`: Estado del producto
- **Registros**: 2,988 productos

#### TCOMBO - Definición de Combos
- **Propósito**: Define qué productos forman parte de combos
- **Relación**: Vincula productos individuales en ofertas combinadas
- **Campos**: `tCombo`, `tCodigoProducto`, `nCantidad`

#### CODIGODELIVERY - Códigos de Delivery
- **Propósito**: Catálogo de códigos para servicios de delivery
- **Estructura simple**: `id`, `letra`

### 2. TABLAS TRANSACCIONALES - PEDIDOS

#### MPEDIDO - Maestro de Pedidos
- **Propósito**: Encabezado de cada pedido realizado
- **Campos críticos**:
  - `tCodigoPedido`: Código único del pedido (PK)
  - `tClienteDelivery`, `tClienteCtaCte`: Identificación del cliente
  - `fFecha`: Fecha del pedido
  - `tEstadoPedido`: Estado actual (1=Activo, 2=Facturado, etc.)
  - `tTipoPedido`: Tipo de atención
  - `tCaja`, `tTurno`: Punto de venta y turno
  - `CodigoOrigenVenta`: Relación con TORIGENVENTA

#### DPEDIDO - Detalle de Pedidos
- **Propósito**: Líneas individuales de cada pedido
- **Relación**: Hijo de MPEDIDO
- **Campos importantes**:
  - `tCodigoPedido`: Referencia al pedido padre
  - `tItem`: Número de línea
  - `tCodigoProducto`: Producto solicitado
  - `nCantidad`: Cantidad pedida
  - `nPrecioVenta`: Precio unitario
  - `tEstadoItem`: Estado del item (N=Normal, A=Anulado)

#### CPEDIDO - Componentes de Pedidos (Combos)
- **Propósito**: Desglosa los productos individuales de los combos
- **Relación**: Detalla DPEDIDO cuando hay combos
- **Estructura**: Similar a DPEDIDO pero para items de combo

#### APEDIDO - Adicionales de Pedidos
- **Propósito**: Maneja modificaciones y adicionales a productos
- **Uso**: Customizaciones del cliente (extra ingredientes, etc.)

#### TPEDIDO - Transferencias de Pedidos
- **Propósito**: Registra transferencias entre puntos de venta
- **Campos**: Pedido origen, destino, productos transferidos

### 3. TABLAS TRANSACCIONALES - FACTURACIÓN

#### MDOCUMENTO - Maestro de Documentos
- **Propósito**: Encabezado de facturas y boletas
- **Campos principales**:
  - `tDocumento`: Número de documento (PK)
  - `tTipoDocumento`: Tipo (factura, boleta, etc.)
  - `nVenta`: Monto total
  - `tEstadoDocumento`: Estado del documento
  - `lEstadoFacturacion`: Estado en SUNAT

#### DDOCUMENTO - Detalle de Documentos
- **Propósito**: Líneas de las facturas/boletas
- **Relación**: Hijo de MDOCUMENTO
- **Vinculación**: Relaciona con pedidos via `tCodigoPedido`

#### MNOTACREDITO - Maestro de Notas de Crédito
- **Propósito**: Encabezado de notas de crédito
- **Uso**: Anulaciones y devoluciones

#### DNOTACREDITO - Detalle de Notas de Crédito
- **Propósito**: Líneas de las notas de crédito
- **Relación**: Hijo de MNOTACREDITO

### 4. TABLAS DE PAGOS

#### DPAGODOCUMENTO - Pagos de Documentos
- **Propósito**: Registra los pagos de facturas
- **Campos importantes**:
  - `tDocumento`: Documento pagado
  - `tTipoPago`: Forma de pago (efectivo, tarjeta, etc.)
  - `nMonto`: Monto pagado
  - `tTarjeta`: Información de tarjeta si aplica

#### DPREPAGO - Prepagos
- **Propósito**: Registra pagos anticipados
- **Uso**: Pagos antes de la entrega del servicio

## RELACIONES PRINCIPALES

### Flujo de Pedido a Factura
```
TCANALVENTA → TORIGENVENTA → MPEDIDO → DPEDIDO → TPRODUCTO
                                   ↓
                            MDOCUMENTO → DDOCUMENTO → DPAGODOCUMENTO
```

### Gestión de Combos
```
TPRODUCTO → TCOMBO → CPEDIDO ← DPEDIDO ← MPEDIDO
```

### Proceso de Anulación
```
MDOCUMENTO → MNOTACREDITO → DNOTACREDITO
```

## ANÁLISIS DE DATOS

### Volumen de Transacciones
- **MPEDIDO**: ~45,000 registros (pedidos)
- **DPEDIDO**: ~310,000 registros (líneas de pedido)
- **MDOCUMENTO**: ~310,000 registros (documentos)
- **MNOTACREDITO**: ~6,000 registros (anulaciones)

### Canales de Venta Principales
1. **EN EL LOCAL**: Venta directa en restaurante
2. **AGREGADORES**: Rappi, Pedidos Ya, Uber, Didi
3. **ECOMMERCE**: Plataforma propia
4. **TO GO**: Para llevar
5. **CANAL CORP**: Clientes corporativos

### Tipos de Productos
- **Grupo 17**: Pasteles y postres (mayoría)
- **Grupo 6**: Bebidas
- **Grupo 14**: Comidas especiales
- **Grupo 15**: Productos adicionales

## INTEGRIDAD Y CONSISTENCIA

### Claves Primarias Identificadas
- `MPEDIDO.tCodigoPedido`
- `MDOCUMENTO.tDocumento`
- `TPRODUCTO.tCodigoProducto`
- `TCAJA.tCaja`
- `TCANALVENTA.tCodigoCanalVenta`

### Relaciones Críticas
- DPEDIDO → MPEDIDO (Integridad referencial)
- DDOCUMENTO → MDOCUMENTO (Integridad referencial)
- DPEDIDO → TPRODUCTO (Validación de productos)
- MPEDIDO → TORIGENVENTA (Canal de venta)

## CONSIDERACIONES TÉCNICAS

### Estado de Datos
- Datos desde abril 2018 hasta julio 2025
- Sistema activo con transacciones recientes
- Múltiples tipos de documentos electrónicos

### Facturación Electrónica
- Integración con SUNAT (Perú)
- Estados de facturación controlados
- Códigos de autorización registrados

### Gestión Multi-tienda
- Soporte para múltiples puntos de venta
- Control de turnos y cajas
- Centralización de productos

## RECOMENDACIONES

1. **Optimización de Índices**: Crear índices en campos de fecha para consultas históricas
2. **Archivado de Datos**: Implementar archivado de transacciones antiguas
3. **Validaciones**: Fortalecer validaciones de integridad referencial
4. **Monitoreo**: Implementar alertas para estados de facturación fallidos
5. **Backup**: Asegurar respaldos regulares dada la criticidad del sistema

## CONCLUSIONES

El sistema INFOREST presenta una arquitectura robusta para la gestión de ventas en restaurantes, con soporte completo para:
- Múltiples canales de venta
- Facturación electrónica
- Gestión de combos y productos
- Control de pagos y prepagos
- Trazabilidad completa del proceso

La estructura permite escalabilidad y es compatible con las regulaciones fiscales peruanas. Los volúmenes de datos indican un negocio activo y en crecimiento. 