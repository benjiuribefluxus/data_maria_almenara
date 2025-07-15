# DICCIONARIO DE DATOS - SISTEMA INFOREST

## INFORMACIÓN GENERAL

**Sistema**: INFOREST - Sistema de Ventas para Restaurantes  
**Versión**: Base de datos operativa 2018-2025  
**SGBD**: SQL Server (inferido por prefijo "dbo.")  
**Propósito**: Gestión integral de ventas multi-canal con facturación electrónica  

---

## TABLAS MAESTRAS - CONFIGURACIÓN

### 1. TCAJA - Configuración de Cajas Registradoras

**Propósito**: Define las estaciones de trabajo y cajas registradoras del sistema.

| Campo | Tipo | Restricciones | Descripción | Ejemplo |
|-------|------|---------------|-------------|---------|
| tCaja | NVARCHAR(10) | PK, NOT NULL | Código único identificador de la caja | "001", "002" |
| tDescripcion | NVARCHAR(100) | NOT NULL | Nombre descriptivo de la caja | "Caja Principal", "Delivery" |
| lActivo | BIT | NOT NULL | Estado activo/inactivo de la caja | 1 = Activo, 0 = Inactivo |
| tPassword | NVARCHAR(50) | NULL | Contraseña de acceso a la caja | Encriptada |
| lImprimeComanda | BIT | DEFAULT 0 | Indica si imprime comandas | 1 = Sí, 0 = No |
| tImpresora | NVARCHAR(50) | NULL | Nombre de la impresora asignada | "HP_Laser_01" |
| nPorcentajeDescuento | DECIMAL(5,2) | DEFAULT 0 | Porcentaje máximo de descuento | 10.00 = 10% |

**Registros**: 15 cajas configuradas  
**Clave Primaria**: tCaja  
**Índices**: tCaja, lActivo  

---

### 2. TCANALVENTA - Canales de Venta

**Propósito**: Define los diferentes canales de venta disponibles en el negocio.

| Campo | Tipo | Restricciones | Descripción | Ejemplo |
|-------|------|---------------|-------------|---------|
| tCodigoCanalVenta | NVARCHAR(10) | PK, NOT NULL | Código único del canal de venta | "LOC", "ECO", "AGR" |
| tDetallado | NVARCHAR(100) | NOT NULL | Descripción completa del canal | "EN EL LOCAL", "ECOMMERCE" |
| lCanalDelivery | BIT | NOT NULL | Indica si es canal de delivery | 1 = Delivery, 0 = Local |
| lActivo | BIT | DEFAULT 1 | Estado del canal | 1 = Activo, 0 = Inactivo |
| nOrden | INT | NULL | Orden de presentación | 1, 2, 3... |

**Valores Estándar**:
- LOC: EN EL LOCAL
- ECO: ECOMMERCE
- TOG: TO GO
- AGR: AGREGADORES
- COR: CANAL CORP

**Registros**: 5 canales principales  
**Clave Primaria**: tCodigoCanalVenta  

---

### 3. TORIGENVENTA - Orígenes Específicos de Venta

**Propósito**: Especifica el origen detallado dentro de cada canal de venta.

| Campo | Tipo | Restricciones | Descripción | Ejemplo |
|-------|------|---------------|-------------|---------|
| tCodigoOrigenVenta | NVARCHAR(10) | PK, NOT NULL | Código único del origen | "RAP", "UBE", "PYA" |
| tCodigoCanalVenta | NVARCHAR(10) | FK, NOT NULL | Canal al que pertenece | "AGR", "ECO" |
| tDescripcion | NVARCHAR(100) | NOT NULL | Nombre del origen específico | "Rappi", "Uber Eats" |
| lDelivery | BIT | NOT NULL | Indica si maneja delivery | 1 = Sí, 0 = No |
| nComision | DECIMAL(5,2) | DEFAULT 0 | Porcentaje de comisión | 15.00 = 15% |
| lActivo | BIT | DEFAULT 1 | Estado del origen | 1 = Activo, 0 = Inactivo |

