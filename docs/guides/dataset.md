# Descargar y congelar las imágenes

## Datos que necesita el SDK

Completa el archivo local `.env` en Windows. La clave se obtiene en el espacio de trabajo de Roboflow. En la dirección de un proyecto, los segmentos de espacio de trabajo y proyecto son sus identificadores; el entero de la versión se consulta en *Versions*. El proyecto debe tener una versión generada que incluya las imágenes normales revisadas y las máscaras de defectos. [Exportación oficial de Roboflow](https://docs.roboflow.com/datasets/versions/dataset-versions/exporting-data)

| Variable | Valor |
| --- | --- |
| `ROBOFLOW_API_KEY` | Clave privada con acceso de lectura y exportación |
| `ROBOFLOW_WORKSPACE` | Identificador del espacio de trabajo |
| `ROBOFLOW_PROJECT` | Identificador del proyecto |
| `ROBOFLOW_VERSION` | Número de la versión congelada |

1. Prepara en Roboflow una versión **real** sin aumentos generados. Conserva TRAIN, VAL y TEST por grupos de captura. Desactiva el filtrado de normales si elimina tapas ya revisadas. Registra los pasos de preprocesamiento para interpretar la resolución y compresión de la exportación.
2. Ejecuta `notebooks/data/01-download.ipynb`. El SDK solicita el formato de segmentación por instancias `coco-segmentation` y guarda imágenes y anotaciones en `data/roboflow/<workspace>/<project>/<version>/raw/`. La descarga se publica localmente al completar las tres particiones. En sesiones posteriores se reutiliza. El entrenamiento lee muestras desde disco. [Ejemplo oficial de exportación con máscaras](https://roboflow.com/blog/fine-tune-paligemma-2)
3. Abre `02-freeze.ipynb` en esa misma carpeta de cuadernos. La primera parte crea `review.csv` al lado de `raw/`. Por cada archivo, completa `group_id`, `origin` y `label`. Usa `real` como origen y `normal` o `defect` como etiqueta confirmada. Recupera el grupo de los registros de adquisición. Si dos capturas comparten pieza, lote **o** sesión, asigna el mismo grupo a todo el conjunto conectado. Si esa relación se desconoce, recupérala antes de congelar. Un identificador único inventado por imagen invalidaría esta prueba.
4. En el cuaderno, copia los nombres de clases COCO en `defect_classes`. Declara en `ignored_classes` las categorías ajenas al defecto, por ejemplo, el contorno de la tapa completa. Se unifican los polígonos o RLE de los defectos en una máscara binaria. Las cajas por sí solas requieren anotar las máscaras antes de continuar.
5. Ejecuta el resto del cuaderno. Revisa los conteos. TRAIN necesita normales reales; VAL y TEST necesitan ambas clases. Los defectos de TRAIN pueden conservarse en el inventario y quedan excluidos del ajuste de los baselines. Comprueba visualmente una muestra de imágenes y máscaras, especialmente defectos pequeños o periféricos. El congelamiento comprueba dimensiones, concordancia etiqueta–máscara, grupos y duplicados exactos entre particiones. La revisión humana cubre casi duplicados y errores de anotación.
6. Conserva `frozen/manifest.csv`, `source.json`, las máscaras y la planilla de auditoría junto con los datos originales. Las huellas SHA-256 permiten verificar imágenes, máscaras y manifiesto al reproducir. Para corregir un conjunto ya usado, genera una nueva versión y conserva la anterior.

## Benchmark público

Descarga la categoría **bottle** de MVTec AD original. Consulta las condiciones del proveedor y extrae el archivo de modo que existan `data/benchmarks/mvtec-ad/bottle/train/good/`, `test/` y `ground_truth/` dentro de la categoría. La prueba pública funciona independientemente de Roboflow. [MVTec AD](https://www.mvtec.com/company/research/datasets/mvtec-ad)

## Pares sintéticos para E3 y E4

Cuando el equipo disponga del generador, guarda sus imágenes y máscaras en `data/synthetic/`. Prepara `pairs.csv` con columnas `source,image,mask,generator,defect,review`. `source` es el campo `file` de una tapa **normal de TRAIN**; `image` y `mask` son rutas relativas a `data/synthetic/`. La imagen alterada debe estar alineada con su tapa original. Usa máscaras binarias con fondo cero. Registra la revisión humana de realismo, diversidad, cobertura y correspondencia imagen–máscara antes de marcar `accepted`. Conserva la versión, parámetros y semilla del generador en sus registros de procedencia. El cuaderno calcula conteos y huellas y comprueba dimensiones, duplicados y procedencia.
