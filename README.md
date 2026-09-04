# proyecto-percepcion-ciudadana-victimas-quibdo

**Equipo de trabajo:**
Roberto

Actividad · Módulo Gerencia de Proyectos y Analítica · Docente: Luis Felipe Ortiz-Clavijo · UNAULA · 2026

> ⚠️ **Datos sintéticos:** todos los registros de PQRSD de este repositorio fueron generados
> por un algoritmo con fines académicos. No corresponden a personas ni a registros reales de
> la Oficina de Enlace de Víctimas. El tratamiento de datos reales exige cumplimiento de la
> **Ley 1581 de 2012** (Habeas Data).

---

# 1. CASO DEL NEGOCIO

**Descripción del problema:** La Oficina de Enlace de Víctimas de la Alcaldía de Quibdó atiende
a la población víctima del conflicto armado del municipio y recibe un alto volumen de PQRSD
(peticiones, quejas, reclamos, sugerencias y denuncias). Hoy, la lectura de esas solicitudes es
**manual y fragmentada**: no existe una clasificación sistemática por tema ni una medición del
sentimiento ciudadano, de modo que las decisiones se toman con información dispersa y tardía.
Cuando se identifica un problema recurrente, muchas veces la insatisfacción ya escaló.

Este conjunto de datos registra **5.209 PQRSD entre 2025 y 2026**, distribuidas en **9 zonas**
(6 comunas y 3 corregimientos) y **8 categorías temáticas** de la ruta de atención a víctimas
(Ley 1448 de 2011).

## 1.1 Objetivo general

Diseñar e implementar una solución de analítica en Databricks que clasifique automáticamente
las PQRSD por tema, mida el sentimiento ciudadano y priorice la atención de la Oficina de
Enlace de Víctimas de Quibdó.

## 1.2 Objetivos específicos

- Construir un pipeline automatizado (**Bronze → Silver → Gold**) que estandarice y consolide
  las PQRSD en un único origen de verdad confiable.
- Clasificar cada combinación zona–tema por **nivel de alerta** de insatisfacción
  (Moderada / Media / Alta / Extrema) para priorizar la respuesta.
- Entrenar un **modelo predictivo** que estime la probabilidad de que una zona–tema entre en
  alerta alta de insatisfacción, a partir del volumen, el tema y el tiempo de respuesta.

Esta solución permite pasar de un enfoque **reactivo** (leer las quejas una por una, después
del hecho) a uno **predictivo y trazable** en la atención a la población víctima.

---

# 2. RELACIÓN BENEFICIO/COSTE

La clasificación automática y el tablero reducen el tiempo de lectura y consolidación de las
PQRSD de **varios días a menos de 24 horas**, liberando horas del equipo que hoy se dedican a
revisar solicitudes manualmente y a armar reportes en hojas de cálculo.

El principal beneficio es **valor público**: priorizar los temas y zonas con más insatisfacción
permite reducir tiempos de respuesta, atender antes los casos urgentes y disminuir quejas
reiteradas. Como referencia de gestión, cada reducción sostenida del tiempo de atención mejora
la satisfacción medible en las encuestas y evita el desgaste institucional de atender
reclamos repetidos.

> *Nota: por tratarse de una oficina pública de atención a víctimas, el beneficio se plantea en
> términos de valor público y eficiencia operativa, no de retorno financiero. Cualquier cifra
> económica debería estimarse con la Secretaría de Hacienda antes de una implementación real.*

---

# 3. ARQUITECTURA PROPUESTA

```
CSV (PQRSD Oficina de Víctimas) → Unity Catalog Volume → Tabla Bronze → Tabla Silver
   → Modelo de Machine Learning → Tabla Gold → Lakeflow Job → Tablero de monitoreo
```

**Fuente de datos:** registros de PQRSD de la población víctima (2025–2026): folio, período,
zona, tipo de zona, categoría temática, sentimiento, días de respuesta y texto de la solicitud.

**Almacenamiento (Volumes):** el CSV se carga en un Volume de Unity Catalog, que actúa como
zona de aterrizaje (landing zone) gobernada.

**Bronze – Datos crudos:** se carga el archivo en una tabla Delta sin transformaciones, para
conservar la información original y permitir trazabilidad completa.

**Silver – Transformación y enriquecimiento:**
- Tipado de `Dias_respuesta` (numérico) y `Anio` (entero)
- Estandarización de texto (zona, categoría, sentimiento)
- Bandera de sentimiento negativo (`es_negativo`)
- Deduplicación por folio y validación de rangos lógicos

**Machine Learning – Scoring de alerta:** modelo **Random Forest** que estima la probabilidad
de que una combinación zona–tema entre en alerta alta o extrema de insatisfacción.

