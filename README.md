<<<<<<< HEAD
# 📊 Data Warehouse de Ventas de Celulares

Proyecto académico de implementación completa de un **Data Warehouse** con esquema estrella, ETL, SCD Tipo 2, y análisis multidimensional usando SQL Server.

---

## 🎯 Descripción

Sistema de Business Intelligence que transforma datos transaccionales (OLTP) de ventas de celulares en un modelo dimensional optimizado para análisis. Incluye:

- **OLTP normalizado** → sistema fuente con datos de ventas
- **DW con esquema estrella** → 7 dimensiones + 1 tabla de hechos
- **ETL inicial e incremental** → con SCD Tipo 2 en vendedores
- **Análisis avanzado** → consultas YoY, MoM, ABC/Pareto, RFM
- **Notebook Python** → visualizaciones estadísticas

---

## 🏗️ Arquitectura

```
OLTP_Celulares (normalizado)
     ↓ ETL
DW_Celulares (esquema estrella)
     ↓ Consultas
Análisis BI + Notebook Python
```

### Modelo Dimensional

**Dimensiones:**
- `DimFecha` - Calendario completo 2020-2030
- `DimCliente` - Información de clientes
- `DimProducto` - Marcas y modelos de celulares
- `DimLocal` - Locales de venta (provincia, ciudad)
- `DimVendedor` - **SCD Tipo 2** con categorización mensual (Top/Medio/Bajo)
- `DimFormaPago` - Métodos de pago
- `DimCanal` - Junk dimension (Online/Presencial)
- `DimMoneda` - Monedas soportadas (ARS, USD, EUR, BRL, CNY con símbolo ¥)
- `DimExchangeRate` - Tipos de cambio mensuales

**Hechos:**
- `FactVentas` - Detalle de ventas con métricas: `cantidad`, `importe`, `margen`, `margen_porcentaje`, `tipo_cambio`

---

## 📂 Estructura del Proyecto

```
proyecto_dw_celulares/
│
├── 01_base_datos/
│   ├── 00_creacion_bases.sql      # Crea OLTP_Celulares y DW_Celulares
│   └── 00_reset_databases.sql     # Elimina ambas bases (reset completo)
│
├── 02_oltp/
│   ├── 01_ddl_oltp.sql            # Estructura OLTP normalizada
│   └── 02_carga_oltp.sql          # Datos de prueba (ventas, productos, clientes)
│
├── 03_datawarehouse/
│   ├── 00_reset_dw.sql            # Elimina SOLO el DW (mantiene OLTP)
│   └── 03_ddl_dw.sql              # Estructura DW (dimensiones + hechos)
│
├── 04_etl/
│   ├── 04_etl_dw_inicial.sql      # Carga completa: dimensiones + hechos
│   ├── 05_reproceso_diario.sql    # ETL incremental + categorización vendedores
│   └── 07_dataset_aplanado.sql    # Vista desnormalizada para análisis
│
├── 05_consultas/
│   ├── 01-06_*.sql                # Consultas básicas con soporte multi-moneda
│   ├── 07_modelo_mas_vendido.sql  # Modelo más vendido
│   ├── 08_analisis_temporal.sql   # YoY, MoM, promedios móviles
│   ├── 09_analisis_abc_pareto.sql # Segmentación 80/20
│   ├── 10_analisis_rfm.sql        # Segmentación de clientes
│   ├── consultas_dw.sql           # Consultas auxiliares y verificaciones
│   └── README_CONSULTAS.md        # Documentación de consultas multi-moneda
│
├── 06_analisis/
│   └── Notebook_Estadistica_Ventas.ipynb  # Visualizaciones con Python/Pandas
│
├── 07_validacion/
│   ├── 06_validacion_calidad.sql  # QA de integridad referencial
│   ├── VALIDACION_COMPLETA.sql    # Validación integral del DW
│   └── VALIDACION_INTEGRAL.sql    # Tests exhaustivos de calidad
│
├── 08_scripts_auxiliares/
│   ├── ALTAS_SIMPLES.sql          # Crear venta completa (testing ETL)
│   ├── BAJAS_SIMPLES.sql          # Eliminar ventas (testing sincronización)
│   ├── BAJA_PRODUCTO.sql          # Eliminar productos sin ventas
│   ├── SOLO_PRODUCTOS.sql         # Agregar solo productos (catálogo)
│   ├── ultimas_vtas.sql           # Ver últimas 10 ventas (debugging)
│   ├── PROBAR_TODO.bat            # Automatización completa del proyecto
│   └── README_SCRIPTS_AUXILIARES.md # Guía de uso concisa
│
└── 09_documentacion/
    ├── CAMBIOS_SEGUNDO_PARCIAL.md # Log de cambios y mejoras
    ├── CHECKLIST_VALIDACION.md   # Lista de verificación del proyecto
    ├── CREAR_DIAGRAMA_DER.md     # Guía para crear diagramas en SSMS
    ├── GUIA_REORGANIZACION.md    # Guía de reorganización del proyecto
    ├── INFORME_VALIDACION_FINAL.md # Informe final de validación
    ├── OLTP_Normalizado.xlsx     # Diagrama del modelo OLTP
    └── Presentacion_*.pptx       # Presentación del proyecto
```

