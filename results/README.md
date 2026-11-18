# Resultados compartidos

Los cuadernos crean un registro ligero al terminar cada corrida y exportan las tablas al evaluar el estudio. Desde la estación Windows, revisa los archivos nuevos con `git status --short -- results` y súbelos con el flujo habitual de commit y push.

| Ruta | Contenido |
| --- | --- |
| `runs/<run-id>/record.json` | Estado, modelo, configuración, semilla, revisión de Git, entorno y costo. En fallos se registra el tipo de error. El diagnóstico completo queda en la estación. |
| `runs/<run-id>/` | Umbral de VAL y métricas de validación o del benchmark público cuando corresponda. |
| `studies/<dataset-hash>/` | Plan congelado, métricas de TEST por corrida, tablas entre semillas y gráfico agregado. El análisis de robustez añade una tabla por grupo. Usa identificadores de grupo aptos para compartir. |
| `studies/<dataset-hash>/synthetic/<pair-hash>/` | Plan, métricas y diferencias pareadas del estudio sintético. |

Conserva los archivos originales de las corridas definitivas en un almacenamiento compartido con respaldo. Para vincular una copia verificada, añade manualmente `backup.json` junto al registro de la corrida o al plan del estudio. Incluye la ubicación estable y la huella del archivo respaldado. Ejemplo de estructura:

```json
{
  "file": "weights.pt",
  "location": "<ubicación compartida sin credenciales>",
  "sha256": "<huella SHA-256 verificada>"
}
```

En PowerShell, `Get-FileHash -Algorithm SHA256 <path>` calcula la huella. Para pesos, compárala con `weights_sha256` en el registro. Para un archivo comprimido de mapas o datos, calcula la huella del archivo completo y comprueba la copia remota. El hash del registro original que aparece en el plan corresponde a `artifacts/runs/<run-id>/record.json` y se conserva en el resumen como `source_record_sha256`. Los mapas individuales, predicciones por captura y galerías con imágenes se conservan en el respaldo privado.
