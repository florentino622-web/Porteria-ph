# Portería · App Unificada

Una sola app con tres formas de entrar, según quién la use:

- **Portería** — solo puede **consultar** (verificar por cédula o QR) quién trata de ingresar.
  No ve la lista completa ni puede editar nada.
- **Administrador** — acceso completo: verificar, ver todo el registro, y corregir cualquier dato.
- **Propietario** — sin contraseña, solo su número de apartamento. Registra a sus visitantes,
  genera el QR para enviarles, y puede ver/editar/eliminar los que ya registró en "Mis visitantes".

Usa el mismo proyecto de Firebase que ya tenías configurado.

---

## Requisito: actualizar las reglas de Firestore

Esta versión agrega el **Saldo Pendiente** por apartamento (lo carga el Administrador, lo ve
el Propietario solo para consultar). Necesita una regla nueva para la colección `saldos`.
Ve a Firebase console → Firestore Database → pestaña **Rules**, y agrega este bloque dentro de
`match /databases/{database}/documents { ... }`, junto al que ya tenías para `personas`:

```
match /saldos/{apartamento} {
  // Cualquiera que haya iniciado sesión puede leer (portería, admin o propietario)
  allow read: if request.auth != null;

  // Solo portería/administrador (correo y contraseña) puede crear, editar o borrar saldos
  allow write: if request.auth != null
               && request.auth.token.firebase.sign_in_provider == 'password';
}
```

