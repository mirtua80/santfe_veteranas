# Santfe Veteranes

App del equipo de veteranas del **Santfeliuenc FC** — Liga FFV, Grupo 2, temporada **2026-2027**.

Un solo archivo `index.html` (HTML + CSS + JavaScript). Sin build ni `npm`. El backend es **Firebase** (Authentication + Firestore).

**Web publicada:** https://mirtua80.github.io/santfe_veteranas/

---

## 1. Índice

- [Qué puede hacer cada una](#2-qué-puede-hacer-cada-una)
- [Roles](#3-roles)
- [Stack](#4-stack)
- [Colecciones de Firestore](#5-colecciones-de-firestore)
- [Puesta en marcha](#6-puesta-en-marcha)
- [Reglas de seguridad](#7-reglas-de-seguridad)
- [Avisos importantes](#8-avisos-importantes)
- [Pendiente](#9-pendiente)

---

## 2. Qué puede hacer cada una

Hay que **entrar con correo y contraseña**. La primera vez se pulsa **Crear cuenta** (no Entrar). La contraseña tiene que tener al menos 6 caracteres. Si se olvida: **He olvidado la contraseña**.

### Inicio

Pensado para el domingo:

- Próximo partido: rival, día, hora, campo, cuenta atrás.
- **Cómo llegar** y **Añadir al móvil** (archivo `.ics` para el calendario del teléfono).
- Disponibilidad: Sí / No / Todavía no lo sé, recuento y **quién viene**.
- Tarjeta del **último partido** con marcador y **enlace a la grabación** (YouTube), si la hay.
- Un aviso destacado, si existe.
- El resto (porra, clasificación, temporada pasada, más partidos) queda detrás de **Ver el resto de la temporada**.

### Calendario

- Partidos del Santfeliuenc (liga y amistosos) y, si se quiere, el grupo entero.
- Filtros, buscador, vista tarjetas o lista.
- En cada ficha del Santfe: disponibilidad, convocatoria (quién ha dicho que sí), grabación si existe, **Cómo llegar** y **Añadir al móvil**.
- La ficha del rival se abre tocando el escudo.
- La convocatoria se cierra **3 días antes** del partido. Solo la admin puede cambiar un voto después.

### Noticias

Apartado propio con **todas** las noticias publicadas. En Inicio solo se ve un avance y el botón para ir a la lista.

### Equipos

- Fichas del grupo (Santfeliuenc primero): campo, dirección, horario, equipación, Instagram y lo que hicieron en 2025-2026.
- Mapa de los campos.
- Plantilla pública **sin DNI ni edad**.

### Porra

Solo partidos de **liga** (jornadas 1-9). Los amistosos no puntúan.

| Puntos | Qué hay que acertar |
| --- | --- |
| 5 | Resultado exacto |
| 3 | Signo (1 / X / 2) con otro marcador |
| +1 | Goles del Santfeliuenc (solo en nuestros partidos) |

Máximo: 6 puntos en partidos del Santfe, 5 en el resto.

- Se vota hasta que empieza el partido.
- Las jugadoras ven los votos de las demás **cuando el partido ya ha empezado**.
- Admin y cuerpo técnico los ven **antes**, para saber quién ha votado.
- El ranking se actualiza al cargar un resultado.

### Administración (rol `admin`)

| Pestaña | Para qué sirve |
| --- | --- |
| Horarios | Crear o editar partidos y amistosos. Un partido **nuevo no se ve** hasta marcar Publicado y guardar. Grabaciones de YouTube. |
| Resultados | Marcadores del grupo, goleadoras, clasificación y recálculo de la porra. |
| Porra | Quién ha votado, qué ha puesto, quién falta. Quitar un voto, puntuar un partido, recalcular todo, rellenar nombres en votos viejos. |
| Plantilla | Alta, edición y baja de jugadoras. |
| Clubes | Fichas y estadística 2025-2026. |
| Disponibilidad | Ver y corregir votos de asistencia. |
| Noticias | Publicar avisos (en Inicio sale un avance; al pulsar se lee entero). |
| Usuarios | Rol, vínculo a una jugadora, activar/desactivar, **borrar**. |
| Ajustes | Auditoría y cargar datos iniciales (sin pisar lo que ya existe). |

La primera admin se marca a mano en Firestore (`users/{uid}.role = "admin"`). A partir de ahí el resto se gestiona en la app.

---

## 3. Roles

| Rol | Calendario, noticias, equipos | Votar asistencia | Porra | Ver nombres de la convocatoria y de la porra (antes del partido) | Admin |
| --- | :---: | :---: | :---: | :---: | :---: |
| `public` | ❌ (hace falta entrar) | ❌ | ❌ | ❌ | ❌ |
| `player` | ✅ | ✅ | ✅ | Convocatoria sí; porra de las demás solo cuando empieza el partido | ❌ |
| `coach` | ✅ | ✅ | ✅ | ✅ | ❌ |
| `admin` | ✅ | ✅ | ✅ | ✅ | ✅ |

Toda cuenta nueva nace como `player`.

---

## 4. Stack

- Un único `index.html`.
- Firebase JS **v11** (CDN): Auth (correo/contraseña) y Firestore en tiempo real (`onSnapshot`).
- Leaflet 1.9.4 — mapa.
- Lucide — iconos.
- Fuente Inter (Google Fonts).
- JavaScript vanilla (`type="module"`).

Se publica en **GitHub Pages** o cualquier hosting estático. El dominio tiene que estar autorizado en Firebase → Authentication → Authorized domains.

---

## 5. Colecciones de Firestore

| Colección | Qué guarda | Quién lee |
| --- | --- | --- |
| `users` | Perfil, rol, alias, jugadora vinculada | Uno mismo, coach y admin |
| `players` | Plantilla (la interfaz pública oculta DNI y edad; ver avisos) | Lectura abierta |
| `playerPrivate` | Pensada para datos sensibles (aún no se usa en el modelo) | Solo admin |
| `clubs` | Fichas y campos | Lectura abierta |
| `matches` | Calendario, resultado, vídeos, publicado / borrador | Quien ha iniciado sesión |
| `attendance` | Voto de asistencia + alias | Quien ha iniciado sesión |
| `goals` | Goleadoras | Lectura abierta |
| `predictions` | Pronósticos + alias + puntos | Quien ha iniciado sesión |
| `announcements` | Noticias | Quien ha iniciado sesión |
| `auditLogs` | Cambios de admin | Solo admin |

---

## 6. Puesta en marcha

1. El `index.html` ya tiene el `firebaseConfig` del proyecto `santfe-veteranes`.
2. En Firebase Console:
   - Authentication → correo y contraseña.
   - Firestore → modo producción.
3. Copia las reglas del comentario final del `index.html` en **Firestore → Reglas** y publícalas. **No se publican solas.**
   - `matches`, `attendance`, `predictions` y `announcements` deben poder **leerse con sesión iniciada** (listas enteras). Si la regla exige `published == true` documento a documento, las jugadoras no ven amistosos, grabaciones ni noticias.
4. Primera admin: crea la cuenta en la app y en Firestore cambia `role` a `"admin"`.
5. Sube `index.html` (y este README) a GitHub Pages. Tras un cambio, recarga fuerte (`Ctrl + F5`); Pages cachea unos 10 minutos.

### Crear cuenta (móvil)

En el teléfono el teclado a menudo tapa **Crear cuenta**. Hay que **bajar** y pulsar ese botón, no Entrar. Si el correo ya existe: Entrar o recuperar contraseña. Hasta que no entre una vez, **no sale en Admin → Usuarios**.

---

## 7. Reglas de seguridad

La propuesta está al final de `index.html` (comentario HTML). Hay que pegarla en la consola de Firebase.

Puntos que no se pueden saltar:

- Lectura de `matches` / `attendance` / `announcements` / `predictions`: `allow read: if signedIn();`
- Escritura de partidos, noticias y plantilla: solo admin.
- Borrar usuarias: `allow delete: if isAdmin();` en `users`.

Borrar una usuaria en la app **quita su ficha** de Firestore. No borra el login de Firebase. Si vuelve a entrar con el mismo correo, se crea otra vez como jugadora.

---

## 8. Avisos importantes

1. Los DNI de la plantilla semilla son **inventados**. Corregirlos o vaciarlos en Admin → Plantilla.
2. La pantalla pública no pinta DNI ni edad, pero la regla actual de `players` es `allow read: if true`. Quien tenga la config de Firebase podría leer el documento entero. Lo recomendable es pasar `dni`, `birthDate`, `phone` y `email` a `playerPrivate/{id}`.
3. Las claves de `firebaseConfig` no son secretas. La seguridad son las reglas, no esconder ese objeto.

---

## 9. Pendiente

- Mover datos sensibles a `playerPrivate`.
- Sustituir o vaciar los DNI de semilla.
- PWA (icono en el móvil): hay un comentario en el `<head>`, aún no está hecha.
- Las reglas se publican a mano en Firebase; este repo no las despliega.
