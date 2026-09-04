# Guía: reconstruir el tablero en Power BI

Los datos viven en `PQRSD_PERCEPCION_VICTIMAS_QUIBDO.csv`. Power BI se alimenta de ese archivo
y tú construyes las visualizaciones dentro de la herramienta.

## Paso 1 — Importar los datos

Power BI Desktop → **Inicio → Obtener datos → Texto/CSV** → selecciona el CSV → **Cargar**.

## Paso 2 — Crear las medidas (DAX)

**Modelado → Nueva medida**, y copia cada una:

```DAX
Total PQRSD = COUNTROWS('PQRSD_PERCEPCION_VICTIMAS_QUIBDO')

PQRSD Negativas = CALCULATE([Total PQRSD], 'PQRSD_PERCEPCION_VICTIMAS_QUIBDO'[Sentimiento] = "Negativo")

% Negativo = DIVIDE([PQRSD Negativas], [Total PQRSD], 0)

Días promedio = AVERAGE('PQRSD_PERCEPCION_VICTIMAS_QUIBDO'[Dias_respuesta])
```

Da formato a `% Negativo` como **Porcentaje**.

## Paso 3 — Armar las visualizaciones

| Visual | Campo(s) | Título |
|---|---|---|
| Tarjetas (KPI) | Total PQRSD · % Negativo · Días promedio | — |
| Barras horizontal | Eje: `Categoria` · Valor: Total PQRSD | Temas predominantes |
| Anillo | Leyenda: `Sentimiento` · Valor: Total PQRSD | Sentimiento ciudadano |
| Líneas | Eje: `Periodo` · Valores: % Negativo y Días promedio | Evolución en el tiempo |
| Barras | Eje: `Zona_corta` · Valor: % Negativo | Insatisfacción por zona |
| Tabla | Folio · Zona · Categoria · Texto_PQRSD · Dias_respuesta · Sentimiento | PQRSD individuales |

## Paso 4 — Filtros

Inserta **Segmentación de datos** con `Zona` y con `Periodo`. Así todo el tablero responde
igual que la demo web.

## Paso 5 — Colores (bandera de Quibdó)

Verde `#0F6B3F` · Azul `#1E5AA8` · Amarillo `#E9B417` · Rojo alerta `#C0392B` · Verde positivo `#2E8B57`.

Para el anillo de sentimiento: Negativo = rojo, Neutral = gris, Positivo = verde.
