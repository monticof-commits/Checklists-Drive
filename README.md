# 3dF Checklist — prototipo con Google Drive

## Qué es
Es la misma app de checklist de obra (mismo diseño blanco/negro, tabla en compu y tarjetas en celular, fotos con editor de lápiz, exportar PDF / Excel / CSV y compartir por WhatsApp), pero la sincronización ya **no usa Supabase**: cada ingeniero inicia sesión con **su propia cuenta de Google** y sus datos quedan en **su Google Drive** (15 GB gratis).

Se sigue editando todo **desde la app**, también en el celular y sin conexión. El Excel en Drive es la copia en la nube: los cambios de la app se suben, y si alguien edita el Excel, la app trae esos cambios al sincronizar.

## Cómo queda en Drive
```
Mi unidad/
└── 3dF Checklist/
    ├── checklist.xlsx      ← una hoja por pestaña + hoja "Config"
    └── fotos/              ← una foto .jpg por archivo
```
Cada pestaña de la app es su propia hoja, con el nombre de la pestaña (recortado a 31 caracteres y sin `\ / * ? : [ ]`). La hoja **Config** guarda la obra, el id de cada pestaña, el próximo id, si está oculta en la app y el nombre de la hoja. Un Excel viejo con una sola hoja "Tareas" se sigue leyendo; la próxima subida lo pasa al formato de una hoja por pestaña.

Ocultar una pestaña solo la saca de la barra de la app. Cada subida a Drive escribe **todas** las pestañas como hojas visibles del Excel, estén ocultas o no en la app. **Mostrar todas** las vuelve a mostrar. Si se borra una hoja en el Excel, la próxima sincronización elimina esa pestaña y sus tareas en la app: no se vuelve a crear desde lo que había quedado en el dispositivo.

Columnas de cada hoja: `id`, `Fecha`, `Tarea`, `Piso`, `Unidad`, `Ambiente`, `Ejecutado` (SI/NO), `Rubro`, `Contratista`, `Ingresado`, `FotosIds` (ids de Drive, separados por coma) y una columna oculta `TabId`. Para agregar una tarea desde el Excel alcanza con completar una fila nueva (sin `id`; la app le asigna uno). No borres la fila de encabezados.

Nombre de cada foto:
- Una sola foto en la tarea: `{pestaña} - {tarea}.jpg` (sin número al final).
- Varias fotos: `{pestaña} - {tarea} - 1.jpg`, `{pestaña} - {tarea} - 2.jpg`, etc. El número es el orden de las fotos de esa tarea.

Los caracteres que no pueden ir en un archivo (`\ / : * ? " < > |`) se reemplazan. Si después se agregan o se borran fotos, los nombres se actualizan en la próxima sincronización.

## Exportar
- **PDF** y **CSV** exportan la pestaña visible, con los filtros activos. En el celular se abre el menú de compartir con el archivo; si eso no entrega el archivo, se descarga. En la compu se descarga directo.
- **Excel** (Exportar Excel), si hay sesión de Google, primero sincroniza con Drive y después arma un libro con una hoja por cada pestaña visible. Las ocultas no entran en esa descarga. El `checklist.xlsx` de Drive sigue teniendo todas.

## Cómo sincroniza
- Los datos se guardan siempre primero en el dispositivo (localStorage + fotos en IndexedDB).
- Se sincroniza al iniciar sesión, al tocar **Sincronizar ahora**, al volver a la app, y ~2 s después de cada cambio.
- Si solo cambió la app → sube el Excel. Si solo cambió el Excel en Drive → lo descarga. Si cambiaron los dos → pregunta: **Aceptar** = subir este dispositivo, **Cancelar** = descargar el Excel de Drive.
- La primera vez que se vincula un dispositivo que ya tiene tareas, también pregunta antes de pisar algo.

## Qué hace falta
Solo un **Client ID de Google** (gratis). Ver [SETUP-GOOGLE.md](SETUP-GOOGLE.md). Resumen:
1. Proyecto gratis en <https://console.cloud.google.com/> y habilitar **Google Drive API**.
2. Pantalla de consentimiento OAuth **Externa**, en modo **Prueba**, con los Gmail de los ingenieros como usuarios de prueba.
3. Credencial **ID de cliente OAuth → Aplicación web**, con orígenes `http://localhost:8765` y luego `https://monticof-commits.github.io`.
4. Scope: `https://www.googleapis.com/auth/drive.file` (la app solo ve los archivos que crea).
5. Pegar el ID en `index.html`: `var GOOGLE_CLIENT_ID = '';`

Sin Client ID la app funciona completa en el dispositivo (incluye PDF, Excel, CSV y WhatsApp); solo el panel de Google Drive muestra los pasos de configuración.

## Respaldo
La app anterior (Supabase) sigue publicada y no se reemplaza:

- Sitio: <https://monticof-commits.github.io/Checklists/>
- Repo: <https://github.com/monticof-commits/Checklists>

Si esta versión con Google Drive falla, usá ese sitio. El origen de Google sigue siendo `https://monticof-commits.github.io` (sin la ruta), así que el mismo Client ID sirve para los dos.

## Probar en la compu
```bash
python3 -m http.server 8765
```
y abrí <http://localhost:8765>.

## Sin suscripciones
No necesita Grok Bot, Spacefast ni ningún servicio pago. Es una página estática (HTML/CSS/JavaScript) que usa Google Identity Services y la API de Google Drive con la cuenta gratuita de cada usuario.

## Limitaciones del prototipo
- El acceso de Google dura ~1 hora; después hay que tocar **Sincronizar ahora** para reconectar (no se guarda sesión permanente, por seguridad).
- Si se edita el Excel, conviene abrirlo en Excel (Drive para escritorio) o en Google Sheets en modo compatible con Office, sin convertirlo a Hoja de Google; una copia convertida no la lee la app.
- Las fotos agregadas a mano en la carpeta `fotos/` no se ven (el permiso `drive.file` solo da acceso a lo que crea la app). Borrar una foto en la app no la borra de Drive.
- El conflicto se resuelve por archivo entero (este dispositivo o Drive), no fila por fila.
- Las fotos se suben comprimidas (máx. 1280 px), igual que hoy.