---

## 🚀 Cómo Ejecutar el Proyecto

### Setup Inicial (Primera Vez)

Ejecutá los scripts **en este orden exacto** desde SQL Server Management Studio (SSMS):

```sql
-- 1. Crear las bases de datos vacías
01_base_datos/00_creacion_bases.sql

-- 2. Crear estructura del OLTP
02_oltp/01_ddl_oltp.sql

-- 3. Cargar datos de prueba en OLTP
02_oltp/02_carga_oltp.sql

-- 4. Crear estructura del DW
03_datawarehouse/03_ddl_dw.sql

-- 5. ETL: Cargar DW desde OLTP
04_etl/04_etl_dw_inicial.sql
```

✅ **Listo!** Ahora tenés ambas bases cargadas y podés ejecutar consultas analíticas.

---

### Reset del Proyecto

**Opción 1: Reset completo (OLTP + DW)**
```sql
01_base_datos/00_reset_databases.sql
-- Luego volvé a ejecutar pasos 1-5 del setup inicial
```

**Opción 2: Reset solo DW (mantiene OLTP intacto)**
```sql
03_datawarehouse/00_reset_dw.sql  -- Elimina solo DW_Celulares
03_datawarehouse/03_ddl_dw.sql    -- Re-crea estructura DW
04_etl/04_etl_dw_inicial.sql      -- Re-carga datos desde OLTP
```

💡 **Usa la opción 2 cuando:** necesites re-cargar el DW sin tocar el OLTP (más rápido).

---

### Actualización Incremental

Para simular procesamiento diario (nuevas ventas → actualizar DW):

```sql
-- 1. Agregar nuevas ventas en OLTP (usa scripts de ejemplo)
08_scripts_auxiliares/ALTAS_SIMPLES.sql

-- 2. Ejecutar ETL incremental
04_etl/05_reproceso_diario.sql
```

Este script:
- Detecta nuevas ventas en OLTP
- Inserta en FactVentas
- Actualiza SCD Tipo 2 de DimVendedor (categorías por desempeño)

---

## 📊 Características Implementadas

### ✅ Requerimientos Cumplidos

| Requisito | Implementación | Ubicación |
|-----------|----------------|-----------|
| **Dimensión Tiempo completa** | Pre-cargada 2020-2030 con atributos (día semana, trimestre, etc.) | `DimFecha` en `03_ddl_dw.sql` |
| **SCD Tipo 2** | Versionado histórico de vendedores con categorías mensuales | `DimVendedor` en `04_etl_dw_inicial.sql` |
| **Dimensión Junk** | Atributo `canal` (Online/Presencial) | `DimCanal` |
| **Registros Unknown** | SK=-1 en todas dimensiones para integridad referencial | Bloque "Unknown" en ETL |
| **Métricas calculadas** | `margen`, `margen_porcentaje` derivadas de precio y costo | `FactVentas` |
| **Consultas avanzadas** | YoY, MoM, ABC/Pareto, RFM con Window Functions | `05_consultas/08-10_*.sql` |
| **Multi-moneda** | ARS, USD, EUR, BRL, CNY con conversión mensual automática | `DimMoneda` + `DimExchangeRate` |
| **Validación cruzada** | SQL Server vs Python/Pandas con coincidencia 100% | Notebook 5M.1-5M.5 |

### 🔑 Conceptos Clave Implementados

