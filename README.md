# Santfe Veteranes

App de gestión del equipo de veteranas del **Santfeliuenc FC** — Liga FFV, Grupo 2, temporada 2026-2027.

Es una **single-page app en un único archivo `index.html`**, sin build ni dependencias instalables: HTML + CSS + JavaScript (ES modules) que corre directamente en el navegador, con **Firebase** (Auth + Firestore + Storage) como backend y un **modo demo** local que funciona sin conexión ni configuración.

---

## 1. Índice

- [Funcionalidades](#2-funcionalidades)
- [Roles y permisos](#3-roles-y-permisos)
- [Stack técnico](#4-stack-técnico)
- [Modelo de datos (Firestore)](#5-modelo-de-datos-firestore)
- [Puesta en marcha](#6-puesta-en-marcha)
- [Modo demo](#7-modo-demo)
- [Reglas de seguridad de Firestore/Storage](#8-reglas-de-seguridad-de-firestorestorage)
- [⚠️ Avisos importantes (leer antes de usar con datos reales)](#9-️-avisos-importantes-leer-antes-de-usar-con-datos-reales)
- [Estructura interna del archivo](#10-estructura-interna-del-archivo)
- [Roadmap / pendiente](#11-roadmap--pendiente)

---

## 2. Funcionalidades

### Inicio
Panel resumen de la temporada (próximo partido, avisos, accesos rápidos).

### Calendario (`Partits`)
- Los 9 partidos de liga del Santfeliuenc, agrupados por jornada.
- Vista en tarjetas o en lista, filtro por jornada y buscador (rival, jornada, campo, municipio).
- Hora y fecha exacta editables desde Admin (el calendario oficial solo da el fin de semana).
- Botón **"Cómo llegar"** al campo del rival (Google Maps) y **exportar a calendario (.ics)** para añadir el partido al calendario del móvil.
- Botón **compartir** (Web Share API, con copia al portapapeles como respaldo).
- **Votación de asistencia**: cada jugadora marca Sí / No / Tal vez para cada partido. **La votación se cierra automáticamente 3 días antes del partido** (`APP.attendanceCloseDays = 3`); pasado ese plazo los botones se deshabilitan salvo para la administradora, que puede corregir una respuesta puntualmente (queda registrado en auditoría).

### Equipos
- Ficha de cada club del grupo (el Santfeliuenc siempre primero), con campo, dirección, horario habitual de partido, equipación y clasificación de la temporada 2025-2026 como referencia.
- Mapa (Leaflet) con la ubicación de todos los campos del grupo.
- Buscador por equipo, municipio, jugadora o campo.
- **Nota**: la ficha pública de plantilla **no** muestra DNI ni edad (solo dorsal, nombre, posición y goles).

### Porra (gamificación)
- **Pronósticos**: cualquier jugadora con sesión iniciada puede predecir el marcador de cada partido del grupo hasta el inicio del partido (se bloquea automáticamente en el saque inicial).
- **Puntuación**:
  - 5 puntos → resultado exacto.
  - 3 puntos → signo correcto (1 / X / 2), marcador distinto.
  - +1 punto extra → si acierta los goles del Santfeliuenc (solo aplica en los partidos del propio equipo).
  - Máximo por partido: 6 puntos (partidos del Santfe) / 5 puntos (resto del grupo).
- **Ranking**: clasificación general o filtrada por jornada, recalculada automáticamente al introducir un resultado desde Admin.

### Administración (solo rol `admin`)
Pestañas:
| Pestaña | Qué gestiona |
|---|---|
| Horarios | Fecha y hora confirmada de cada partido propio |
| Resultados | Marcadores de todo el grupo (alimenta clasificación y porra) + goleadoras |
| Plantilla | Alta, edición y baja de jugadoras (dorsal, posición, rol, **DNI, fecha de nacimiento** — ver aviso de seguridad más abajo) |
| Clubes | Ficha, campo, equipación y clasificación 2025-2026 de cada club |
| Disponibilidad | Consulta y corrige (con auditoría) las respuestas de asistencia |
| Noticias | Avisos que se muestran en Inicio |
| Usuarios | Asignar roles (`public` / `player` / `coach` / `admin`) y vincular cuenta ↔ jugadora |
| Ajustes | Registro de auditoría y herramientas de datos (restablecer, etc.) |

---

## 3. Roles y permisos

| Rol | Ver calendario/resultados/clasificación/ranking | Votar asistencia | Pronosticar | Ver quién ha votado (nominal) | Panel Admin |
|---|:---:|:---:|:---:|:---:|:---:|
| `public` (sin sesión) | ✅ | ❌ | ❌ | ❌ | ❌ |
| `player` | ✅ | ✅ | ✅ | ❌ | ❌ |
| `coach` | ✅ | ✅ | ✅ | ✅ | ❌ |
| `admin` | ✅ | ✅ | ✅ | ✅ | ✅ |

El rol se guarda en `users/{uid}.role` y solo puede cambiarlo una administradora desde **Admin → Usuarios**. Toda cuenta nueva se crea con rol `player` por defecto.

---

## 4. Stack técnico

- **Sin build ni npm**: un único `index.html`, abrible directamente o servible como archivo estático.
- **Firebase v11 (modular, vía CDN `gstatic.com`)**: Authentication (email/contraseña + Google), Firestore (datos en tiempo real vía `onSnapshot`), Storage (fotos).
- **Leaflet 1.9.4** (CDN unpkg) — mapa de campos.
- **Lucide Icons** (CDN unpkg) — iconografía.
- **Google Fonts (Inter)**.
- Vanilla JS con `type="module"`, sin framework. Render por `innerHTML` + delegación de eventos.
- Persistencia local de respaldo en `localStorage` (clave `santfe_veteranes_v1`) para el modo demo y como caché.

No requiere servidor propio: puede alojarse en **Firebase Hosting**, GitHub Pages, o cualquier hosting estático (ya que toda la lógica de backend vive en Firebase).

---

## 5. Modelo de datos (Firestore)

| Colección | Contenido | Lectura pública |
|---|---|---|
| `users` | Perfil, rol, alias, jugadora vinculada | No (solo propio usuario / coach / admin) |
| `players` | Plantilla: dorsal, nombre, posición, foto, goles, **y también DNI/fecha de nacimiento** (ver aviso ⚠️) | Sí, tal como están configuradas las reglas actuales |
| `clubs` | Ficha de cada club: campo, dirección, coordenadas, equipación, clasificación 25-26 | Sí |
| `matches` | Los 9 partidos: jornada, rivales, fecha/hora, campo, resultado, estado | Solo con sesión iniciada |
| `attendance` | Voto de asistencia por partido y usuaria | Solo con sesión iniciada |
| `goals` | Goleadoras por partido | Pública |
| `predictions` | Pronósticos por partido y usuaria, con puntos calculados | Solo con sesión iniciada |
| `announcements` | Noticias/avisos | Solo con sesión iniciada |
| `auditLogs` | Registro de cambios sensibles (ediciones de admin) | Solo admin |

---

## 6. Puesta en marcha

1. **Proyecto Firebase**: el archivo ya incluye una configuración de Firebase (bloque `firebaseConfig` al inicio del `<script type="module">`) apuntando al proyecto `santfe-veteranes`. Si quieres usar tu propio proyecto, sustituye ese objeto por el de **Firebase Console → Configuración del proyecto → tus apps → SDK setup**.
2. **Habilita en Firebase Console**:
   - *Authentication* → método Email/contraseña (y Google, si se quiere usar ese botón).
   - *Firestore Database* → modo producción.
   - *Storage* (opcional, solo si se van a subir fotos de jugadoras/escudos).
3. **Aplica las reglas de seguridad**: al final del `index.html` hay un bloque comentado con la propuesta de reglas de Firestore y Storage. **No se despliegan solas** — hay que copiarlas manualmente en *Firestore → Reglas* y *Storage → Reglas* y publicarlas. Ver también el aviso del punto 9.
4. **Primera administradora**: registra una cuenta desde la app (botón "Crear cuenta") y luego, directamente en Firestore Console, cambia a mano el campo `role` de ese documento en `users/{uid}` a `"admin"`. A partir de ahí ya puedes gestionar el resto de roles desde **Admin → Usuarios**.
5. **Despliegue**: sube el `index.html` a Firebase Hosting (`firebase deploy`) o a cualquier hosting estático. Al ser un solo archivo, también puede abrirse localmente, aunque Auth/Firestore requieren que el dominio esté autorizado en Firebase (*Authentication → Settings → Authorized domains*).

---

## 7. Modo demo

Si Firebase no está configurado, no responde, o falla la conexión, la app cae automáticamente en **modo demostración**:

- Aviso visible en la parte superior ("Modo demostración").
- Los datos se guardan solo en `localStorage` del navegador (no se sincronizan entre dispositivos).
- Login de prueba: **`admin@demo.santfe.cat` / `demo1234`** entra como administradora; cualquier otro correo/contraseña (mín. 6 caracteres) entra como jugadora normal.
- Útil para probar la interfaz sin tocar el proyecto Firebase real.

---

## 8. Reglas de seguridad de Firestore/Storage

La propuesta completa de reglas está documentada como comentario HTML al final del propio `index.html` (colecciones `users`, `players`, `playerPrivate`, `clubs`, `matches`, `attendance`, `goals`, `predictions`, `announcements`, `auditLogs`, más las reglas de Storage). Cópialas a Firebase Console antes de usar la app con datos reales — **sin ellas, Firestore puede quedar en modo de prueba abierto y cualquiera con la config podría leer o escribir datos.**

---

## 9. ⚠️ Avisos importantes (leer antes de usar con datos reales)

1. **Los DNI de la plantilla son datos inventados.** La lista de jugadoras que se cargó como semilla (`buildSeedPlayers()`) incluye un número de DNI por jugadora, pero esos DNI **no fueron proporcionados por el club** — son valores ficticios generados en algún momento del desarrollo. Antes de dar por buena la ficha de cualquier jugadora hay que **corregir o borrar esos DNI manualmente desde Admin → Plantilla**, para no dejar un documento de identidad falso asociado al nombre real de una persona.
2. **El DNI y la fecha de nacimiento no están realmente protegidos a nivel de base de datos todavía.** La interfaz pública oculta esos campos (usa `publicPlayerFields()` para no pintarlos en pantalla), pero la regla de Firestore propuesta para la colección `players` es `allow read: if true`, es decir, **cualquiera que conozca la configuración de Firebase podría leer el documento completo directamente**, DNI y fecha de nacimiento incluidos, sin pasar por la interfaz. El propio código deja una nota para esto: mover `dni`, `birthDate`, `phone` y `email` a una colección separada `playerPrivate/{id}` con `allow read, write: if isAdmin()`, tal como ya aparece en la propuesta de reglas. **Esa separación está descrita pero no implementada en el modelo de datos actual** — es el primer cambio recomendable antes de tratar esto como app de producción con datos sensibles reales.
3. Las claves de `firebaseConfig` visibles en el archivo (`apiKey`, etc.) no son secretas por diseño de Firebase — la seguridad real depende de las reglas de Firestore/Storage del punto 8, no de ocultar ese objeto.

---

## 10. Estructura interna del archivo

El `<script type="module">` está organizado en bloques numerados (buscar `* N. NOMBRE` para saltar directamente):

1. Configuración · 2. Inicialización de Firebase · 3. Estado global · 4. Autenticación · 5. Acceso a Firestore/almacén local · 6. Datos iniciales (seed) · 7. Navegación · 11. Disponibilidad · 12. Resultados y goles · 13. Pronósticos · 14. Ranking · 15. Administración · 16. Campos y mapas / 17. Compartir e ICS · 18. Utilidades · 19. Toasts y modales · 20. Inicialización de la app.

(Los números 8, 9 y 10 no aparecen como bloque propio; su contenido está integrado en las secciones de Navegación/Render de cada vista.)

---

## 11. Roadmap / pendiente

- Separar `dni`/`birthDate`/`phone`/`email` en `playerPrivate/{id}` (ver aviso 9.2) y actualizar las reglas de Firestore en consecuencia.
- Sustituir los DNI ficticios de la plantilla semilla por los reales (o dejarlos en blanco) desde Admin.
- PWA instalable: el `<head>` ya deja un comentario `<!-- PWA futura: añadir manifest.json y service-worker.js -->`, pero ambos archivos aún no existen.
- Las reglas de Firestore/Storage deben aplicarse manualmente en Firebase Console; no se despliegan desde este repositorio.