Las reglas completas quedarían así (con las de `personas` que ya tenías, más esta nueva):

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /personas/{cedula} {
      allow read, write: if request.auth != null
                          && request.auth.token.firebase.sign_in_provider == 'password';

      allow read: if request.auth != null
                  && request.auth.token.firebase.sign_in_provider == 'anonymous'
                  && resource.data.tipo == 'visitante';

      allow create: if request.auth != null
                    && request.auth.token.firebase.sign_in_provider == 'anonymous'
                    && request.resource.data.tipo == 'visitante';

      allow update: if request.auth != null
                    && request.auth.token.firebase.sign_in_provider == 'anonymous'
                    && request.resource.data.tipo == 'visitante'
                    && resource.data.tipo == 'visitante';

      allow delete: if request.auth != null
                    && request.auth.token.firebase.sign_in_provider == 'anonymous'
                    && resource.data.tipo == 'visitante';
    }

    match /saldos/{apartamento} {
      allow read: if request.auth != null;
      allow write: if request.auth != null
                   && request.auth.token.firebase.sign_in_provider == 'password';
    }
  }
}
```

Clic en **Publicar**. Sin este cambio, la pestaña "Saldos" del administrador dará error de
permisos al intentar guardar.

Confirma también que en **Authentication → Sign-in method** tengas activados:
- Correo electrónico/contraseña
- Anonymous

(Si ya los tenías activados de las apps anteriores, no hay que hacer nada más aquí.)

## Publicar en GitHub Pages

Puedes reemplazar el repositorio de la app de portería con esta versión (recomendado, para
tener un solo link para todo el mundo), o crear uno nuevo:

1. Sube todos los archivos de esta carpeta con "Add file → Upload files".
2. Settings → Pages → Branch: main / (root).
3. Comparte **un solo link** con todos: porteros, administrador y propietarios. Cada quien
   elige su opción en la primera pantalla.

## Cómo se ve para cada quien

**Portería / Administrador**: pantalla de inicio de sesión con correo y contraseña — la misma
cuenta que ya tenías, o crea una segunda cuenta en Authentication → Users si quieres una
diferente para el administrador.

**Propietario**: sin contraseña. Solo pide el número de apartamento la primera vez, y queda
guardado en ese dispositivo. Tiene dos pestañas:
- **Nuevo**: el formulario para registrar un visitante y generar su QR (igual que antes, ahora
  con foto opcional).
- **Mis visitantes**: la lista de todos los visitantes que ese apartamento ha registrado, con
  botones para editar o eliminar.

Cada pantalla tiene un botón **"← Elegir otra opción"** o **"Salir"** para cambiar de rol o
cerrar sesión en el mismo dispositivo.

## Seguridad — qué puede hacer cada rol

| | Portería / Admin | Propietario |
|---|---|---|
| Ver inquilinos | ✅ | ❌ |
| Ver todos los visitantes | ✅ | ❌ (solo los que él mismo registró, por diseño de la app) |
| Crear/editar/borrar inquilinos | ✅ | ❌ |
| Crear/editar/borrar visitantes | ✅ | ✅ (cualquier visitante, ver nota abajo) |
| Ver fotos | ✅ | Solo las que él mismo subió |

**Nota importante:** el número de apartamento que escribe un propietario no está verificado
contra ninguna identidad real (no hay una cuenta por apartamento). Las reglas de Firestore
protegen que un propietario jamás pueda tocar datos de **inquilinos**, pero, a nivel técnico,
cualquier persona con el link podría —si se lo propusiera— leer o editar visitantes de
**otro** apartamento (la app solo se lo oculta en la lista "Mis visitantes", filtrando por el
apartamento guardado en su propio dispositivo). Para un edificio pequeño y de confianza esto
suele ser suficiente. Si más adelante quieres bloquear esto por completo (que cada propietario
solo pueda tocar su propio apartamento, verificado de verdad), se puede hacer creando una
cuenta de correo/contraseña por apartamento — es un paso más de configuración; avísame si
quieres avanzar a esa versión.

## ¿Qué pasa con las apps anteriores?

Esta app unificada reemplaza a las tres anteriores (`porteria-app-cloud`, `visitantes-inquilinos`).
Puedes seguir usando las viejas si ya las tienes publicadas — comparten la misma base de datos,
así que no hay conflicto — pero ya no hace falta mantener dos links distintos.

---

## Novedades de esta versión (v-unif3)

- **Foto más grande y a la derecha** en las tarjetas de "Registro" y "Mis visitantes".
- **Botón "Aa+"** (arriba, junto a ES/EN) para agrandar el texto en toda la app — útil para
  personas con dificultad visual. Se activa por dispositivo, queda guardado.
- **Varios apartamentos por propietario**: en "cambiar apartamento" ahora se pueden agregar
  varios con el botón "+ Agregar otro apartamento". Si tienen más de uno, el formulario de
  nuevo visitante muestra un selector para elegir a cuál apartamento pertenece cada visita, y
  "Mis visitantes" trae los de todos sus apartamentos juntos.
- **Español / Inglés**: botones ES/EN en cada pantalla. Cubre selección de rol, login,
  configuración de propietario, formulario de visitante, resultados de verificación y los
  botones principales. Algunos mensajes de error poco frecuentes (fallos puntuales de conexión)
  siguen apareciendo solo en español.

No se necesita ningún cambio de configuración de Firebase para esta actualización — solo sube
el `index.html` nuevo.

---

## Corrección v-unif7 — El escáner QR no leía nada

La librería usada para leer el código QR (`jsQR`) estaba cargándose desde una dirección que ya
no existe, así que la cámara se abría pero nunca detectaba nada, sin mostrar ningún aviso de
error. Se cambió a una dirección que sí funciona. Si en el futuro vuelve a fallar la carga, ahora
la app muestra un mensaje claro en pantalla en vez de quedarse escaneando en silencio.

No requiere ningún cambio de Firebase — solo sube el `index.html` nuevo.

---

## Corrección v-unif8 — La cámara abría pero no detectaba el QR

Después de arreglar la carga de la librería, la cámara ya abría bien pero el lector genérico en
JavaScript (jsQR) es poco confiable en condiciones reales (distancia, enfoque, brillo de pantalla).

Se agregó el **lector nativo de códigos del propio navegador** (disponible en la mayoría de
Android con Chrome actualizado), que es mucho más preciso y rápido — jsQR queda solo como
respaldo por si un dispositivo no lo soporta. También se mejoró la calidad de video solicitada
a la cámara y se activa enfoque continuo cuando el dispositivo lo permite.

**Consejos si aún cuesta leer un código:**
- Que el código QR ocupe buena parte del recuadro (ni muy lejos ni pegado a la cámara).
- Buena luz, evitando reflejos si el QR se muestra en otra pantalla.
- Si el QR es de otro celular, aumentar el brillo de esa pantalla ayuda bastante.

No requiere ningún cambio de Firebase — solo sube el `index.html` nuevo.

---

## Cambio importante v-unif9 — Apartamentos y propietarios reales

Este cambio es grande: ahora el **Administrador registra cada apartamento y a su propietario
principal**, con usuario y contraseña reales. Los propietarios ya NO escriben su apartamento
libremente — solo pueden ver y usar los apartamentos que la administración les asignó.

### Qué cambia para cada quien

- **Administrador**: pestaña nueva **"Apartamentos"**. Ahí registra: número de apartamento,
  nombre del propietario, usuario y contraseña. Si vuelve a usar el mismo usuario para un
  segundo apartamento (mismo propietario, dos unidades), se vincula a la misma cuenta — no
  hace falta crear una cuenta nueva por cada apartamento.
- **Propietario**: ya no escribe su número de apartamento — ahora inicia sesión con el
  **usuario y contraseña** que le dio la administración. Después de entrar, solo ve y puede
  registrar visitantes para los apartamentos que tiene asignados.
- **Portería**: sin cambios.

### ⚠️ Paso obligatorio — actualizar las reglas de Firestore

Las reglas cambian bastante porque ahora hay que verificar de verdad quién es cada quien
(antes era más simple, basado solo en si la sesión era anónima o no). Ve a Firebase console →
Firestore Database → pestaña **Rules**, borra todo y pega esto:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    function getRole() {
      return get(/databases/$(database)/documents/roles/$(request.auth.uid)).data.role;
    }
    function isAdmin() { return request.auth != null && getRole() == 'admin'; }
    function isConcierge() { return request.auth != null && getRole() == 'concierge'; }
    function isStaff() { return isAdmin() || isConcierge(); }
    function isOwnerOfApto(apto) {
      return request.auth != null &&
        get(/databases/$(database)/documents/apartamentos/$(apto)).data.uid == request.auth.uid;
    }

    match /roles/{uid} {
      allow read: if request.auth != null && request.auth.uid == uid;
      allow write: if isAdmin();
    }

    match /apartamentos/{numero} {
      allow read: if isStaff() || (request.auth != null && resource.data.uid == request.auth.uid);
      allow write: if isAdmin();
    }

    match /personas/{cedula} {
      allow read, write: if isStaff();

      allow read: if request.auth != null && resource.data.tipo == 'visitante'
                  && isOwnerOfApto(resource.data.apartamento);

      allow create: if request.auth != null && request.resource.data.tipo == 'visitante'
                    && isOwnerOfApto(request.resource.data.apartamento);

      allow update: if request.auth != null && request.resource.data.tipo == 'visitante'
                    && resource.data.tipo == 'visitante'
                    && isOwnerOfApto(resource.data.apartamento);

      allow delete: if request.auth != null && resource.data.tipo == 'visitante'
                    && isOwnerOfApto(resource.data.apartamento);
    }

    match /saldos/{apartamento} {
      allow read: if isStaff() || isOwnerOfApto(apartamento);
      allow write: if isStaff();
    }
  }
}
```