**Registros**: 58 orígenes específicos  
**Clave Primaria**: tCodigoOrigenVenta  
**Clave Foránea**: tCodigoCanalVenta → TCANALVENTA  

---

### 4. TPRODUCTO - Catálogo de Productos

**Propósito**: Maestro principal de productos disponibles para la venta.

| Campo | Tipo | Restricciones | Descripción | Ejemplo |
|-------|------|---------------|-------------|---------|
| tCodigoProducto | NVARCHAR(15) | PK, NOT NULL | Código único del producto | "PAS001", "BEB015" |
| tGrupo | NVARCHAR(10) | NOT NULL | Código del grupo principal | "17", "6", "14" |
| tSubGrupo | NVARCHAR(10) | NULL | Código del subgrupo | "001", "002" |
| tDetallado | NVARCHAR(200) | NOT NULL | Descripción completa del producto | "Torta Chocolate Individual" |
| nPrecioVenta | DECIMAL(10,2) | NOT NULL | Precio de venta base | 25.90 |
| nPrecioCosto | DECIMAL(10,2) | NULL | Costo del producto | 15.50 |
| lActivo | BIT | DEFAULT 1 | Estado del producto | 1 = Activo, 0 = Inactivo |
| lCombo | BIT | DEFAULT 0 | Indica si es un combo | 1 = Combo, 0 = Individual |
| nStock | INT | DEFAULT 0 | Cantidad en stock | 50 |
| tUnidadMedida | NVARCHAR(10) | DEFAULT 'UNI' | Unidad de medida | "UNI", "KG", "LT" |

**Grupos Principales**:
- Grupo 17: Pasteles y postres
- Grupo 6: Bebidas
- Grupo 14: Comidas especiales
- Grupo 15: Productos adicionales

**Registros**: 2,988 productos  
**Clave Primaria**: tCodigoProducto  
**Índices**: tGrupo, tSubGrupo, lActivo  

---

### 5. TCOMBO - Composición de Combos

**Propósito**: Define qué productos individuales conforman cada combo.

| Campo | Tipo | Restricciones | Descripción | Ejemplo |
|-------|------|---------------|-------------|---------|
| tCombo | NVARCHAR(15) | FK, NOT NULL | Código del producto combo | "CMB001" |
| tCodigoProducto | NVARCHAR(15) | FK, NOT NULL | Producto componente del combo | "PAS001", "BEB015" |
| nCantidad | DECIMAL(10,3) | NOT NULL | Cantidad del producto en el combo | 1.000, 0.500 |
| lPrincipal | BIT | DEFAULT 0 | Indica si es el producto principal | 1 = Principal, 0 = Accesorio |
| nOrden | INT | DEFAULT 1 | Orden de presentación | 1, 2, 3... |

**Registros**: 562 definiciones de combo  
**Clave Primaria**: tCombo + tCodigoProducto  
**Claves Foráneas**: 
- tCombo → TPRODUCTO.tCodigoProducto
- tCodigoProducto → TPRODUCTO.tCodigoProducto

---

### 6. CODIGODELIVERY - Códigos de Delivery

**Propósito**: Catálogo de códigos utilizados para identificar servicios de delivery.

| Campo | Tipo | Restricciones | Descripción | Ejemplo |
|-------|------|---------------|-------------|---------|
| id | INT | PK, IDENTITY | Identificador secuencial | 1, 2, 3... |
| letra | NVARCHAR(5) | NOT NULL, UNIQUE | Código alfanumérico | "A", "B", "C1" |
| descripcion | NVARCHAR(50) | NULL | Descripción del código | "Zona Norte", "Express" |

**Registros**: Variable según configuración  
**Clave Primaria**: id  

---

## TABLAS TRANSACCIONALES - GESTIÓN DE PEDIDOS

### 7. MPEDIDO - Maestro de Pedidos

**Propósito**: Encabezado principal de cada pedido realizado en el sistema.

