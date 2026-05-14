# Buscador web GCBA - Actualización automatizada desde informes de decretos

Proyecto para mantener actualizada una base de funcionarios/as del Gobierno de la Ciudad de Buenos Aires (GCBA) a partir de informes oficiales en PDF, incorporando detecciones de designaciones, renuncias, ratificaciones y otros cambios relevados en informes periódicos de seguimiento de decretos.

La lógica central del proyecto es transformar documentos administrativos no estructurados —por ejemplo, informes PDF enviados por mail— en datos reutilizables, trazables y publicables en una versión web.

---

## Objetivo del proyecto

Automatizar, total o parcialmente, el circuito de actualización de la base `data/data2.csv` a partir de informes PDF del tipo:

```text
Seguimiento de Decretos - Marzo.pdf
Seguimiento de Decretos - Abril.pdf
Seguimiento de Decretos - Mayo.pdf
```

El flujo permite:

1. Recibir o descargar el PDF del informe.
2. Extraer texto del PDF.
3. Identificar registros vinculados a designaciones, renuncias o ratificaciones.
4. Detectar personas mencionadas en los decretos.
5. Cruzar esas personas con la base existente.
6. Incorporar nuevas filas con trazabilidad del decreto.
7. Generar backups y reportes de actualización.
8. Exportar la base actualizada a `web/data.json`.
9. Publicar los datos en GitHub Pages.

---

## Estructura del proyecto

```text
VERSION NUEVA/
│
├── data/
│   └── data2.csv
│
├── web/
│   ├── index.html
│   ├── data.json
│   └── README.md
│
├── backups/
│   └── backups automáticos de data2.csv
│
├── reportes_actualizacion/
│   └── reportes Excel/CSV de cambios detectados
│
├── pdfs_entrada/
│   └── PDFs descargados desde Outlook o guardados manualmente
│
├── actualizar_base_gcba.py
├── extraer_cambios_decretos.py
├── incorporar_decretos_a_base.py
├── exportar_web.py
└── README.md
```

---

## Scripts principales

### `extraer_cambios_decretos.py`

Extrae del PDF la sección relevante de designaciones y renuncias, detecta acciones posibles y genera un reporte de revisión.

Este script **no modifica la base principal**. Sirve para auditar qué información se está detectando antes de incorporarla.

Uso:

```powershell
python extraer_cambios_decretos.py "C:\ruta\al\pdf\Seguimiento de Decretos - Mayo.pdf"
```

Salida esperada:

```text
Registros detectados: X
Reporte: reportes_actualizacion/cambios_decretos_YYYYMMDD_HHMMSS.xlsx
```

---

### `incorporar_decretos_a_base.py`

Procesa uno o más PDFs y agrega a `data/data2.csv` las personas detectadas en los decretos.

Si la persona ya existe en la base, reutiliza datos como CUIL, mail, sigla y ministerio. Si no encuentra una coincidencia segura, incorpora la fila con los datos disponibles y marca el origen como `Detectado en decreto`.

También agrega columnas de trazabilidad:

```text
fuente_pdf
fecha_decreto
decreto
accion_decreto
detalle_decreto
```

Uso:

```powershell
python incorporar_decretos_a_base.py "C:\ruta\al\pdf\Seguimiento de Decretos - Mayo.pdf"
```

También permite procesar varios PDFs juntos:

```powershell
python incorporar_decretos_a_base.py "Seguimiento de Decretos - Marzo.pdf" "Seguimiento de Decretos - Abril.pdf" "Seguimiento de Decretos - Mayo.pdf"
```

---

### `exportar_web.py`

Lee `data/data2.csv` y genera el archivo `web/data.json`, que es utilizado por la versión web del buscador.

Uso:

```powershell
python exportar_web.py
```

Salida esperada:

```text
Exportados X registros a web/data.json
```

---

### `actualizar_base_gcba.py`

Actualiza `data/data2.csv` desde un CSV o Excel estructurado, normalizando columnas y generando reportes de altas, bajas y modificaciones.

Uso:

```powershell
python actualizar_base_gcba.py "C:\ruta\archivo_entrada.xlsx"
```

Modo simulación:

```powershell
python actualizar_base_gcba.py "C:\ruta\archivo_entrada.xlsx" --simular
```

---

## Actualización manual con un PDF nuevo

Guardar el PDF en una carpeta local, por ejemplo:

```text
C:\Users\Administrador\Downloads\Seguimiento de Decretos - Mayo.pdf
```

Luego ejecutar desde PowerShell:

```powershell
cd "C:\Users\Administrador\Desktop\PYTHON\version 1-2\VERSION NUEVA"

C:\Users\Administrador\.cache\codex-runtimes\codex-primary-runtime\dependencies\python\python.exe incorporar_decretos_a_base.py "C:\Users\Administrador\Downloads\Seguimiento de Decretos - Mayo.pdf"

C:\Users\Administrador\.cache\codex-runtimes\codex-primary-runtime\dependencies\python\python.exe exportar_web.py
```

Después subir los cambios a GitHub:

```powershell
git add data/data2.csv web/data.json reportes_actualizacion backups
git commit -m "Actualización estructura GCBA desde informe de decretos"
git push
```

Si no hubiera cambios nuevos, puede usarse:

```powershell
git add data/data2.csv web/data.json reportes_actualizacion backups
git diff --cached --quiet
if ($LASTEXITCODE -eq 0) {
  echo "No hay cambios para commitear"
} else {
  git commit -m "Actualización automática estructura GCBA"
  git push
}
```

---

## Publicación web

La carpeta `web/` está pensada para ser publicada con GitHub Pages.

Archivos principales:

```text
web/index.html
web/data.json
web/README.md
```

Cada vez que se actualiza `data/data2.csv`, debe ejecutarse:

```powershell
python exportar_web.py
```

Esto regenera:

```text
web/data.json
```

Luego se realiza el commit y push a GitHub para que la web quede actualizada.

---

## Automatización propuesta con Outlook + n8n

El proyecto puede automatizarse para que cada 15 días revise una cuenta de Outlook y procese automáticamente el PDF recibido.

### Flujo esperado

```text
Schedule cada 15 días
   ↓
Buscar mails en Outlook
   ↓
Filtrar remitente y asunto
   ↓
Descargar adjunto PDF
   ↓
Ejecutar scripts Python
   ↓
Actualizar data2.csv
   ↓
Exportar web/data.json
   ↓
Commit y push a GitHub
   ↓
Enviar notificación de resultado
```

### Condiciones del mail

Remitente esperado:

```text
Victoria Ghoglione
```

Asunto esperado:

```text
Informe de Estructura Orgánica del GCBA
```

Adjunto esperado:

```text
Seguimiento de Decretos - Marzo.pdf
Seguimiento de Decretos - Abril.pdf
Seguimiento de Decretos - Mayo.pdf
```

El nombre del archivo puede variar por mes, pero debería contener:

```text
Seguimiento de Decretos
```

y terminar en:

```text
.pdf
```

---

## Automatización con n8n local

Se recomienda usar n8n instalado localmente o self-hosted, porque el proyecto necesita acceder a carpetas locales y ejecutar scripts Python ubicados en la computadora.

### Nodos sugeridos

1. `Schedule Trigger`
2. `Microsoft Outlook`
3. `IF`
4. `Write Binary File`
5. `Execute Command`
6. `Send Email` o notificación final

### Ejemplo de comando para n8n

```powershell
cd "C:\Users\Administrador\Desktop\PYTHON\version 1-2\VERSION NUEVA"
C:\Users\Administrador\.cache\codex-runtimes\codex-primary-runtime\dependencies\python\python.exe incorporar_decretos_a_base.py "pdfs_entrada\Seguimiento de Decretos - Mayo.pdf"
C:\Users\Administrador\.cache\codex-runtimes\codex-primary-runtime\dependencies\python\python.exe exportar_web.py
git add data/data2.csv web/data.json reportes_actualizacion backups
git diff --cached --quiet
if ($LASTEXITCODE -eq 0) {
  echo "No hay cambios para commitear"
} else {
  git commit -m "Actualización automática estructura GCBA"
  git push
}
```

---

## Recomendación de mejora

Para simplificar la automatización, conviene crear más adelante un script único, por ejemplo:

```text
automatizar_actualizacion_gcba.py
```

Ese script debería:

1. Recibir la ruta del PDF.
2. Ejecutar la incorporación de decretos.
3. Exportar `web/data.json`.
4. Ejecutar `git add`.
5. Crear commit solo si hay cambios.
6. Hacer `git push`.
7. Generar un log final con el resultado.

De esta forma, n8n solo tendría que descargar el PDF y ejecutar un único comando.

---

## Consideraciones sobre calidad de datos

Los informes PDF de seguimiento de decretos no necesariamente incluyen todos los datos necesarios para completar una fila de la base.

En particular, pueden faltar:

```text
CUIL
mail_laboral
sigla
ministerio
escalafon
```

Por eso, cuando una persona detectada en el PDF ya existe en la base, el script intenta completar esos campos usando la información previa. Si no encuentra una coincidencia segura, la fila se incorpora con información parcial para evitar cargar datos incorrectos.

Esto permite mantener trazabilidad sin forzar coincidencias dudosas.

---

## Salidas generadas

El proceso puede generar:

```text
data/data2.csv
web/data.json
backups/data2_YYYYMMDD_HHMMSS.csv
reportes_actualizacion/cambios_decretos_YYYYMMDD_HHMMSS.xlsx
reportes_actualizacion/filas_incorporadas_decretos_YYYYMMDD_HHMMSS.xlsx
```

---

## Buenas prácticas

Antes de automatizar completamente, se recomienda:

1. Probar varios PDFs históricos.
2. Revisar los reportes generados.
3. Confirmar que las personas detectadas sean correctas.
4. Verificar que no haya duplicados.
5. Revisar los backups.
6. Ejecutar primero en modo controlado antes de activar el flujo automático.

---

## Idea central

Este proyecto busca reducir tareas manuales y mejorar la trazabilidad en procesos de gestión pública.

La automatización no reemplaza el criterio humano: permite liberar tiempo operativo para revisar mejor, controlar mejor y mantener datos más actualizados.

```text
Menos tarea manual.
Más trazabilidad.
Mejores datos para gestionar.
```

---

## Tecnologías utilizadas

```text
Python
pandas
pypdf
Git
GitHub Pages
n8n
Outlook / Microsoft Graph
PowerShell
```