- **Surrogate Keys**: Claves artificiales (`sk_*`) para independencia del OLTP
- **SCD Tipo 1**: Sobrescribe datos (usado en DimCliente, DimLocal, DimProducto)
- **SCD Tipo 2**: Versionado histórico con `fecha_inicio`, `fecha_fin`, `es_actual`, `version`
  - Caso real: vendedores categorizados por desempeño mensual (Top/Medio/Bajo/SinVentas)
- **Unknown Pattern**: FK=-1 para late-arriving dimensions
- **Junk Dimension**: Atributos de baja cardinalidad (canal)
- **Esquema Estrella**: 1 tabla de hechos rodeada de dimensiones

---

## 📈 Análisis Disponibles

### Consultas Básicas Multi-moneda (`05_consultas/01-06_*.sql`)

**Todas las consultas incluyen conversiones automáticas a USD, EUR, BRL, CNY:**

- **01** Marca más vendida (unidades + facturación en 5 monedas)
- **02** Vendedor con más ventas (nombre completo + multi-moneda)
- **03** Local con mayor ganancia (márgenes en todas las monedas)
- **04** Método de pago más usado (transacciones + importes)
- **05** Trimestre con menores ventas (comparación multi-moneda)
- **06** Trimestre con mayores ventas (análisis estacional)
- **07** Modelo más vendido (ranking por unidades)

### Análisis Avanzados

**Temporal (`08_analisis_temporal.sql`):**
- Year-over-Year (YoY)
- Month-over-Month (MoM)
- Promedios móviles (3 meses)
- Running totals
- Estacionalidad por día de semana

**ABC/Pareto (`09_analisis_abc_pareto.sql`):**
- Clasificación 80/20 de productos
- Segmentación de clientes (VIP, Regular, Ocasional)
- Ranking de vendedores

**RFM (`10_analisis_rfm.sql`):**
- Segmentación de clientes en 10 categorías
- Champions, Loyal, At Risk, Lost, etc.
- Recomendaciones de acción por segmento

---

## 📓 Análisis con Python

El notebook `06_analisis/Notebook_Estadistica_Ventas.ipynb` incluye:

**🔧 Funcionalidades:**
- Conexión directa a `DW_Celulares` con SQLAlchemy + reconnection automática
- Construcción del modelo estrella en Pandas con helper multi-moneda
- Gráficos de evolución temporal por categoría vendedores (Top/Medio/Bajo)
- Distribución de quintiles (histograma + KDE + análisis estadístico)
- **5 consultas multi-moneda SQL vs Pandas** con validación automática
- Símbolos Unicode correctos (¥, €, R$) en todas las visualizaciones
- Paleta de colores Splatoon para categorías de vendedores
- **Tabla comparativa final** con ✅/❌ por moneda y consulta

**📊 Validaciones implementadas:**
- Marca más facturación (5M.1)
- Vendedor más ventas (5M.2)  
- Local más margen (5M.3)
- Forma pago más usada (5M.4)
- Análisis trimestral (5M.5)

**Requisitos:**
```bash
pip install pandas numpy sqlalchemy pyodbc matplotlib seaborn scipy
```

**Ejecución:**
1. Abrí el notebook en VS Code o Jupyter
2. Ejecutá las celdas en orden (filtro de warnings incluido)
3. La tabla final muestra coincidencias 100% entre SQL Server y Pandas

---

## 🔍 Validación de Calidad

Script: `07_validacion/06_validacion_calidad.sql`

Verifica:
- ✅ Integridad referencial (sin FKs huérfanas excepto Unknown)
- ✅ Claves primarias únicas
- ✅ Business keys duplicados en SCD2
- ✅ Métricas consistentes (margen = cantidad × (precio - costo))
- ✅ Fechas SCD2 coherentes (fecha_fin > fecha_inicio)
- ✅ Valores nulos en columnas críticas

---

## 🐛 Problemas Comunes

### "No veo el símbolo ¥ en SSMS"

**Causa:** La fuente de SSMS no soporta Unicode extendido.

**Solución:**
1. En SSMS: `Tools > Options > Environment > Fonts and Colors`
2. Cambiar fuente a **Consolas** o **Courier New**
3. Reiniciar SSMS

**Verificación:**
```sql
-- El dato está correcto si el hex es 0x00A5
SELECT codigo_moneda, simbolo, CONVERT(VARBINARY(20), simbolo) AS hex
FROM DimMoneda WHERE codigo_moneda = 'CNY';
```

✅ En el **notebook Python siempre se ve correctamente** porque usa UTF-8 nativo.

---

### "Error: Database already exists"