| Campo | Tipo | Restricciones | Descripción | Ejemplo |
|-------|------|---------------|-------------|---------|
| tCodigoPedido | NVARCHAR(20) | PK, NOT NULL | Código único del pedido | "PED20240115001" |
| fFecha | DATETIME | NOT NULL | Fecha y hora del pedido | 2024-01-15 14:30:00 |
| tClienteDelivery | NVARCHAR(100) | NULL | Nombre del cliente delivery | "Juan Pérez" |
| tClienteCtaCte | NVARCHAR(20) | NULL | Código de cliente cuenta corriente | "CLI001" |
| tEstadoPedido | NVARCHAR(2) | NOT NULL | Estado actual del pedido | "1", "2", "3" |
| tTipoPedido | NVARCHAR(10) | NOT NULL | Tipo de atención | "NORMAL", "EXPRESS" |
| tCaja | NVARCHAR(10) | FK, NOT NULL | Caja que registró el pedido | "001" |
| tTurno | NVARCHAR(10) | NOT NULL | Turno de trabajo | "MAÑANA", "TARDE" |
| CodigoOrigenVenta | NVARCHAR(10) | FK, NULL | Origen específico del pedido | "RAP", "LOC" |
| nSubTotal | DECIMAL(10,2) | DEFAULT 0 | Subtotal sin descuentos | 45.90 |
| nDescuento | DECIMAL(10,2) | DEFAULT 0 | Monto de descuento aplicado | 5.00 |
| nTotal | DECIMAL(10,2) | NOT NULL | Total final del pedido | 40.90 |
| tObservaciones | NTEXT | NULL | Observaciones especiales | "Sin cebolla" |

**Estados de Pedido**:
- 1: Activo/Pendiente
- 2: Facturado
- 3: Anulado
- 4: En preparación
- 5: Entregado

**Registros**: ~45,000 pedidos  
**Clave Primaria**: tCodigoPedido  
**Claves Foráneas**:
- tCaja → TCAJA.tCaja
- CodigoOrigenVenta → TORIGENVENTA.tCodigoOrigenVenta

---

### 8. DPEDIDO - Detalle de Pedidos

**Propósito**: Líneas individuales de productos dentro de cada pedido.

| Campo | Tipo | Restricciones | Descripción | Ejemplo |
|-------|------|---------------|-------------|---------|
| tCodigoPedido | NVARCHAR(20) | FK, NOT NULL | Código del pedido padre | "PED20240115001" |
| tItem | NVARCHAR(10) | NOT NULL | Número de línea del item | "001", "002" |
| tCodigoProducto | NVARCHAR(15) | FK, NOT NULL | Producto solicitado | "PAS001" |
| nCantidad | DECIMAL(10,3) | NOT NULL | Cantidad pedida | 2.000 |
| nPrecioVenta | DECIMAL(10,2) | NOT NULL | Precio unitario de venta | 25.90 |
| nDescuento | DECIMAL(10,2) | DEFAULT 0 | Descuento aplicado al item | 2.00 |
| nTotal | DECIMAL(10,2) | NOT NULL | Total de la línea | 49.80 |
| tEstadoItem | NVARCHAR(2) | DEFAULT 'N' | Estado del item | "N", "A", "P" |
| tObservaciones | NVARCHAR(500) | NULL | Observaciones del item | "Extra queso" |

**Estados de Item**:
- N: Normal
- A: Anulado
- P: Pendiente
- E: Entregado

**Registros**: ~310,000 líneas de pedido  
**Clave Primaria**: tCodigoPedido + tItem  
**Claves Foráneas**:
- tCodigoPedido → MPEDIDO.tCodigoPedido
- tCodigoProducto → TPRODUCTO.tCodigoProducto

---

### 9. CPEDIDO - Componentes de Combos en Pedidos

**Propósito**: Detalla los productos individuales cuando se pide un combo.

| Campo | Tipo | Restricciones | Descripción | Ejemplo |
|-------|------|---------------|-------------|---------|
| tCodigoPedido | NVARCHAR(20) | FK, NOT NULL | Código del pedido | "PED20240115001" |
| tItem | NVARCHAR(10) | NOT NULL | Número de línea del combo | "001" |
| tItemCombo | NVARCHAR(10) | NOT NULL | Sublínea del componente | "001", "002" |
| tProductoCombo | NVARCHAR(15) | FK, NOT NULL | Producto componente | "PAS001" |
| nCantidad | DECIMAL(10,3) | NOT NULL | Cantidad del componente | 1.000 |
| nPrecio | DECIMAL(10,2) | NOT NULL | Precio del componente | 0.00 |
| tEstadoItem | NVARCHAR(2) | DEFAULT 'N' | Estado del componente | "N", "A" |

