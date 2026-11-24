# materia-integradora

## Documentación del proyecto

Guarda los documentos en PDF o Markdown según su propósito.

- [Contexto](docs/context/). La propuesta de tesis describe el problema, los objetivos y el alcance del estudio. El protocolo define cómo seleccionar y evaluar las arquitecturas. La guía IEEEtran HOWTO fija el formato del paper.
- [Informes](docs/reports/). Aquí se registran avances y resultados parciales, como el recuento de imágenes revisadas y los tipos de anomalías encontrados en las tapas.

## Reproducir los experimentos

El estudio compara PatchCore, EfficientAD-S, DRAEM y Dinomaly para detectar y localizar defectos en tapas industriales. El flujo comienza con una prueba corta de cada arquitectura en MVTec AD. Después se congelan los modelos y se evalúan sobre las mismas tapas reales.

1. [Preparar el equipo Windows](docs/guides/windows-setup.md). Instalar con `uv`, configurar Roboflow y verificar CUDA.
2. [Preparar los datos](docs/guides/dataset.md). Descargar una versión, revisar máscaras y comprobar las particiones por pieza, lote o sesión.
3. [Ejecutar el protocolo](docs/guides/experiments.md). Abrir los cuadernos en orden, estimar el tiempo con la GPU real e interpretar las métricas.

| Ruta | Contenido |
| --- | --- |
| `configs/models/` | Configuración versionada de cada arquitectura |
| `notebooks/data/` | Descarga, auditoría y congelamiento |
| `notebooks/models/` | Un punto de entrada por arquitectura |
| `notebooks/shared/` | Código Python común a los cuadernos |
| `notebooks/evaluation/` | TEST, estadísticas, ejemplos de errores y ablaciones |
| `data/` | Exportaciones, máscaras, datos públicos y pares sintéticos locales |
| `artifacts/` | Pesos, registros, predicciones, imágenes de errores y tablas locales |
| `docs/reports/` | Informes en español seleccionados para compartir |
| `docs/thesis/` | Tesis en LaTeX, lista para comprimir y subir a Overleaf |
| `docs/paper/` | Paper IEEE en LaTeX, lista para comprimir y subir a Overleaf |

Los cuadernos crean los directorios de trabajo cuando los necesitan. Los datos privados, las credenciales y las salidas voluminosas quedan fuera de Git.

Los [resultados compartidos](results/README.md) reúnen el registro de cada corrida y las métricas finales que se pueden consultar desde GitHub.
