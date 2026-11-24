# Paper IEEE

Proyecto LaTeX autocontenido: `paper.tex` (plantilla IEEE *conference* con IEEEtran) y sus imágenes en `figures/`. Las referencias están dentro del mismo archivo. `figures/fig1.png` es un marcador gris de la plantilla; reemplázalo por la figura real.

El formato sigue la guía oficial [IEEEtran HOWTO](../context/ieeetran-howto.pdf). No se modifican márgenes, fuentes ni la clase.

Para compilar, comprime esta carpeta y súbela a Overleaf (New Project → Upload Project). Compila con pdfLaTeX sin cambios.

```bash
cd docs/paper && zip -r ../../paper.zip . -x README.md
```
