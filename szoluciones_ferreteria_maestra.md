# SZoluciones — Ferretería Maestra
### El tornillero unificado que se lee ANTES de arrancar cualquier proyecto, sin importar el stack

> **Qué es esto.** La síntesis de los tres tornilleros de SZoluciones —el de TypeScript (60 tornillos), el de Python (FastAPI, 72 tornillos) y el condensado de universales (32 tornillos)— en UN solo documento. Acá no hay "lo de TS" y "lo de Python": hay **principios** que sobrevivieron a la prueba de aparecer, con sintaxis distinta, en proyectos reales de SZoluciones.

> **Cómo se construyó.** Se leyeron los tres archivos completos. Se verificó cada uno de los 32 universales declarados contra el texto REAL de TS y Python (no contra los números). Se revisaron además los mapeos "Confirma #N" que el tornillero Python hizo de forma *inferencial* (sin haber leído el texto TS). Resultado: varios se confirmaron, algunos se reclasificaron, y aparecieron universales nuevos que un repo había escondido como "Solo TS" o "Solo Python".

> **Cómo leerlo.** Donde diga "en TypeScript hacemos X", un dev Python tiene que entender el principio sin saber TS. Y viceversa. El ejemplo concreto está para anclar la idea, no para que lo copies tal cual.

**38 universales confirmados · 8 candidatos · tensiones documentadas · mapa de dependencias · receta de arranque.**

---

## Nota de verificación (lo que cambió respecto del archivo de universales)

El archivo `szoluciones_tornillos_universales.md` listaba 32 universales tomados **del autoetiquetado del repo TS**. Verificando contra el texto Python real:

- **Se cayeron de "universal confirmado" (no hay principio gemelo en Python):**
  - **TS #8** (catálogo de permisos como strings `recurso.accion` compartido front/back): Python autoriza por **roles gruesos** (`require_rol(Rol.PSICOLOGA)`), no tiene catálogo de permisos finos. → pasa a **Solo TypeScript**.
  - **TS #2** (apps/ vs packages/ en monorepo): el repo Python es un **monolito** (una app FastAPI + un front), no hay packages que emitan tipos. → pasa a **Candidato**.
  - **TS #20** (onDelete Cascade/SetNull): Python no documenta política de borrado en FK; usa **ledgers inmutables** y flags `activo`. → pasa a **Candidato**.
  - **TS #55** (billing de plataforma con 402): Python no tiene capa de suscripción del propio SaaS. → pasa a **Candidato**.
