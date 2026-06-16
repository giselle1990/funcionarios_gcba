# Buscador GCBA para GitHub Pages

Subir estos archivos a GitHub Pages:

- `index.html`
- `data.json`

## Actualizar cuando llega un PDF nuevo

Desde PowerShell:

```powershell
cd "C:\Users\Administrador\Desktop\PYTHON\version 1-2"
C:\Users\Administrador\.cache\codex-runtimes\codex-primary-runtime\dependencies\python\python.exe incorporar_decretos_a_base.py "C:\Users\Administrador\Downloads\Seguimiento de Decretos - Mayo.pdf"
C:\Users\Administrador\.cache\codex-runtimes\codex-primary-runtime\dependencies\python\python.exe exportar_web.py
```

Despues subir de nuevo `web\data.json` al repositorio de GitHub Pages. Si tambien cambiaste el diseño, subir `web\index.html`.
