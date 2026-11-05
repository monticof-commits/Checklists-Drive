# Configurar el acceso a Google Drive (Client ID)

Se hace una sola vez y es gratis. El Client ID **no es secreto**; no hace falta ningún "client secret".

1. Entrá a <https://console.cloud.google.com/> con una cuenta de Google y creá un proyecto nuevo (por ejemplo `3dF Checklist`). No requiere tarjeta.
2. **APIs y servicios → Biblioteca** → buscá **Google Drive API** → **Habilitar**.
3. **APIs y servicios → Pantalla de consentimiento OAuth** (o "Google Auth Platform → Branding/Público"):
   - Tipo de usuario: **Externo**.
   - Nombre de la app, correo de soporte y correo del desarrollador.
   - Estado de publicación: **Prueba (Testing)**. En **Usuarios de prueba** agregá el Gmail de cada ingeniero (hasta 100).
   - Permisos (scopes): agregá `https://www.googleapis.com/auth/drive.file`.
4. **APIs y servicios → Credenciales → Crear credenciales → ID de cliente de OAuth**:
   - Tipo: **Aplicación web**.
   - **Orígenes de JavaScript autorizados**:
     - `http://localhost:8765` (pruebas locales)
     - `https://monticof-commits.github.io` (GitHub Pages; solo el dominio, sin la ruta `/Checklists/`)
   - No hace falta completar "URI de redireccionamiento".
5. Copiá el **ID de cliente** (termina en `.apps.googleusercontent.com`) y pegalo en `index.html`:

   ```js
   var GOOGLE_CLIENT_ID = 'XXXXXXXXXXXX-xxxxxxxx.apps.googleusercontent.com';
   ```

Scope usado: `https://www.googleapis.com/auth/drive.file` — la app solo puede ver y modificar los archivos que ella misma creó en el Drive de cada usuario (no ve el resto del Drive).

Notas:
- En modo Prueba, Google muestra el aviso "Google no verificó esta app" (se sigue con *Avanzado → Ir a…*), y solo pueden entrar los usuarios de prueba cargados.
- Si servís la app en otro puerto o dominio, agregalo como origen autorizado (los cambios pueden tardar unos minutos).
