# 3dF Checklist — prototipo con Google Drive

## Qué es
Es la app de checklist de obra (diseño blanco/negro, varias obras, tabla en compu y tarjetas en celular, fotos, exportar PDF / Excel / CSV y dashboard mensual). La sincronización **no usa Supabase**: cada ingeniero inicia sesión con **su propia cuenta de Google** y sus datos quedan en **su Google Drive** (15 GB gratis).

Se sigue editando todo **desde la app**, también en el celular y sin conexión. El Excel en Drive es la copia en la nube: los cambios de la app se suben, y si alguien edita el Excel, la app trae esos cambios al sincronizar.

## Cómo queda en Drive
```
Mi unidad/
└── 3dF Checklist/
    ├── Oficial/
    │   └── Planillas/
    │       └── {obra}/
    │           ├── checklist.xlsx
    │           └── fotos/{piso}/{tarea}/*.jpg
    ├── Versiones/
    │   └── {empleado}_{YYYY-MM-DD-HHMM}/
    │       └── Planillas/
    │           └── {obra}/
    │               ├── checklist.xlsx
    │               └── fotos/…
    └── Dashboard Mensual/
```
**Oficial** es la versión oficial. Si la pantalla dice `Viendo: Oficial`, la sincronización escribe solo en `Oficial/Planillas/{obra}/`.

**Versiones** es hermana de Oficial (no vive adentro de Planillas). **Importar información** crea `Versiones/{empleado}_{YYYY-MM-DD-HHMM}/` y deja esa carpeta como versión activa: la sync siguiente sube ahí y no pisa Oficial. **Versión oficial** vuelve el puntero a Oficial y recarga esa copia; no borra ninguna carpeta de Versiones.

Si el Drive todavía tiene el árbol viejo `3dF Checklist/Planillas/{obra}/` (sin `Oficial/`), la primera sincronización **mueve** esa carpeta a `Oficial/Planillas/` (el id no cambia y no se copia ni se vacía). Si `Oficial/Planillas` ya tiene archivos, el `Planillas` viejo se deja en la raíz sin borrarlo. Un Excel suelto (`3dF Checklist/checklist.xlsx` o `Oficial/Planillas/checklist.xlsx`) se parte por obra en la próxima sincronización **estando en Oficial** y después va a la papelera. La carpeta global `fotos/` también se mueve adentro de cada obra de Oficial y se tira; eso no corre mientras se está viendo una versión importada.

Cada obra tiene su Excel. Cada pestaña de esa obra es su propia hoja (nombre recortado a 31 caracteres y sin `\ / * ? : [ ]`). La hoja **Config** guarda la obra, el id de cada pestaña, el próximo id, si está oculta, si es importante y el nombre de la hoja.

Ocultar una pestaña solo la saca de la barra de esa obra. Cada subida a Drive escribe **todas** las pestañas de la obra. **Mostrar todas** las vuelve a mostrar en esa obra. Si se borra una hoja en el Excel, la próxima sincronización elimina esa pestaña y sus tareas en la app.

Columnas de cada hoja: `id`, `Fecha`, `Tarea`, `Piso`, `Unidad`, `Ambiente`, `Ejecutado` (SI/NO), `Rubro`, `Contratista`, `Ingresado`, `FotosIds`, `TabId` (oculta), `Plazo`, `Importante`, `CompletadoEn` y `CreadoEn`. Para agregar una tarea desde el Excel alcanza con completar una fila nueva (sin `id`; la app le asigna uno). No borres la fila de encabezados.

El nombre de cada foto usa la pestaña, la tarea y la fecha y hora de la toma. Si la tarea cambia de nombre, el archivo se renombra en la próxima sincronización. Borrar una foto en la app la manda a la papelera de Drive, sin borrar la tarea.

Los caracteres que no pueden ir en un archivo (`\ / : * ? " < > |`) se reemplazan.

## Exportar
- **PDF** y **CSV** exportan la pestaña visible, con los filtros activos. En el celular se abre el menú de compartir con el archivo; si eso no entrega el archivo, se descarga. En la compu se descarga directo.
- **Excel** (Exportar Excel), si hay sesión de Google, primero sincroniza con Drive y después arma un libro con una hoja por cada pestaña visible. Las ocultas no entran en esa descarga. El `checklist.xlsx` de Drive sigue teniendo todas.
- **Exportar información** arma un ZIP de la obra que está en pantalla (la versión activa): `manifest.json` (schema 1, obra, quién exportó, fecha), `checklist.json`, las fotos en binario y, si el generador de Excel ya cargó, `checklist.xlsx`. Sale del estado local; no hace falta Drive. **Importar información** lee ese ZIP, pide el nombre del empleado si Google no tiene nombre para mostrar, y abre una versión nueva. El id queda en `localStorage` (`3df-gdrive-version-activa`) y se conserva al recargar. Si Drive no está conectado, los datos quedan en el dispositivo con esa versión; al conectar, se crea `Versiones/…` y se sincroniza ahí.

## Cómo sincroniza
- Los datos se guardan siempre primero en el dispositivo (localStorage + fotos en IndexedDB).
- Se sincroniza al iniciar sesión, al tocar **Sincronizar ahora**, al volver a la app, y ~2 s después de cada cambio.
- Cada subida pasa por la versión activa. Con `Viendo: Oficial` el destino es `Oficial/Planillas/`. Con una versión importada el destino es `Versiones/{empleado}_{fecha}/Planillas/`. Si esa carpeta no está dentro de `Versiones`, la subida se cancela.
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
- Las fotos agregadas a mano en Drive no se ven (el permiso `drive.file` solo da acceso a lo que crea la app). Borrar una foto en la app la manda a la papelera de Drive.
- El conflicto se resuelve por archivo entero (este dispositivo o Drive), no fila por fila.
- Las fotos se suben comprimidas (máx. 1280 px), igual que hoy.
