# 🏗️ Decisiones de Diseño - E-commerce Analytics

Este documento explica las decisiones técnicas tomadas durante el desarrollo del proyecto.

## 📊 Arquitectura: ¿Por qué 3 capas?

### Seeds (Raw Data)
**Decisión:** Usar seeds en lugar de conectar a fuente real.
**Razón:** Proyecto de portfolio - seeds permiten reproducibilidad sin depender de APIs externas.
**En producción:** Se reemplazarían por sources conectadas a sistemas transaccionales.

### Staging Layer
**Decisión:** Transformaciones 1:1 con tablas raw (un staging por cada raw).
**Razón:** 
- Separación clara entre limpieza y lógica de negocio
- Facilita debugging
- Permite reutilización en múltiples marts
- Estándar de industria en DBT

### Marts Layer
**Decisión:** Star Schema en lugar de normalizado.
**Razón:**
- Optimizado para análisis (menos JOINs)
- Desnormalización controlada para queries rápidas
- Fácil de entender para usuarios de negocio
- Compatible con herramientas de BI

## 🎯 Modelado de Datos

### ¿Por qué Star Schema y no Snowflake?
**Decisión:** Star Schema con dimensiones desnormalizadas.
**Razón:**
- Query performance > espacio en disco
- BigQuery cobra por datos procesados, no almacenados
- Menos JOINs = menos costos
- Proyecto pequeño donde normalización no aporta valor

### Granularidad de fact_ventas
**Decisión:** Una fila = una línea de pedido (no pedido completo).
**Razón:**
- Máxima flexibilidad para análisis
- Permite agregaciones a cualquier nivel
- Estándar de industria para fact tables

## 💡 Decisiones de Transformación

### Cálculo de monto_total en staging
**Decisión:** Calcular monto_total (cantidad × precio) en `stg_pedidos`.
**Razón:**
- Dato derivado usado en múltiples modelos downstream
- Calcularlo una vez evita inconsistencias
- Staging es el lugar correcto para cálculos básicos

### Segmentación de clientes
**Decisión:** VIP (≥€500), Regular (≥€200), Nuevo (<€200).
**Razón:**
- Umbrales basados en distribución de datos actual
- En producción se ajustarían según análisis de cohortes
- CASE statement permite fácil modificación

### Clasificación de productos
**Decisión:** Nivel de stock (Alto/Medio/Bajo) y popularidad (Best Seller/Popular/Regular).
**Razón:**
- Facilita análisis sin necesidad de recordar rangos numéricos
- Útil para dashboards y alertas
- Se recalcula automáticamente en cada run

## 🧪 Estrategia de Testing

### Tests en Staging
**Decisión:** Tests de integridad básica (unique, not_null, relationships).
**Razón:**
- Staging debe garantizar que datos limpios sean válidos
- Catch errores temprano en el pipeline
- Relationships validan integridad referencial

### Tests en Marts
**Decisión:** Tests de relaciones con dimensiones + validación de valores calculados.
**Razón:**
- Validar foreign keys del Star Schema
- Asegurar que segmentaciones sean correctas
- Prevenir datos inválidos en capa de consumo

### ¿Por qué no tests en seeds?
**Decisión:** No agregar tests a seeds.
**Razón:**
- Seeds son datos estáticos controlados manualmente
- En producción los tests irían en la fuente real (sources)

## 📈 Escalabilidad

### ¿Por qué no incremental models?
**Decisión:** Todos los modelos son full-refresh.
**Razón:**
- Dataset pequeño (15 pedidos) no justifica complejidad
- Full-refresh toma segundos
- En producción con millones de filas: staging incremental, marts full-refresh

### Futuras optimizaciones
Si el volumen de datos crece:
1. `stg_pedidos` → incremental por fecha
2. `fact_ventas` → incremental con merge
3. Particionamiento por fecha en BigQuery
4. Clustering por cliente_id y producto_id

## 🎨 Convenciones de Nomenclatura

### Prefijos
- `raw_*` - Datos crudos (seeds)
- `stg_*` - Staging models
- `dim_*` - Dimensiones
- `fact_*` - Fact tables

### Columnas
- `*_id` - Primary/Foreign keys
- `*_at` / `*_fecha` - Timestamps
- `total_*` - Agregaciones
- `es_*` - Flags booleanos

**Razón:** Consistencia facilita mantenimiento y onboarding de nuevos desarrolladores.

## 🔄 Dependencias

### Orden de ejecución