**Registros**: Variable según combos pedidos  
**Clave Primaria**: tCodigoPedido + tItem + tItemCombo  
**Claves Foráneas**:
- tCodigoPedido → MPEDIDO.tCodigoPedido
- tProductoCombo → TPRODUCTO.tCodigoProducto

---

### 10. APEDIDO - Adicionales de Pedidos

**Propósito**: Registra adicionales y modificaciones solicitadas por el cliente.

| Campo | Tipo | Restricciones | Descripción | Ejemplo |
|-------|------|---------------|-------------|---------|
| tCodigoPedido | NVARCHAR(20) | FK, NOT NULL | Código del pedido | "PED20240115001" |
| tItem | NVARCHAR(10) | NOT NULL | Línea del pedido base | "001" |
| tItemAdicional | NVARCHAR(10) | NOT NULL | Sublínea del adicional | "001" |
| tCodigoProducto | NVARCHAR(15) | FK, NOT NULL | Producto adicional | "ADD001" |
| nCantidad | DECIMAL(10,3) | NOT NULL | Cantidad adicional | 1.000 |
| nPrecio | DECIMAL(10,2) | NOT NULL | Precio del adicional | 3.50 |
| tTipoAdicional | NVARCHAR(10) | NOT NULL | Tipo de adicional | "EXTRA", "SIN" |

**Tipos de Adicional**:
- EXTRA: Ingrediente adicional con costo
- SIN: Remover ingrediente sin costo
- CAMBIO: Cambiar ingrediente

**Registros**: Variable según personalizaciones  
**Clave Primaria**: tCodigoPedido + tItem + tItemAdicional  

---

### 11. TPEDIDO - Transferencias de Pedidos

**Propósito**: Registra transferencias de productos entre puntos de venta.

| Campo | Tipo | Restricciones | Descripción | Ejemplo |
|-------|------|---------------|-------------|---------|
| nCorrelativo | INT | PK, IDENTITY | Correlativo de transferencia | 1, 2, 3... |
| fFecha | DATETIME | NOT NULL | Fecha de transferencia | 2024-01-15 16:00:00 |
| tPedidoIni | NVARCHAR(20) | FK, NOT NULL | Pedido origen | "PED20240115001" |
| tPedidoFin | NVARCHAR(20) | FK, NOT NULL | Pedido destino | "PED20240115002" |
| tProducto | NVARCHAR(15) | FK, NOT NULL | Producto transferido | "PAS001" |
| nCantidad | DECIMAL(10,3) | NOT NULL | Cantidad transferida | 2.000 |
| tMotivo | NVARCHAR(200) | NULL | Motivo de la transferencia | "Cambio de local" |

**Registros**: Variable según transferencias  
**Clave Primaria**: nCorrelativo  

---

## TABLAS TRANSACCIONALES - FACTURACIÓN

### 12. MDOCUMENTO - Maestro de Documentos Fiscales

**Propósito**: Encabezado de facturas, boletas y otros documentos fiscales.

| Campo | Tipo | Restricciones | Descripción | Ejemplo |
|-------|------|---------------|-------------|---------|
| tDocumento | NVARCHAR(20) | PK, NOT NULL | Número del documento fiscal | "F001-00000123" |
| fFecha | DATETIME | NOT NULL | Fecha de emisión | 2024-01-15 14:35:00 |
| tTipoDocumento | NVARCHAR(5) | NOT NULL | Tipo de documento | "01", "03", "07" |
| tCliente | NVARCHAR(100) | NULL | Nombre/razón social del cliente | "Juan Pérez SAC" |
| tRucCliente | NVARCHAR(15) | NULL | RUC/DNI del cliente | "20123456789" |
| nSubTotal | DECIMAL(10,2) | NOT NULL | Subtotal gravado | 40.68 |
| nIGV | DECIMAL(10,2) | NOT NULL | Impuesto IGV | 7.32 |
| nVenta | DECIMAL(10,2) | NOT NULL | Total del documento | 48.00 |
| tEstadoDocumento | NVARCHAR(2) | DEFAULT '1' | Estado del documento | "1", "2", "3" |
| lEstadoFacturacion | BIT | DEFAULT 0 | Estado en SUNAT | 1 = Enviado, 0 = Pendiente |
| tCodigoAutorizacion | NVARCHAR(50) | NULL | Código de autorización SUNAT | "ABC123456" |