- **Subieron a universal confirmado (el TS los tenía como "Solo TS/proyecto" pero el principio está en ambos):** 401-interceptor en el cliente (TS #27 → U-07), enums tipados para estados (TS #45 → U-11), barrel/fachada de paquete (TS #42 → U-38), defaults de runtime en el arranque (TS #35 → U-29), lectura de entorno tipada (TS #35/#37 → U-30).
- **Los mapeos "Confirma #N" del Python:** la mayoría se confirman. La excepción notable es que Python asumió que TS tenía permisos como universal — y sí los tiene, pero como **catálogo de strings** (TS #8) que Python NO replica. La discrepancia queda registrada como tensión (ver Sección 4).

---

## Índice de universales confirmados

| U | Nombre coloquial | TS # | PY # | Familia |
|---|------------------|------|------|---------|
| U-01 | El tenant sale del token firmado, nunca del cliente | 9, 14 | 6 | Multi-tenant |
| U-02 | Cada query lleva su filtro de tenant, sin excepción | 10, 14, 29 | 6 | Multi-tenant |
| U-03 | Recurso de otro tenant = 404, nunca 403 | 9, 29 | 5 | Multi-tenant |
| U-04 | Fail-closed: sin contexto, negás (no devolvés todo) | 11 | 7, 71 | Multi-tenant |
| U-05 | No confíes solo en el token: revalidá contra la DB | 5 | 7 | Auth |
| U-06 | Ownership guard antes de mutar un id que vino del cliente | 9, 10 | 11 | Multi-tenant |
| U-07 | 401 global en el cliente: limpiá sesión y redirigí | 27 | 13, 34 | Auth |
| U-08 | Molde de tabla: PK opaca + timestamps de auditoría | 18 | 21, 23 | DB |
| U-09 | Dinero con tipo decimal, jamás float | 18 | 22 | DB |
| U-10 | Índices y unicidad tenant-first | 19 | 25 | DB |
| U-11 | Estados y catálogos como enums tipados, no strings mágicos | 45 | 24 | DB |
| U-12 | Credenciales de terceros cifradas en reposo, clave de env | 34, 52 | 26 | Secrets |
| U-13 | Schema de entrada ≠ de salida; la salida nunca es el modelo crudo | 46, 50 | 14, 19 | Contrato |
| U-14 | Validación en capas: estructural en la frontera, negocio en el service | 15, 25 | 16 | Contrato |
| U-15 | El cliente normaliza al contrato y parsea el error del backend | 16 | 20 | Contrato |
| U-16 | La transacción como guardián de la regla de negocio | 13 | 15 | Transacciones |
| U-17 | Lock pesimista para recursos finitos y transiciones (FOR UPDATE) | 13 | 18 | Transacciones |
| U-18 | Error de dominio → status HTTP traducido en el borde | 23 | 17 | Errores |
| U-19 | Stacktrace adentro, mensaje genérico afuera | 23, 24 | 29 | Errores |
| U-20 | Logging estructurado con niveles según gravedad | 23 | 30 | Errores |
| U-21 | Proveedor externo caído no rompe el flujo (degradación) | 26 | 32 | Resiliencia |
| U-22 | Firma de webhook validada timing-safe, fail-closed | 26, 53 | 31 | Dinero |
| U-23 | Idempotencia: repetir sin duplicar, y testeado | 26, 53, 60 | 55 | Dinero |
| U-24 | Job de fondo resiliente: no mata al scheduler | 60 | 33 | Resiliencia |
| U-25 | El .env.example como contrato; el .env real, ignorado | 33 | 44, 45 | Secrets |
| U-26 | Validar config crítica al arranque (fail-fast), rechazar secretos débiles | 41 | 8, 9, 47 | Secrets |
| U-27 | Default seguro y barato; lo caro o riesgoso se enciende a mano | 37 | 46, 72 | Secrets |
| U-28 | Secretos productivos los inyecta la plataforma, no el repo | 36, 41 | 48 | Secrets |
| U-29 | El arranque pone defaults de runtime y migra una sola vez | 35 | 50 | Secrets |
| U-30 | Lectura de entorno tipada, validada y con fallback | 35, 37 | 36 | Secrets |
| U-31 | Tests contra DB real, sin mockear el ORM (solo el IO externo) | 38 | 52, 53, 57 | Tests |
| U-32 | Probar el caso negativo, el contrato y las invariantes | 39 | 55, 51 | Tests |
| U-33 | QA/CI como gate del release | 41 | 57 | Tests |
| U-34 | Un test que actúa como atacante: cruzar tenants DEBE fallar | 40 | 5 | Tests |
| U-35 | Composition root: registro central explícito, módulo auto-instalable | 28, 51 | 1 | Arquitectura |
| U-36 | Route flaca, service gordo | (services/) | 2 | Arquitectura |
| U-37 | SQL parametrizado, nunca concatenado | 58 | 42 | Arquitectura |
| U-38 | Fachada/barrel: el paquete expone una API pública curada | 42 | 3 | Arquitectura |

---

## SECCIÓN 1 — TORNILLOS UNIVERSALES CONFIRMADOS

> Cada uno aparece en AMBOS repos con distinta sintaxis pero el mismo principio. El principio no depende de un lenguaje ni de un framework, y se traduciría a un tercer stack sin cambiar la idea central.

### Familia A — La cadena de seguridad multi-tenant

#### U-01 — El tenant sale del token firmado, nunca del cliente
**(TS #9 / TS #14 · PY #6)**

**En una línea:** El identificador de inquilino (org, cuenta, inquilino, clínica) se toma SIEMPRE del token de sesión ya verificado, jamás de un parámetro, body o header que el cliente controle.

**La regla universal:** El dato que decide "qué datos podés ver" no puede venir de quien quiere verlos. Extraé el `tenant_id` del sujeto autenticado del token y usalo como primer filtro de toda operación. Los ids que sí manda el cliente (un `:id` de ruta, un campo del body) se usan únicamente combinados con `AND tenant_id = <el del token>`. Si el token no trae tenant, ni siquiera tocás la base.

**Cómo se ve en TypeScript:** Cada handler hace `const user = request.user as any` y usa `user.org_id` como primer parámetro del `WHERE`; un middleware (`tenancy.ts`) corta con 401 código `INVALID_TOKEN` si el JWT no trae `org_id`. El `JwtPayload` marca `org_id` con el comentario literal `CRITICAL: multi-tenancy`. Un `:id` de otra org nunca devuelve datos porque siempre va acompañado de `AND organization_id = $org`.

**Cómo se ve en Python:** El tenant es `usuario.id` (el usuario autenticado vía `get_usuario_actual`), nunca un `?tenant_id=` (ese parámetro directamente no existe en la API). El filtro `.filter(Model.psicologa_id == usuario.id)` aparece en ~160 consultas a lo largo de ~36 archivos. Un atacante no puede pasar el id de otra psicóloga porque el scope es implícito y server-side.

**Señal de que lo estás usando bien:** No existe NINGUNA query de negocio donde el tenant provenga del request. Buscás el nombre del tenant en una ruta y todas las ocurrencias apuntan al usuario del token. Cambiar el id en la URL por uno de otro tenant no filtra nada.

---

#### U-02 — Cada query lleva su filtro de tenant, sin excepción
**(TS #10 / TS #14 / TS #29 · PY #6)**

**En una línea:** Toda consulta a una tabla con datos de inquilino —SELECT, INSERT, UPDATE, DELETE, y también los COUNT y agregaciones de reportes— incluye el filtro de tenant, como disciplina manual omnipresente.

**La regla universal:** Si tu capa de datos no garantiza el filtro automáticamente, el filtro manual no es opcional: es la única barrera real. No confíes en que un id sea único globalmente; combinalo siempre con el tenant. Propagá el `tenant_id` también a las funciones de servicio (como parámetro explícito), no lo dejes implícito. Tratá un endpoint sin ese filtro como un bug de seguridad, no como un detalle de performance.

**Cómo se ve en TypeScript:** Como los routes corren sobre un `Pool` de `pg` crudo (no Prisma) y la RLS de Postgres no se activa en esas conexiones, el Senior compensa filtrando a mano: hasta los `DELETE FROM member_notes` exigen `AND organization_id = $3`. El tenant se propaga a los services explícitamente (`restoreReservationAccess(organizationId, ...)`). Un grep de `organization_id` da más de 400 ocurrencias en 23 archivos.

**Cómo se ve en Python:** El mismo `psicologa_id` lidera el WHERE incluso en los conteos: `db.query(func.count(Cliente.id)).filter(Cliente.psicologa_id == psicologa_id, ...)`. Hay >1400 referencias a `psicologa_id` en `app/`. Los reportes agregados también están scopeados, no solo los GET por id.

**Señal de que lo estás usando bien:** Un grep del filtro de tenant devuelve una ocurrencia por cada operación de DB. No hay un solo SELECT/UPDATE/DELETE de negocio que lo omita, ni siquiera en reportes.

---

#### U-03 — Recurso de otro tenant = 404, nunca 403
**(TS #9 / TS #29 · PY #5)**

**En una línea:** Un id válido pero de otro inquilino "no existe" para vos: misma respuesta que un id inexistente, sin confirmar jamás la existencia del dato ajeno.

**La regla universal:** No separes "¿existe?" de "¿es tuyo?". Filtrá por `id AND tenant_id` en una sola query y devolvé 404 si no aparece. Un 403 ("esto existe pero no es tuyo") ya filtra información: confirma que el recurso existe. Encapsulá la dueñez en un helper que devuelva el objeto ya autorizado.

**Cómo se ve en TypeScript:** Los UPDATE/DELETE llevan `organization_id` en el WHERE además del id, y si no matchea ninguna fila se responde `reply.code(404).send({ message: 'Sede no encontrada' })`. Pedir un id de otra org da 404 idéntico al de un id inventado.

**Cómo se ve en Python:** Centralizado en helpers `obtener_*_de_psicologa` que filtran por `id AND psicologa_id` en la misma query y lanzan `HTTPException(404)` si no aparece. Los 5 helpers (cliente, turno, disponibilidad, transacción, conversación) usan el patrón idéntico. No hay un 403 "no es tuyo" en ningún lado.

**Señal de que lo estás usando bien:** Pedir un id real de otro tenant devuelve exactamente el mismo 404 que un id que no existe; ningún endpoint hace `.filter(id==x).first()` y después chequea dueñez aparte.

---

#### U-04 — Fail-closed: sin contexto, negás (no devolvés todo)
**(TS #11 · PY #7 / PY #71)**

**En una línea:** Ante la ausencia o la duda de contexto de inquilino, el sistema deniega o devuelve vacío — nunca "todo". El default seguro es negar.

**La regla universal:** Diseñá el aislamiento de forma que un fallo a medias produzca cero resultados o un rechazo explícito, jamás acceso total. La pregunta de diseño es siempre: "si esto falla a la mitad, ¿se filtran datos o se niegan?" — la respuesta correcta es "se niegan". Esto aplica también al ruteo: si un canal compartido (webhook, número, dominio) no resuelve a un único tenant, cortá; no adivines ni caigas a un default.

**Cómo se ve en TypeScript:** El principio está escrito en el middleware (`Security principle: Fail closed`): si el JWT no trae `org_id`, 401 antes de cualquier query. La idea de diseño es "tenant indefinido nunca se traduce en sin filtro".

**Cómo se ve en Python:** `get_usuario_actual` revalida contra DB y un usuario inexistente o inactivo devuelve el MISMO 401 (sin distinguir el caso). Y el webhook de Meta (#71), compartido por todas las psicólogas, resuelve el tenant desde el `phone_number_id` entrante contra las integraciones ACTIVAS; si no matchea una, lanza `HTTPException(400)` en vez de mandar el mensaje a un default. No existe la rama "si no sé de quién es, mandáselo a alguien".

**Señal de que lo estás usando bien:** Quitar el contexto de tenant produce un rechazo o cero filas, nunca una respuesta con datos. Un canal de entrada que no resuelve a un tenant único no procesa nada.

---

#### U-05 — No confíes solo en el token: revalidá contra la DB
**(TS #5 · PY #7)**

**En una línea:** El token dice quién sos; la base de datos, en cada request, dice si seguís siendo válido y qué podés hacer ahora.

**La regla universal:** Meté en el token solo identidad estable. Para autorizar (o incluso para confirmar que la cuenta sigue viva), releé el estado vivo desde la DB en un hook/middleware en cada request sensible, filtrando por sujeto activo y no borrado. Así un permiso revocado o una cuenta desactivada pega al instante, sin esperar a que expire el token (revocación efectiva sin lista negra).

**Cómo se ve en TypeScript:** El JWT incluye un array de `permissions`, pero el hook `requirePermission` lo IGNORA y vuelve a consultar la DB (`organization_users JOIN roles JOIN users WHERE is_active = true AND deleted_at IS NULL`) para traer los permisos vivos. Si le sacás un permiso a alguien, su próximo request ya recibe 403 aunque su access token siga vigente.

**Cómo se ve en Python:** Tras decodificar el token, `get_usuario_actual` valida contra DB que el usuario exista y esté activo (`if not cast(bool, usuario.activo): raise HTTPException(401)`). Desactivar un usuario invalida sus tokens vigentes al instante, sin blacklist.

**Señal de que lo estás usando bien:** Cambiar permisos o desactivar una cuenta en la DB se refleja en el siguiente request, no cuando expira el token. Si tenés que esperar a la expiración, lo hiciste mal.

---

#### U-06 — Ownership guard antes de mutar un id que vino del cliente
**(TS #9 / TS #10 · PY #11)**

**En una línea:** Cuando un id de recurso llega por body o query y vas a operar sobre él, validás la dueñez con el helper de tenant ANTES de la lógica de negocio, aunque no uses el objeto devuelto.

**La regla universal:** No asumas que un id en el payload pertenece al tenant del usuario. Antes de cobrar, transaccionar o mutar contra un id que mandó el cliente, pasalo por el guard de dueñez (que corta con 404 si es de otro tenant). El efecto importante es la autorización + 404, no el objeto.

**Cómo se ve en TypeScript:** La regla está en la disciplina de #9/#10: los ids del cliente "se usan solo como filtro adicional, siempre combinados con `AND tenant_id`". Un `memberId` o `classId` que venga en el body solo resuelve dentro del scope del org del token.

**Cómo se ve en Python:** Explícito como guard: `obtener_cliente_de_psicologa(db, datos.cliente_id, usuario.id)` se invoca antes de `procesar_pago_parcial` aunque a veces se descarte el return. Repetido en `transacciones.py` y `turnos.py`. Cobrar contra un cliente de otra psicóloga da 404, no un error de negocio más adentro.

**Señal de que lo estás usando bien:** Operar contra un id de otro tenant corta con 404 al principio del handler, no revienta a mitad de la lógica de negocio.

---

#### U-07 — 401 global en el cliente: limpiá sesión y redirigí
**(TS #27 · PY #13 / PY #34)**

**En una línea:** Un interceptor único en el cliente convierte cualquier 401 del backend en logout + redirect al login, en vez de repetir ese manejo en cada llamada.

**La regla universal:** El backend es la autoridad. En el front tratá el token como opaco para autorizar (decodificalo solo para UX: rol, expiración) y centralizá el manejo de 401 en un interceptor que limpie la sesión y redirija. No dupliques ese catch por endpoint.

**Cómo se ve en TypeScript:** El interceptor (axios en mobile, wrapper de fetch en web) ante un 401 intenta un refresh único deduplicado y, si falla, hace `clearAuth()` + redirect a `/login` con un mensaje humano ("Sesión expirada"). El detalle del refresh con deduplicación es un idiom propio de TS; el núcleo —401 centralizado → logout— es lo universal.

**Cómo se ve en Python (front React):** `client.js` tiene `if (error.response?.status === 401) { clearAuth(); window.location.href = "/login"; }` en un solo interceptor; `hasValidSession()` valida `exp` localmente solo para gating de navegación, no de datos. Un `AppErrorBoundary` cubre además los crashes de render.

**Señal de que lo estás usando bien:** Un 401 desde cualquier llamada termina siempre en logout+redirect sin código ad-hoc por endpoint; la validación de rol del front es solo para mostrar/ocultar, nunca la barrera real.

---

### Familia B — El archivo del negocio (diseño de datos)

#### U-08 — Molde de tabla: PK opaca + timestamps de auditoría
**(TS #18 · PY #21 / PY #23)**

**En una línea:** Cada tabla de negocio nace del mismo esqueleto: clave primaria opaca generada en la app y la tripleta de auditoría, no se improvisa modelo por modelo.

**La regla universal:** Definí un molde estándar y aplicalo sin excepción: PK opaca (uuid/cuid generado en la aplicación, nunca un entero autoincremental en un sistema multi-tenant) y marcas de tiempo `created_at`/`updated_at`. Antes de agregar una tabla, copiás el molde y después sumás los campos específicos. (Variante deliberada: para tablas tipo *ledger* inmutable, omitís `updated_at` a propósito — la ausencia documenta la intención.)

**Cómo se ve en TypeScript:** Patrón fijo en todos los modelos Prisma: `id String @id @default(cuid())`, `organization_id String`, y `created_at`/`updated_at`/`deleted_at` (con soft-delete). Idéntico en Organization, User, Facility, Class, Reservation, Payment.

**Cómo se ve en Python:** `id = Column(String, primary_key=True, index=True)` con uuid str generado en el service, idéntico en los ~25 modelos. `created_at` con default y `updated_at` con `default + onupdate`, apuntando a un callable de tiempo compartido (`utc_now_naive`). Los ledgers inmutables (`PagoRecibido`, `EventoFinanciero`, `AjusteContable`) solo tienen `created_at`.

**Señal de que lo estás usando bien:** Abrís cualquier modelo nuevo y encontrás las mismas columnas base; ninguna tabla de negocio carece de PK opaca ni de timestamps. (Diferencia entre repos sobre el borrado: ver U-08 vs Tensión T-1.)

---

#### U-09 — Dinero con tipo decimal, jamás float
**(TS #18 · PY #22)**

**En una línea:** Los montos se modelan con un tipo decimal de precisión y escala explícitas; los defaults y constantes monetarias también son decimales, nunca literales float.

**La regla universal:** El dinero en binario flotante acumula errores de redondeo que en finanzas son inaceptables. Usá el tipo decimal del motor (precisión 10-14, escala 2 para moneda; escala mayor para tasas de cambio) y escribí los defaults como decimal-desde-string, no como número flotante. Prohibido `float` en cualquier columna de dinero.

**Cómo se ve en TypeScript:** `amount Decimal @db.Decimal(10, 2)` en el schema Prisma; "El dinero NUNCA es Float" es regla explícita del molde de tabla.

**Cómo se ve en Python:** `Column(Numeric(precision=12, scale=2))` para montos, `Numeric(14,2)` para acumulados, `Numeric(_, scale=6)` para tasas; los defaults se escriben `Decimal("1.0")`, jamás `1.0`.

**Señal de que lo estás usando bien:** No aparece un solo `Float`/`number` en columnas de dinero; los defaults monetarios son decimales construidos desde string.

---

#### U-10 — Índices y unicidad tenant-first
**(TS #19 · PY #25)**

**En una línea:** Indexás a mano cada FK, la columna de tenant y las columnas de filtro frecuente; y cada unicidad de negocio empieza por el tenant.

**La regla universal:** No dejes que el motor adivine los índices. Indexá toda FK, la columna de tenant y lo que aparezca seguido en WHERE/ORDER BY. Convertí cada invariante "no puede haber dos X iguales" en un unique compuesto en la base, y hacelo **por tenant** (teléfono/email únicos por inquilino, no globalmente) poniendo el `tenant_id` como primera columna del índice/constraint.

**Cómo se ve en TypeScript:** `@@unique([user_id, class_id, reservation_date])`, `@@unique([organization_id, name])`, `@@index([organization_id])`, `@@index([deleted_at])` declarados explícitamente en cada modelo.

**Cómo se ve en Python:** `__table_args__` con `UniqueConstraint("psicologa_id","telefono", name="uq_cliente_psicologa_telefono")` e `Index("idx_psicologa_fecha","psicologa_id","fecha_inicio")` — siempre liderando con `psicologa_id`. Convención de nombres `idx_<tabla>_<cols>` / `uq_<tabla>_<cols>`.

**Señal de que lo estás usando bien:** No hay FK sin su índice; toda unicidad de negocio lidera con el tenant; no existen unicidades globales de teléfono/email que romperían el multi-tenant.

---

#### U-11 — Estados y catálogos como enums tipados, no strings mágicos
**(TS #45 · PY #24)**

**En una línea:** Cada conjunto cerrado de estados/tipos es un enum tipado, reusado en la capa de datos, la validación y la lógica — nunca strings sueltos repetidos.

**La regla universal:** Modelá los catálogos cerrados (estados, métodos, roles) como un tipo enumerado fuerte y reusalo en todas las capas. Cambiar o renombrar un miembro debe tener un solo lugar obvio donde tocarlo, y el resto del código habla ese vocabulario exacto con autocompletado y chequeo.

**Cómo se ve en TypeScript:** Por cada enum del schema Prisma (`ReservationStatus`, `PaymentMethod`, etc.) hay un union type idéntico en `shared/types.ts` (`'PENDING' | 'CONFIRMED' | ...`), así el front habla el vocabulario de la DB sin importar el cliente de Prisma. Esos literales se reusan en los `z.enum` de validación.

**Cómo se ve en Python:** `class Rol(str, Enum)`, `class EstadoTurno(str, Enum)` (~16 enums) montados con `Column(SQLEnum(X))`, reusados en `models.py`, `schemas.py` y la lógica. Donde el catálogo debe evolucionar sin `ALTER TYPE`, usa `String` guardando `Enum.MIEMBRO.value`.

**Señal de que lo estás usando bien:** No hay strings mágicos sueltos para estados; el mismo enum aparece en datos, validación y negocio. (Ojo: en ninguno de los dos hay verificación automática de sincronía entre capas; es disciplina.)

---

#### U-12 — Credenciales de terceros cifradas en reposo, clave de env
**(TS #34 / TS #52 · PY #26)**

**En una línea:** Las credenciales sensibles de cada inquilino se guardan cifradas con cifrado autenticado y una clave derivada de una variable de entorno; nunca en texto plano.

**La regla universal:** Centralizá encrypt/decrypt en UN módulo, usá cifrado autenticado (que dé confidencialidad + integridad), derivá la clave de un secreto de entorno dedicado, y fallá si la clave no está. Nombrá las columnas para que se note que están cifradas y dejá en claro solo lo que no es secreto (ej: una public key). El código de negocio trabaja con texto plano sin saber que está cifrado en reposo.

**Cómo se ve en TypeScript:** `encryptSecret`/`decryptSecret` en `payment-settings.ts`: clave por SHA-256 de `PAYMENT_SETTINGS_SECRET` (con cascada de fallbacks), AES-256-GCM, IV aleatorio de 12 bytes, y formato versionado `v1:iv:tag:cipher`. El prefijo `v1:` permite rotar el esquema; valores sin prefijo se tratan como legacy. Columnas `access_token_encrypted`.

**Cómo se ve en Python:** La columna persistida es `X_encriptado = Column(Text)` y un `@hybrid_property` cifra al escribir y descifra al leer con Fernet (`ENCRYPTION_KEY` del env), con `.expression` para filtrar en SQL si hace falta. `get_cipher` exige la clave o falla. Repetido en `IntegracionMeta`, `IntegracionMercadoPago`, `IntegracionMobbex`.

**Señal de que lo estás usando bien:** Si abrís la tabla ves strings ilegibles, nunca el token real; rotar el secreto maestro invalida (no expone) los datos viejos; el código de negocio nunca toca la columna cruda. (Diferencia de robustez en rotación: ver Tensión T-5.)

---

### Familia C — El viaje de un dato (contratos y validación)

#### U-13 — Schema de entrada ≠ de salida; la salida nunca es el modelo crudo
**(TS #46 / TS #50 · PY #14 / PY #19)**

**En una línea:** Cada recurso tiene schemas de entrada (`Create`/`Update`, solo lo que el cliente controla) separados del de salida (`Response`, con los campos derivados y el id), y la respuesta nunca es el objeto crudo de la base.

**La regla universal:** Nunca reutilices un mismo modelo para request y response. El schema de entrada no expone campos que el backend calcula (estado, tarifa base, timestamps): así evitás por construcción el *mass-assignment* y la fuga de campos derivados. La salida pasa siempre por un contrato que decide qué se serializa.

**Cómo se ve en TypeScript:** Los inputs se validan con schemas Zod compartidos (`CreateXSchema`, `UpdateXSchema = CreateXSchema.partial()`); la salida tiene su propio contrato (`ApiResponse<T>`, tipos de entidad en `shared`). El tipo se deriva del schema con `z.infer`, una sola fuente de verdad.

**Cómo se ve en Python:** `TurnoCreate` (lo que el cliente manda) está separado de `TurnoResponse` (id, estado, `tarifa_base`, `origen_politica`, timestamps); el `Response` hereda de un Base con `from_attributes` para leer del ORM. El endpoint declara `response_model=TurnoResponse` y devuelve el objeto ORM: Pydantic decide qué sale y un encoder global normaliza datetime a UTC. La salida nunca es un dict armado a mano.

**Señal de que lo estás usando bien:** El body tipado es un `*Create`/`*Update` que NO tiene id ni campos derivados; la respuesta es un `*Response`/envelope que sí. Ningún handler arma dicts crudos de la DB para la respuesta normal.

---

#### U-14 — Validación en capas: estructural en la frontera, negocio en el service
**(TS #15 / TS #25 · PY #16)**

**En una línea:** La forma, los rangos y la coherencia entre campos se validan en la frontera del handler antes de tocar la base; las reglas que necesitan la DB (slot libre, transición válida, unicidad real) viven en el service.

**La regla universal:** Dos capas que no se mezclan. En la frontera, validá todo lo verificable sin DB con un validador de esquema que NO lance excepción (para que el error de validación sea un caso de negocio, no del sistema) y cortá con un 400 estructurado por campo. Lo que requiere consultar la base lo decide el service. Las reglas de validación de la frontera no reciben acceso a la DB.

**Cómo se ve en TypeScript:** `Schema.safeParse(request.body)`; si `!parsed.success`, `reply.code(400).send({ message: 'Datos inválidos', errors: parsed.error.flatten().fieldErrors })` antes de cualquier query. Las reglas cross-field (hora fin > inicio, un pack requiere créditos) viven en `refine`/`superRefine` del schema; la disponibilidad real la chequea el service.

**Cómo se ve en Python:** Field constraints (`ge=0`, `le=999999.99`) y `@model_validator(mode="after")` atrapan lo malformado; "ese horario está libre" y "esa transición de estado es válida" viven en services porque necesitan la DB. Los validators de Pydantic nunca reciben `Session`.

**Señal de que lo estás usando bien:** Un body malformado nunca ejecuta una query: siempre 400 con errores por campo. Las reglas que necesitan datos externos están en services y no hay queries dentro de los validators estructurales.

---

#### U-15 — El cliente normaliza al contrato y parsea el error del backend
**(TS #16 · PY #20)**

**En una línea:** El frontend convierte los tipos al contrato del backend antes de enviar y sabe leer la forma de la respuesta y del error, sin acoplarse a un formato exacto frágil.

**La regla universal:** En el cliente, normalizá al contrato antes de enviar (fechas a ISO-UTC, números a número) y validá lo barato para UX, pero tratá al backend como la autoridad. Al recibir, no asumas una única forma: tolerá las variantes de envelope y distinguí los formatos de error (lista de errores de validación vs string).

**Cómo se ve en TypeScript:** Un helper `unwrapList` con tipo `T[] | { data: T[] }` normaliza listas antes de devolverlas del `queryFn`, así el front sobrevive si el endpoint cambia entre array crudo y `{ data }`.

**Cómo se ve en Python (front React):** `toISO()` manda siempre UTC, hay validaciones espejo para UX, y al fallar distingue si `err.response.data.detail` es un array (errores de validación, 422) y lo une, o un string (excepción HTTP) y lo muestra directo.

**Señal de que lo estás usando bien:** Cambiar el envelope del backend no rompe componentes (solo el helper de unwrap conoce la forma); las fechas viajan como ISO-UTC; el manejo de error contempla "es lista o es string".

---

### Familia D — Transacciones y concurrencia

#### U-16 — La transacción como guardián de la regla de negocio
**(TS #13 · PY #15)**

**En una línea:** Toda escritura que toca varias tablas o consume un recurso finito corre dentro de una transacción explícita y hace rollback ante cualquier fallo, sea de la base o de validación de negocio.

**La regla universal:** Una transacción por request, services componibles. Cuando una operación toca más de una tabla o consume un recurso escaso (créditos, cupo, stock), envolvela en una transacción. Diseñá los services para que NO commiteen: mutan y dejan los cambios pendientes; el handler commitea recién cuando todos terminaron, y hace rollback ante un fallo de negocio igual que ante un error de base. Capturá los errores nativos de la DB (ej. violación de unicidad) y traducilos a un status entendible.

**Cómo se ve en TypeScript:** El handler abre un cliente del pool, hace `BEGIN`, y dentro consume el crédito de membresía; si el service devuelve `{ok:false}`, hace `ROLLBACK` manual ANTES de responder con el statusCode de negocio. Captura el código `23505` (unique violation) y lo traduce a 409.

**Cómo se ve en Python:** El service construye el objeto, aplica la política y hace `session.flush()` (NUNCA `commit()`); el handler hace `db.commit()` + `db.refresh()` recién cuando todos los services terminaron. Rodea el `commit()` con `try/except IntegrityError → rollback → HTTPException(400/409)`.

**Señal de que lo estás usando bien:** Buscás `commit` en los services y no aparece (solo en los handlers). Si la validación de negocio falla a mitad de camino, no queda ninguna fila insertada ni recurso gastado.

---

#### U-17 — Lock pesimista para recursos finitos y transiciones (FOR UPDATE)
**(TS #13 · PY #18)**

**En una línea:** Antes de mutar el estado de algo que dispara efectos con dinero o que no debe duplicarse, releés la fila con un lock pesimista para serializar la concurrencia.

**La regla universal:** Dos requests simultáneos no pueden gastar el mismo crédito ni crear la misma obligación dos veces. Cuando una transición de estado dispara efectos financieros o irrepetibles, volvé a leer la fila objetivo con `SELECT ... FOR UPDATE` dentro de la transacción (no reuses la instancia que ya cargó el handler) y reverificá el estado actual antes de aplicar el efecto. La transacción queda abierta hasta el commit.

**Cómo se ve en TypeScript:** `consumeReservationAccess` usa `SELECT ... FOR UPDATE OF mm` para lockear la fila de membresía y evitar que dos reservas concurrentes gasten el mismo crédito.

**Cómo se ve en Python:** Un helper `_obtener_turno_bloqueado` hace `session.query(Turno).filter(...).with_for_update().first()`, usado por `marcar_realizado`, `cancelar_turno` y `reprogramar_turno` para que dos requests no dupliquen la obligación financiera ni la multa.

**Señal de que lo estás usando bien:** Simulás dos operaciones concurrentes sobre el último recurso y solo una lo consume; existe un helper `_obtener_X_bloqueado` con el lock que revalida el estado antes de aplicar el efecto.

---

### Familia E — Cuando algo se rompe (errores, logging, resiliencia)

#### U-18 — Error de dominio → status HTTP traducido en el borde
**(TS #23 · PY #17)**

**En una línea:** El dominio no conoce HTTP: lanza errores de negocio legibles; el borde (handler) los traduce a 400/404/409/422, y reserva el 500 + log para lo inesperado.

**La regla universal:** Separá el error esperable del usuario del error inesperado del sistema. En el service, lanzá un error de dominio con mensaje legible (sin importar nada de HTTP). En el handler, mapeá los errores conocidos a su status y, para todo lo demás, logueá y devolvé un 500 genérico. El `try/catch` local nunca traga un error que no entiende: solo mapea los que conoce.

**Cómo se ve en TypeScript:** Los errores de dominio conocidos se traducen a un status explícito; lo inesperado dentro de una transacción hace `ROLLBACK` y `throw err` para que suba al error handler global de Fastify (donde pino loguea el stack y devuelve 500). El `log.error` aparece recién en el último escalón, nunca para los 4xx de validación.

**Cómo se ve en Python:** Los services lanzan `ValueError("Horario no disponible")` (jamás importan `HTTPException`). El handler captura `ValueError` y elige el status por contexto (`409 if "disponible" in mensaje.lower() else 400`) y captura `Exception` aparte para loguear y devolver 500. Un grep de `HTTPException` no aparece en `services/`.

**Señal de que lo estás usando bien:** Tus logs de error solo contienen fallas reales del sistema, no validaciones fallidas. Mandar basura nunca termina en 500; cae en un 4xx mapeado.

---

#### U-19 — Stacktrace adentro, mensaje genérico afuera
**(TS #23 / TS #24 · PY #29)**

**En una línea:** El error real con traceback va al log; al cliente solo le llega un mensaje corto, uniforme y seguro, nunca el detalle interno.

**La regla universal:** No filtres internals al cliente. Elegí UN solo shape de error de cara al cliente y respetalo en todas las rutas. En un 500, el mensaje es una constante; el `str(error)` y el stack viven solo en el log. Si definís un tipo de envelope de error, usalo en runtime o no lo definas (un tipo que nadie aplica es una mentira que confunde).

**Cómo se ve en TypeScript:** Contrato `{ message }` en español, 182 ocurrencias en 15 archivos; el cliente lee ese campo con un único fallback. Deuda viva honesta: el archivo más viejo (`auth.ts`) quedó con `{ error }` en inglés, y el tipo `ApiResponse` está definido pero no se usa en runtime (el envelope teórico y el real divergieron).

**Cómo se ve en Python:** `except Exception as exc: logger.exception(...); raise HTTPException(500, detail="Error procesando webhook Meta")` — `detail` fijo, sin `str(exc)`. Contraejemplo honesto que el propio tornillero marca: `turnos.py:451` sí filtra con `detail=f"...: {e}"`.

**Señal de que lo estás usando bien:** El `detail`/`message` de un 500 es una constante; el stack aparece solo en el log. Un grep del shape de error legacy da resultados concentrados en un solo archivo viejo (o cero).

---

#### U-20 — Logging estructurado con niveles según gravedad
**(TS #23 · PY #30)**

**En una línea:** Cada componente loguea de forma identificable y elige el nivel por consecuencia: warning para fallos recuperables con fallback, error/exception con stacktrace para lo que quedó roto.

**La regla universal:** No loguees todo como info ni todo como error. Usá un logger que identifique al emisor (nombre de módulo) y un formato central, y elegí el nivel por gravedad real. Un fallo con degradación es WARNING; uno sin recuperación es ERROR con stacktrace.

**Cómo se ve en TypeScript:** Logger pino centralizado con handler global; el 500 lo loguea con stack el error handler de Fastify, no cada route. Los 4xx de validación no ensucian los logs de error.

**Cómo se ve en Python:** `logger = logging.getLogger(__name__)` a nivel de módulo (en 40+ módulos) y formato global con `%(name)s`. Warning recuperable cuando hay fallback; `logger.error(..., exc_info=True)` cuando algo quedó roto.

**Señal de que lo estás usando bien:** Podés saber qué componente emitió cada línea; los fallos con fallback aparecen como WARNING y los sin recuperación como ERROR con traceback.

---

#### U-21 — Proveedor externo caído no rompe el flujo (degradación)
**(TS #26 · PY #32)**

**En una línea:** Un fallo de una dependencia externa no determinista (LLM, mensajería, OCR, pago, firma) se captura y devuelve un fallback definido, en vez de propagar y tirar todo abajo.

**La regla universal:** Para toda dependencia externa que puede fallar o estar caída, envolvé la llamada, logueá con identificación del proveedor y devolvé un valor seguro de degradación (un mensaje de disculpa, `False`, `None`, un plan B). El endpoint siempre debe poder responder; el detalle del fallo queda en el log.

**Cómo se ve en TypeScript:** El webhook de pago, si la firma HMAC falla pero hay `webhookSecret`, no corta duro: loguea un `warn` y cae al plan B (validación por lookup del pago). El cliente traduce errores de red/timeout (`AbortError`) a mensajes humanos en español en vez de mostrar el error crudo.

**Cómo se ve en Python:** Cada llamada a un proveedor externo va en `try/except`: el LLM devuelve `_fallback_intent()`, el envío Twilio devuelve `False`, el webhook ante fallo de transcripción responde un mensaje genérico. El usuario recibe algo usable.

**Señal de que lo estás usando bien:** Apagar el proveedor externo no produce un 500: produce una respuesta de fallback y una línea de log con el error real.

---

### Familia F — Dinero, webhooks y jobs

#### U-22 — Firma de webhook validada timing-safe, fail-closed
**(TS #26 / TS #53 · PY #31)**

**En una línea:** Todo webhook valida su firma/token con comparación de tiempo constante y rechaza ante firma ausente, malformada o que no matchea, antes de ejecutar cualquier efecto.

**La regla universal:** El camino por defecto de un webhook es denegar. Validá la firma con comparación constant-time (resistente a timing attacks) sobre buffers, nunca con `===`/`==`, y que un input malformado devuelva "inválido" en vez de explotar. Sin firma válida, no tocás lógica de negocio.

**Cómo se ve en TypeScript:** Un helper `timingSafeEqualHex` usa `crypto.timingSafeEqual` sobre buffers hex (devuelve `false` en el catch ante input malformado); reconstruye el manifest `id:..;request-id:..;ts:..;` y compara el HMAC-SHA256. Además NUNCA confía en el status del body: re-consulta el pago real al proveedor (`fetchMercadoPagoPayment`) antes de activar.

**Cómo se ve en Python:** `if not hmac.compare_digest(expected_signature, received_signature): raise HTTPException(403, ...)` antes de tocar lógica, repetido para Twilio y el token de Meta.

**Señal de que lo estás usando bien:** Un atacante no puede distinguir por tiempo si la firma estuvo cerca de ser válida; toda rama que no confirma autenticidad termina en rechazo. (Refuerzo que TS agrega y conviene universalizar: re-verificar el estado server-to-server, ver Tensión T-6.)

---

#### U-23 — Idempotencia: repetir sin duplicar, y testeado
**(TS #26 / TS #53 / TS #60 · PY #55)**

**En una línea:** Toda operación con efecto secundario (cobro, multa, aviso, activación) se puede ejecutar dos veces sin duplicar el efecto, y hay un test que lo prueba llamándola dos veces.

**La regla universal:** Hacé cada efecto idempotente con una clave determinística persistida y un "insertá-si-no-existe" (índice/constraint único que respalde el corte). Si no insertaste fila, ya estaba procesado: respondé OK y no hagas nada. Y convertí la idempotencia en invariante testeada: el test llama la operación dos veces y afirma que el segundo intento no duplica (mismo id / count 0).

**Cómo se ve en TypeScript:** El webhook inserta en `payment_webhook_events` con `UNIQUE (provider, event_key)` y `ON CONFLICT DO NOTHING RETURNING id`; si no devuelve fila, responde `{ duplicate: true }`. Los jobs usan una `dedupeKey` con la fecha + índice único parcial; `rowCount === 1` distingue "creado" de "omitido".

**Cómo se ve en Python:** Los tests llaman la misma operación dos veces y afirman la no-duplicación: `assert queue_debt_notices(...) == 1; assert queue_debt_notices(...) == 0` y `assert multa_1.id == multa_2.id; assert len(multas) == 1`. Las reglas de negocio (R1-R4, `idempotency_key`) tienen su test de monto exacto y no-duplicación.

**Señal de que lo estás usando bien:** Reenviar el mismo webhook o correr el mismo job dos veces no duplica nada; existe un test "es idempotente" por cada operación mutante de dinero/estado.

---

#### U-24 — Job de fondo resiliente: no mata al scheduler
**(TS #60 · PY #33)**

**En una línea:** Cada job programado abre su recurso, captura sus errores, los cuenta y los loguea, y libera todo en un `finally`; un fallo no detiene el scheduler ni las corridas siguientes.

**La regla universal:** Un job debe sobrevivir a sus propios errores. Estructuralo con try (commit) / except (rollback + log con stacktrace, sumando un contador de errores) / finally (cerrar la sesión), devolvé un resumen con contadores y NO relances. Registrá cada corrida (estado RUNNING → COMPLETED/FAILED + resumen) para poder auditar cuándo corrió y qué tocó.

**Cómo se ve en TypeScript:** Cada suite de jobs abre un registro en `notification_job_runs` con status `RUNNING` y al final lo marca `COMPLETED`/`FAILED` con un `details` JSONB, todo en try/catch. El endpoint que dispara el job (lo llama un cron externo) está protegido por un secret en header.

**Cómo se ve en Python:** `db = SessionLocal(); try: ...; db.commit(); except Exception as e: db.rollback(); logger.error(..., exc_info=True); finally: db.close()`. El job devuelve un dict con contadores (encolados/enviados/errores) y nunca relanza, así el scheduler sigue vivo.

**Señal de que lo estás usando bien:** Tras un error, el log muestra el stacktrace, la sesión quedó cerrada sin transacción colgada y el siguiente disparo corre normal.

---

### Familia G — Las llaves del local (config y secrets)

#### U-25 — El .env.example como contrato; el .env real, ignorado
**(TS #33 · PY #44 / PY #45)**

**En una línea:** Un único template versionado, agrupado por dominio y comentado, documenta cada variable que la app espera; el `.env` real nunca entra al repo.

**La regla universal:** Tratá el `.env.example` como documentación de primera clase: agrupá por secciones, y en cada variable sensible explicá qué hace, en qué ambiente aplica y si es la opción recomendada o un fallback. Usá placeholders obviamente inseguros (`change_me`, `replace_with_...`) para que nadie los confunda con prod. Ignorá `.env`/`.env.*` con una excepción `!.env.example`. (Refinamiento que conviene adoptar: poner al lado de cada secreto el comando exacto para generarlo.)

**Cómo se ve en TypeScript:** `.env.example` agrupado por dominio (`# ===== MERCADO PAGO =====`) con comentarios que explican la decisión arquitectónica (por qué WhatsApp está apagado por costo, qué cifra `PAYMENT_SETTINGS_SECRET`); placeholders tipo `dev_secret_change_in_production_...`. Lista paralela de "variables mínimas por servicio" en el doc de deploy.

**Cómo se ve en Python:** `.env.example` con encabezado "safe to commit", secciones y placeholders `replace_with_*`; `.gitignore` con `.env` / `.env.*` / `!.env.example`. Y embebe el comando para generar cada clave: `python -c "from cryptography.fernet import Fernet; print(...)"` y `secrets.token_hex(64)`, repetido en el mensaje de error cuando la clave falta.

**Señal de que lo estás usando bien:** Un dev nuevo copia el ejemplo, levanta todo en local sin pedir secretos, y entiende leyendo los comentarios qué es obligatorio en prod. Un `git ls-files | grep .env` devuelve únicamente `.env.example`.

---

#### U-26 — Validar config crítica al arranque (fail-fast), rechazar secretos débiles
**(TS #41 · PY #8 / PY #9 / PY #47)**

**En una línea:** La config de seguridad inválida hace fallar el arranque (o el deploy), no el primer request: secreto corto/placeholder o algoritmo fuera de allowlist abortan.

**La regla universal:** Validá los secretos y la config crítica lo más temprano posible (al cargar el proceso o en un preflight de deploy), con un error accionable. Rechazá secretos débiles: no solo que existan, sino que no sean cortos ni contengan marcadores de placeholder. Distinguí dev (permisivo) de prod (estricto) por una variable de ambiente explícita.

**Cómo se ve en TypeScript:** Un `preflight.mjs` previo al deploy valida `JWT_SECRET >= 32 && != 'dev-secret'`, `DATABASE_URL` postgresql sin `localhost`, migraciones, type-check y `/health`; cualquier fallo aborta con `exit 1`.

**Cómo se ve en Python:** Al importar el módulo corre `_validar_configuracion_jwt()`: algoritmo en allowlist `{HS256,HS384,HS512}` (evita `none`), y en prod exige secreto fuerte + issuer + audience, fallando con `RuntimeError`. `_secreto_debil` rechaza longitud <32 o marcadores como `change_in_production`, `dev_secret`, `example`.

**Señal de que lo estás usando bien:** Desplegar en prod con el secreto por defecto hace fallar el contenedor al iniciar (o el deploy), no en el primer login; nunca se acepta `none`.

---

#### U-27 — Default seguro y barato; lo caro o riesgoso se enciende a mano
**(TS #37 · PY #46 / PY #72)**

**En una línea:** Un ambiente recién provisionado arranca en el modo más barato y seguro; encender algo costoso o riesgoso requiere un acto deliberado.

**La regla universal:** El valor por defecto debe ser el comportamiento seguro de producción. Gobierná lo caro/opcional/en-desarrollo con flags que vienen apagados, chequeados de forma estricta; y donde el comportamiento seguro difiere entre dev y prod, derivá el default del ambiente en vez de hardcodearlo. Nada riesgoso se enciende por accidente al provisionar.

**Cómo se ve en TypeScript:** Batería de flags `ENABLE_*`/`FEATURE_*` todos en `false` en el `.env.example`, chequeados estrictamente con `=== 'true'` (no truthy). El preflight avisa si una feature requiere una condición de infra que no se cumple.

**Cómo se ve en Python:** El default se deriva de `ENV`: en dev/test autocrea el esquema, en prod no (fuerza migraciones); `LOG_TO_FILE` off en prod. El proveedor de notificaciones por defecto es `log` (no-op): el comportamiento caro (mandar mensajes reales) requiere setear `NOTIFICATION_PROVIDER` explícitamente.

**Señal de que lo estás usando bien:** Sin tocar ninguna variable, la app levanta en modo seguro (no autocrea schema, no manda mensajes reales, features beta ocultas); prender algo es deliberado.

---

#### U-28 — Secretos productivos los inyecta la plataforma, no el repo
**(TS #36 / TS #41 · PY #48)**

**En una línea:** Los secretos de producción se generan o se cargan en la plataforma de deploy (Secret Manager / dashboard / IaC con referencias), nunca como valores en el repositorio ni en capas de la imagen.

**La regla universal:** Separá la config pública del cliente (se hornea en build) de los secretos del servidor (se inyectan en runtime por el orquestador). En el IaC declarás los nombres de las variables sensibles como generadas o "no sincronizar" y referenciás recursos, pero nunca ponés el valor del secreto en el yaml versionado. Un check de infra verifica que los secretos vienen del gestor y que la base no tiene redes abiertas.

**Cómo se ve en TypeScript:** Config pública (`NEXT_PUBLIC_*`) como `--build-arg`; secretos del servidor (`DATABASE_URL`, `JWT_SECRET`, `PAYMENT_SETTINGS_SECRET`) como env de runtime de Cloud Run. El `hardening-check.mjs` verifica que vengan de Secret Manager y que Cloud SQL no tenga authorized networks abiertas.

**Cómo se ve en Python:** `render.yaml` usa `generateValue: true` para `JWT_SECRET`, `sync: false` para `ENCRYPTION_KEY`/`META_ACCESS_TOKEN`/`OPENAI_API_KEY` (se cargan a mano en el panel) y `fromDatabase`/`fromService` para `DATABASE_URL`/`REDIS_URL`. (Deuda honesta marcada en el propio repo: `WEBHOOK_VERIFY_TOKEN` quedó hardcodeado y debería ser `sync:false`.)

**Señal de que lo estás usando bien:** El yaml de deploy no contiene ningún secreto real; todos son generados, "no sincronizar" o referencias a recursos. Ningún secreto del servidor aparece en el bundle del cliente.

---

#### U-29 — El arranque pone defaults de runtime y migra una sola vez
**(TS #35 · PY #50)**

**En una línea:** Un script/launcher de arranque centraliza los defaults de runtime y corre las migraciones/init una sola vez, antes de levantar los workers.

**La regla universal:** Centralizá el arranque: poné defaults para puerto/host/concurrencia (con `${VAR:-default}` o equivalente), cargá las variables críticas ANTES de importar el cliente de datos, y corré migraciones/init una vez (no una por worker). La app debe levantar en local sin ningún `.env`, con defaults de dev claramente inseguros para forzar configurarlos en prod.

**Cómo se ve en TypeScript:** El launcher (`start.ts`/`bootstrap.ts`) setea `DATABASE_URL`/`JWT_SECRET`/`NODE_ENV` con defaults de dev ANTES de importar la app (import dinámico), evitando bugs de orden de carga del ORM.

**Cómo se ve en Python:** `start.sh` lee tuning con `${WEB_CONCURRENCY:-4}` etc., corre migraciones y setea `INIT_DB_ON_STARTUP=false` para no reinicializar la BD en cada worker; `main.py` respeta ese flag.

**Señal de que lo estás usando bien:** La app levanta sin setear ninguna variable de tuning (usa defaults) y la base se inicializa una vez, no una por worker.

---

#### U-30 — Lectura de entorno tipada, validada y con fallback
**(TS #35 / TS #37 · PY #36)**

**En una línea:** Las variables de entorno se leen siempre a través de helpers que parsean, validan, clampan y caen a un default — nunca `getenv` crudo desparramado.

**La regla universal:** No leas booleanos/enteros del entorno crudos. Pasá toda lectura por un helper que haga strip, coerción al tipo final, clamp a un mínimo razonable y fallback con warning ante valor inválido. Para cada pieza de config, implementá una cascada explícita: fuente principal → alias tolerantes → default de desarrollo, marcando el origen cuando caés al fallback para que sea auditable.

**Cómo se ve en TypeScript:** Patrón `process.env.X ?? process.env.X_ALIAS ?? 'default-local'` consistente; flags chequeados estrictamente con `=== 'true'`; cuando la config de pagos cae al fallback de env, lo marca con `lastValidationStatus: 'env_fallback'` para que sea auditable.

**Cómo se ve en Python:** `_env_int(nombre, default, *, minimum=1)` (parsea, clampa, loguea warning ante inválido) y `_env_bool` (normaliza a `{1,true,yes,on}`); se usan en `database.py`, `main.py`, `cron/runner.py`. No hay `os.getenv(...) == 'true'` suelto.

**Señal de que lo estás usando bien:** Ninguna lectura de env devuelve un string crudo sin validar; un valor inválido cae al default con un warning, no rompe el arranque silenciosamente.

---

### Familia H — Cuándo está terminado (tests y QA)

#### U-31 — Tests contra DB real, sin mockear el ORM (solo el IO externo)
**(TS #38 · PY #52 / PY #53 / PY #57)**

**En una línea:** Las rutas se prueban levantando el framework real con los plugins de producción y pegando contra una base de datos de verdad con datos conocidos; lo que se mockea es el IO externo, nunca la capa de datos.

**La regla universal:** Escribí tests de integración que levanten el framework HTTP real (inyección en memoria) y peguen contra una base real con seed determinista, no contra mocks del ORM. Mockeá solo las dependencias externas no deterministas (mensajería, LLM, pagos, cache) y hacé que el fake registre sus llamadas para afirmar el contrato exacto. Aislá el estado entre tests.

**Cómo se ve en TypeScript:** Cada `.test.ts` arma un Fastify real con los mismos plugins (cors, jwt, authenticate), registra el `setupXRoutes` real y usa `app.inject()`. No mockea Prisma: apunta `DATABASE_URL` a una Postgres local fija con seed, desactiva el paralelismo entre archivos y cierra conexiones en un teardown global.

**Cómo se ve en Python:** Fixture `db_session` que hace `create_all` en `sqlite:///:memory:`, yield y `drop_all` en finally (aislamiento total por test); `monkeypatch.setattr` corta WhatsApp/LLM/pagos/Redis apuntando al módulo consumidor, y el fake acumula llamadas para afirmar el payload. El CI, además, corre la suite contra Postgres 16 real.

**Señal de que lo estás usando bien:** Los tests fallan cuando rompés una query real, una constraint o un status code, no solo cuando cambia un mock; corren sin red ni claves reales pero verifican el payload que se habría enviado. (Diferencia de motor de DB: ver Tensión T-3.)

---

#### U-32 — Probar el caso negativo, el contrato y las invariantes
**(TS #39 · PY #55 / PY #51)**

**En una línea:** No alcanza el happy path: se testean explícitamente los rechazos (400/401/409), la forma exacta de la respuesta y las invariantes del dominio (como la idempotencia), y el nombre del test documenta qué protege.

**La regla universal:** Por cada endpoint cubrí, además del happy path: falta de auth, cada regla de validación relevante, conflictos de unicidad, y la forma exacta del cuerpo de respuesta. Para operaciones con efecto, testeá la invariante (ej. ejecutarla dos veces no duplica). Nombrá cada test por el comportamiento o el bug que previene, no por la función que llama.

**Cómo se ve en TypeScript:** Bloques de casos negativos por ruta (400 por campo faltante, monto negativo, método no permitido; 409 por duplicado), `toHaveProperty` sobre la forma del body, y en CSV verifica content-type, header de columnas y la AUSENCIA de un error serializado filtrado. Cuando el resultado es ambiguo, acepta el set válido con `expect([201,409]).toContain(...)`.

**Cómo se ve en Python:** Nombres largos de dominio en español (`test_cancelar_turno_es_idempotente_y_crea_una_sola_multa`) con docstring del root-cause del bug; ~290 tests. La idempotencia se afirma llamando la operación dos veces y verificando el efecto numérico exacto.

**Señal de que lo estás usando bien:** Mandar un payload basura nunca termina en 500 ni en un 200 silencioso; al leer el nombre de un test fallido sabés qué invariante se violó sin abrir el archivo.

---

#### U-33 — QA/CI como gate del release
**(TS #41 · PY #57)**

**En una línea:** Antes de liberar, una corrida automatizada (cobertura, matriz de versiones, DB real, smoke contra entorno real) tiene que pasar; cualquier fallo aborta y ningún release se firma sin ella en verde.

**La regla universal:** El "terminado" tiene un gate ejecutable. Componé un único comando/pipeline que valide lo que importa (contratos, permisos por rol, cobertura mínima, salud del entorno) contra el motor de datos de producción y los proveedores en modo no-op, y que salga con error listando exactamente qué falló. Documentá en un checklist qué corridas son obligatorias.

**Cómo se ve en TypeScript:** `pnpm qa:release` corre una `routeMatrix` declarativa (status esperado por endpoint × rol), un envío push real verificando `deliveryStatus === 'PUSH_SENT'`, y acumula TODOS los fallos antes de `process.exit(1)`; más `preflight` + `hardening-check`. El `RELEASE_CHECKLIST.md` lo declara obligatorio.

**Cómo se ve en Python:** El workflow de CI levanta Postgres 16 con healthcheck, inyecta env de test, corre pytest en matriz 3.11/3.12 y falla el build si la cobertura baja de 53% (`--cov-fail-under=53`), con los proveedores de notificación en modo `log`.

**Señal de que lo estás usando bien:** Existe un único comando/pipeline que se pone rojo si algo de permisos, contrato, cobertura o infra está roto, listando qué falló.

---

#### U-34 — Un test que actúa como atacante: cruzar tenants DEBE fallar
**(TS #40 · PY #5)**

**En una línea:** Hay una verificación que se pone en el rol del atacante e intenta acceder/mutar datos de otro inquilino, exigiendo que falle (o devuelva 404), no que pase.

**La regla universal:** No confíes solo en que escribiste el `WHERE` de tenant: probalo activamente. Parado en el tenant A, intentá leer/insertar/actualizar/borrar datos del tenant B y exigí el fallo. Probá también el fail-closed: sin contexto, cero filas (no todo). Si tu base soporta filtrado a nivel de fila (RLS y equivalentes), sumalo como red final — pero como red, no como piso, y verificá que esté realmente conectada al camino de runtime.

**Cómo se ve en TypeScript:** Una suite `rls.test.ts` crea dos orgs y, parada en org1, intenta INSERTAR datos de org2 exigiendo `rejects.toThrow()` por la policy `WITH CHECK`; prueba el fail-closed (COUNT sin contexto → 0). Honestidad del repo: la RLS hoy NO está conectada al pool real de runtime (los routes usan `pg` crudo), así que en la práctica la barrera viva es el filtro app-layer.

**Cómo se ve en Python:** No hay RLS; la verdad la impone el filtro app-layer (U-02). El cruce de tenants se exige vía el 404 de los helpers `obtener_*_de_tenant` (pedir un id de otro inquilino da el mismo 404 que uno inexistente), reforzado por los tests de invariantes.

**Señal de que lo estás usando bien:** Un test que se hace pasar por otro tenant falla; si comentás el filtro de tenant en una query, algún test se pone rojo. (Honestidad: el repo TS muestra que una RLS "declarada pero desconectada" es seguridad teórica — verificá que la red exista de verdad.)

---

### Familia I — Cómo se organiza y crece

#### U-35 — Composition root: registro central explícito, módulo auto-instalable
**(TS #28 / TS #51 · PY #1)**

**En una línea:** Hay un solo lugar que arma la app registrando cada módulo explícitamente; nada se auto-descubre por magia, y cada módulo se expone como una función que recibe la app y se instala sola.

**La regla universal:** Registrá los módulos/rutas explícitamente en un composition root. Exportá por dominio una función `setup<X>(app, ...deps)` que reciba la instancia del servidor (y los servicios que necesite) como parámetro y registre ahí sus endpoints — inyección por parámetro, no imports de un singleton global. Agregar un módulo = crear su archivo y sumar UNA línea de registro.

**Cómo se ve en TypeScript:** `export async function setupXRoutes(app: FastifyInstance)` en los 19 archivos de routes; el `index.ts` llama a cada uno en orden. Cuando un módulo necesita más colaboradores, los suma a la firma (`setupAuthRoutes(fastify, authService)`). Podés instanciar el módulo en un test con un `app` falso sin levantar todo.

**Cómo se ve en Python:** `main.py` importa cada router por nombre y los engancha con `app.include_router()`; cada router declara su `prefix` y `tags`. Cero lógica de negocio en el composition root: solo orquesta logging, CORS, lifespan y montaje. Agregar un endpoint = un archivo en `routes/` + una línea `include_router`.

**Señal de que lo estás usando bien:** Ningún archivo de rutas importa una instancia global del framework; agregar un módulo toca un único lugar predecible de registro.

---

#### U-36 — Route flaca, service gordo
**(TS services/ · PY #2)**

**En una línea:** El handler HTTP solo inyecta dependencias, valida y delega; la regla de negocio vive en una capa de servicios separada de la persistencia.

**La regla universal:** En el handler poné solo: dependencias, validación de schema y delegación a una función de servicio. Cualquier comportamiento de dominio nuevo (cálculos financieros, políticas, transiciones) va al service, no al handler; la capa de modelos es solo persistencia. Un handler que entra en pocas líneas y no tiene queries complejas ni cálculos de negocio está bien ubicado.

**Cómo se ve en TypeScript:** La lógica vive en `services/` (`auth.ts`, `membership-access.ts`, `payment-settings.ts`, `platform-billing.ts`); los handlers delegan (`consumeReservationAccess`, `canUseOrganizationPermission`) y se quedan con validar + transaccionar + responder.

**Cómo se ve en Python:** Cada endpoint recibe `db: Session = Depends(get_db)` y `usuario = Depends(require_psicologa)`, valida con schemas y llama a una función de `services/` (~30 archivos `*_service.py`). La lógica financiera, de políticas y de turnos no toca el route. Declarado como regla del repo ("Keep route handlers thin").

**Señal de que lo estás usando bien:** Si estás escribiendo lógica financiera dentro de un route, está mal ubicada; toda consulta compleja y todo cálculo de dominio vive en el service.

---

#### U-37 — SQL parametrizado, nunca concatenado
**(TS #58 · PY #42)**

**En una línea:** Los valores de usuario nunca se interpolan en el string SQL: entran siempre como parámetros bindeados, y el SQL crudo está confinado a lugares puntuales.

**La regla universal:** Ningún dato de usuario aparece literal en el SQL; solo el placeholder posicional/nombrado. Construí los filtros opcionales empujando valores a un array de params y referenciando su índice, o con el constructor de queries del ORM. Reservá el SQL crudo para casos muy puntuales (migraciones de bootstrap) y siempre con binds.

**Cómo se ve en TypeScript:** Constructor incremental: `params.push(valor)` y `AND col = $${params.length}`; el valor jamás se interpola, solo el número de posición. Pasar `' OR 1=1` en un search no inyecta, solo no matchea.

**Cómo se ve en Python:** Todo el acceso de negocio usa el ORM (`db.query(...).filter(...)`); el único `text()` está en `init_db` para migraciones legacy, con valores bindeados (`{"legacy":..., "nuevo":...}`). Ninguna route arma SQL por concatenación.

**Señal de que lo estás usando bien:** Agregar o quitar un filtro no descoordina los binds; un grep de SQL crudo aparece solo en el lugar confinado; ningún valor de usuario está concatenado.

---

#### U-38 — Fachada/barrel: el paquete expone una API pública curada
**(TS #42 · PY #3)**

**En una línea:** Un paquete o módulo grande expone su API pública desde un único punto de entrada que re-exporta solo lo curado, ocultando la fragmentación interna.

**La regla universal:** Cuando partas un módulo grande en submódulos, dejá un punto de entrada (barrel/fachada) que re-exporte solo la API pública; marcá los submódulos internos como privados y que los consumidores importen siempre desde la fachada, nunca de las tripas. Así podés reorganizar el interior sin romper a quien lo usa.

**Cómo se ve en TypeScript:** `packages/shared` es un paquete con un `index.ts` que es puro barrel (`export * from './types'`, `./validations`, `./permissions`); todos importan `from '@szoluciones/shared'`, nunca con rutas relativas que cruzan paquetes.

**Cómo se ve en Python:** `operativo/__init__.py` importa desde `_core`/`_handlers` lo público y lo re-exporta con `__all__`; los submódulos internos llevan prefijo `_`. Quien usa el paquete importa desde `app.services.operativo`, no desde `._core`.

**Señal de que lo estás usando bien:** Ningún consumidor importa de las tripas internas; el punto de entrada lista exactamente lo público y reorganizar el interior no rompe imports.

---

### Candidatos a Universal (sospechados pero documentados en un solo repo)

> Cada uno parece un principio cross-stack, pero solo está respaldado por evidencia en UNO de los dos repos. Para confirmarlo falta verlo implementado en el otro.

- **CANDIDATO A UNIVERSAL — falta verificar en Python: separar deployables de librerías en el build (TS #2).** El repo TS distingue `apps/` (deployables, no emiten tipos) de `packages/` (librerías, emiten `.d.ts`/dist). Lo sospecho universal porque "separar lo que se despliega de lo que se reusa" vale en cualquier monorepo; pero el repo Python es un monolito (una app + un front), así que no hay con qué confirmarlo.

- **CANDIDATO A UNIVERSAL — falta verificar en Python: onDelete con intención, Cascade para hijos / desligar para histórico (TS #20).** Elegir el borrado en cascada según si el dato es accesorio (cae con el padre) o evidencia que debe sobrevivir (se desliga) es relacional puro, no un idiom de TS. Lo sospecho universal; pero Python no documenta política de FK onDelete (resuelve la supervivencia del histórico con ledgers inmutables), así que queda sin verificar.

- **CANDIDATO A UNIVERSAL — falta verificar en Python: billing del propio SaaS como gate de permisos con 402 (TS #55).** Gatear features por estado de suscripción de la plataforma (degradar a solo-lectura, responder 402 para distinguir "no pagaste" de "no tenés permiso") es un patrón de arquitectura SaaS reaplicable; pero el repo Python no tiene capa de suscripción de plataforma, así que no hay evidencia gemela.

- **CANDIDATO A UNIVERSAL — falta verificar en TypeScript: proveedores intercambiables tras una interfaz, seleccionados por config (PY #72 / #69).** Modelar varios proveedores del mismo comportamiento (LLM, notificaciones, pagos) como una interfaz + una implementación por proveedor + un factory por env es Strategy clásico (GoF), agnóstico al lenguaje. El repo TS tiene un solo proveedor de pago real, así que nunca necesitó abstraerlo: no hay con qué confirmarlo.

- **CANDIDATO A UNIVERSAL — falta verificar en TypeScript: policy chain con precedencia + snapshot inmutable (PY #70).** Resolver una regla por cascada de fuentes (DEFAULT → entidad padre → override puntual), etiquetar el origen, exigir justificación para overrides manuales y CONGELAR un snapshot en la entidad para que cambios futuros de config no reescriban el histórico. Patrón de dominio muy valioso para cualquier sistema con tarifas/penalidades; el TS no lo documenta.

- **CANDIDATO A UNIVERSAL — falta verificar en TypeScript: guardrails por nivel de riesgo + confirmación (PY #68).** Etiquetar cada acción con un `risk_level` y exigir un paso de confirmación antes de ejecutar mutaciones (sobre todo financieras). La idea "antes de una acción destructiva o con dinero, confirmá" es universal; el TS no la aísla como tornillo (no tiene un agente conversacional que la motive).

- **CANDIDATO A UNIVERSAL — falta verificar en TypeScript: pool de conexiones endurecido y configurable (PY #27).** `pre_ping`, `recycle`, LIFO y overflow acotado, todo por env con parser defensivo, aplican a cualquier driver con pool. El TS gestiona las conexiones deliberadamente (singleton + health check + shutdown ordenado, su #22) pero no documenta el tuning del pool, así que el principio específico queda sin confirmar del lado TS.

- **CANDIDATO A UNIVERSAL — falta verificar en TypeScript: el nombre del test ES la documentación del bug (PY #51).** Nombrar cada test por la regla de negocio o el bug que previene (y documentar el root-cause en el docstring) es disciplina de testing cross-lenguaje. El TS prueba casos negativos (U-32) pero no aísla la convención de nomenclatura como tornillo propio.

---

## SECCIÓN 2 — TORNILLOS SOLO TYPESCRIPT

> Los que no tienen equivalente confirmado en Python: o son idioms de TS/JS, o son específicos de un stack en particular (monorepo + Prisma + Cloud Run). Úsalos si tu stack es TypeScript; en otro stack, leelos como referencia.

- **TS #1 — `workspace:*` + alias `@scope/*` para deps internas.** Idiom de monorepo de pnpm/yarn/npm; no aplica al monolito Python.
- **TS #3 — tsconfig raíz único + override por proyecto.** Config DRY de TypeScript; no existe equivalente directo en Python.
- **TS #4 — Turborepo respeta el grafo (`^build`) y cachea lo determinístico.** Orquestación de build de monorepo JS; Python no tiene ese pipeline.
- **TS #6 — Cadena `onRequest`: autenticar → autorizar (y de paso cobrar).** Específico de Fastify + el gate de billing 402; Python compone deps de FastAPI distinto y no tiene el gate.
- **TS #7 — Access token en memoria, refresh token persistido y rotado.** Patrón de cliente Zustand/axios; Python trata el JWT como opaco en el front (U-07) pero no documenta rotación de refresh.
- **TS #8 — Catálogo de permisos como array de strings, fuente única compartida.** Permisos finos `recurso.accion` con `'*'` y ADMIN normalizado, compartidos front/back. Python autoriza por roles gruesos (`require_rol`), no tiene catálogo de permisos. (Ver Tensión T-2.)
- **TS #12 / #21 — RLS de Postgres con GUC como red de DB.** Defensa en profundidad específica de Postgres + Prisma; Python no usa RLS. (Y en TS está hoy desconectada del pool real — deuda honesta del repo.)
- **TS #15 / #25 / #30 / #43 / #50 — Zod como fuente única: el validador ES el tipo.** El principio "validar en la frontera" es universal (U-14), pero derivar el tipo con `z.infer` y componer schemas con `.partial()/.extend()/.superRefine()` es idiom de TS. Python lo hace con Pydantic v2.
- **TS #16 — El cliente tolera el envelope (`unwrapList`).** El principio (cliente normaliza, U-15) es universal; el helper genérico con union type es TS.
- **TS #17 — Invalidación dirigida de React Query con update optimista.** Idiom de TanStack Query.
- **TS #22 — Conexiones como singleton con shutdown ordenado y escape hatch a `pg` crudo.** El "manejar conexiones deliberadamente" roza el candidato de pool (PY #27); el detalle (singleton lazy + workaround por bug de Prisma en Windows) es TS/proyecto.
- **TS #27 — Refresh silencioso en 401 con deduplicación.** El núcleo (401 → logout, U-07) es universal; la deduplicación con una promesa módulo-level es idiom de axios/fetch.
- **TS #31 / #47 / #48 / #49 / #51 — Auto-migración `ensureXTable` + patrón de expansión de módulo.** DDL idempotente en el arranque y el "molde de módulo" (setupXRoutes + guards gemelos + CRUD canónico + registro) son específicos de Movete; Python usa Alembic versionado.
- **TS #35 / #36 — Fallbacks de env en cascada / config de cliente build-arg vs servidor runtime.** El principio de defaults de runtime y secretos por plataforma se universalizó (U-29/U-28/U-30); el detalle de `--build-arg` de Next/Expo y Cloud Run es del proyecto.
- **TS #42 / #44 / #45 — Barrel `@szoluciones/shared` / permisos como array const / espejo Prisma↔TS.** El barrel se universalizó (U-38) y los enums tipados también (U-11); el `as const` que deriva el union type y el espejo manual enum-Prisma↔union-TS son idioms de TS.
- **TS #56 — Fallback por entorno con bandera de origen explícita (`source`).** El principio de auditar el origen del valor es bueno, pero está acoplado a la config de Mercado Pago del proyecto.
- **TS #58 — Constructor incremental de `WHERE` + params.** El principio (SQL parametrizado) se universalizó (U-37); el patrón de armado manual con `$${params.length}` sobre `pg` crudo es de este repo (Python usa el ORM).

---

## SECCIÓN 3 — TORNILLOS SOLO PYTHON

> Los que no tienen equivalente confirmado en TypeScript: ergonomías que Python hace de forma idiomática y que en TS se replican con más fricción o no existen con la misma naturalidad. Para cada uno, lo que Python aporta.

- **PY #4 — Banner "AUTO-GENERATED" en archivos derivados.** *Lo que aporta:* cuando un módulo grande se parte con un script, el derivado avisa en la cabecera su fuente y cómo regenerarlo, evitando que alguien edite el archivo equivocado. Es una convención de "fuente única de verdad" a nivel archivo que el repo TS no necesitó (no parte módulos por script).

- **PY #10 — Dependency factory para roles (`require_rol(*roles)`).** *Lo que aporta:* una función que cierra sobre sus parámetros y devuelve la dependency lista para `Depends`, con shortcuts pre-aplicados (`require_psicologa`). El closure + first-class function de Python lo vuelve trivial y sin metadata/reflection; TS lo haría con decoradores y guards (NestJS), más ceremonia.

- **PY #12 — bcrypt con fallback PBKDF2 y verify por prefijo.** *Lo que aporta:* marcar cada hash con un prefijo de esquema y elegir el verificador por ese prefijo (no por config global), así los hashes viejos siguen validando tras cambiar de algoritmo, con comparación constant-time y `False` ante hash corrupto en vez de excepción. Sobrevive entornos sin bcrypt (Python 3.14+/tests).

- **PY #26 — Encriptación transparente vía `hybrid_property` (Fernet).** *Lo que aporta:* el `@hybrid_property` de SQLAlchemy deja que un atributo se vea como texto plano en Python pero esté cifrado en la columna, e incluso exponer la columna cruda para SQL vía `.expression`. El principio (cifrar en reposo) es universal (U-12); la integración transparente con el query layer es marca de Python. (En TS se resuelve con transformers de TypeORM, menos integrados.)

- **PY #28 — `init_db` detecta drift y exige migración explícita.** *Lo que aporta:* al arrancar compara esquema real vs modelos con `inspect(engine)`; crea tablas faltantes solo bajo flag y solo en dev, pero FALLA si faltan columnas (drift), forzando migración explícita en prod. Más migración idempotente de valores legacy. Una red de seguridad de esquema que el repo TS reemplaza con `ensureXTable`.

- **PY #35 — Docstring de módulo en español al tope.** *Lo que aporta:* cada archivo abre con un docstring de 1-3 líneas que dice qué es el módulo antes de los imports, así un humano sabe el rol del archivo de un vistazo. Convención de estilo del repo Python.

- **PY #37 — `typing.cast` para domar columnas SQLAlchemy.** *Lo que aporta:* `cast(str, usuario.id)` es no-op en runtime y reconcilia los `Column[...]` genéricos con los tipos Python reales, manteniendo el type checker estricto sin `# type: ignore` dispersos. Es específico del gradual typing de Python (TS, tipado de raíz, no lo necesita).

- **PY #38 — Triada `get_`/`require_`/`upsert_`/`serialize_`/`delete_` por entidad.** *Lo que aporta:* una nomenclatura canónica donde `require_` se construye SIEMPRE sobre `get_` + guard clause (nunca duplica la query) y `delete_` sobre `require_`, así cambiar el filtro de scope toca un solo lugar. Convención del repo.

- **PY #40 / #41 — Helpers privados `_` reutilizados + type hints con keyword-only tras `*`.** *Lo que aporta:* micro-operaciones puras (`_money`, `_normalize_text`, `_bucket_key`) extraídas y compuestas, y firmas donde los ids/flags semánticos van después de `*` para que la llamada sea autoexplicativa e imposible de invertir por posición. Ergonomía idiomática de Python.

- **PY #43 — Imports explícitos de helpers con guion bajo.** *Lo que aporta:* como `from .module import *` no trae nombres `_privados`, los helpers privados se importan por nombre explícito, evitando el `NameError` silencioso de creer que el wildcard los trajo. Convención que nace de la semántica de `import *` de Python.

- **PY #49 — Normalizar `DATABASE_URL` y validar el driver.** *Lo que aporta:* convierte el `postgres://` que dan algunos PaaS al `postgresql://` que espera SQLAlchemy y rechaza cualquier otro motor, sin fallback a SQLite (mono-motor deliberado). Detalle del ecosistema Python/SQLAlchemy.

- **PY #52 / #53 / #54 / #58 — Fixtures in-memory + `monkeypatch` + autouse + reloj determinista.** *Lo que aporta:* DB SQLite efímera por test, `monkeypatch.setattr` quirúrgico apuntando al módulo consumidor (que además espía las llamadas), un fixture `autouse` que mata una fuente global de flakiness, y parcheo del reloj para tests deterministas. El principio "tests aislados y deterministas" es universal; estas herramientas (pytest) tienen una densidad y uniformidad que es marca de Python.

- **PY #59 — Generator como recurso con cierre garantizado (`get_db` con `yield`/`finally`).** *Lo que aporta:* un generador que cede el recurso en el `yield` y lo libera en el `finally`, consumido directo por el sistema de dependencias, que ejecuta el teardown aunque el endpoint explote. En TS no hay un equivalente tan limpio de "dependency que se autolimpia post-handler"; se termina con middlewares o `try/finally` manuales.

- **PY #60 — Imports opcionales con `try/except ModuleNotFoundError`.** *Lo que aporta:* que un módulo importe aunque falte una dependencia, dejando un sentinela `None` y degradando o difiriendo el error al punto de uso. Idiomático de los imports en runtime de Python; en TS los imports son estáticos y replicarlo requiere `import()` dinámico y ceremonia.

- **PY #61 — Context managers de SQLAlchemy (`with engine.begin()` / `execution_options(AUTOCOMMIT)`).** *Lo que aporta:* el protocolo `with` que delimita transacción/conexión y la libera al salir, ajustando isolation por bloque (autocommit para DDL como `ALTER TYPE`). TS no tiene `with`; lo emula con callbacks, menos visualmente delimitado.

- **PY #62 — Registry inmutable con dataclasses frozen (matcher→prepare→execute).** *Lo que aporta:* cada acción es una `@dataclass(frozen=True)` que agrupa sus callables, juntadas en una tupla inmutable, con `TypedDict` para payloads; el dispatcher itera o usa un dict por id. Agregar un comando no toca el motor de dispatch. La combinación dataclass frozen + tuple + funciones de firma uniforme da un registry declarativo muy compacto.

- **PY #63 — `@asynccontextmanager` para el lifespan.** *Lo que aporta:* una sola función async donde el `yield` parte startup y shutdown, decorada y pasada como `lifespan=`, con setup y teardown visiblemente emparejados. En TS el ciclo de vida del servidor se hace con hooks o eventos separados.

- **PY #65 / #66 / #67 — Migraciones idempotentes con inspector + enum Postgres idempotente + cadena fecha+secuencia (Alembic).** *Lo que aporta:* helpers `_has_table`/`_has_column` sobre `sa.inspect`, el bloque `DO $$ ... EXCEPTION WHEN duplicate_object` para crear enums sin romper en re-run, y revisiones `<YYYYMMDD>_<NN>` en cadena lineal. Disciplina de migraciones del ecosistema Alembic/Python; el repo TS usa auto-migración `ensure*` en su lugar.

> **El resumen del lado Python:** TypeScript te da seguridad de tipos de fábrica y un front de primera; Python te da `with`, generators con teardown, closures como dependencies, imports diferidos, `monkeypatch` quirúrgico y un ORM con properties híbridas. Esa es la ferretería extra que cada lado suma al tornillero común.

---

## SECCIÓN 4 — TENSIONES Y CONTRADICCIONES

> Casos donde los dos repos resuelven lo mismo de forma distinta y una solución es más robusta. No son bugs: son decisiones, y conviene saber cuál preferir en un proyecto nuevo.

### T-1 — Borrado: soft-delete uniforme (TS) vs ledger inmutable + flag `activo` (Python)
- **Qué resuelven los dos:** cómo borrar sin perder historia ni dejar huérfanos.
- **TypeScript:** soft-delete pervasivo — columna `deleted_at` en cada tabla, los DELETE son `UPDATE SET deleted_at = NOW()`, y toda lectura filtra `deleted_at IS NULL`. Para histórico financiero usa `onDelete: SetNull` (el pago sobrevive con FK en NULL).
- **Python:** no usa `deleted_at`; las entidades mutables tienen un flag `activo` y los registros financieros viven en **ledgers inmutables** (`PagoRecibido`, `EventoFinanciero`, `AjusteContable`) que solo tienen `created_at` — no se borran ni se editan, se compensan con asientos nuevos.
- **Cuál es más robusto:** depende del dato. Para **entidades de negocio mutables** (un socio, una sede), el **soft-delete de TS** es más simple y reversible. Para **dinero y auditoría**, el **ledger inmutable de Python** es más robusto: append-only elimina por construcción la clase de bugs de "editaron un pago viejo".
- **Regla unificada recomendada:** soft-delete (`deleted_at`) para entidades de negocio mutables; ledger append-only e inmutable para dinero y auditoría. (Ambos repos ya coinciden en que el dinero no se destruye — U-12 y la decisión SetNull/ledger lo confirman desde dos ángulos.)

### T-2 — Autorización: permisos finos compartidos (TS) vs roles gruesos (Python)
- **Qué resuelven los dos:** quién puede hacer qué.
- **TypeScript:** catálogo de permisos `recurso.accion` (strings) en un paquete compartido front/back, con `'*'` y ADMIN normalizado; guard declarativo `requirePermission('payments.manage')` por endpoint y el front filtra el menú con el mismo modelo.
- **Python:** roles gruesos vía `require_rol(Rol.PSICOLOGA)`/`require_superadmin`; la dueñez de los datos la resuelve el scope por tenant (U-01/U-03), no un permiso fino.
- **Cuál es más robusto:** para apps con muchos roles y acciones granulares, el **modelo de permisos de TS** escala mejor (agregar un permiso es un string en un solo catálogo; el front y el back lo reconocen sin desincronizarse). Para apps con pocos roles bien diferenciados, los **roles de Python** son más simples y suficientes.
- **Regla unificada recomendada:** arrancá con roles si la matriz es chica; cuando empieces a escribir `if rol == X and recurso == Y`, migrá a un catálogo de permisos `recurso.accion` definido UNA vez y compartido entre back y front. La verdad siempre la impone el back (U-05); el front solo oculta.

### T-3 — Tests de DB: Postgres real siempre (TS) vs SQLite in-memory + Postgres en CI (Python)
- **Qué resuelven los dos:** probar la lógica que toca la base sin mockear el ORM.
- **TypeScript:** los tests pegan contra una **Postgres real** de test (mismo motor que prod) con seed; capturan bugs de constraint, dialecto SQL y status code en el momento.
- **Python:** los tests unitarios usan **SQLite in-memory** efímera por test (rapidísimos, aislados), y el **CI corre la suite completa contra Postgres 16 real**.
- **Cuál es más robusto:** el enfoque de TS (motor real siempre) es más fiel: SQLite y Postgres difieren en tipos, constraints y SQL, y un test verde en SQLite puede ocultar un bug que solo aparece en Postgres. El de Python es más rápido localmente, pero **delega la fidelidad al CI** (U-33 lo mitiga).
- **Regla unificada recomendada:** como mínimo, el motor real en CI (Python ya lo hace). Para lógica pesada de DB (constraints, transacciones, SQL específico), preferí el motor real también localmente (Postgres de test o testcontainers). SQLite in-memory está bien para lógica pura que apenas roza el ORM.

### T-4 — RLS: declarada pero desconectada (TS) vs sin RLS, solo filtro app-layer (Python)
- **Qué resuelven los dos:** el aislamiento entre tenants a nivel base.
- **TypeScript:** tiene policies de RLS de Postgres escritas y testeadas... pero **hoy desconectadas del camino real** (los routes corren sobre un `Pool` de `pg` crudo donde el GUC nunca se setea). El propio tornillero lo declara como brecha honesta: la barrera viva es el filtro app-layer.
- **Python:** directamente **no usa RLS**; la única barrera es el filtro app-layer (U-02) + los helpers de dueñez (U-03).
- **Cuál es más robusto:** en la práctica, **los dos dependen del filtro app-layer**, así que hoy están parejos. Una RLS bien conectada (GUC seteado en TODA conexión que toca datos) sería estrictamente más segura que ninguna — pero una RLS declarada y NO conectada es peor que no tenerla, porque da una falsa sensación de red.
- **Regla unificada recomendada:** el filtro app-layer es el piso, no negociable (U-02). Si sumás RLS, verificá con un test de integración sobre un endpoint REAL que una query sin contexto devuelve 0 filas — si no, es seguridad teórica. Nunca uses RLS como reemplazo del filtro: es red, no piso.

### T-5 — Cifrado en reposo: AES-256-GCM versionado (TS) vs Fernet (Python)
- **Qué resuelven los dos:** guardar credenciales de terceros cifradas (U-12).
- **TypeScript:** AES-256-GCM con IV aleatorio y formato **versionado `v1:iv:tag:cipher`**; el prefijo permite rotar el esquema, y valores sin prefijo se tratan como legacy.
- **Python:** Fernet (cifrado autenticado: AES-CBC + HMAC) con `ENCRYPTION_KEY` del env, sin prefijo de versión visible.
- **Cuál es más robusto:** ambos son cifrado autenticado correcto. El de TS gana en **rotabilidad**: el prefijo de versión le permite cambiar de algoritmo sin romper datos viejos. Fernet tiene rotación de claves vía `MultiFernet`, pero el repo no la documenta.
- **Regla unificada recomendada:** cifrado autenticado siempre (cualquiera de los dos sirve), pero prefijá el ciphertext con una versión de esquema (`v1:`) para poder rotar algoritmo/clave sin migración traumática. El patrón TS es el más a prueba de futuro.

### T-6 — Webhook de pago: re-verificación server-to-server (TS) vs solo firma (Python documentado)
- **Qué resuelven los dos:** confiar (o no) en lo que llega en el webhook.
- **TypeScript:** valida la firma Y además **re-consulta el pago al proveedor** (`fetchMercadoPagoPayment`) antes de activar — nunca confía en el status del body.
- **Python:** valida la firma con `hmac.compare_digest` y fail-closed (U-22), pero el texto no documenta una re-consulta server-to-server del estado del pago.
- **Cuál es más robusto:** **TS**. La firma garantiza autenticidad del mensaje, pero re-consultar al proveedor garantiza la **verdad del estado** (un body autenticado podría llegar con un status que después cambió). "No confíes en el payload, preguntale al proveedor" es la barra más alta.
- **Regla unificada recomendada:** firma timing-safe + idempotencia (ambos repos) **y** re-verificación server-to-server del estado antes de mover plata o activar accesos (patrón TS). Es el complemento que conviene universalizar.

> **Nota:** fuera de estas seis, los dos repos son ampliamente **complementarios, no contradictorios**: comparten la columna vertebral (tenant del token, filtro en cada query, fail-closed, transacción con lock, idempotencia, config validada al boot) y difieren sobre todo en el "cómo" idiomático de cada lenguaje.

---

## SECCIÓN 5 — MAPA DE DEPENDENCIAS UNIVERSAL

> Solo entre tornillos U-XX. Si vas a aplicar uno, mirá de qué se apoya.

- **U-02** (filtro de tenant en cada query) se apoya en **U-01** (de dónde sale el tenant).
- **U-03** (404 no 403) se apoya en **U-02** (el filtro que hace "desaparecer" el dato ajeno).
- **U-04** (fail-closed) se apoya en **U-01** (si no hay tenant en el token, negá).
- **U-05** (revalidar contra DB) se apoya en **U-01** (la identidad del token es el punto de partida).
- **U-06** (ownership guard) se apoya en **U-01** y **U-02** (valida el id del cliente contra el tenant del token).
- **U-07** (401 global en el cliente) se apoya en **U-05** (el 401 que el back emite al revalidar).
- **U-09** (dinero decimal) y **U-10** (índices tenant-first) se apoyan en **U-08** (el molde de tabla donde viven).
- **U-13** (schema entrada≠salida) se apoya en **U-38** (el paquete/fachada donde se define el contrato).
- **U-14** (validación en capas) se apoya en **U-13** (los schemas de entrada) y habilita **U-16**.
- **U-15** (cliente normaliza) se apoya en **U-13** (el contrato que el cliente respeta).
- **U-16** (transacción) se apoya en **U-02** (las queries scopeadas) y **U-14** (validar antes de tocar la DB).
- **U-17** (lock pesimista) se apoya en **U-16** (vive dentro de la transacción).
- **U-19** (genérico afuera) se apoya en **U-18** (primero mapeás lo conocido, después el 500 genérico).
- **U-20** (logging) sostiene a **U-18/U-19** (el 500 se loguea con stack).
- **U-22** (firma webhook) se apoya en **U-18/U-19** (rechazo limpio) y precede a **U-23**.
- **U-23** (idempotencia) se apoya en **U-16** (la activación va en transacción) y **U-22**.
- **U-24** (job resiliente) se apoya en **U-23** (cada efecto del job es idempotente).
- **U-26/U-27/U-28/U-29/U-30** (config) se apoyan en **U-25** (el `.env.example` que documenta el contrato).
- **U-12** (cifrado) se apoya en **U-30** (la clave sale del env tipado) y **U-26** (validada al boot).
- **U-32** (negativo/contrato) se apoya en **U-31** (los tests de integración con DB real).
- **U-33** (gate de release) se apoya en **U-31** y **U-32**.
- **U-34** (test atacante) se apoya en **U-02** (lo que prueba) y **U-31** (la infra de test).
- **U-36** (route flaca) se apoya en **U-35** (el módulo que registra las rutas).
- **U-37** (SQL parametrizado) se apoya en **U-02** (toda query, parametrizada y scopeada).

### La cadena de seguridad multi-tenant (sin atarse a ningún motor)

Estos eslabones se implementan en cualquier base y cualquier lenguaje. Si se rompe uno, se filtran datos entre clientes:

```
  U-01  El tenant sale del token firmado, NUNCA del cliente
          |
          v
  U-04  Fail-closed: si no hay tenant en el token -> negá, no devuelvas nada
          |
          v
  U-02  Cada query lleva su filtro de tenant (WHERE tenant_id = <del token>)
          |   (+ U-06: los ids que manda el cliente se validan contra el tenant)
          v
  U-03  Un recurso de otro tenant devuelve 404, no 403 ni los datos
          |
          v
  U-34  Un test que actúa como atacante intenta cruzar tenants y EXIGE el fallo
```

> Quinta capa OPCIONAL: si tu base soporta filtrado a nivel de fila (RLS en Postgres, VPD en Oracle, políticas equivalentes), sumalo — pero como **red, no como piso**, y verificá que esté realmente conectado al runtime (ver Tensión T-4). Nunca reemplaza a U-02.

> Regla mental: el `tenant_id` se origina en el token y viaja en el `WHERE` de toda query. Las demás capas (fail-closed, 404, test atacante, RLS) existen para cuando un humano se olvida de la primera.

---

## SECCIÓN 6 — CÓMO USAR ESTA FERRETERÍA EN UN PROYECTO NUEVO

No importa si el proyecto nuevo es Python, Node, PHP, Go, Ruby o Java. Estos son principios, no APIs. Donde un universal diga "validá en la frontera", elegí tu validador (Zod, Pydantic, Joi, class-validator, Bean Validation); donde diga "capa de datos", elegí tu ORM. El tornillo te dice *qué* garantizar; vos elegís *con qué*.

### Los 5 pasos (independientes del stack)

1. **Poné los cimientos antes que las features.** Estructura del proyecto con un composition root que registra módulos explícitamente (U-35), routes flacas y services gordos (U-36), un paquete/fachada para el contrato compartido (U-38), lectura de entorno tipada (U-30) y un `.env.example` que documenta cada variable (U-25) con validación de config al arranque (U-26).

2. **Si es multi-tenant, la cadena de seguridad va PRIMERO, sí o sí, y en orden.** U-01 (tenant del token) → U-04 (fail-closed) → U-02 (filtro en cada query) → U-06 (ownership guard de ids del cliente) → U-03 (404, no 403), y cerrás con U-34 (test que actúa como atacante). Sumá U-05 (revalidar identidad/permisos contra DB) y U-07 (401 global en el cliente). Esto es idéntico en SQL, NoSQL o microservicios: cambia la sintaxis, no el principio.

3. **Montá la base con el molde y los contratos.** Molde de tabla con PK opaca + timestamps (U-08), dinero con decimal (U-09), índices y unicidad tenant-first (U-10), estados como enums tipados (U-11), credenciales cifradas en reposo (U-12). Definí los schemas de entrada≠salida (U-13) y la validación en capas (U-14) desde el día uno, con un contrato de error uniforme (U-19).

4. **Para cada endpoint, repetí el mismo molde.** Autenticar → autorizar → validar la entrada (U-14) → envolver la regla que toca varios registros en una transacción (U-16) con lock pesimista si consume un recurso finito (U-17) → traducir errores de dominio en el borde (U-18) → responder con el contrato uniforme. Si recibís webhooks o callbacks externos, firma timing-safe fail-closed (U-22), idempotencia (U-23) y, para dinero, re-verificación server-to-server (Tensión T-6). Lo costoso/riesgoso, apagado por default (U-27); lo externo no determinista, con degradación (U-21); los jobs, resilientes (U-24).

5. **Definí "terminado" igual en todos lados.** No alcanza el happy path: tests contra DB real sin mockear el ORM (U-31), casos negativos + contrato + invariantes (U-32), el test atacante (U-34), y todo gateado por un comando único de QA/CI con cobertura mínima (U-33). Si algo termina con `exit != 0`, no está terminado.

### Ejemplo trabajado: sistema de turnos para profesionales de la salud

Imaginá un nuevo proyecto SZoluciones: **una plataforma de turnos para profesionales de la salud** (médicos, kinesiólogos, nutricionistas). Cada profesional (o clínica) es un tenant: ve solo sus pacientes, su agenda y sus pagos. Hay turnos con estados (PENDIENTE/CONFIRMADO/ATENDIDO/CANCELADO/AUSENTE), copagos, recordatorios automáticos, y datos sensibles (motivo de consulta, notas clínicas). Distinto del de fidelización, pero estructuralmente las mismas piezas.

**Checklist de arranque, en orden:**

**Fase 0 — Esqueleto (antes de cualquier feature)**
- [ ] **U-35** — Composition root que registra cada módulo (agenda, pacientes, pagos, recordatorios) explícitamente.
- [ ] **U-36** — Routes flacas; la lógica de turnos/copagos en services.
- [ ] **U-38** — Paquete/fachada compartido para tipos, validaciones y catálogos.
- [ ] **U-25 + U-26 + U-30** — `.env.example` documentado, lectura de entorno tipada, validación de config al boot (rechazar `JWT_SECRET` débil; falla si falta `ENCRYPTION_KEY`).
- [ ] **U-29** — Script de arranque con defaults de runtime y migración una sola vez.

**Fase 1 — La cadena multi-tenant (NO negociable, va antes que las features)**
- [ ] **U-01** — `profesional_id`/`clinica_id` sale del token, nunca de `?profesional_id=`.
- [ ] **U-04** — Sin tenant en el token → 401, antes de tocar la base.
- [ ] **U-02** — Cada query de pacientes/turnos/pagos lleva `WHERE tenant_id = <del token>`.
- [ ] **U-06** — Antes de reagendar/cobrar contra un `turno_id` o `paciente_id` del body, ownership guard.
- [ ] **U-03** — Un turno o paciente de otro profesional devuelve 404, no 403.
- [ ] **U-05** — Revalidar contra DB que el profesional sigue activo en cada request.
- [ ] **U-07** — 401 global en el cliente → logout + redirect.
- [ ] **U-34** — Test atacante: parado en el profesional A, intentar ver/mutar la agenda del profesional B y exigir el fallo.

**Fase 2 — La base del negocio**
- [ ] **U-08** — Molde de tabla (PK opaca + timestamps) para Paciente, Turno, Pago; ledger inmutable para los pagos (Tensión T-1).
- [ ] **U-09** — Copagos y aranceles con decimal, jamás float.
- [ ] **U-10** — Índices tenant-first; unicidad "un turno por profesional+franja horaria" como constraint compuesto liderado por el tenant.
- [ ] **U-11** — `EstadoTurno` como enum tipado, reusado en datos, validación y lógica.
- [ ] **U-12** — Notas clínicas / motivo de consulta cifrados en reposo (datos de salud).
- [ ] **U-13 + U-14** — `TurnoCreate` (sin estado ni arancel calculado) ≠ `TurnoResponse`; "¿la franja está libre?" en el service, no en el validador estructural.

**Fase 3 — Por cada endpoint y el dinero**
- [ ] **U-16 + U-17** — Reservar un turno corre en transacción con lock pesimista sobre la franja: dos pacientes no toman el mismo horario (el caso de oro del lock).
- [ ] **U-18 + U-19 + U-20** — "Horario no disponible" → 409 en el borde; stack al log, mensaje genérico al paciente; logging por gravedad.
- [ ] **U-22 + U-23** — Webhook de copago: firma timing-safe fail-closed + idempotencia (reenviar el webhook no cobra dos veces) + re-verificación server-to-server del estado (T-6).
- [ ] **U-21 + U-24 + U-27** — El recordatorio por WhatsApp/SMS degrada si el proveedor está caído; el job de recordatorios es resiliente e idempotente (no manda dos veces el mismo recordatorio el mismo día); el envío real, apagado por default hasta configurarlo.
- [ ] **U-15** — El front manda fechas en ISO-UTC y parsea el error (lista de validación vs string).

**Fase 4 — Terminado**
- [ ] **U-31** — Tests de integración contra DB real (o al menos en CI), mockeando solo WhatsApp/pagos.
- [ ] **U-32** — Casos negativos: doble reserva de la misma franja, turno de otro tenant (404), payload inválido (400); + la invariante de idempotencia del cobro.
- [ ] **U-33** — Un comando único de QA/CI con cobertura mínima y la matriz de permisos por rol (profesional vs recepción vs paciente) gatea el release.

> Fijate el patrón: **no inventás arquitectura nueva, ensamblás tornillos ya probados.** Un sistema de turnos de salud es, estructuralmente, cualquier sistema de turnos/reservas multi-tenant con otro nombre y datos específicos del dominio.

---

## Regla de oro (vale en cualquier lenguaje)

> El identificador de tenant sale del token firmado y viaja en el filtro de toda consulta; el contrato (entrada, salida y error) vive en un solo lugar y la salida nunca es el modelo crudo; el dinero corre en transacción con lock e idempotencia; y nada se da por "terminado" hasta que un test que actúa como atacante intenta cruzar tenants y **falla**. Si dudás entre filtrar de más o devolver de más, **negá**: una fuga de datos no se perdona, un 404 de más sí.