```sql
-- Usar reset según necesidad:
01_base_datos/00_reset_databases.sql       -- Reset completo
-- O
03_datawarehouse/00_reset_dw.sql           -- Reset solo DW (NUEVO)
```

### "Vendedores aparecen como 'Inicial'"

**Causa:** El ETL inicial asigna categoria='Inicial'. La categorización real se hace en reproceso diario.

**Solución:**
```sql
-- Ejecutar categorización manual:
04_etl/05_reproceso_diario.sql
-- O desde notebook: celda de categorización automática
```

### "No coinciden resultados SQL vs Pandas"

**Causa:** Orden de agregación diferente en conversiones multi-moneda.

**Solución:** El notebook incluye lógica corregida que agrega ARS primero y luego convierte (igual que SQL).

---

### "Foreign key constraint violation"

Verificá el **orden de ejecución**. Las dimensiones deben cargarse antes que FactVentas.

---

## 🆕 Novedades de la Segunda Entrega

### ✨ Mejoras Implementadas (v2.1)

**🌍 Soporte Multi-moneda Completo:**
- Conversiones automáticas ARS → USD, EUR, BRL, CNY
- Tasas de cambio mensuales en `DimExchangeRate`
- Patrón CTE + LEFT JOIN por moneda (evita multiplicación de filas)
- Símbolo ¥ correcto para CNY (Yuan chino)

**🔄 Validación SQL vs Python:**
- 5 consultas críticas implementadas en ambos lenguajes
- Algoritmo de agregación corregido en Pandas (coincidencia 100%)
- Tabla comparativa automática con ✅/❌ por moneda
- Reconnection automática para conexiones SQL perdidas

**⚡ ETL Mejorado:**
- Categorización automática de vendedores (Top/Medio/Bajo/SinVentas)
- Script de reproceso con manejo de errores y transacciones
- Consolidación de variables y eliminación de comandos GO
- Validación integral con scripts específicos

**📊 Análisis Avanzado:**
- Gráficos temporales por categoría de vendedores
- Paleta de colores consistente (Splatoon theme)
- Manejo de warnings deprecados en Pandas
- Análisis estadístico con percentiles y distribuciones

### 📋 Documentación Actualizada:
- Guías específicas para cada componente
- Checklist de validación completo
- Troubleshooting para problemas comunes
- Log detallado de cambios implementados

---

## 📚 Tecnologías

- **Motor:** SQL Server 2016+ (compatible con Azure SQL, SQL Server 2019/2022)
- **Lenguaje:** Transact-SQL (T-SQL)
- **Análisis:** Python 3.8+ (Pandas, Matplotlib, Seaborn, Scipy)
- **Herramientas:** SSMS, Azure Data Studio, VS Code, Jupyter

---

## 🎓 Conceptos Académicos

Este proyecto demuestra:

- **Modelado dimensional** (Kimball) con esquema estrella
- **Slowly Changing Dimensions** (Tipo 1 y 2) con versionado histórico
- **ETL** (Extract, Transform, Load) inicial e incremental
- **Data Quality** (validaciones, integridad referencial, tests)
- **Window Functions** (RANK, ROW_NUMBER, LAG, SUM OVER, PERCENTILE_CONT)
- **Análisis multidimensional** (OLAP) con drill-down temporal
- **Business Intelligence** (KPIs, categorización automática, dashboards)
- **Multi-currency support** con tipos de cambio históricos
- **Cross-platform validation** (SQL Server ↔ Python/Pandas)

---

## 👨‍💻 Autor

**Proyecto:** Data Warehouse de Ventas de Celulares  
**Autor:** Ramiro Ottone Villar  
**Fecha:** Noviembre 2025  
**Versión:** 2.1 - Segunda Entrega  
**Propósito:** Proyecto académico de Modelado de Minería de Datos  
**Estado:** ✅ Funcional con soporte multi-moneda y validación completa

---

## 📝 Licencia

Este proyecto está licenciado bajo la Licencia MIT - ver el archivo [LICENSE](LICENSE) para más detalles.

**Uso académico y educativo libre.**

=======
# retail-sales-data-warehouse-sql
Data Warehouse project for retail sales analytics. Includes dimensional modeling (star schema), SQL DDL/DML scripts, ETL pipeline in Python, and Jupyter notebooks for EDA and KPI reporting.
>>>>>>> ab794cfb45f4ececdd5ebff5c6728fe24f4352f0
