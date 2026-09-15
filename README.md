# Portería · Control de Acceso — PH

App para verificar identidad de inquilinos y visitantes al ingresar al edificio.
Guarda cédula, nombre, apartamento, tipo (inquilino/visitante), fechas de estadía y foto.

Los archivos de esta carpeta son:
- `index.html` — la app
- `manifest.json` — configuración para que se pueda instalar como app
- `sw.js` — permite que funcione sin conexión a internet
- `icon-192.png`, `icon-512.png`, `icon-512-maskable.png` — íconos de la app

La app incluye:
- Registro de inquilinos y visitantes (cédula, nombre, apartamento, fechas de estadía, foto)
- Verificación rápida por cédula
- **Escaneo de código QR** (botón "QR" junto al campo de verificación) para leer la cédula sin escribirla
- Importación masiva desde Excel/CSV

---

## Publicar en GitHub Pages (una sola vez)

1. Entra a **github.com** y crea una cuenta gratis si no tienes.
2. Clic en **New** (o el ícono `+` arriba a la derecha) → **New repository**.
   - Nombre sugerido: `porteria-ph`
   - Marca **Public**
   - Clic en **Create repository**
3. En la página del repositorio, clic en **"uploading an existing file"**.
4. Arrastra **todos los archivos de esta carpeta** (no la carpeta en sí, su contenido: `index.html`, `manifest.json`, `sw.js` y los 3 íconos).
5. Abajo, clic en **Commit changes**.
6. Ve a la pestaña **Settings** → menú lateral **Pages**.
7. En "Branch" elige **main** y carpeta **/ (root)** → **Save**.
8. Espera 1-2 minutos. Tu dirección final va a ser:

   ```
   https://TU-USUARIO.github.io/porteria-ph/
   ```

   (cambia `TU-USUARIO` por tu usuario real de GitHub)

Esta dirección no cambia nunca — puedes guardarla, compartirla con otros porteros, y siempre va a apuntar a la misma app.

---

## Instalar en el celular Android

1. Abre la dirección `https://TU-USUARIO.github.io/porteria-ph/` en **Chrome**.
2. Toca el menú de tres puntos (⋮) arriba a la derecha.
3. Toca **"Instalar aplicación"** (o espera el banner que aparece solo).
4. Confirma. Queda un ícono en la pantalla de inicio, como cualquier app.

Una vez instalada:
- Abre en pantalla completa, sin barra de navegador.
- Funciona sin internet (excepto la función de importar Excel/CSV, que si necesita conexión la primera vez para cargar la librería).
- La cámara y la galería funcionan sin restricciones, porque la página corre bajo `https`.

---

## Actualizar la app más adelante

Si en el futuro Claude te genera una versión nueva de `index.html` (con más funciones), solo tienes que:
1. Entrar al repositorio en GitHub.
2. Abrir el archivo `index.html` → ícono de lápiz (editar) → pegar el contenido nuevo → **Commit changes**.
   - O simplemente subir el archivo nuevo con el mismo nombre, para que reemplace al anterior.
3. Los cambios se publican solos en 1-2 minutos, en la misma dirección de siempre.

---

## Dónde viven los datos

Los registros (personas, fotos, fechas) se guardan en el navegador del celular donde se instaló la app — no en la nube ni en GitHub. Si desinstalas la app o borras los datos de navegación de Chrome, el registro se pierde. Para varios celulares/porterías compartiendo la misma base, se necesitaría una versión con una base de datos central (backend), que es un desarrollo aparte.
