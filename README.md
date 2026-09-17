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