**Tipos de Documento SUNAT**:
- 01: Factura
- 03: Boleta de Venta
- 07: Nota de Crédito
- 08: Nota de Débito

**Estados de Documento**:
- 1: Emitido
- 2: Anulado
- 3: Enviado a SUNAT

**Registros**: ~310,000 documentos  
**Clave Primaria**: tDocumento  

---

### 13. DDOCUMENTO - Detalle de Documentos Fiscales

**Propósito**: Líneas individuales de los documentos fiscales.

| Campo | Tipo | Restricciones | Descripción | Ejemplo |
|-------|------|---------------|-------------|---------|
| tDocumento | NVARCHAR(20) | FK, NOT NULL | Número del documento | "F001-00000123" |
| tItem | NVARCHAR(10) | NOT NULL | Número de línea | "001", "002" |
| tCodigoPedido | NVARCHAR(20) | FK, NULL | Pedido relacionado | "PED20240115001" |
| tCodigoProducto | NVARCHAR(15) | FK, NOT NULL | Producto facturado | "PAS001" |
| tDescripcion | NVARCHAR(200) | NOT NULL | Descripción para factura | "Torta Chocolate Individual" |
| nCantidad | DECIMAL(10,3) | NOT NULL | Cantidad facturada | 2.000 |
| nPrecioUnitario | DECIMAL(10,4) | NOT NULL | Precio unitario | 25.9000 |
| nDescuento | DECIMAL(10,2) | DEFAULT 0 | Descuento aplicado | 0.00 |
| nValorVenta | DECIMAL(10,2) | NOT NULL | Valor de venta sin IGV | 43.73 |
| nIGV | DECIMAL(10,2) | NOT NULL | IGV de la línea | 7.87 |
| nTotal | DECIMAL(10,2) | NOT NULL | Total de la línea | 51.60 |

**Registros**: Variable según líneas de documento  
**Clave Primaria**: tDocumento + tItem  
**Claves Foráneas**:
- tDocumento → MDOCUMENTO.tDocumento
- tCodigoPedido → MPEDIDO.tCodigoPedido
- tCodigoProducto → TPRODUCTO.tCodigoProducto

---

### 14. MNOTACREDITO - Maestro de Notas de Crédito

**Propósito**: Encabezado de notas de crédito para anulaciones y devoluciones.

| Campo | Tipo | Restricciones | Descripción | Ejemplo |
|-------|------|---------------|-------------|---------|
| tNotaCredito | NVARCHAR(20) | PK, NOT NULL | Número de la nota de crédito | "NC01-00000001" |
| fFecha | DATETIME | NOT NULL | Fecha de emisión | 2024-01-15 16:00:00 |
| tDocumento | NVARCHAR(20) | FK, NOT NULL | Documento que anula | "F001-00000123" |
| tMotivo | NVARCHAR(200) | NOT NULL | Motivo de la anulación | "Anulación por error" |
| nSubTotal | DECIMAL(10,2) | NOT NULL | Subtotal de la nota | 40.68 |
| nIGV | DECIMAL(10,2) | NOT NULL | IGV de la nota | 7.32 |
| nVenta | DECIMAL(10,2) | NOT NULL | Total de la nota | 48.00 |
| tEstadoNota | NVARCHAR(2) | DEFAULT '1' | Estado de la nota | "1", "2" |
| lEstadoFacturacion | BIT | DEFAULT 0 | Estado en SUNAT | 1 = Enviado, 0 = Pendiente |

