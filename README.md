# TP-Analitica-Descriptiva

## Estructura y adquisicion de los datos.

El proyecto utiliza información proveniente de la plataforma Inside Airbnb, específicamente de la sección correspondiente a la ciudad de Buenos Aires.

En el repositorio, dentro de la carpeta /data/, se incluyen archivos de muestra extraídos del paquete de datos publicado el 29 de enero de 2025.
Estos archivos son una versión reducida del dataset original, seleccionada para facilitar la manipulación y el análisis en entornos colaborativos (por ejemplo, Google Colab o GitHub). La muestra conserva la misma estructura de columnas y tipos de datos que el dataset completo, por lo que es plenamente representativa para el desarrollo del análisis exploratorio inicial.

El dataset contiene información detallada de publicaciones activas en Airbnb, incluyendo tanto características del alojamiento como del anfitrión.

Para quienes deseen replicar o ampliar el análisis con la totalidad de los datos, el dataset completo se encuentra disponible públicamente en: https://insideairbnb.com/get-the-data/
Allí pueden descargarse los archivos .csv.gz correspondientes, que incluyen todas las observaciones y variables utilizadas.

En caso de trabajar con el dataset completo, se recomienda ubicarlo en la misma carpeta /data/ y actualizar las rutas en el notebook de análisis (TP_Descriptiva.ipynb).

## Limpieza de datos.

Se realizó una estandarización inicial de tipos y valores:

 - Conversión de campos monetarios (price) a float, eliminando símbolos “$” y comas.
 - Conversión de porcentajes (host_response_rate, host_acceptance_rate) a enteros (0–100).
 - Campos booleanos (host_is_superhost, instant_bookable, host_identity_verified) mapeados a True/False.
 - Fechas (host_since) transformadas al tipo datetime.
 - Se reemplazaron valores imposibles o incoherentes por NaN:
   - price == 0
   - maximum_nights == 0
   - porcentajes o fechas vacías.

## Tratamiento de 0s y nulos.

Reemplazo de valores inválidos: los 0 en price y maximum_nights fueron tratados como faltantes, ya que representan datos inexistentes y no valores válidos del fenómeno.
Detección de nulos: se generó una tabla de porcentaje de faltantes por columna.
Eliminación de columnas: aquellas con más del 20 % de valores nulos fueron descartadas por bajo aporte al análisis y alto costo de imputación.

## Eliminacion de columnas.

Poner todas las columnas que borramos. O no usamos del dataset original. Cuentan las que no se importan, no se usan o se eliminan de por medio.

## Visualizaciones y conclusiones preliminares.
