# DIAGRAMA ENTIDAD-RELACIÓN - SISTEMA VENTAS INFOREST

## DIAGRAMA ASCII

```
                    SISTEMA DE VENTAS INFOREST - DIAGRAMA ER
                               
    ┌─────────────────────────────── TABLAS MAESTRAS ───────────────────────────────┐
    │                                                                               │
    │  ┌─────────────┐   ┌──────────────┐   ┌──────────────┐   ┌──────────────┐   │
    │  │    TCAJA    │   │ TCANALVENTA  │   │ TORIGENVENTA │   │ TPRODUCTO    │   │
    │  │             │   │              │   │              │   │              │   │
    │  │ tCaja (PK)  │   │ tCodigoCanalV│   │ tCodigoOrigenV│   │ tCodigoProducto│ │
    │  │ tDescripcion│   │ Venta (PK)   │   │ Venta (PK)   │   │ (PK)         │   │
    │  │ lActivo     │   │ tDetallado   │   │ tDescripcion │   │ tDetallado   │   │
    │  └─────────────┘   │ lCanalDelivery│   │ lDelivery    │   │ nPrecioVenta │   │
    │         │          └──────────────┘   └──────────────┘   └──────────────┘   │
    │         │                   │                   │                   │       │
    │         │                   └───────────────────┘                   │       │
    │         │                            1:N                            │       │
    │         │                             │                             │       │
    │  ┌─────────────┐                     │                      ┌──────────────┐│
    │  │CODIGODELIVERY│                    │                      │   TCOMBO     ││
    │  │             │                     │                      │              ││
    │  │ id (PK)     │                     │                      │ tCombo       ││
    │  │ letra       │                     │                      │ tCodigoProducto││
    │  └─────────────┘                     │                      │ nCantidad    ││
    │                                      │                      └──────────────┘│
    └──────────────────────────────────────┼─────────────────────────────────────┘
                                           │                             │
                                           │ 1:N                         │ N:1
                                           ▼                             │
    ┌─────────────────────────── GESTIÓN DE PEDIDOS ─────────────────────▼─────────┐
    │                                                                             │
    │  ┌─────────────┐                   ┌──────────────┐                        │
    │  │   APEDIDO   │                   │   MPEDIDO    │                        │
    │  │             │                   │              │                        │
    │  │ tCodigoPedido│◄──────────────────│ tCodigoPedido│                        │
    │  │ (FK)        │       1:N         │ (PK)         │                        │
    │  │ tItem       │                   │ fFecha       │                        │
    │  │ tCodigoProducto│                │ tEstadoPedido│                        │
    │  └─────────────┘                   │ tCaja        │                        │
    │         │                          │ CodigoOrigenV│                        │
    │         │ N:1                      └──────────────┘                        │
    │         │                                   │                              │
    │         │                                   │ 1:N                          │
    │         │                          ┌────────┼────────┐                     │
    │         │                          │                 │                     │
    │         ▼                          ▼                 ▼                     │
    │  ┌──────────────┐         ┌──────────────┐  ┌──────────────┐              │
    │  │ TPRODUCTO    │◄────────│   DPEDIDO    │  │   CPEDIDO    │              │
    │  │              │  N:1    │              │  │              │              │
    │  │ tCodigoProducto│        │ tCodigoPedido│  │ tCodigoPedido│              │
    │  │ (PK)         │         │ (FK)         │  │ (FK)         │              │
    │  │ tDetallado   │         │ tItem        │  │ tItem        │              │
    │  │ nPrecioVenta │         │ tCodigoProducto│ │ tProductoCombo│             │
    │  └──────────────┘         │ nCantidad    │  │ nCantidad    │              │
    │         ▲                 │ tEstadoItem  │  └──────────────┘              │
    │         │                 └──────────────┘                                │
    │         │ N:1                      │                                      │
    │         │                          │                                      │
    │  ┌──────────────┐                  │                                      │
    │  │   TPEDIDO    │                  │                                      │
    │  │              │                  │                                      │
    │  │ nCorrelativo │                  │                                      │
    │  │ tPedidoIni   │                  │                                      │
    │  │ tProducto(FK)│                  │                                      │
    │  └──────────────┘                  │                                      │
    └────────────────────────────────────┼─────────────────────────────────────┘
                                         │
                                         │ N:1 (via tCodigoPedido)
                                         ▼
    ┌─────────────────────── FACTURACIÓN Y DOCUMENTOS ───────────────────────────┐
    │                                                                             │
    │  ┌──────────────┐                   ┌──────────────┐                       │
    │  │ MDOCUMENTO   │                   │ MNOTACREDITO │                       │
    │  │              │                   │              │                       │
    │  │ tDocumento   │◄──────────────────│ tNotaCredito │                       │
    │  │ (PK)         │       N:1         │ (PK)         │                       │
    │  │ tTipoDocumento│                  │ tDocumento   │                       │
    │  │ nVenta       │                   │ (FK)         │                       │
    │  │ tEstadoDocumento│                │ nVenta       │                       │
    │  │ lEstadoFacturacion│              └──────────────┘                       │
    │  └──────────────┘                           │                              │
    │          │                                  │ 1:N                          │
    │          │ 1:N                              ▼                              │
    │          ▼                          ┌──────────────┐                       │
    │  ┌──────────────┐                   │ DNOTACREDITO │                       │
    │  │ DDOCUMENTO   │                   │              │                       │
    │  │              │                   │ tNotaCredito │                       │
    │  │ tDocumento   │                   │ (FK)         │                       │
    │  │ (FK)         │                   │ tItem        │                       │
    │  │ tItem        │                   │ tCodigoProducto│                     │
    │  │ tCodigoPedido│ ──────────────────│ nCantidad    │                       │
    │  │ tCodigoProducto│                 └──────────────┘                       │
    │  │ nCantidad    │                                                          │
    │  └──────────────┘                                                          │
    │          │                                                                 │
    └──────────┼─────────────────────────────────────────────────────────────────┘
               │
               │ 1:N
               ▼
    ┌─────────────────────────── GESTIÓN DE PAGOS ───────────────────────────────┐
    │                                                                             │
    │  ┌──────────────┐                   ┌──────────────┐                       │
    │  │DPAGODOCUMENTO│                   │  DPREPAGO    │                       │
    │  │              │                   │              │                       │
    │  │ tDocumento   │                   │ tDocumento   │                       │
    │  │ (FK)         │                   │ tTipoPago    │                       │
    │  │ tTipoPago    │                   │ nMonto       │                       │
    │  │ nMonto       │                   │ tTarjeta     │                       │
    │  │ tTarjeta     │                   │ tCodigoPedido│                       │
    │  │ tReferencia  │                   └──────────────┘                       │
    │  └──────────────┘                                                          │
    │                                                                             │
    └─────────────────────────────────────────────────────────────────────────────┘
```