**Registros**: ~6,000 notas de crédito  
**Clave Primaria**: tNotaCredito  
**Clave Foránea**: tDocumento → MDOCUMENTO.tDocumento  

---

### 15. DNOTACREDITO - Detalle de Notas de Crédito

**Propósito**: Líneas individuales de las notas de crédito.

| Campo | Tipo | Restricciones | Descripción | Ejemplo |
|-------|------|---------------|-------------|---------|
| tNotaCredito | NVARCHAR(20) | FK, NOT NULL | Número de la nota | "NC01-00000001" |
| tItem | NVARCHAR(10) | NOT NULL | Número de línea | "001" |
| tCodigoProducto | NVARCHAR(15) | FK, NOT NULL | Producto anulado | "PAS001" |
| tDescripcion | NVARCHAR(200) | NOT NULL | Descripción del producto | "Torta Chocolate Individual" |
| nCantidad | DECIMAL(10,3) | NOT NULL | Cantidad anulada | 1.000 |
| nPrecioUnitario | DECIMAL(10,4) | NOT NULL | Precio unitario | 25.9000 |
| nValorVenta | DECIMAL(10,2) | NOT NULL | Valor sin IGV | 21.86 |
| nIGV | DECIMAL(10,2) | NOT NULL | IGV anulado | 3.94 |
| nTotal | DECIMAL(10,2) | NOT NULL | Total anulado | 25.80 |

**Registros**: Variable según líneas anuladas  
**Clave Primaria**: tNotaCredito + tItem  
**Claves Foráneas**:
- tNotaCredito → MNOTACREDITO.tNotaCredito
- tCodigoProducto → TPRODUCTO.tCodigoProducto

---

## TABLAS TRANSACCIONALES - GESTIÓN DE PAGOS

### 16. DPAGODOCUMENTO - Pagos de Documentos

**Propósito**: Registra los pagos realizados contra documentos fiscales.

| Campo | Tipo | Restricciones | Descripción | Ejemplo |
|-------|------|---------------|-------------|---------|
| tDocumento | NVARCHAR(20) | FK, NOT NULL | Documento pagado | "F001-00000123" |
| tItem | NVARCHAR(10) | NOT NULL | Línea de pago | "001" |
| fFecha | DATETIME | NOT NULL | Fecha del pago | 2024-01-15 14:40:00 |
| tTipoPago | NVARCHAR(20) | NOT NULL | Forma de pago | "EFECTIVO", "TARJETA" |
| nMonto | DECIMAL(10,2) | NOT NULL | Monto pagado | 48.00 |
| tTarjeta | NVARCHAR(50) | NULL | Tipo de tarjeta si aplica | "VISA", "MASTERCARD" |
| tReferencia | NVARCHAR(50) | NULL | Número de referencia | "TXN123456789" |
| tBanco | NVARCHAR(50) | NULL | Banco emisor | "BCP", "BBVA" |
| tEstadoPago | NVARCHAR(2) | DEFAULT '1' | Estado del pago | "1", "2", "3" |

**Tipos de Pago**:
- EFECTIVO: Pago en efectivo
- TARJETA: Tarjeta de crédito/débito
- TRANSFERENCIA: Transferencia bancaria
- YAPE: Aplicativo Yape
- PLIN: Aplicativo Plin

**Estados de Pago**:
- 1: Pagado
- 2: Pendiente
- 3: Rechazado

**Registros**: Variable según pagos  
**Clave Primaria**: tDocumento + tItem  
**Clave Foránea**: tDocumento → MDOCUMENTO.tDocumento  

---

### 17. DPREPAGO - Prepagos

**Propósito**: Registra pagos anticipados antes de la entrega del servicio.