**Gold – Datos para negocio:** tabla `gold_percepcion_zonas`, con volumen de PQRSD, porcentaje
de sentimiento negativo, tiempo promedio de respuesta y nivel de alerta por zona–tema–período.
Lista para consumo analítico.

**Lakeflow Jobs:** automatización y orquestación del pipeline completo.

---

# 4. PIPELINE DE INGESTA DE DATOS

La información se procesa mediante el notebook
[`Pipeline_Percepcion_Victimas_Quibdo.ipynb`](Pipeline_Percepcion_Victimas_Quibdo.ipynb), que
ejecuta cada etapa del flujo de datos y las almacena en tablas Delta Lake, integrables con
**Lakeflow Jobs** para ejecuciones automáticas.

**Estrategia Medallion:**

| Capa   | Contenido                                                            | Tabla generada                  |
| ------ | ------------------------------------------------------------------- | ------------------------------- |
| Bronze | Carga cruda del CSV, sin transformaciones                           | `bronze_percepcion_victimas`    |
| Silver | Tipado, limpieza, estandarización, deduplicación, bandera de sentim.| `silver_percepcion_limpio`      |
| Gold   | Agregados por zona–tema–período y nivel de alerta                    | `gold_percepcion_zonas`         |

---

# 5. MODELOS DE CIENCIA DE DATOS

**Análisis descriptivo:** sobre 5.209 PQRSD (2025–2026) en 9 zonas y 8 categorías. La
indemnización/reparación y la vivienda concentran los mayores niveles de insatisfacción,
mientras que los corregimientos rurales muestran tiempos de respuesta más altos que las comunas.

**Modelado:** se entrenó un **Random Forest** sobre la tabla Gold (288 combinaciones
zona–tema–período) para estimar la probabilidad de que una combinación caiga en alerta alta,
usando como variables la categoría, el tipo de zona, el volumen de PQRSD y el tiempo de
respuesta.

El dataset se dividió 75% entrenamiento / 25% prueba. Resultado obtenido al ejecutar el
notebook: **AUC-ROC = 0.859**, **F1-score = 0.69**.

El experimento se registra en **MLflow** (parámetros, métricas y modelo serializado),
permitiendo comparar versiones y justificar la elección del modelo en lugar de presentarlo
como una caja negra.

---

# 6. APP O VISUALIZACIÓN

La tabla Gold alimenta un **tablero de monitoreo** con tarjetas KPI (total de PQRSD, %
negativo, días de respuesta, F1 del clasificador), temas predominantes, sentimiento ciudadano,
evolución temporal e insatisfacción por zona.

Este repositorio incluye además una versión web interactiva del tablero
([`dashboard_percepcion_victimas.jsx`](dashboard_percepcion_victimas.jsx)) y una guía para
reconstruirlo en **Power BI** a partir del CSV.

**Serving:** la tabla Gold se expone mediante un **Databricks SQL Warehouse** como punto único
de consumo para herramientas de BI, con acceso controlado por permisos de Unity Catalog.

---

# Cómo ejecutar este proyecto

1. Crea una cuenta gratuita en **Databricks Free Edition**:
   <https://www.databricks.com/try-databricks> (no requiere tarjeta de crédito).
2. En **Catalog Explorer**, crea un Volume y sube el archivo
   [`PQRSD_PERCEPCION_VICTIMAS_QUIBDO.csv`](PQRSD_PERCEPCION_VICTIMAS_QUIBDO.csv).
3. Importa el notebook
   [`Pipeline_Percepcion_Victimas_Quibdo.ipynb`](Pipeline_Percepcion_Victimas_Quibdo.ipynb)
   a tu workspace (Workspace → Import).
4. Ajusta las variables `catalog`, `schema` y `volume_path` en la primera celda de código.
5. Ejecuta el notebook completo (Run All).

## Contenido del repositorio

```
proyecto-percepcion-ciudadana-victimas-quibdo/
├── README.md
├── PQRSD_PERCEPCION_VICTIMAS_QUIBDO.csv          # datos sintéticos (5.209 PQRSD)
├── Pipeline_Percepcion_Victimas_Quibdo.ipynb     # pipeline Bronze/Silver/Gold + ML + MLflow
├── dashboard_percepcion_victimas.jsx             # tablero web interactivo (React)
├── Documento_Definitivo_Gerencia_Proyectos_Analitica.docx  # documento del proyecto
├── guia_powerbi.md                               # guía para reconstruir el tablero en Power BI
└── .gitignore
```

---

*Proyecto académico. Datos sintéticos. El tratamiento de datos reales de población víctima
exige cumplimiento de la Ley 1581 de 2012 y el uso de datos sensibles bajo autorización.*
