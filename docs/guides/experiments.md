# Ejecutar y evaluar las arquitecturas

La autoridad metodológica es el [protocolo de selección y evaluación, versión 1.1](../context/architecture-selection-and-evaluation-protocol-v1.1.pdf). Los cuatro cuadernos usan las implementaciones de Anomalib y comparten los cargadores, la calibración y el evaluador. [Modelos de Anomalib](https://anomalib.readthedocs.io/en/latest/markdown/guides/reference/models/image/index.html)

## Una corrida por vez

Abre `notebooks/models/<model>/run.ipynb`, selecciona la etapa y la semilla, y ejecuta todas las celdas. El identificador impreso corresponde a un directorio nuevo en `artifacts/runs/`. Usa un kernel nuevo al cambiar de modelo. Una corrida fallida conserva `record.json` con la causa. Tras corregirla, el nuevo intento obtiene otro identificador.

| Carpeta | Ajuste nativo | Configuración inicial y evidencia |
| --- | --- | --- |
| `patchcore` | Banco de memoria con normales reales | WRN50-2, capas 2 y 3, coreset 0,10, FP32, 256 × 256. Registra los bytes del banco. Si hace falta, reduce el coreset y repite el preflight. |
| `efficientad-s` | Docente, alumno y autocodificador | Variante pequeña, lote 1, 256 × 256. Imagenette cumple su papel auxiliar en el método original. Presupuesto inicial de 70 000 pasos. |
| `draem-ref` | Reconstrucción y discriminación | Lote 1, 256 × 256, normales reales y síntesis procedural DTD. Presupuesto inicial de 700 épocas. |
| `dinomaly` | Reconstrucción de características DINOv2 | ViT-Base/14 con registros, lote 1, redimensionado a 448 y recorte central de 392, 5 000 pasos. Pesos maestros FP32 y operaciones compatibles en FP16 mediante Lightning. |

En el paquete de Anomalib 2.6.2, el constructor de Dinomaly convierte el modelo a BF16 cuando su argumento interno es `float16`. Por eso la configuración conserva `params.precision = "float32"` y selecciona `precision = "16-mixed"` en el entrenador. Si Base falla o deja poco margen, guarda el registro, cambia exclusivamente `encoder_name` a `vit_small_patch14_reg4_dinov2` y repite E0 tras reiniciar el kernel. Congela el encoder antes de las corridas definitivas. [Distribución de referencia](https://pypi.org/project/anomalib/2.6.2/)

### E0 y nivel A. Factibilidad y referencia pública

1. Ejecuta `preflight` con la primera semilla para cada arquitectura. Usa hasta ocho imágenes por partición y clase de MVTec AD, una época, ajuste e inferencia. Revisa el registro, memoria reservada y tiempos. El techo operativo configurado es 9,5 GiB de memoria CUDA reservada por PyTorch. Comprueba también la ocupación total en `nvidia-smi`, que incluye el escritorio y otros procesos. El banco completo de PatchCore puede requerir más memoria que esta muestra.
2. Ejecuta `benchmark` para comparar implementaciones en la categoría pública completa. Se reserva para VAL una de cada diez imágenes normales de TRAIN en orden de nombre. El percentil 95 de sus puntuaciones fija el umbral público. TEST público mantiene su partición oficial. Las métricas por píxel del benchmark usan la vista nativa. Revisa `public-test/metrics.json` antes de pasar a tapas.
3. Documenta en `docs/reports/` la referencia publicada **por categoría**, encoder, resolución, cantidad de normales, coreset o pasos, definición de las métricas y diferencia en puntos porcentuales. Este reparto retiene normales para calibrar EfficientAD y el umbral; decláralo al comparar con una referencia entrenada con el 100 %. Una reproducción comparable dentro de dos puntos es el objetivo del protocolo. Una tabla global de todas las categorías o un resultado de ViT-Base no sirven de referencia equivalente para una corrida individual de ViT-Small.

### E1 y E6. Baselines reales y evaluación final

1. Congela los datos con el [procedimiento de auditoría](dataset.md). Para cada modelo ejecuta `baseline` con 17, 29 y 43. Entrena con normales de TRAIN. Usa el último estado del presupuesto elegido. El umbral de imagen maximiza F1 en VAL; en empates, elige el umbral más bajo. Conserva `weights.pt`, `threshold.json`, las predicciones de VAL y el registro.
2. Abre `notebooks/evaluation/01-final-test.ipynb`. Revisa la tabla y coloca en `selected_runs` los doce identificadores definitivos. El cuaderno congela la selección, los pesos, umbrales y metadatos antes de calcular métricas sobre el único TEST real. Un reinicio retoma la misma selección y omite las evaluaciones ya terminadas.
3. Abre `02-report.ipynb`. Obtendrás métricas por corrida, media, desviación estándar muestral e intervalo t del 95 % por arquitectura sobre tres semillas. La galería muestra hasta cuatro errores por corrida. Interpreta el tamaño de muestra y examina los casos con falsos positivos y falsos negativos; una media por sí sola es insuficiente.

Las puntuaciones por imagen producen I-AUROC, I-AP, F1, precisión y sensibilidad. Para las tapas, el evaluador mide la localización sobre el mismo rectángulo central que ve Dinomaly. Proyecta todas las máscaras originales en esa región con vecino más cercano, y los mapas con interpolación bilineal. Usa histogramas de 65 536 niveles para aproximar P-AUROC, P-AP y AU-PRO con límite de falsos positivos 0,30 y componentes de ocho vecinos. Procesa un mapa a la vez. Informa los defectos que desaparecen del campo común o al rasterizar. Una pérdida de defectos periféricos debe figurar en el informe junto a los resultados de localización.

El evaluador cronometra el modelo PyTorch con lote 1, diez llamadas de calentamiento y sincronización CUDA. La ventana excluye lectura de disco, preprocesamiento en CPU y copia de la entrada. Informa p50, p95 y rendimiento en imágenes/s para ese mismo alcance. El registro de ajuste recoge los picos de memoria asignada y reservada; el TEST toma un nuevo pico de inferencia. Si se ensaya otro motor, registra su nombre y reporta sus medidas por separado.

### E2–E5 y E7. Preguntas adicionales

| Bloque | Cuaderno y comparación |
| --- | --- |
| E2 | `evaluation/03-ablations.ipynb`. Compara 10, 25, 50 y 100 % de las normales con subconjuntos anidados por semilla. Registra las imágenes usadas y compara primero en VAL. |
| E3 | `models/draem-ref/synthetic-study.ipynb`. Audita procedencia, máscaras y cobertura de defectos de los pares sintéticos. Adjunta la revisión visual de realismo y diversidad. |
| E4 | El mismo cuaderno entrena `draem-project` y opcionalmente `draem-mixed`. Todas las variantes ven una anomalía en el 50 % de los pasos, igual que DRAEM-Ref, para que el delta mida el tipo de anomalía y no su cantidad. En la rama del proyecto, la anomalía es un par generado elegido al azar y el objetivo de reconstrucción es su tapa original. En la mixta, la mitad de esas anomalías es procedural y la otra mitad son pares. Congela las corridas y calcula deltas por semilla frente a DRAEM-Ref con el mismo TEST. |
| E5 | `evaluation/03-ablations.ipynb`. Compara aumento horizontal activado/desactivado usando todas las normales. Documenta por qué la simetría es válida para la cámara. |
| E7 | `evaluation/04-robustness.ipynb`. Desglosa el TEST ya evaluado por grupo de adquisición, con conteos y errores. Los grupos con ambas clases admiten AUROC. |

Los experimentos de escasez y aumento generan resultados de exploración en VAL. Antes de elevar una variante al análisis final de tesis, registra la hipótesis y congela su propia selección y condiciones de comparación. La generación de imágenes sintéticas se incorpora cuando el equipo entregue los pares revisados.

## Cuánto tiempo reservar

Son **ejemplos de planificación**, pendientes de medición en la estación. La conexión, el volumen de datos, el controlador y las cachés pueden cambiar los tiempos. Presupuesta inicialmente 0,5–2 horas para entorno y descargas con una conexión rápida, además de la revisión humana. Ejecuta primero E0 de las cuatro arquitecturas y usa una corrida exploratoria suficientemente larga para medir el tiempo estable por paso. El tiempo de E0 incluye arranque y posibles descargas.

| Supuesto medido en Windows | Proyección para una semilla |
| --- | --- |
| EfficientAD-S tarda 0,10–0,40 s por paso | 70 000 pasos requieren 1,9–7,8 h de optimización. |
| Dinomaly tarda 0,5–2 s por paso | 5 000 pasos requieren 0,7–2,8 h de optimización. |
| DRAEM tarda 0,2–0,8 s por paso y hay 500 normales | 700 épocas con lote 1 requieren 19–78 h de optimización. |
| PatchCore | Mide extracción y construcción del coreset con conteos crecientes. La selección del banco puede crecer de forma no lineal. |

Multiplica el tiempo por semilla por tres para la tabla final. Añade validación, evaluación por píxel y escritura de mapas. Recalcula antes de lanzar DRAEM completo. Si el presupuesto exige cambiar pasos o épocas, documenta el cambio en el JSON antes de congelar, repite el preflight y usa el mismo presupuesto para la referencia y sus variantes.
