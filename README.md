# P1 · ¿Qué tiempo hizo?

Proyecto de análisis de datos meteorológicos diarios: descarga, inspección, limpieza, análisis y visualización con pandas, medición de eficiencia y carga en PostgreSQL. Todo el trabajo está en el notebook `notebooks/Task1.ipynb`.

## Estación

| Dato | Valor |
|---|---|
| Nombre | A Coruña / Alvedro (aeropuerto) |
| Indicativo | 08002 |
| Latitud | 43,3000 |
| Longitud | -8,3833 |
| Altitud | 97 m |
| Periodo | 01/01/2021 – 31/12/2025 (datos diarios) |

Variables: temperatura media, mínima y máxima (°C), humedad relativa (%), precipitación (mm), viento medio (km/h) y presión (hPa).

## Estructura

```
├── README.md
├── requirements.txt
├── .gitignore
├── .env.example         # plantilla de credenciales (el .env real no se sube)
├── docker-compose.yml   # PostgreSQL para el apartado 4.7
├── datos/
│   ├── raw/             # fichero original, tal como se descarga (latin-1)
│   └── clean/           # datos limpios (UTF-8)
├── notebooks/
│   └── Task1.ipynb      # todo el trabajo
└── output/              # gráficas generadas
```

## Fuente de los datos

