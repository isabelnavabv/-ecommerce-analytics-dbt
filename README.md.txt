# 📊 E-commerce Analytics - DBT Data Warehouse

Proyecto end-to-end de Analytics Engineering que implementa un data warehouse dimensional para análisis de ventas de e-commerce.

## 🎯 Objetivo del Proyecto

Crear un data warehouse con arquitectura Star Schema que permita análisis de:
- Comportamiento de clientes y segmentación
- Performance de productos y categorías
- Tendencias de ventas por tiempo
- Métricas de negocio clave (ticket promedio, conversión, etc.)

## 🏗️ Arquitectura
### Flujo de Datos
┌─────────────────────────────────────────────────────────┐
│                     SEEDS (Raw Data)                     │
├─────────────────────────────────────────────────────────┤
│  raw_clientes  │  raw_productos  │  raw_pedidos         │
└────────┬────────────────┬────────────────┬──────────────┘
│                │                │
▼                ▼                │
┌─────────────────────────────────────────▼───────────────┐
│                  STAGING LAYER (1:1)                     │
├──────────────────────────────────────────────────────────┤
│  stg_clientes  │  stg_productos  │  stg_pedidos         │
│                │                 │  (JOIN productos)     │
└────────┬────────────────┬────────────────┬──────────────┘
│                │                │
│                │                │
├────────────────┼────────────────┤
│                │                │
▼                ▼                ▼
┌─────────────────────────────────────────────────────────┐
│              MARTS LAYER (Star Schema)                   │
├──────────────────────────────────────────────────────────┤
│                                                          │
│   dim_clientes          dim_productos                    │
│   (+ métricas)          (+ popularidad)                  │
│         │                      │                         │
│         └──────────┬───────────┘                         │
│                    │                                     │
│                    ▼                                     │
│              fact_ventas                                 │
│         (granularidad: línea)                            │
│                                                          │
└──────────────────────────────────────────────────────────┘
│
▼
BI Tools / Analytics
### Descripción de capas:

**Seeds (Raw Data)**
- `raw_clientes` - Datos crudos de clientes
- `raw_productos` - Catálogo de productos  
- `raw_pedidos` - Transacciones de pedidos

**Staging Layer**
- `stg_clientes` - Clientes limpios y estandarizados
- `stg_productos` - Productos con clasificación de stock
- `stg_pedidos` - Pedidos con montos calculados y dimensiones de tiempo

**Marts Layer (Star Schema)**
- `dim_clientes` - Dimensión de clientes con segmentación (VIP/Regular/Nuevo)
- `dim_productos` - Dimensión de productos con métricas de popularidad
- `fact_ventas` - Fact table con granularidad de línea de pedido