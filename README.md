# Predicción de tiempos de resolución e incumplimiento de ANS en mesas de servicio TI

**Proyecto académico de Ciencia de Datos y Big Data · GLPI · 2024–2026**

Este repositorio documenta el análisis de casos históricos registrados en GLPI para estudiar los tiempos de resolución y construir modelos iniciales que permitan estimar la duración de un caso y el riesgo de incumplimiento de los Acuerdos de Nivel de Servicio (**ANS**).

> **Estado del proyecto:** notebook de análisis y modelado inicial disponible. La ejecución y validación del componente PySpark/Parquet, la optimización de modelos y el tablero de visualización permanecen pendientes. La publicación del código no equivale a una ejecución reproducida en GitHub.

## 1. Problema y objetivos

Las mesas de servicio TI deben atender casos dentro de plazos definidos. Analizar su historial puede ayudar a reconocer patrones de demanda, comprender la duración de las soluciones e identificar factores asociados con incumplimientos de ANS.

**Objetivo general:** analizar datos históricos de casos GLPI y desarrollar modelos predictivos iniciales de tiempo de resolución e incumplimiento de ANS, incorporando una propuesta de procesamiento Big Data.

**Objetivos específicos:**

- Integrar y depurar exportaciones históricas de casos.
- Explorar distribuciones de casos, tiempos de resolución e incumplimiento por categorías y periodos.
- Comparar modelos de regresión y clasificación contra líneas base.
- Proponer y verificar un flujo de procesamiento con PySpark y almacenamiento Parquet.
- Documentar resultados, limitaciones y siguientes pasos sin divulgar información institucional.

## 2. Datos y alcance

El análisis se basa en tres exportaciones CSV de GLPI identificadas como **2024, 2025 y 2026**. En el análisis inicial se contabilizaron **200.375 casos**: 58.647, 83.709 y 58.019 registros por archivo, respectivamente. Estas cifras corresponden a las **exportaciones**, no necesariamente al año real de apertura de cada caso.

Los campos considerados son:

| Campo | Uso |
| --- | --- |
| `ID` | Identificador del caso y control de duplicados |
| `Título` | Descripción textual; no se publica por posibles datos sensibles |
| `Departamento` | Agrupación territorial o administrativa |
| `Estado` | Estado registrado en la exportación |
| `Fecha de Apertura` | Fecha y hora de creación del caso |
| `Tipo` | Clasificación del caso |
| `Fecha de solución` | Fecha y hora de resolución |
| `Tiempo de solución` | **Fecha y hora límite** para resolver el caso según el dato exportado; **no es una duración** |

La **duración observada** se calcula como `Fecha de solución - Fecha de Apertura`, en horas calendario. El **incumplimiento de ANS** se define provisionalmente como `Fecha de solución > Tiempo de solución`, cuando ambas fechas existen. Esta comparación debe contrastarse con las reglas institucionales, posibles excepciones y calendarios de atención.

**Restricciones de interpretación:** las exportaciones analizadas contienen casos cerrados; los resultados descriptivos no se extrapolan automáticamente a casos abiertos. Algunas fechas reales de apertura son anteriores a 2024.

## 3. Metodología

Se utiliza **CRISP-DM** como guía de trabajo:

1. **Comprensión del negocio:** definir la necesidad de anticipar tiempos e incumplimientos de ANS.
2. **Comprensión de los datos:** revisar campos, tipos, fechas, distribución y calidad de registros.
3. **Preparación:** integrar CSV, convertir fechas, detectar duplicados y calcular variables analíticas.
4. **Modelado:** entrenar modelos de regresión y clasificación con variables disponibles al abrir un caso.
5. **Evaluación:** contrastar métricas contra modelos base y examinar limitaciones.
6. **Despliegue:** previsto como visualización y documentación de resultados; no se presenta como finalizado.

### Prevención de fuga de información

Las variables predictoras iniciales son `Departamento`, `Tipo`, hora, día de semana y mes de apertura. No se utilizan como predictores la fecha real de solución ni el estado final. El plazo ANS tampoco se incorpora al modelo mientras no se verifique que estaba disponible desde la apertura del caso.

## 4. Análisis exploratorio y resultados iniciales

El trabajo previo identificó, para el conjunto histórico estudiado:

| Indicador | Resultado inicial |
| --- | ---: |
| Casos integrados | 200.375 |
| Requerimientos | 188.855 (94,25 %) |
| Incidencias | 11.520 (5,75 %) |
| Incumplimientos ANS observados | 40.449 (aprox. 20,2 %) |
| Mediana del tiempo de resolución | 33,5 horas calendario |

Estos valores son **resultados del análisis inicial**, sujetos a reproducción y comprobación con los archivos originales autorizados. El notebook contiene código para recalcular indicadores y generar visualizaciones; sus salidas deben ejecutarse y guardarse para que sean visibles directamente en GitHub.

## 5. Modelos predictivos

Se plantean dos tareas:

- **Regresión:** estimar horas de resolución. Se comparan una línea base de mediana, **Ridge** y **Random Forest Regressor** mediante MAE y otras métricas.
- **Clasificación:** estimar riesgo de incumplimiento de ANS. Se comparan una línea base mayoritaria, **regresión logística** y **Random Forest Classifier** mediante precisión, recall, F1 y matriz de confusión.