| Campo | Tipo | Restricciones | Descripción | Ejemplo |
|-------|------|---------------|-------------|---------|
| nCorrelativo | INT | PK, IDENTITY | Correlativo del prepago | 1, 2, 3... |
| fFecha | DATETIME | NOT NULL | Fecha del prepago | 2024-01-15 12:00:00 |
| tDocumento | NVARCHAR(20) | FK, NULL | Documento relacionado | "F001-00000123" |
| tCodigoPedido | NVARCHAR(20) | FK, NULL | Pedido relacionado | "PED20240115001" |
| tTipoPago | NVARCHAR(20) | NOT NULL | Forma de prepago | "TARJETA", "TRANSFERENCIA" |
| nMonto | DECIMAL(10,2) | NOT NULL | Monto prepagado | 25.00 |
| tTarjeta | NVARCHAR(50) | NULL | Tipo de tarjeta | "VISA" |
| tReferencia | NVARCHAR(50) | NULL | Referencia del pago | "PRE123456" |
| tEstadoPrepago | NVARCHAR(2) | DEFAULT '1' | Estado del prepago | "1", "2", "3" |

**Estados de Prepago**:
- 1: Activo
- 2: Aplicado
- 3: Anulado

**Registros**: Variable según prepagos  
**Clave Primaria**: nCorrelativo  

---

## RELACIONES ENTRE TABLAS

### Relaciones Principales

```
TCANALVENTA (1) ──→ (N) TORIGENVENTA
TORIGENVENTA (1) ──→ (N) MPEDIDO
MPEDIDO (1) ──→ (N) DPEDIDO
MPEDIDO (1) ──→ (N) CPEDIDO
MPEDIDO (1) ──→ (N) APEDIDO
TPRODUCTO (1) ──→ (N) DPEDIDO
TPRODUCTO (1) ──→ (N) TCOMBO
MPEDIDO (1) ──→ (N) DDOCUMENTO
MDOCUMENTO (1) ──→ (N) DDOCUMENTO
MDOCUMENTO (1) ──→ (N) DPAGODOCUMENTO
MDOCUMENTO (1) ──→ (N) MNOTACREDITO
MNOTACREDITO (1) ──→ (N) DNOTACREDITO
```

### Integridad Referencial

**Críticas (CASCADE)**:
- DPEDIDO → MPEDIDO
- DDOCUMENTO → MDOCUMENTO
- DNOTACREDITO → MNOTACREDITO

**Estándar (RESTRICT)**:
- DPEDIDO → TPRODUCTO
- TORIGENVENTA → TCANALVENTA
- MPEDIDO → TORIGENVENTA

---

## ÍNDICES RECOMENDADOS

### Índices de Performance
```sql
-- Consultas frecuentes por fecha
CREATE INDEX IX_MPEDIDO_FECHA ON MPEDIDO(fFecha DESC)
CREATE INDEX IX_MDOCUMENTO_FECHA ON MDOCUMENTO(fFecha DESC)

-- Búsquedas por cliente
CREATE INDEX IX_MPEDIDO_CLIENTE ON MPEDIDO(tClienteDelivery)

-- Estados de documentos
CREATE INDEX IX_MDOCUMENTO_ESTADO ON MDOCUMENTO(tEstadoDocumento, lEstadoFacturacion)

-- Productos activos
CREATE INDEX IX_TPRODUCTO_ACTIVO ON TPRODUCTO(lActivo, tGrupo)
```

---

## CONSIDERACIONES TÉCNICAS

### Tipos de Datos
- **NVARCHAR**: Campos de texto con soporte Unicode
- **DECIMAL**: Campos monetarios y cantidades precisas
- **BIT**: Campos booleanos (0/1)
- **DATETIME**: Fechas con hora completa
- **INT**: Identificadores numéricos
- **NTEXT**: Textos largos (observaciones)

### Restricciones de Negocio
- Precios siempre positivos
- Cantidades mayor a cero
- Estados controlados por catálogo
- Fechas no futuras para transacciones
- RUC válido según SUNAT

### Respaldos y Mantenimiento
- Respaldo diario de tablas transaccionales
- Archivado mensual de datos antiguos
- Reindexación semanal
- Verificación de integridad referencial

---

## CONCLUSIÓN

Este diccionario de datos documenta la estructura completa del sistema INFOREST, diseñado para la gestión integral de ventas en restaurantes con múltiples canales. La arquitectura permite trazabilidad completa desde el pedido hasta el pago, cumpliendo con regulaciones fiscales peruanas y soportando operaciones multi-canal escalables. 