- **Fuente original:** [Meteostat](https://meteostat.net/). El fichero de la práctica simula la exportación de un portal de datos y ha sido **modificado con fines didácticos** para incluir problemas de calidad.
- **URL de descarga:** <https://raw.githubusercontent.com/bigdatawirtz/datasets/main/datasets/meteo_alvedro_2021_2025.csv>
- **Condiciones de uso:** los datos de Meteostat se publican bajo licencia [Creative Commons Attribution-NonCommercial 4.0 (CC BY-NC 4.0)](https://creativecommons.org/licenses/by-nc/4.0/): se pueden usar y modificar citando la fuente y sin fines comerciales. Ver <https://dev.meteostat.net/terms.html>.
- **Fecha de descarga:** _AAAA-MM-DD_ (el notebook la imprime en el paso 1).

## Cómo ejecutar el proyecto

1. Clonar el repositorio y entrar en la carpeta.
2. Crear y activar un entorno virtual (opcional pero recomendable):
   ```bash
   python -m venv .venv
   # Windows
   .venv\Scripts\activate
   # Linux / macOS
   source .venv/bin/activate
   ```
3. Instalar las dependencias:
   ```bash
   pip install -r requirements.txt
   ```
4. Arrancar Docker Desktop y levantar PostgreSQL (necesario solo para el paso 10 del notebook; también se puede hacer desde la propia celda del notebook):
   ```bash
   docker compose up -d
   ```
5. Abrir el notebook y ejecutarlo entero (*Kernel → Restart & Run All*):
   ```bash
   jupyter notebook notebooks/Task1.ipynb
   ```
   El notebook funciona con la carpeta `datos/` vacía: crea las carpetas, descarga el fichero en `datos/raw/`, guarda el resultado limpio en `datos/clean/` y las gráficas en `output/`.
6. Al terminar, parar PostgreSQL:
   ```bash
   docker compose down
   ```

La tabla limpia se carga en PostgreSQL desde el notebook con `to_sql` (tabla `meteo`).

**Credenciales de PostgreSQL:** están en el fichero `.env` de la raíz (`POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB`, `POSTGRES_HOST`, `POSTGRES_PORT`). Lo leen `docker-compose.yml` y el notebook (con `python-dotenv`). El `.env` está en `.gitignore` y no se sube al repositorio. En un equipo nuevo hay que crearlo a partir de la plantilla `.env.example`, que sí está en el repositorio:

```bash
cp .env.example .env   # y editar POSTGRES_PASSWORD
```

Si se cambia la contraseña después de haber levantado el contenedor, hay que borrar el volumen (`docker compose down -v`), porque PostgreSQL solo aplica las credenciales al crear la base de datos.

## Características del fichero original

| Aspecto | Valor |
|---|---|
| Codificación | latin-1 (ISO-8859-1), saltos de línea CRLF |
| Líneas que no son datos | 5 de metadatos + 1 en blanco antes de la cabecera |
| Separador de campos | `;` |
| Separador decimal | `,` |
| Separador de miles | `.` (en la presión: `1.015,7`) |
| Códigos de valor ausente | campo vacío, `NA`, `-9999` |
| Código especial | `Ip` en precipitación = inapreciable (< 0,1 mm) |

## Decisiones de limpieza

| # | Problema | Cómo se detectó | Qué se hizo |
|---|---|---|---|
| 1 | Codificación latin-1, no UTF-8 | `decode('utf-8')` falla; las letras acentuadas ocupan un solo byte (`\xf3` = ó) | Leer con `encoding='latin-1'`; guardar el limpio en UTF-8 |
| 2 | 6 líneas antes de la cabecera (metadatos y línea en blanco) | Viendo las primeras líneas del fichero | `skiprows=6` |
| 3 | Separador `;` y coma decimal | Primeras líneas del fichero | `sep=';'` y `decimal=','` |
| 4 | Punto como separador de miles en la presión | La presión se lee como texto (`1.015,7`) | `thousands='.'` |
| 5 | Código `Ip` en la precipitación (121 días) | La precipitación se lee como texto | `Ip` → 0 mm (llovió algo, pero no se pudo medir) |
| 6 | Valores ausentes como campo vacío y `NA` | `isna()` | pandas los lee como `NaN` |
| 7 | Valores ausentes como `-9999` (14 valores) | `describe()` muestra un mínimo de -9999 | `na_values=['NA', '-9999']` |
| 8 | Fechas en dos formatos: `DD/MM/AAAA` en el segundo semestre de 2023 (164 filas) | Algunas fechas no se convierten con el formato `AAAA-MM-DD` | Convertir cada formato por separado |
| 9 | Filas desordenadas al final del fichero | Últimas líneas del fichero; `is_monotonic_increasing` es `False` | Ordenar por fecha |
| 10 | 13 filas duplicadas | `duplicated()` | `drop_duplicates()` |
| 11 | 27 días que faltan (21 seguidos del 03/07/2023 al 23/07/2023) | Comparación con un calendario completo | Se añaden como filas vacías; no se inventan valores |
| 12 | Temperaturas que no encajan (por ejemplo, máxima de 39,2 °C el 19/03/2024 con media de 10,6 °C) | Gráfica antes de limpiar; la media se aleja más de 6 °C de `(tmin + tmax) / 2` | Se anulan las tres temperaturas de esos días |
| 13 | Valores fuera de rango (humedad fuera de 0–100, lluvia negativa, mínima mayor que máxima) | Comprobación sencilla de cada condición | No aparece ninguno |

### Valores ausentes

- **No se rellenan:** son pocos (alrededor del 1 % por variable) y rellenarlos sería inventar datos.
- **No se borran las filas**, para no perder las demás variables de ese día.
- Se dejan como `NaN` / `NULL`: pandas y SQL los ignoran en medias y sumas, así que los resultados coinciden.
- Julio de 2023 está casi vacío, así que su total de lluvia sale más bajo de lo real.

## Criterios del análisis

- **Día de lluvia:** precipitación ≥ 1,0 mm.
- **Día más cálido / más frío:** según la temperatura máxima / mínima.
- **Pregunta nueva en SQL:** número de días de calor (máxima ≥ 25 °C) por año.

## Uso de la IA

| Herramienta | Para qué |
|---|---|
| Claude Code (Anthropic) | Revisar el fichero en bruto en busca de problemas de calidad, proponer la estructura del notebook, el código de limpieza, análisis, gráficas, medición con `%timeit` y carga en PostgreSQL, y redactar este README. |

Las interpretaciones de los resultados se comprueban con la salida del notebook.
