# Portería · Control de Acceso — versión con base de datos en la nube (Firebase)

Esta versión sincroniza los registros **en tiempo real** entre todos los dispositivos
(celulares, tablet, laptop) usando Firebase (Google), que es gratis para este tamaño de uso.

---

## Paso 1 — Crear el proyecto de Firebase (una sola vez)

1. Entra a **console.firebase.google.com** con tu cuenta de Google.
2. Clic en **"Crear un proyecto"** (o "Add project").
3. Ponle un nombre, por ejemplo `porteria-ph`. Sigue los pasos (puedes desactivar Google Analytics, no hace falta).
4. Cuando termine de crearse, entras al panel del proyecto.

## Paso 2 — Activar la base de datos (Firestore)

1. En el menú izquierdo: **Compilación (Build) → Firestore Database**.
2. Clic en **"Crear base de datos"**.
3. Elige la ubicación más cercana (por ejemplo `southamerica-east1` o la que te sugiera).
4. Selecciona **"Iniciar en modo de producción"** → Crear.
5. Ve a la pestaña **"Reglas"** dentro de Firestore, borra lo que haya y pega esto:

   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /personas/{cedula} {
         // Portero: acceso total (inicia sesión con correo/contraseña)
         allow read, write: if request.auth != null
                             && request.auth.token.firebase.sign_in_provider == 'password';

         // Inquilinos: solo pueden crear o actualizar visitantes.
         // Nunca pueden leer datos, ni tocar registros de tipo "inquilino".
         allow create: if request.auth != null
                       && request.auth.token.firebase.sign_in_provider == 'anonymous'
                       && request.resource.data.tipo == 'visitante';

         allow update: if request.auth != null
                       && request.auth.token.firebase.sign_in_provider == 'anonymous'
                       && request.resource.data.tipo == 'visitante'
                       && resource.data.tipo == 'visitante';
       }
     }
   }
   ```

6. Clic en **Publicar**.

   Esto dice: el portero (con su correo/contraseña) puede leer y escribir todo. Los inquilinos
   (sin contraseña, "anónimos") solo pueden crear un visitante nuevo o actualizar uno que ya exista
   como visitante — nunca pueden ver la lista completa, ni las fotos, ni tocar los datos de otros inquilinos.

## Paso 3 — Activar los métodos de inicio de sesión

1. Menú izquierdo: **Compilación → Authentication**.
2. Clic en **"Comenzar" / "Get started"**.
3. En "Sign-in method", activa **dos** proveedores:
   - **Correo electrónico/contraseña** → Activar → Guardar. (para el portero)
   - **Anónimo** → Activar → Guardar. (para los inquilinos, no piden contraseña)
4. Ve a la pestaña **"Users" (Usuarios)** → **"Add user"**.
5. Crea el usuario que van a compartir todos los porteros, por ejemplo:
   - Correo: `porteria@tuedificio.com` (no tiene que ser un correo real que exista, solo un identificador)
   - Contraseña: la que tú definas (guárdala bien, la vas a usar en cada dispositivo)

## Paso 4 — Obtener la configuración del proyecto

1. Clic en el ícono de engranaje ⚙️ (arriba, junto a "Project Overview") → **"Configuración del proyecto"**.
2. Baja hasta **"Tus apps"** → clic en el ícono `</>` (Web).
3. Ponle un apodo (ej. "porteria-web") → **Registrar app**. NO actives Firebase Hosting.
4. Te va a mostrar un bloque de código con algo así:

   ```js
   const firebaseConfig = {
     apiKey: "AIzaSy...",
     authDomain: "porteria-ph.firebaseapp.com",
     projectId: "porteria-ph",
     storageBucket: "porteria-ph.appspot.com",
     messagingSenderId: "123456789",
     appId: "1:123456789:web:abc123"
   };
   ```

5. Copia esos 6 valores.

## Paso 5 — Pegar la configuración en el archivo de la app

1. Abre `index.html` (de este paquete) con el Bloc de notas, VS Code, o el editor de GitHub.
2. Busca este bloque cerca del principio del `<script>`:

   ```js
   const firebaseConfig = {
     apiKey: "PEGA_AQUI_TU_API_KEY",
     authDomain: "PEGA_AQUI_TU_PROYECTO.firebaseapp.com",
     projectId: "PEGA_AQUI_TU_PROYECTO",
     storageBucket: "PEGA_AQUI_TU_PROYECTO.appspot.com",
     messagingSenderId: "PEGA_AQUI",
     appId: "PEGA_AQUI"
   };
   ```

3. Reemplaza cada valor `"PEGA_AQUI_..."` por el valor real que copiaste en el Paso 4.
4. Guarda el archivo.

## Paso 6 — Publicar en GitHub Pages

Igual que siempre:
1. Sube todos los archivos de esta carpeta a tu repositorio (Add file → Upload files).
2. Settings → Pages → confirma que esté en main /(root).
3. Espera 1-2 minutos y abre tu link `https://tu-usuario.github.io/tu-repo/`.

## Paso 7 — Iniciar sesión en cada dispositivo

La primera vez que abras la app en cada celular/tablet/laptop, va a pedir **correo y contraseña**
— usa el mismo usuario que creaste en el Paso 3. Después de iniciar sesión una vez, el dispositivo
queda conectado (no hay que volver a escribirlo, a menos que toques "Salir").

A partir de ahí, **cualquier cambio que hagas en un dispositivo (agregar, editar, eliminar, importar)
aparece automáticamente en todos los demás**, en tiempo real.

---

## Costos

Firebase tiene un plan gratis (Spark) con:
- 50,000 lecturas y 20,000 escrituras al día
- 1 GB de almacenamiento

Para el tamaño de un PH normal esto no tiene costo. Si algún día crece mucho, Firebase avisa antes
de cobrar nada.

## Sobre las fotos

Las fotos se guardan comprimidas dentro de cada registro. Si en algún momento ves un error al guardar
una persona con foto, es porque la imagen quedó demasiado pesada — dile a Claude que reduzca más la
compresión de las fotos en el código.

## ¿Y si no quiero usar la nube?

El archivo sigue funcionando exactamente igual sin conexión a Firebase, solo que sin configurar
`firebaseConfig` no va a poder iniciar sesión. Si prefieres volver a la versión simple (datos solo
en el dispositivo, sin login), usa la versión anterior que ya tienes (`porteria-app.zip`, sin Firebase).

---

## App de inquilinos (Visitantes)

Hay una **segunda app**, separada de esta, para que cada inquilino registre a sus propios visitantes
y les genere un código QR. Vive en su propia carpeta/repositorio, conecta al mismo proyecto de Firebase
y ya está configurada con las reglas de este mismo README (Paso 2). No necesita usuario ni contraseña:
cada quien solo escribe su número de apartamento la primera vez.

Instrucciones de publicación para esa app están en su propio README, dentro del paquete
`visitantes-inquilinos.zip`.

