# Preparar la estación de investigación

El equipo del protocolo es un Windows x64 con una GeForce RTX 2080 Ti de 11 GB, un Intel Core i9-10850K y 16 GB de RAM. Usa una GPU a la vez. Reserva espacio en un SSD para el entorno CUDA, los datos, las cachés de pesos y los mapas por imagen. Como presupuesto inicial, deja unos 30 GB libres más el tamaño de los conjuntos de datos y de las salidas. Comprueba el espacio tras la primera corrida.

## Instalación en PowerShell

Instala Git y uv. Si ya están disponibles, empieza por clonar. Usa un controlador NVIDIA que admita CUDA 12.4 o posterior y comprueba que `nvidia-smi` identifica la tarjeta. La distribución CUDA seleccionada está en el archivo de dependencias. [Instalación de uv](https://docs.astral.sh/uv/getting-started/installation/) · [Versiones de PyTorch para CUDA 12.4](https://pytorch.org/get-started/previous-versions/)

```powershell
winget install --id Git.Git -e
winget install --id astral-sh.uv -e
```

Abre otra consola después de la instalación. Clona la revisión que contenga los cuadernos en una ruta corta. Desde la raíz, ejecuta lo siguiente.

```powershell
git clone https://github.com/waldaara/materia-integradora.git
cd materia-integradora
nvidia-smi
uv python install 3.11
uv sync --locked
Copy-Item .env.example .env
notepad .env
uv run --locked jupyter lab
```

Usa el enlace local que imprime Jupyter. Abre `notebooks/00-environment.ipynb` y ejecuta todas las celdas con el kernel del proyecto. La prueba verifica el intérprete, CUDA, una multiplicación FP16 y tres métricas de ejemplo. Si falla, comprueba primero la ruta de Python y la versión del controlador. Reanuda desde esta prueba después de corregirlos.

El archivo `uv.lock` fija las dependencias para Windows x64 y Python 3.11, entre ellas PyTorch 2.6.0+cu124, Torchvision 0.21.0+cu124 y Anomalib 2.6.2. Las ruedas CUDA proceden del índice explícito de PyTorch. Las primeras ejecuciones pueden requerir Internet para los pesos, Imagenette y las texturas DTD. [Integración oficial de uv con PyTorch](https://docs.astral.sh/uv/guides/integration/pytorch/) · [Distribución de Anomalib](https://pypi.org/project/anomalib/2.6.2/)

## Reproducir una corrida

Conserva la revisión de Git, el lockfile, los JSON de configuración y la carpeta de resultados. El registro guarda versiones de Python, paquetes, CUDA y controlador, GPU, semilla, huellas de datos y configuración, tiempos y memoria. Después del preflight de los cuatro modelos, archiva el entorno aceptado con esas evidencias. El flujo suministrado es nativo de Windows. Una imagen Docker requeriría su propia validación de CUDA y del sistema operativo antes de usarla para reproducir resultados.

En otra sesión vuelve a la raíz del clon y ejecuta `uv run --locked jupyter lab`. Reinicia el kernel al cambiar de arquitectura o después de un error de memoria. Evita abrir dos entrenamientos en la misma GPU.