## LEYENDA DE RELACIONES

- **1:N** = Uno a Muchos (Un registro padre puede tener múltiples hijos)
- **N:1** = Muchos a Uno (Múltiples registros apuntan a uno)
- **PK** = Clave Primaria (Primary Key)
- **FK** = Clave Foránea (Foreign Key)
- **◄──** = Relación de dependencia

## FLUJOS PRINCIPALES

### 1. FLUJO DE PEDIDO
```
TCANALVENTA → TORIGENVENTA → MPEDIDO → DPEDIDO → TPRODUCTO
                                   ↘
                                    CPEDIDO (para combos)
                                   ↘
                                    APEDIDO (adicionales)
```

### 2. FLUJO DE FACTURACIÓN
```
MPEDIDO → DDOCUMENTO → MDOCUMENTO → DPAGODOCUMENTO
              ↓
         DNOTACREDITO ← MNOTACREDITO (anulaciones)
```

### 3. FLUJO DE COMBOS
```
TPRODUCTO → TCOMBO → CPEDIDO ← DPEDIDO ← MPEDIDO
```

## TIPOS DE TABLAS POR COLOR/FUNCIÓN

- **🔵 MAESTRAS**: Configuración y catálogos base
- **🟡 PEDIDOS**: Gestión del proceso de pedidos
- **🟢 DOCUMENTOS**: Facturación y comprobantes
- **🔴 PAGOS**: Gestión de cobranzas y pagos

## CARDINALIDADES IMPORTANTES

| Relación | Tipo | Descripción |
|----------|------|-------------|
| MPEDIDO → DPEDIDO | 1:N | Un pedido tiene múltiples líneas |
| DPEDIDO → TPRODUCTO | N:1 | Cada línea referencia un producto |
| MDOCUMENTO → DDOCUMENTO | 1:N | Un documento tiene múltiples líneas |
| TCANALVENTA → TORIGENVENTA | 1:N | Un canal tiene múltiples orígenes |
| MPEDIDO → CPEDIDO | 1:N | Un pedido puede tener combos |
| TPRODUCTO → TCOMBO | 1:N | Un producto puede ser parte de varios combos | 