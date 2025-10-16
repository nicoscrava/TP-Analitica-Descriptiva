# TP-Analitica-Descriptiva

## Estructura y adquisición de los datos

El proyecto utiliza información proveniente de la plataforma **Inside Airbnb**, específicamente de la sección correspondiente a la ciudad de **Buenos Aires**.

En el repositorio, dentro de la carpeta `/data/`, se incluyen **archivos de muestra** extraídos del paquete de datos publicado el **29 de enero de 2025**.  
Estos archivos son una versión reducida del dataset original, seleccionada para facilitar la manipulación y el análisis en entornos colaborativos (por ejemplo, Google Colab o GitHub).  
La muestra conserva la misma estructura de columnas y tipos de datos que el dataset completo, por lo que es plenamente representativa para el desarrollo del análisis exploratorio inicial.

El dataset contiene información detallada de **publicaciones activas en Airbnb**, incluyendo tanto características del alojamiento como del anfitrión.

Para quienes deseen replicar o ampliar el análisis con la totalidad de los datos, el dataset completo se encuentra disponible públicamente en:  
👉 https://insideairbnb.com/get-the-data/  

Allí pueden descargarse los archivos `.csv.gz` correspondientes, que incluyen todas las observaciones y variables utilizadas.

En caso de trabajar con el dataset completo, se recomienda ubicarlo en la misma carpeta `/data/` y actualizar las rutas en el notebook de análisis (`TP_Descriptiva.ipynb`).

---

# Limpieza de datos
Estandarizaciones aplicadas (según cada dataset):

- **`calendar`**
  - `price`: eliminación de símbolos (`$`, `,`) y conversión a `float`.
  - `available`: mapeo `'t'/'f'` → `True/False`.
  - `date`: conversión a `datetime`.
  - Se elimina la columna vacía `adjusted_price`.

- **`reviews`**
  - `date`: conversión a `datetime`.

- **`listings`**
  - Se selecciona un subconjunto de columnas relevantes en `df_listings_limpio`.
  - `host_since`: `datetime`.
  - `host_response_rate` y `host_acceptance_rate`: conversión de `'85%'` → **proporción** `0.85` (no 0–100).
  - `host_is_superhost`, `instant_bookable`, `host_identity_verified`: mapeo `'t'/'f'` → `True/False`.  
    > Tras la limpieza, `instant_bookable` queda como `bool`; en las otras dos puede persistir `object` por presencia de nulos.
  - `price`: eliminación de símbolos (`$`, `,`) y conversión a `float`.

---

# Tratamiento de 0s y nulos
- **Valores imposibles**
  - En `calendar`, **`price <= 0`** se marca como faltante (`NaN`).

- **Detección de nulos**
  - Se reportan conteos y **porcentajes de nulos** por columna (todos los datasets).
  - Visualización del patrón de faltantes con `missingno`.

- **MAR/MCAR (diagnóstico)**
  - Se ejecuta el **Little’s MCAR test** (pyampute) en `reviews`, `calendar` y `listings_limpio`; el resultado indica **p≈0** (no MCAR), por lo que se procede con imputación.

- **Imputación**
  - Se usa **`KNNImputer(n_neighbors=5)`** para **variables numéricas** en `listings_limpio` y `calendar`, **excluyendo** columnas no numéricas o con >20% nulos.
  - En `calendar`, tras la imputación se **redondean hacia arriba** `minimum_nights` y `maximum_nights` y se tipan como `Int64`.

> No se aplicó imputación categórica por moda (global o por barrio).


---

## Eliminación de columnas
Se comienza el analisis unicamente con estas columnas de listings: 'id', 'name', 'description', 'neighborhood_overview', 'host_id', 'host_since', 'host_location', 'host_about', 'host_response_time', 'host_response_rate', 'host_acceptance_rate', 'host_is_superhost', 'host_identity_verified','neighbourhood_cleansed', 'property_type', 'room_type', 'accommodates', 'bathrooms', 'bathrooms_text', 'bedrooms', 'beds', 'amenities', 'price', 'minimum_nights', 'maximum_nights', 'availability_90', 'number_of_reviews_ltm','review_scores_rating','review_scores_accuracy','review_scores_cleanliness','review_scores_checkin','review_scores_communication','review_scores_location','review_scores_value','instant_bookable'.

Las demas se pueden considerar eliminadas.
De calendar se borra 'adjusted_price' .
Las columnas con >20% de datos faltantes no se imputaron y pueden considerarse eliminadas.


---

# Detección y análisis de outliers
Enfoque aplicado:

1. **Exploratorio** con transformaciones (Box–Cox con *shift* cuando ≤0) y **z-scores** para cuantificar atipicidad por variable.
2. **Detección multivariada**:
   - **Isolation Forest** (`contamination=0.05`)
   - **Local Outlier Factor** (`n_neighbors=20`, `contamination=0.05`)
   - Se agregan columnas **`outlier_iso`** y **`outlier_lof`**.

**Decisión adoptada:** se **eliminó** el **5%** de registros de `listings_limpio` marcados por **Isolation Forest** (criterio principal de poda). LOF se usó como contraste, sin eliminar por LOF.

> Nota: aunque se probaron transformaciones y normalidad, las variables se mantienen mayormente no normales; los histogramas se interpretan con ese criterio.

---

# Contexto de negocio, hipótesis y objetivos
El análisis busca comprender **drivers de precio y valoración** de alojamientos en CABA: características del alojamiento, **amenities**, ubicación y desempeño del anfitrión.

---

# Hipótesis planteadas
| Nº | Hipótesis | Variables involucradas | Visualizacion |
|----|-----------|------------------------|-----------------------|
| 1  | Los precios en barrios turísticos (p.ej. Palermo, Recoleta, San Telmo) son sistemáticamente más altos que en el resto. | `price`, `neighbourhood_cleansed` | **Parcial**: se clasifican barrios y se grafican distribuciones en histogramas. |
| 2  | Amenities claves (p.ej. gym, pool, free parking) incrementan precio y calificación. | `amenities`, `price`, `review_scores_rating` | **Implementada (amenities seleccionadas)**: se crean flags `has_gym`, `has_pool`, `has_free_parking`, combinaciones y comparativas de medias en un bar plot. |
| 3  | Menor tiempo de respuesta del host → mejores reviews. | `host_response_time`, `review_scores_rating` | Boxplot por categorías de `host_response_time`. |
| 4  | Existe relación entre score de limpieza y de ubicación. | `review_scores_cleanliness`, `review_scores_location` | Scatter para inspección visual. |

---

## Visualizaciones y conclusiones preliminares

**Gráficos y conclusiones preliminares:**
1. **Distribución de precios:**  
   Muestra asimetría hacia la derecha y outliers explicables por capacidad o ubicación.
   Los **barrios turísticos** parecen presentar precios más altos.  
2. **Precio y review score por combinacion de ammenities:**  
   Pareceria que la combinacion de los ammenities gym, parking gratis y pileta hace que el promedio del precio de la publicacion y el promedio del review score cambie.  
3. **El tiempo de respuesta impacta en los reviews que recibe un host:**  
   El promedio de score que recibe un host pareceria no cambiar segun el tiempo de respuesta.  
4. **Hay una relación entre el score de reviews de limpieza y el score de reviews de la ubicación:**  
   Pareceria no haber relacion entre el score de reviews de limpieza y el de ubicacion.  