La estrategia temporal del notebook utiliza aperturas de **2024 para entrenamiento**, **2025 para validación futura** y **2026 para prueba inicial**. La primera comparación de modelos no constituye aún una selección final optimizada mediante validación 2025.

**Hallazgo importante:** en el análisis inicial, los modelos de regresión no superaron el MAE de la línea base de mediana. No se declara una mejora predictiva que las métricas no respalden.

## 6. Arquitectura propuesta de Big Data

```text
Exportaciones GLPI (CSV autorizados)
              |
              v
   Ingesta y validación de esquema
              |
              v
     Limpieza y transformación
      Pandas / PySpark (previsto)
              |
              v
      Datos analíticos / Parquet
              |
              v
   EDA y modelos scikit-learn
              |
              v
   Métricas, gráficas e informe
              |
              v
     Dashboard (pendiente)
```

El notebook incluye una sección de **PySpark y Parquet como código propuesto**, pero **su ejecución, validación y comparación de rendimiento todavía están pendientes**. No se atribuyen tiempos de procesamiento ni mejoras de rendimiento sin mediciones reales.

## 7. Contenido del repositorio

```text
GLPI-Ciencia-Datos-Big-Data/
├── README.md
└── Avance_Proyecto_GLPI_Ciencia_Datos_Big_Data.ipynb
```

**Notebook:** [Abrir análisis y modelos](Avance_Proyecto_GLPI_Ciencia_Datos_Big_Data.ipynb).

Los CSV originales **no se incluyen** por motivos de confidencialidad. Los archivos de resultados derivados que contengan registros individuales tampoco deben publicarse sin autorización y revisión.

## 8. Requisitos y ejecución local

**Entorno recomendado:** Python 3.10 o posterior, Jupyter Notebook o VS Code con extensión Jupyter. Para el componente Big Data se requiere además Java y PySpark configurados.

Instalar dependencias principales:

```bash
python -m pip install notebook pandas numpy matplotlib scikit-learn pyarrow
```

El componente opcional de Spark requiere:

```bash
python -m pip install pyspark
```

**Procedimiento:**

1. Clonar el repositorio y entrar en su carpeta:

   ```bash
   git clone https://github.com/MateusJg/GLPI-Ciencia-Datos-Big-Data.git
   cd GLPI-Ciencia-Datos-Big-Data
   ```

2. Obtener **por un canal institucional autorizado** los CSV necesarios y ubicarlos localmente junto al notebook, con estos nombres exactos:

   ```text
   GLPI-2024.csv
   GLPI-2025.csv
   GLPI-2026.csv
   ```

3. Abrir `Avance_Proyecto_GLPI_Ciencia_Datos_Big_Data.ipynb` en Jupyter o VS Code y seleccionar el intérprete de Python con las dependencias instaladas.
4. Ejecutar las celdas de forma secuencial. El código lee CSV con separador `;`, codificación UTF-8 y fechas interpretadas con día primero. Si las exportaciones tienen otro formato, será necesario adaptar la lectura.
5. Revisar los resultados y las advertencias de calidad de datos antes de interpretar métricas. La sección PySpark está comentada y requiere habilitación y validación independiente.

**Atención:** el notebook genera una carpeta `resultados_notebook/` que puede contener `casos_analiticos.csv` con registros individuales. **No subir esa carpeta ni los CSV institucionales a un repositorio público.**

Para impedir inclusiones accidentales se recomienda crear un archivo `.gitignore` con, al menos:

```gitignore
GLPI-*.csv
resultados_notebook/
.ipynb_checkpoints/
__pycache__/
.venv/
```

Antes de cada publicación, verificar los archivos con `git status` y revisar que las salidas del notebook no expongan nombres, títulos, identificadores o información institucional sensible.

## 9. Limitaciones y trabajo pendiente

- Reproducir y conservar salidas verificables del notebook con datos autorizados.
- Ejecutar PySpark y Parquet y medir su desempeño frente al flujo Pandas/CSV.
- Usar 2025 para ajustar modelos y establecer una evaluación temporal rigurosa sin reutilizar repetidamente 2026 para selección.
- Verificar las reglas de ANS, horarios hábiles, festivos y excepciones aplicables.
- Evaluar mejoras predictivas y desarrollar un tablero de visualización.
- Completar el informe académico, conclusiones y referencias conforme a las exigencias de la asignatura.

## 10. Equipo de trabajo

| Integrante | Responsabilidad propuesta |
| --- | --- |
| Brayan Stiven Sandoval Reyes | Ingeniería de datos y componente PySpark |
| José Gregorio Mateus Malagón | Análisis de datos y modelos predictivos |
| Sara Milena Villacorte Pedraza | Visualización, integración y documentación |

Las responsabilidades indicadas son una **distribución propuesta**, no una certificación de tareas terminadas por cada integrante.

## 11. Uso y confidencialidad

Proyecto con fines académicos. Los datos GLPI son de origen institucional y su utilización, publicación o redistribución depende de las autorizaciones correspondientes. El repositorio público debe contener solamente código, documentación y resultados agregados aprobados para divulgación.

---

**Repositorio:** https://github.com/MateusJg/GLPI-Ciencia-Datos-Big-Data