Clic en **Publicar**.

### ⚠️ Otro paso obligatorio — migrar tus cuentas de portería/administrador ya existentes

Las reglas nuevas verifican el rol de cada quien en una colección llamada `roles`, que antes
no existía. Si no haces este paso, **tu cuenta de portería/administrador actual va a perder
acceso** en cuanto publiques las reglas nuevas.

1. Firebase console → Authentication → pestaña **Users**.
2. Busca tu cuenta de portero (o administrador) y copia su **User UID** (una cadena larga de
   letras y números — hay un ícono de copiar al lado).
3. Ve a Firestore Database → pestaña **Data** → **"Start collection"** (o "+ Add collection"
   si ya tienes otras).
4. Nombre de la colección: `roles`
5. ID del documento: pega ahí el UID que copiaste.
6. Agrega un campo: nombre `role`, tipo `string`, valor `admin` (o `concierge` si esa cuenta es
   solo de portería).
7. Guarda. Repite para cada cuenta de portero/administrador que ya tengas.

### Cómo registrar tu primer propietario

1. Entra a la app como Administrador.
2. Pestaña **"Apartamentos"**.
3. Llena: número de apartamento, nombre del propietario, usuario (puede ser el mismo número
   de apartamento, o un nombre corto — lo que sea fácil de recordar) y una contraseña de al
   menos 6 caracteres.
4. Guarda. Comparte ese usuario y contraseña con el propietario (por WhatsApp, en persona, etc.).
5. El propietario entra a la misma app, elige "Propietario", y escribe ese usuario y contraseña.

### Un propietario con más de un apartamento

Simplemente regístralo dos veces en "Apartamentos" (una por cada número de apartamento), usando
el **mismo usuario y la misma contraseña** las dos veces. La app detecta que ese usuario ya
existe y vincula el nuevo apartamento a la misma cuenta — el propietario inicia sesión una sola
vez y ve todos sus apartamentos.

### Limitaciones a tener en cuenta

- Borrar un apartamento desde "Apartamentos" quita el acceso de ese propietario a ESE
  apartamento, pero **no borra su cuenta de acceso** (por seguridad, eso requiere hacerlo desde
  Firebase console → Authentication → Users → buscar y eliminar). Si un propietario tenía solo
  ese apartamento, después de borrarlo su usuario simplemente no podrá entrar a ningún lado
  (sin apartamentos asignados), aunque la cuenta técnicamente siga existiendo hasta que la
  borres allá también.
- No hay un botón de "olvidé mi contraseña" todavía. Si un propietario la olvida, el
  administrador debe:
  1. Firebase console → Authentication → Users → buscar la cuenta de ese usuario → eliminarla
     (el ícono de basura). Esto no borra sus datos de apartamento ni sus visitantes ya
     registrados, solo su acceso.
  2. Volver a la pestaña "Apartamentos" de la app y guardar de nuevo **cada uno** de sus
     apartamentos con el mismo usuario y la contraseña nueva (como se explica arriba en
     "Un propietario con más de un apartamento" — al no existir ya la cuenta vieja, se crea
     una nueva y se vincula).


---

## Novedades v-unif6 — Saldo Pendiente

- El **Administrador** tiene una pestaña nueva, **"Saldos"** (Portería no la ve — solo consulta).
  Ahí escribe el número de apartamento y el monto, y guarda. También puede editar o eliminar
  saldos ya cargados, todo desde una lista debajo del formulario.
- El **Propietario** ve su saldo automáticamente arriba de sus pestañas, marcado como
  **"Solo lectura"** — no lo puede tocar, solo consultarlo. Si tiene varios apartamentos, ve
  el saldo de cada uno por separado.
- Todo se sincroniza en tiempo real: en cuanto el administrador guarda un monto, el propietario
  lo ve sin recargar la página.
- **Requiere la regla nueva de Firestore para la colección `saldos`** — ver la sección de
  "Requisito: actualizar las reglas de Firestore" más arriba. Sin eso, el administrador no va
  a poder guardar saldos (dará error de permisos).

