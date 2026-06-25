# Szoluciones — Tornillero Python (FastAPI + SQLAlchemy 2.0 + Pydantic v2 + Alembic + PostgreSQL + React)

Bienvenido. Esto es el "tornillero": la caja de tornillos que un Senior usa una y otra vez en esta API real de gestión de psicólogas (multi-tenant por `psicologa_id`, con integraciones de WhatsApp, LLM y pagos). Cada tornillo es un patrón concreto, anclado a un archivo y una línea del repo, con la regla destilada para que lo reuses en tu próximo proyecto.

Te lo explico como si recién entraras al equipo: cuando aparezca un término técnico, lo aclaro entre paréntesis la primera vez. Tono de colega, precisión de manual.

Un par de definiciones que se repiten todo el tiempo:

- **Multi-tenant** (multi-inquilino): una sola base de datos y una sola app sirven a muchas psicólogas, y cada una solo ve lo suyo. El "inquilino" (tenant) se identifica por `psicologa_id`.
- **Fail-closed** (falla cerrado): ante la duda o el error, el sistema deniega/aborta en vez de dejar pasar. Lo opuesto es fail-open (peligroso).
- **ORM** (Object-Relational Mapping): la capa que mapea filas de la base a objetos Python; acá es SQLAlchemy.
- **Service** (servicio): módulo Python donde vive la regla de negocio, separado del handler HTTP.

---

## ÍNDICE DE TORNILLOS

### Área 1 — EL PLANO DE LA CASA (organización del proyecto)
- TORNILLO #1 — El composition root: un solo lugar arma la app *(Universal)*
- TORNILLO #2 — Capas estrictas: route flaca, service gordo *(Universal)*
- TORNILLO #3 — Fachada `__init__` que re-exporta la API pública *(Universal)*
- TORNILLO #4 — Archivos auto-generados con banner de "fuente única de verdad" *(Solo este proyecto)*
- TORNILLO #72 — Proveedores intercambiables tras una interfaz: ABC + Strategy + factory por env *(Universal)*

### Área 2 — LA PUERTA Y EL PORTERO (auth y autorización)
- TORNILLO #5 — 404 en vez de 403 para datos de otro tenant *(Universal)*
- TORNILLO #6 — Scoping por tenant en cada query, derivado del token *(Universal)*
- TORNILLO #7 — Fail-closed en `get_usuario_actual` (revalidar contra DB) *(Universal)*
- TORNILLO #8 — Validar config de seguridad crítica al boot *(Universal)*
- TORNILLO #9 — Detección de secretos débiles por marcadores *(Universal)*
- TORNILLO #10 — Dependency factory para roles (`require_rol`) *(Solo Python)*
- TORNILLO #11 — Scope helper como guard previo en escrituras *(Universal)*
- TORNILLO #12 — bcrypt con fallback PBKDF2 y verify por prefijo *(Solo Python)*
- TORNILLO #13 — 401 global en el cliente: limpia sesión y redirige *(Universal)*
- TORNILLO #71 — Ruteo de tenant fail-closed por identificador entrante (`phone_number_id`) *(Universal)*

### Área 3 — EL VIAJE DE UN DATO (pantalla → route → service → model → schema → response)
- TORNILLO #14 — Schema de entrada distinto del de salida *(Universal)*
- TORNILLO #15 — El commit vive en el handler, el service hace flush *(Solo Python)*
- TORNILLO #16 — Validación en capas: Pydantic estructural + service de negocio *(Solo Python)*
- TORNILLO #17 — `ValueError` del service → `HTTPException` con status por contexto *(Universal)*
- TORNILLO #18 — Lock pesimista en transiciones de estado (`SELECT FOR UPDATE`) *(Solo Python)*
- TORNILLO #19 — La salida nunca es el ORM crudo: `response_model` + encoder UTC *(Solo Python)*
- TORNILLO #20 — El cliente normaliza el dato y parsea el error de Pydantic *(Universal)*

### Área 4 — EL ARCHIVO DEL NEGOCIO (modelos y conexión a BD)
- TORNILLO #21 — PK uuid como String, no autoincrement *(Solo este proyecto)*
- TORNILLO #22 — Dinero con Numeric/Decimal, nunca float *(Universal)*
- TORNILLO #23 — Timestamps `created_at`/`updated_at` con `onupdate` *(Universal)*
- TORNILLO #24 — Enums `str, Enum` para estados y catálogos *(Universal)*
- TORNILLO #25 — Índices y unicidad tenant-first *(Universal)*
- TORNILLO #26 — Encriptación transparente vía `hybrid_property` (Fernet) *(Solo Python)*
- TORNILLO #27 — Pool de conexiones endurecido y configurable *(Universal)*
- TORNILLO #28 — `init_db` detecta drift y exige migración explícita *(Solo este proyecto)*

### Área 5 — CUANDO ALGO SE ROMPE (errores, logging, mensajes)
- TORNILLO #29 — Stacktrace adentro, mensaje genérico afuera *(Universal)*
- TORNILLO #30 — Un logger por módulo con `__name__` y niveles con intención *(Universal)*
- TORNILLO #31 — Fail-closed en firmas/tokens de webhook *(Universal)*
- TORNILLO #32 — Proveedor externo caído no rompe el flujo (degradación) *(Universal)*
- TORNILLO #33 — Cron con try/rollback/finally que cuenta errores en vez de morir *(Universal)*
- TORNILLO #34 — Error boundary de render + interceptor 401 en el frontend *(Universal)*

### Área 6 — LA LETRA DEL SENIOR (estilo personal Python)
- TORNILLO #35 — Docstring de módulo en español al tope *(Solo este proyecto)*
- TORNILLO #36 — Helpers `_env_bool`/`_env_int` para leer entorno tipado *(Universal)*
- TORNILLO #37 — `typing.cast` para domar columnas SQLAlchemy *(Solo Python)*
- TORNILLO #38 — Triada `get_`/`require_`/`upsert_`/`serialize_`/`delete_` por entidad *(Solo este proyecto)*
- TORNILLO #39 — Guard clauses con `HTTPException`, sin else anidado *(Universal)*
- TORNILLO #40 — Helpers privados `_` de normalización/parseo reutilizados *(Solo Python)*
- TORNILLO #41 — Type hints completos + keyword-only para parámetros de dominio *(Solo Python)*
- TORNILLO #42 — Raw SQL confinado y parametrizado (anti-patrón evitado) *(Universal)*
- TORNILLO #43 — Imports explícitos de helpers con guion bajo *(Solo este proyecto)*

### Área 7 — LAS LLAVES DEL LOCAL (env, config por ambiente, secrets)
- TORNILLO #44 — `.env.example` versionado como fuente de verdad, `.env` ignorado *(Universal)*
- TORNILLO #45 — Comandos para generar secretos embebidos en el template *(Universal)*
- TORNILLO #46 — Defaults distintos por ambiente (dev/test vs prod) *(Universal)*
- TORNILLO #47 — Validación fail-fast de config al importar el módulo *(Universal)*
- TORNILLO #48 — Secretos productivos inyectados por la plataforma, no en el repo *(Universal)*
- TORNILLO #49 — Normalizar `DATABASE_URL` y validar el driver *(Solo Python)*
- TORNILLO #50 — Defaults de runtime en el script de arranque *(Universal)*

### Área 8 — CUANDO ESTÁ TERMINADO (testing, regresión, CI)
- TORNILLO #51 — El nombre del test ES la documentación del bug *(Universal)*
- TORNILLO #52 — Fixture `db_session` efímera in-memory por test *(Solo Python)*
- TORNILLO #53 — `monkeypatch` para cortar todo el IO externo *(Solo Python)*
- TORNILLO #54 — Fixture global autouse que mata el cache no-determinista *(Solo Python)*
- TORNILLO #55 — Idempotencia y reintentos como invariante testeada *(Universal)*
- TORNILLO #56 — `tests/manual` excluido del runner por diseño *(Universal)*
- TORNILLO #57 — CI gatea por cobertura y prueba contra la DB real *(Universal)*
- TORNILLO #58 — Reloj y entorno deterministas en tests *(Solo Python)*

### Área 9 — LO QUE PYTHON HACE MEJOR (idiomático)
- TORNILLO #59 — Generator como recurso con cierre garantizado (`get_db`) *(Solo Python)*
- TORNILLO #60 — Imports opcionales con degradación elegante *(Solo Python)*
- TORNILLO #61 — Context managers de SQLAlchemy con `execution_options` *(Solo Python)*
- TORNILLO #62 — Registry inmutable con dataclasses frozen (matcher→prepare→execute) *(Solo Python)*
- TORNILLO #63 — `@asynccontextmanager` para el lifespan de la app *(Solo Python)*

### Área 10 — CÓMO CRECE ESTO (expansión vertical)
- TORNILLO #64 — El orden sagrado de 8 capas para una feature nueva *(Solo este proyecto)*
- TORNILLO #65 — Migración idempotente con inspector y guards *(Solo este proyecto)*
- TORNILLO #66 — Enum Postgres idempotente (`DO $$ ... duplicate_object`) *(Solo Python)*
- TORNILLO #67 — Migración encadenada por fecha+secuencia *(Solo Python)*
- TORNILLO #68 — Guardrails por nivel de riesgo + confirmación *(Universal)*
- TORNILLO #69 — Integración paralela clonable (Meta/MercadoPago/Mobbex) + facade *(Universal)*
- TORNILLO #70 — Policy chain con precedencia y snapshot congelado *(Universal)*

### Área 11 — AGENTES CONVERSACIONALES (un agente de IA que habla con personas reales)
- TORNILLO PY-16 — El bot se calla cuando hay un humano *(Universal)*
- TORNILLO PY-17 — La confianza tiene umbral: debajo de 0.55 no actúes *(Universal)*
- TORNILLO PY-18 — Resolver antes de gastar tokens *(Universal)*
- TORNILLO PY-19 — El handoff lleva contexto, no solo una notificación *(Universal)*
- TORNILLO PY-20 — Historial acotado para el LLM *(Universal)*

---

## ÁREA 1 — EL PLANO DE LA CASA

### TORNILLO #1 — El composition root: un solo lugar arma la app
**En una línea:** Todos los routers se registran centralizadamente en `main.py`; nada se auto-descubre por magia.
**Lo que hace el Senior:** Importa cada router por nombre desde `app.routes` y los engancha con `app.include_router()`; cada router declara su propio `prefix` y `tags`. `main.py` solo orquesta logging, CORS, lifespan (init de DB + scheduler) y montaje de routers. Cero lógica de negocio en el composition root (la raíz donde se "compone" la app conectando sus piezas).
**Ejemplo real del repo:** `app/main.py:149` `app.include_router(auth.router)` ... `:158` `app.include_router(consultorios.router)`; y `app/routes/clientes.py:56` `router = APIRouter(prefix="/clientes", tags=["Clientes"])`.
**La regla para el próximo proyecto:** Registrá los routers explícitamente en el composition root con `include_router`; cada módulo de ruta crea su propio router con `prefix` y `tags`. Nada de auto-discovery ni setup de app desparramado.
**Señal de que lo estás usando bien:** Agregar un endpoint nuevo = crear/editar un archivo en `routes/` con su router y agregar UNA línea `include_router` en `main.py`; nada más se toca.
**Aplica a:** [x] Universal

### TORNILLO #2 — Capas estrictas: route flaca, service gordo
**En una línea:** El handler HTTP solo inyecta dependencias y delega; la regla de negocio vive en `services/`.
**Lo que hace el Senior:** Cada endpoint recibe `db: Session = Depends(get_db)` y `usuario = Depends(require_psicologa)`, valida con schemas Pydantic y llama a una función de `services/`. La lógica financiera, de políticas y de turnos no toca el route; `models.py` es solo persistencia.
**Ejemplo real del repo:** `app/routes/clientes.py:129-130` `db: Session = Depends(get_db)` / `usuario=Depends(require_psicologa)` (repetido en 7+ endpoints); ~30 archivos `*_service.py` en `app/services/`. Declarado en `.github/copilot-instructions.md:6` ("Keep route handlers thin").
**La regla para el próximo proyecto:** En `routes/` poné solo: deps vía `Depends`, validación con schemas y delegación a una función de `services/`. Cualquier comportamiento de dominio nuevo va a un `*_service.py`, no al handler.
**Señal de que lo estás usando bien:** Un handler entra en pocas líneas y no contiene queries complejas ni cálculos de negocio. Si estás escribiendo lógica financiera dentro de un route, está mal ubicada.
**Aplica a:** [x] Universal

### TORNILLO #3 — Fachada `__init__` que re-exporta la API pública
**En una línea:** El paquete expone una API curada vía `__init__` con `__all__`, ocultando la fragmentación interna.
**Lo que hace el Senior:** `operativo/__init__.py` importa desde `_core`/`_handlers` las funciones que son la interfaz pública del módulo y las re-exporta con `__all__`. Los archivos internos llevan prefijo `_` para señalar "privado del paquete" (`_core`, `_shared`, `_action_registry`, `_handlers_*`, `_wizards`, etc.).
**Ejemplo real del repo:** `app/services/operativo/__init__.py:4-10` `from ._core import (procesar_mensaje_operativo_psicologa, ...)` y `:13-20` `__all__ = [...]`.
**La regla para el próximo proyecto:** Cuando partas un módulo grande en submódulos internos, dejá un `__init__` que re-exporte solo la API pública con `__all__`; nombrá los submódulos con prefijo `_` para marcar que son detalle interno.
**Señal de que lo estás usando bien:** Quien usa el paquete importa desde `app.services.operativo`, no desde `._core`/`._shared`; el `__init__` lista en `__all__` exactamente lo público.
**Aplica a:** [x] Universal

### TORNILLO #4 — Archivos auto-generados con banner de "fuente única de verdad"
**En una línea:** Los módulos derivados de un split avisan en la cabecera de dónde se generan, para no editarlos a mano.
**Lo que hace el Senior:** Los submódulos grandes (resultado de un split automático) llevan un banner en la línea 2 que indica la fuente y cómo regenerarlos, documentando la relación fuente→derivado en el propio código.
**Ejemplo real del repo:** `app/services/operativo/_core.py:2` `# THIS FILE IS AUTO-GENERATED — edit psicologa_whatsapp_operativo_service.py and re-run _split_operativo.py`; banner idéntico en `_shared.py:2`.
**La regla para el próximo proyecto:** Si un archivo es generado/derivado, poné en la cabecera un banner que indique la fuente y cómo regenerarlo; editá la fuente, no el derivado.
**Señal de que lo estás usando bien:** Antes de editar un archivo grande, leés la línea 2; si dice AUTO-GENERATED, vas a la fuente en lugar de tocar el derivado.
**Aplica a:** [x] Solo este proyecto

### TORNILLO #72 — Proveedores intercambiables tras una interfaz (ABC + Strategy + factory por env)
**En una línea:** Cada familia de proveedores externos (LLM, notificaciones) se modela como una clase base abstracta con una subclase por proveedor, y un factory elige cuál instanciar según una variable de entorno.
**Lo que hace el Senior:** Para servicios con comportamiento real (no solo CRUD de credenciales) define una ABC (Abstract Base Class, clase base abstracta de Python) con `@abstractmethod`, una subclase concreta por proveedor que comparte la firma y un atributo `provider_label`, y una función factory `get_*_provider()` que lee la env (`LLM_PROVIDER`, `NOTIFICATION_PROVIDER`) y devuelve la instancia correcta. El resto del código depende de la interfaz, no del proveedor concreto: agregar un proveedor = una subclase + una rama en el factory. Es el patrón Strategy (estrategias intercambiables detrás de una interfaz común). Convive A PROPÓSITO con el "molde clonable de funciones libres + facade" del #69: aquel se usa para la config/credenciales (CRUD repetitivo de `IntegracionX`), este para la lógica polimórfica (comportamiento que varía por proveedor).
**Ejemplo real del repo:** `app/services/llm_service.py:256` `class LLMProvider(ABC):` con `@abstractmethod` (`:259,269,279`), `:319` `class BaseLLMProvider(LLMProvider): provider_label = "llm"`, `:378` `class OpenAIProvider(BaseLLMProvider)`, `:588` `class AnthropicProvider(BaseLLMProvider)`, y factory `:799` `def get_llm_provider() -> LLMProvider: provider_name = os.getenv("LLM_PROVIDER", "openai")...`. Mismo esquema en `notification_service.py:61` `class BaseNotificationProvider(ABC)` → `LogNotificationProvider`(`:76`)/`TwilioNotificationProvider`(`:92`)/`MetaNotificationProvider`(`:146`) + factory `get_provider()` (`:274`) leyendo `NOTIFICATION_PROVIDER`.
**La regla para el próximo proyecto:** Cuando tengas varios proveedores intercambiables de un mismo comportamiento, definí una interfaz/ABC con los métodos que todos deben cumplir, una implementación por proveedor y un único factory que seleccione por env/config. El código cliente depende de la abstracción; sumar un proveedor no toca a los consumidores. Reservá este patrón polimórfico para lógica que varía por proveedor; para el CRUD de config repetitivo, el molde clonable + facade (#69) alcanza.
**Señal de que lo estás usando bien:** Cambiar de proveedor es cambiar una variable de entorno, no tocar código; los consumidores nunca hacen `if provider == "openai"`, solo llaman a métodos de la interfaz; agregar un proveedor es una subclase nueva + una rama en el factory.
**Aplica a:** [x] Universal

---

## ÁREA 2 — LA PUERTA Y EL PORTERO

### TORNILLO #5 — 404 en vez de 403 para datos de otro tenant
**En una línea:** El recurso de otra psicóloga "no existe" para vos: misma URL, distinta respuesta, sin filtrar la existencia del dato ajeno.
**Lo que hace el Senior:** Centraliza el control de ownership (dueñez) en helpers `obtener_*_de_psicologa` que filtran por `id` AND `psicologa_id` en la misma query; si no aparece, lanzan `HTTPException(404)`. No hay un 403 "esto existe pero no es tuyo" que confirmaría la existencia. El mismo helper hace autorización y carga del objeto en un solo paso.
**Ejemplo real del repo:** `app/services/psicologa_scope_service.py:11` `cliente = db.query(Cliente).filter(Cliente.id == cliente_id, Cliente.psicologa_id == psicologa_id).first(); if cliente is None: raise HTTPException(status_code=404, detail="Cliente no encontrado")`. Los 5 helpers (cliente/turno/disponibilidad/transacción/conversación) usan el mismo patrón en `:11,21,31,45,59`.
**La regla para el próximo proyecto:** En datos multi-tenant no separes "¿existe?" de "¿es tuyo?". Filtrá por `id` AND `tenant_id` en una sola query y devolvé 404 (no 403) si no aparece. Encapsulalo en un helper reutilizable que devuelva el objeto ya autorizado.
**Señal de que lo estás usando bien:** Pedir un id válido de otro tenant devuelve 404 idéntico al de un id inexistente; ningún endpoint hace su propio `.filter(Model.id==x).first()` seguido de un check de ownership aparte.
**Aplica a:** [x] Universal

### TORNILLO #6 — Scoping por tenant en cada query, derivado del token
**En una línea:** Toda consulta de negocio lleva `.filter(Model.psicologa_id == usuario.id)`, derivado del token, no de un parámetro del request.
**Lo que hace el Senior:** El tenant nunca viene del cliente: se toma de `usuario.id` (el sujeto autenticado del JWT vía `get_usuario_actual`). Cada query de listados, conteos, reportes y agregaciones incluye explícitamente el filtro. Así un atacante no puede pasar `?psicologa_id=otra`: ese parámetro no existe; el scope es implícito y server-side.
**Ejemplo real del repo:** `app/routes/admin.py:103` `total_clientes = db.query(func.count(Cliente.id)).filter(Cliente.psicologa_id == psicologa_id, Cliente.activo == True).scalar()`. El filtro directo (`psicologa_id ==`) aparece en ~160 consultas a lo largo de ~36 archivos, dentro de >1400 referencias totales a `psicologa_id` en `app/` (verificado por `grep`).
**La regla para el próximo proyecto:** Derivá el `tenant_id` del usuario autenticado (token), nunca de un query param o body. Aplicá el filtro de tenant en TODA query de lectura y escritura, incluso conteos y agregaciones, no solo en los GET por id.
**Señal de que lo estás usando bien:** Ningún endpoint acepta `psicologa_id` como parámetro de entrada para datos propios; toda query referencia `usuario.id`, y los reportes agregados también están filtrados.
**Aplica a:** [x] Universal

### TORNILLO #7 — Fail-closed en `get_usuario_actual` (revalidar contra DB)
**En una línea:** Usuario inexistente o inactivo → 401 con el mismo mensaje, sin distinguir el caso.
**Lo que hace el Senior:** Tras decodificar el token valida que el usuario exista en DB y esté activo. Ambos fallos devuelven 401 "Usuario no encontrado" idéntico, sin revelar si fue desactivado vs nunca existió. Un token firmado válido de un usuario ya desactivado no da acceso (revocación efectiva sin lista negra de tokens).
**Ejemplo real del repo:** `app/services/jwt_service.py:188` `if not cast(bool, usuario.activo): raise HTTPException(status_code=status.HTTP_401_UNAUTHORIZED, detail="Usuario no encontrado")`. Login refuerza con `Usuario.activo.is_(True)` y mensaje genérico "Credenciales incorrectas" (`auth.py:64,72,77`).
**La regla para el próximo proyecto:** No confíes solo en la firma del JWT: revalidá contra DB que el sujeto existe y sigue activo en cada request. Usá el mismo mensaje 401 para "no existe" e "inactivo" para no filtrar estado de cuentas.
**Señal de que lo estás usando bien:** Desactivar un usuario invalida sus tokens vigentes al instante; login y checks devuelven mensajes genéricos.
**Aplica a:** [x] Universal

### TORNILLO #8 — Validar config de seguridad crítica al boot
**En una línea:** Si `ENV=production` y el secreto es corto/débil o el algoritmo no está en allowlist, el proceso no arranca.
**Lo que hace el Senior:** Al importar el módulo (import-time) corre `_validar_configuracion_jwt()`: valida que el algoritmo esté en allowlist `{HS256,HS384,HS512}` (evita `none`), y solo en producción exige secreto fuerte, issuer y audience. Falla con `RuntimeError` que aborta el arranque. En dev es permisivo.
**Ejemplo real del repo:** `app/services/jwt_service.py:97` `_validar_configuracion_jwt()` (invocación top-level); `:86` `if _es_produccion(): if _secreto_debil(JWT_SECRET): raise RuntimeError(...)`. `jwt.decode` usa `algorithms=[JWT_ALGORITHM]` (`:166`).
**La regla para el próximo proyecto:** Validá la config de seguridad crítica al boot, no en runtime: secreto débil o algoritmo fuera de allowlist → abortar el arranque en producción. Distinguí dev (permisivo) de prod (estricto) por una variable `ENV` explícita, y exigí issuer/audience en prod.
**Señal de que lo estás usando bien:** Desplegar en prod con `JWT_SECRET=dev_secret_change_in_production` o <32 chars hace fallar el contenedor al iniciar, no en el primer request; nunca se acepta `none`.
**Aplica a:** [x] Universal

### TORNILLO #9 — Detección de secretos débiles por marcadores
**En una línea:** En producción rechaza secretos cortos o que contengan strings típicos de placeholder.
**Lo que hace el Senior:** Define `_secreto_debil`: longitud < 32, o que contenga marcadores típicos de placeholder/ejemplo. Así un `.env` copiado sin editar no llega a prod.
**Ejemplo real del repo:** `app/services/jwt_service.py:38-46` `_WEAK_SECRET_MARKERS = ("replace_with", "tu_secreto", "change_in_production", "dev_secret", "example", "password", "123456")` y `:71-76` `def _secreto_debil(secret): if len(secret) < 32: return True; ... return any(marker in secret_lower for marker in _WEAK_SECRET_MARKERS)`.
**La regla para el próximo proyecto:** En prod no solo exijas que el secreto exista: rechazalo si es corto (<32) o contiene marcadores de placeholder. Mantené una lista de markers de los defaults/ejemplos que usás.
**Señal de que lo estás usando bien:** Dejar el secreto por defecto hace fallar el arranque en prod; un secreto random de 64 hex pasa.
**Aplica a:** [x] Universal

### TORNILLO #10 — Dependency factory para roles (`require_rol`)
**En una línea:** Una factory (función que devuelve una función) genera dependencies de FastAPI por rol; los endpoints solo declaran `Depends(require_superadmin/require_psicologa)`.
**Lo que hace el Senior:** `require_rol(*roles)` recibe roles variádicos y devuelve una closure `check` que captura `roles` en su scope léxico y depende de `get_usuario_actual`; si el rol no coincide lanza 403. Crea shortcuts pre-aplicados (`require_psicologa = require_rol(Rol.PSICOLOGA)`) reusados en todos los routers. La autorización queda declarativa en la firma, sin `if-rol` disperso.
**Ejemplo real del repo:** `app/services/jwt_service.py:193` `def require_rol(*roles): def check(usuario=Depends(get_usuario_actual)): if usuario.rol not in roles: raise HTTPException(403, ...); return usuario; return check`; `:223` `require_superadmin = require_rol(Rol.SUPERADMIN)`.
**La regla para el próximo proyecto:** Cuando necesites una dependency configurable (rol, permiso, scope), escribí una factory que cierre sobre los parámetros y devuelva la función interna usada en `Depends`. Creá constantes pre-aplicadas para los casos comunes.
**Señal de que lo estás usando bien:** El endpoint declara `usuario=Depends(require_psicologa)` sin repetir el chequeo; agregar un rol es una línea, no copiar/pegar un `if`.
**Aplica a:** [x] Solo Python

### TORNILLO #11 — Scope helper como guard previo en escrituras
**En una línea:** Antes de mutar un recurso referenciado por id en el body, se llama al helper de ownership aunque no se use el objeto devuelto.
**Lo que hace el Senior:** En operaciones donde un `cliente_id`/`turno_id` llega en el body (cobro manual, crear transacción), invoca `obtener_*_de_psicologa(db, id, usuario.id)` como guard: si el id es de otro tenant, corta con 404 antes de procesar. A veces descarta el return; lo importante es el efecto de autorización+404.
**Ejemplo real del repo:** `app/routes/admin.py:339` `obtener_cliente_de_psicologa(db, datos.cliente_id, usuario.id)` (guard antes de `procesar_pago_parcial`; el `cliente_id` viene del body). Repetido en `transacciones.py:184,187,221` y `turnos.py:116`.
**La regla para el próximo proyecto:** Cuando un id de recurso llega por body/query y vas a operar sobre él, validá ownership con el helper de tenant ANTES de la lógica de negocio, incluso si no necesitás el objeto. No asumas que un id en el payload es del tenant del usuario.
**Señal de que lo estás usando bien:** Intentar cobrar/transaccionar contra un cliente de otra psicóloga da 404, no un error de negocio más adentro.
**Aplica a:** [x] Universal

### TORNILLO #12 — bcrypt con fallback PBKDF2 y verify por prefijo
**En una línea:** `hash_password` usa bcrypt y degrada a PBKDF2-sha256 si falta la dependencia; `verify_password` detecta el esquema por el prefijo del hash.
**Lo que hace el Senior:** Para sobrevivir entornos sin bcrypt (Python 3.14+/tests), `hash_password` usa bcrypt si está disponible y si no genera un hash con prefijo `pbkdf2_sha256$salt$digest`. `verify_password` detecta el esquema por el prefijo del hash almacenado y usa `hmac.compare_digest` (comparación constant-time, resistente a timing attacks) para el fallback. Errores de parseo devuelven `False`, no excepción.
**Ejemplo real del repo:** `app/services/jwt_service.py:125` `if hashed.startswith("pbkdf2_sha256$"): ... return hmac.compare_digest(actual, expected) ... if bcrypt is None: return False; return bcrypt.checkpw(...)`. Doble esquema en `:102-142`, `try/except (ValueError, TypeError) -> False`.
**La regla para el próximo proyecto:** Si necesitás un fallback de hashing, marcá cada hash con un prefijo de esquema y seleccioná el verificador por ese prefijo (no por config global), así los hashes viejos siguen verificando tras cambiar de algoritmo. Usá comparación constant-time y devolvé `False` ante hash corrupto en vez de lanzar.
**Señal de que lo estás usando bien:** Un hash bcrypt existente sigue validando aunque bcrypt esté ausente (devuelve False sin romper); los hashes nuevos llevan prefijo identificable; verify nunca propaga `ValueError`/`TypeError`.
**Aplica a:** [x] Solo Python

### TORNILLO #13 — 401 global en el cliente: limpia sesión y redirige
**En una línea:** Un interceptor de respuesta convierte cualquier 401 del backend en logout + redirect a `/login`.
**Lo que hace el Senior:** El cliente trata el token como opaco para la auth real: lo inyecta en cada request solo si `hasValidSession()` (chequea `exp` y `rol` localmente para UX). Si el backend responde 401 (token vencido/revocado/usuario inactivo), un interceptor hace `clearAuth()` y redirige. La fuente de verdad es el backend; el decode local es solo presentación.
**Ejemplo real del repo:** `frontend/src/api/client.js:42` `if (error.response?.status === 401) { clearAuth(); window.location.href = "/login"; }`. `hasValidSession` valida `exp` fail-closed en `auth.js:101-116`.
**La regla para el próximo proyecto:** En el frontend tratá el JWT como opaco para autorizar (el backend decide); decodificalo solo para UX (rol, expiración). Centralizá el manejo de 401 en un interceptor que limpie sesión y redirija, en vez de repetir el catch en cada llamada.
**Señal de que lo estás usando bien:** El token expira o el server revoca y la UI salta sola al login sin código ad-hoc por endpoint; la validación de rol del front es solo gating de navegación, no de datos.
**Aplica a:** [x] Universal

### TORNILLO #71 — Ruteo de tenant fail-closed por identificador entrante (`phone_number_id`)
**En una línea:** Un webhook compartido por todas las psicólogas resuelve a qué tenant pertenece el mensaje mirando el identificador entrante (el `phone_number_id` de Meta); si no matchea una integración activa, corta — no adivina ni cae a un default.
**Lo que hace el Senior:** El endpoint de Meta no recibe un `psicologa_id` confiable en el payload: resuelve el tenant buscando el `phone_number_id` entrante contra las integraciones Meta ACTIVAS (`get_meta_config_by_phone_number_id(db, phone_number_id, only_active=True)`). Si hay config activa, usa su `psicologa_id`; si no hay ninguna (o no vino ni `psicologa_id` ni `phone_number_id`), lanza `HTTPException(400)` indicando el id que no pudo resolver. Es el caso "mismo endpoint, distinto dueño según quién escribe" resuelto fail-closed: ante ambigüedad, deniega en vez de adivinar el destinatario.
**Ejemplo real del repo:** `app/routes/webhooks.py:105` `if phone_number_id: config = get_meta_config_by_phone_number_id(db, phone_number_id, only_active=True); if config is not None: return cast(str, config.psicologa_id)`; `:109` `raise HTTPException(status_code=400, detail=f"No existe una integración Meta activa para el phone_number_id '{phone_number_id}'.")`; `:120` `raise HTTPException(status_code=400, detail="psicologa_id o phone_number_id son obligatorios para enrutar mensajes Meta")`. Helper en `app/services/meta_integration_service.py`. Declarado como invariante de producción en `.github/copilot-instructions.md:7` ("must not weaken the fail-closed Meta WhatsApp routing by `phone_number_id`").
**La regla para el próximo proyecto:** Cuando un canal de entrada (webhook, número, dominio) es compartido por varios tenants, resolvé el tenant desde el identificador entrante contra un registro de integraciones ACTIVAS y fallá cerrado si no hay match único: nunca caigas a un tenant por defecto ni adivines. El identificador entrante decide el destino con el mismo rigor con que la firma decide el origen.
**Señal de que lo estás usando bien:** Un mensaje a un `phone_number_id` sin integración activa devuelve error y no se procesa contra ningún tenant; no existe una rama "si no sé de quién es, mandáselo al default".
**Aplica a:** [x] Universal

---

## ÁREA 3 — EL VIAJE DE UN DATO

### TORNILLO #14 — Schema de entrada distinto del de salida
**En una línea:** Cada recurso tiene `Create`/`Update` separados del `Response`, nunca un schema único reutilizado.
**Lo que hace el Senior:** Define `TurnoCreate` (lo que el cliente puede mandar) aparte de `TurnoResponse` (lo que el server controla: `id`, `estado`, `tarifa_base`, `origen_politica`, timestamps). El `Response` hereda de un Base con `from_attributes` (lee del ORM); los `Create`/`Update` jamás exponen campos derivados/calculados.
**Ejemplo real del repo:** `app/schemas.py:224` `class TurnoCreate(BaseModel): cliente_id: str; ...; tarifa: Optional[Decimal] = Field(None, ge=0, le=Decimal("999999.99"))` vs `:298` `class TurnoResponse(TurnoBase): id: str; estado: EstadoTurno; ... model_config = {"from_attributes": True}`. Repetido en Cliente, Transaccion, Disponibilidad, Consultorio.
**La regla para el próximo proyecto:** Para cada entidad creá schemas separados: `XCreate`/`XUpdate` (entrada, solo campos que el usuario controla) y `XResponse` (salida, con `from_attributes`). Nunca uses el mismo modelo para request y response, ni expongas en el Create campos que el backend calcula.
**Señal de que lo estás usando bien:** El `response_model` del endpoint es un `*Response` con `id`/`created_at`/campos derivados, y el body tipado es un `*Create`/`*Update` que NO los tiene.
**Aplica a:** [x] Universal

### TORNILLO #15 — El commit vive en el handler, el service hace flush
**En una línea:** Los services mutan el ORM y hacen `flush()`, nunca `commit()`; el handler commitea al final.
**Lo que hace el Senior:** El service construye el objeto, aplica la política y hace `session.flush()` (envía los cambios a la DB pero sin cerrar la transacción). El handler hace `db.commit()` + `db.refresh()` recién cuando todos los services terminaron. Así una transacción por request permite componer varios services antes de commitear.
**Ejemplo real del repo:** `app/routes/turnos.py:78` `turno = reservar_turno(db, ...)` luego `:96` `db.commit(); db.refresh(turno); return turno`; el service `turno_booking_service.py:221` hace `session.add(turno); session.flush()` SIN commit.
**La regla para el próximo proyecto:** Los services mutan el ORM y hacen `flush()`, nunca `commit()`. El handler es quien hace `commit()`+`refresh()` después de que todos los services involucrados terminaron. Una transacción por request, services componibles.
**Señal de que lo estás usando bien:** Buscás `commit()` en los services y no aparece; aparece solo en los handlers. `flush()` aparece en services.
**Aplica a:** [x] Solo Python

### TORNILLO #16 — Validación en capas: Pydantic estructural + service de negocio
**En una línea:** Pydantic valida forma/rango/coherencia de campos; el service valida reglas que necesitan la DB (slot libre, transición válida).
**Lo que hace el Senior:** Field constraints (`ge=0`, `le=999999.99`) y `model_validator` (`fecha_fin > fecha_inicio`, fecha futura) atrapan datos malformados antes de tocar la DB; pero "ese horario está libre" o "esa transición de estado es válida" vive en services porque necesita consultar la DB. Las dos capas no se mezclan: los validators de Pydantic nunca reciben `Session`.
**Ejemplo real del repo:** Pydantic `app/schemas.py:238` `@model_validator(mode="after") def validar_fechas(self): if self.fecha_fin is not None and self.fecha_fin <= self.fecha_inicio: raise ValueError(...)`. Negocio `turnos.py:55` `TRANSICIONES_VALIDAS` + `validar_slot_disponible` en `booking:199`.
**La regla para el próximo proyecto:** Usá Pydantic para todo lo verificable sin DB: rangos, formato, coherencia entre campos del mismo payload. Dejá para el service lo que requiere consultar la DB (unicidad real, disponibilidad, transiciones). No metas queries en validators de Pydantic.
**Señal de que lo estás usando bien:** Tus `model_validator` no reciben `Session` ni tocan la BD; las reglas que necesitan datos externos están en services y devuelven `ValueError` que el handler convierte.
**Aplica a:** [x] Solo Python

### TORNILLO #17 — `ValueError` del service → `HTTPException` con status por contexto
**En una línea:** Los services lanzan `ValueError` plano con mensaje legible; el handler los traduce a 400/404/409 y reserva 500+log para lo inesperado.
**Lo que hace el Senior:** Los services no conocen HTTP: lanzan `ValueError("Horario no disponible")`. El handler captura `ValueError` y decide el status según el contexto (409 si el mensaje habla de disponibilidad, 400 si no), y captura `Exception` genérico aparte para loguear y devolver 500 sin filtrar internals. Usa `raise ... from exc` para preservar la cadena.
**Ejemplo real del repo:** `app/routes/turnos.py:91` `except ValueError as exc: mensaje = str(exc); status_code = 409 if "disponible" in mensaje.lower() else 400; raise HTTPException(status_code=status_code, detail=mensaje) from exc`. Distinción `ValueError`(4xx) vs `Exception`(500) en `turnos.py:447-451`.
**La regla para el próximo proyecto:** En los services lanzá `ValueError` con mensaje user-facing para errores esperables; nunca importes `HTTPException` en un service. En el handler, capturá `ValueError`→4xx (eligiendo el código por contexto) y `Exception`→`logger.error`+500. Reservá `str(exc)` para el `ValueError` controlado, nunca para la `Exception` genérica de un 500.
**Señal de que lo estás usando bien:** `grep HTTPException` no aparece en `services/`; aparece solo en `routes/` y en `scope_service`. Los services solo hacen `raise ValueError`.
**Aplica a:** [x] Universal

### TORNILLO #18 — Lock pesimista en transiciones de estado (`SELECT FOR UPDATE`)
**En una línea:** Antes de mutar el estado de un turno y disparar efectos financieros, se relee la fila con `with_for_update()` para serializar la concurrencia.
**Lo que hace el Senior:** `marcar_realizado` y `cancelar_turno` no confían en el objeto que ya cargó el handler: vuelven a leer el turno con `.with_for_update()` (lock de fila pesimista) para que dos requests simultáneos no dupliquen la obligación financiera (R1) ni la multa (R4). Reverifica el estado actual ANTES de aplicar el efecto.
**Ejemplo real del repo:** `app/services/turno_lifecycle_service.py:77` `def _obtener_turno_bloqueado(session, turno_id): turno = session.query(Turno).filter(Turno.id==turno_id).with_for_update().first()`, usado por `marcar_realizado:96`, `cancelar_turno:133` y `reprogramar_turno:200`.
**La regla para el próximo proyecto:** Cuando una transición de estado dispara efectos con dinero o que no deben duplicarse, releé la fila con `.with_for_update().first()` dentro del service (no reuses la instancia del handler) y verificá el estado actual antes de aplicar el efecto. La transacción queda abierta hasta el commit del handler.
**Señal de que lo estás usando bien:** El service tiene un helper `_obtener_X_bloqueado` con `with_for_update` y revalida el estado antes de crear la obligación/multa.
**Aplica a:** [x] Solo Python

### TORNILLO #19 — La salida nunca es el ORM crudo: `response_model` + encoder UTC
**En una línea:** Todo endpoint declara `response_model` y devuelve el objeto ORM; Pydantic lo serializa filtrando campos y normalizando datetime a UTC.
**Lo que hace el Senior:** El handler hace `return turno` (objeto ORM), pero el `response_model=TurnoResponse` (con `from_attributes`) decide qué campos salen, y un `json_encoder` global convierte todo datetime a UTC ISO (así la DB naive-UTC se serializa consistente). Para colecciones o DTOs no-ORM usa `model_validate` explícito o helpers `_serialize_*`.
**Ejemplo real del repo:** Encoder global `app/schemas.py:47` `class BaseModel(PydanticBaseModel): model_config = ConfigDict(json_encoders={datetime: serialize_datetime_utc})`; serialización explícita `clientes.py:323` `detalle = [TransaccionResponse.model_validate(t) for t in transacciones]`.
**La regla para el próximo proyecto:** Declará siempre `response_model` en el decorador y devolvé el objeto ORM directamente (dejá que `from_attributes` serialice). Centralizá el formato de datetime en un `BaseModel` con `json_encoders`. Cuando la salida no es un ORM 1:1, usá `Model.model_validate(obj)` o un helper `_serialize_*` por item.
**Señal de que lo estás usando bien:** Ningún handler arma dicts a mano para la respuesta normal; todos tienen `response_model` y devuelven el ORM o pasan por `model_validate`/`_serialize_*`.
**Aplica a:** [x] Solo Python

### TORNILLO #20 — El cliente normaliza el dato y parsea el error de Pydantic
**En una línea:** El frontend convierte fecha local a ISO-UTC, valida lo básico antes de pegarle al backend, y sabe parsear el `detail` (array vs string).
**Lo que hace el Senior:** `Turnos.jsx` usa `toISO` (`new Date(...).toISOString()`) para mandar siempre UTC, hace validaciones espejo (tarifa>0, fecha_fin>fecha_inicio) para UX rápida, y al fallar distingue si `err.response.data.detail` es un array (errores de validación Pydantic 422) y lo une, o un string (`HTTPException`).
**Ejemplo real del repo:** `frontend/src/pages/Turnos.jsx:5` `function toISO(localStr){ return new Date(localStr).toISOString(); }` y `:111` `if (Array.isArray(detalle)) setErrModal(detalle.map(d=>d.msg).join(" | ")); else setErrModal(detalle || ...)`.
**La regla para el próximo proyecto:** En el cliente, normalizá tipos al contrato del backend antes de enviar (fechas a ISO-UTC, números a `Number`), validá lo barato para UX pero confiando en el backend como autoridad, y al manejar errores distinguí `detail` array (Pydantic 422) de string (`HTTPException`).
**Señal de que lo estás usando bien:** Las llamadas no setean `Authorization` a mano (lo hace el interceptor), las fechas viajan como ISO-UTC, y el manejo de error contempla `Array.isArray(detail)`.
**Aplica a:** [x] Universal

---

## ÁREA 4 — EL ARCHIVO DEL NEGOCIO

### TORNILLO #21 — PK uuid como String, no autoincrement
**En una línea:** Toda primary key (clave primaria) es un `String` (uuid generado en la app) indexado, nunca un `Integer` autoincremental.
**Lo que hace el Senior:** Define `id = Column(String, primary_key=True, index=True)` idéntico en TODOS los modelos. El id se genera en la capa de servicio (uuid str), no en la BD, lo que evita coupling con secuencias del motor y permite generar ids antes del commit.
**Ejemplo real del repo:** `models.py:167` `id = Column(String, primary_key=True, index=True)` (Cliente); idéntico en Turno(220), Consultorio(294), Transaccion(407), Usuario(666). Repetido en los ~25 modelos.
**La regla para el próximo proyecto:** Usá `id = Column(String, primary_key=True, index=True)` con uuid str generado en la aplicación como PK universal. No uses `Integer` autoincremental ni dejes que la BD genere la PK.
**Señal de que lo estás usando bien:** Cada modelo nuevo arranca con la misma línea exacta de PK y los FK que apuntan a él son `Column(String, ForeignKey(...))`.
**Aplica a:** [x] Solo este proyecto

### TORNILLO #22 — Dinero con Numeric/Decimal, nunca float
**En una línea:** Los montos son `Numeric` con precision/scale explícitas y defaults `Decimal`, jamás `Float`.
**Lo que hace el Senior:** Usa `Numeric(precision=12, scale=2)` para montos, `Numeric(14,2)` para acumulados en moneda base, y `Numeric(_, scale=6)` para tasas de cambio. Los defaults se escriben como `Decimal("...")`, no como float literal, evitando errores de redondeo binario en finanzas.
**Ejemplo real del repo:** `models.py:411` `monto = Column(Numeric(precision=12, scale=2), nullable=False)`; `:413` `tasa_cambio = Column(Numeric(precision=10, scale=6), default=Decimal("1.0"), nullable=False)`; `:477` `monto_original_ars = Column(Numeric(precision=14, scale=2), nullable=False)`.
**La regla para el próximo proyecto:** Para cualquier monto usá `Numeric(precision=12, scale=2)` (14,2 si es acumulado en moneda base) y para tasas `Numeric(_, scale=6)`. Defaults y constantes monetarias siempre `Decimal("...")`. Prohibido `Float`.
**Señal de que lo estás usando bien:** No aparece `Float` en ninguna columna de dinero; los defaults monetarios son `Decimal` con string.
**Aplica a:** [x] Universal

### TORNILLO #23 — Timestamps `created_at`/`updated_at` con `onupdate`
**En una línea:** Mixin de facto: `created_at` con default y `updated_at` con default + `onupdate`, apuntando a un callable de tiempo compartido.
**Lo que hace el Senior:** Casi todos los modelos repiten `created_at = Column(DateTime, default=utc_now_naive, nullable=False)` y `updated_at = Column(DateTime, default=utc_now_naive, onupdate=utc_now_naive, nullable=False)`. El callable `utc_now_naive` delega en un servicio de timezone compartido para UTC naive consistente. Los modelos tipo ledger inmutable (PagoRecibido, AplicacionPago, EventoFinanciero, AjusteContable) deliberadamente solo tienen `created_at`.
**Ejemplo real del repo:** `models.py:174-175` (Cliente); `:28-30` `def utc_now_naive() -> datetime: return shared_utc_now_naive()`; solo-`created_at` en PagoRecibido(536), AplicacionPago(578), EventoFinanciero(607), AjusteContable(1515).
**La regla para el próximo proyecto:** Agregá siempre `created_at` (default) y `updated_at` (default + `onupdate`) usando un único callable de tiempo centralizado. Para tablas-ledger inmutables omití `updated_at` a propósito (la ausencia documenta la intención).
**Señal de que lo estás usando bien:** Todo modelo mutable tiene el par idéntico; los registros inmutables solo `created_at`.
**Aplica a:** [x] Universal

### TORNILLO #24 — Enums `str, Enum` para estados y catálogos
**En una línea:** Cada conjunto de estados/tipos es una clase `class X(str, Enum)` mapeada con `SQLEnum`, reusable en ORM, Pydantic y lógica.
**Lo que hace el Senior:** Define ~16 enums heredando de `(str, Enum)` (son strings reales, serializables y comparables) y los monta con `Column(SQLEnum(X), default=X.VALOR, nullable=False)`, generando un tipo enum nativo en Postgres. Donde el catálogo puede evolucionar por política usa `String` guardando `Enum.MIEMBRO.value`, para no requerir `ALTER TYPE`.
**Ejemplo real del repo:** `models.py:101` `class Rol(str, Enum): SUPERADMIN = "SUPERADMIN"; PSICOLOGA = "PSICOLOGA"; CLIENT = "CLIENT"`; `:673` `rol = Column(SQLEnum(Rol), default=Rol.PSICOLOGA, nullable=False)`; patrón `String+.value` en Turno(230,237).
**La regla para el próximo proyecto:** Modelá estados y catálogos cerrados como `class X(str, Enum)` + `SQLEnum(X)`, y reusalos en Pydantic en vez de strings sueltos. Cuando el catálogo deba evolucionar sin migrar el tipo de la BD, usá `String(n)` almacenando `Enum.MIEMBRO.value`.
**Señal de que lo estás usando bien:** No hay strings mágicos sueltos para estados; o hay `SQLEnum` o hay un `String` alimentado por `.value` de un Enum definido arriba. El mismo enum aparece en `models.py`, `schemas.py` y la lógica.
**Aplica a:** [x] Universal

### TORNILLO #25 — Índices y unicidad tenant-first
**En una línea:** Los índices compuestos y unique constraints empiezan por `psicologa_id`, porque toda query filtra por tenant.
**Lo que hace el Senior:** Casi cada modelo tiene `psicologa_id` como primer FK indexado y define `__table_args__` con `Index` compuestos que arrancan por `psicologa_id` más el campo de filtro/orden. La unicidad también es por-tenant: teléfono/email son únicos POR psicóloga, no globalmente, vía `UniqueConstraint` compuesto.
**Ejemplo real del repo:** `models.py:177-181` `UniqueConstraint("psicologa_id","telefono", name="uq_cliente_psicologa_telefono")` + `Index("idx_cliente_psicologa_nombre","psicologa_id","nombre")`; `:265` `Index('idx_psicologa_fecha','psicologa_id','fecha_inicio')`.
**La regla para el próximo proyecto:** En sistemas multi-tenant poné `tenant_id` (acá `psicologa_id`) como primera columna de cada índice compuesto y dentro de cada `UniqueConstraint` de negocio. Nombrá los índices `idx_<tabla>_<cols>` y las constraints `uq_<tabla>_<cols>`.
**Señal de que lo estás usando bien:** Todo `Index`/`UniqueConstraint` de negocio lidera con `psicologa_id`; no existen unicidades globales de teléfono/email que romperían el multi-tenant.
**Aplica a:** [x] Universal

### TORNILLO #26 — Encriptación transparente vía `hybrid_property` (Fernet)
**En una línea:** Los campos sensibles se guardan en columna `_encriptado` y se exponen desencriptados con un `hybrid_property` Fernet.
**Lo que hace el Senior:** La columna persistida es `X_encriptado = Column(Text)` y se define `@hybrid_property def X` que desencripta al leer, un setter que cifra al escribir, y (donde hace falta filtrar en SQL) una `@X.expression` que expone la columna cruda. `encrypt_text`/`decrypt_text` usan Fernet (AES-256) con `ENCRYPTION_KEY` del entorno, fallando si no está. El resto del código trabaja con texto plano sin saber que está cifrado en reposo.
**Ejemplo real del repo:** `models.py:434-451` hybrid_property `notas` de Transaccion (getter decrypt, setter cifra, `.expression` para SQL); `:36-39` `get_cipher` exige `ENCRYPTION_KEY`. Repetido en IntegracionMeta access_token(755), IntegracionMercadoPago(817), IntegracionMobbex(885).
**La regla para el próximo proyecto:** Para datos sensibles persistí `<campo>_encriptado = Column(Text)` y exponé un `hybrid_property` que cifra/descifra con Fernet; agregá `.expression` si necesitás filtrarlo en SQL. Fallá fuerte si falta la clave.
**Señal de que lo estás usando bien:** El código de negocio nunca toca la columna `_encriptado`; siempre el property en claro. La clave viene de env y su ausencia lanza error.
**Aplica a:** [x] Solo Python

### TORNILLO #27 — Pool de conexiones endurecido y configurable
**En una línea:** El engine usa `pre_ping`, `recycle`, LIFO y overflow acotado, todo ajustable por env con parser defensivo.
**Lo que hace el Senior:** `create_engine` con `pool_pre_ping=True` (descarta conexiones muertas antes de usarlas), `pool_recycle=1800s` (evita conexiones rancias que el server cierra), `pool_use_lifo=True` (reusa conexiones calientes), y `pool_size`/`max_overflow`/`timeout` leídos vía `_env_int` que clampa a un mínimo y cae al default ante valores inválidos.
**Ejemplo real del repo:** `app/database.py:47-56` `engine = create_engine(DATABASE_URL, pool_pre_ping=True, pool_size=DB_POOL_SIZE, max_overflow=DB_MAX_OVERFLOW, pool_timeout=DB_POOL_TIMEOUT, pool_recycle=DB_POOL_RECYCLE_SECONDS, pool_use_lifo=True)`; `_env_int` defensivo en `:18-24`.
**La regla para el próximo proyecto:** Configurá el engine con `pool_pre_ping`, `pool_recycle` (~1800s), `pool_use_lifo` y `pool_size`/`max_overflow`/`timeout` parametrizados por env con un parser que valide y clampe. No uses el pool por defecto en producción.
**Señal de que lo estás usando bien:** Cada parámetro de pool está explícito y los numéricos vienen de env validados; conexiones muertas/rancias se reciclan solas.
**Aplica a:** [x] Universal

### TORNILLO #28 — `init_db` detecta drift y exige migración explícita
**En una línea:** Al arrancar compara esquema real vs modelos: crea tablas faltantes (bajo flag, solo en dev) pero FALLA si faltan columnas, exigiendo migración.
**Lo que hace el Senior:** `init_db` usa `inspect(engine)` para comparar tablas y columnas reales contra `modelo.__table__.columns`. Si faltan COLUMNAS (drift de esquema), lanza `RuntimeError` pidiendo migraciones explícitas. Si faltan TABLAS, solo las crea con `DB_AUTO_CREATE_SCHEMA=true` (default true solo en dev/test). Además migra valores de enum legacy (`ADMIN`→`SUPERADMIN`) y renombres de columnas legacy idempotentemente. Nunca usa `create_all()` como sustituto de migraciones en prod.
**Ejemplo real del repo:** `app/database.py:246-268` detecta columnas faltantes y `raise RuntimeError("Esquema de base de datos desactualizado... Ejecuta migraciones explicitas")`; mapas `LEGACY_COLUMN_RENAMES`/`LEGACY_ROLE_RENAMES` en `:71-79`.
**La regla para el próximo proyecto:** En el bootstrap inspeccioná el esquema real vs el ORM: creá tablas faltantes solo bajo flag y solo en dev; ante columnas faltantes FALLÁ y exigí migración explícita en vez de `create_all()`. Centralizá renombres legacy en mapas y aplicalos idempotentemente.
**Señal de que lo estás usando bien:** Un drift de columnas en producción impide arrancar la API (no se auto-parchea); las tablas nuevas solo se crean con el flag; los renombres legacy son idempotentes.
**Aplica a:** [x] Solo este proyecto

---

## ÁREA 5 — CUANDO ALGO SE ROMPE

### TORNILLO #29 — Stacktrace adentro, mensaje genérico afuera
**En una línea:** El error real con traceback va al log; al cliente solo le llega un `detail` corto y seguro.
**Lo que hace el Senior:** En cada handler de webhook envuelve la lógica en try/except donde `except HTTPException: raise` re-propaga los errores ya saneados, y `except Exception` hace `logger.exception(...)` (stacktrace + contexto de identidad) y luego `raise HTTPException(500, detail="...")` con un texto fijo que NO contiene `str(exc)`.
**Ejemplo real del repo:** `app/routes/webhooks.py:766` `except Exception as exc: logger.exception("Error procesando webhook Meta: %s", exc); raise HTTPException(status_code=500, detail="Error procesando webhook Meta") from exc`. Contraejemplo (fuga) en `turnos.py:451` donde `detail=f"...: {e}"` sí filtra.
**La regla para el próximo proyecto:** En handlers de API/webhook capturá `Exception`, logueá con `logger.exception` (stacktrace + IDs de contexto) y devolvé un `HTTPException` con `detail` fijo y genérico; nunca interpoles `str(exc)` en un `detail` de 500.
**Señal de que lo estás usando bien:** El `detail` de un 500 es una constante string; el `str(exc)` y el traceback aparecen solo en el log, nunca en el body.
**Aplica a:** [x] Universal

### TORNILLO #30 — Un logger por módulo con `__name__` y niveles con intención
**En una línea:** Cada archivo declara `logger = logging.getLogger(__name__)`, y el nivel se elige por gravedad (warning recuperable vs error/exception con stacktrace).
**Lo que hace el Senior:** No usa el root logger ni uno compartido: define el logger a nivel de módulo, y la config central en `main.py` usa `%(name)s` para identificar el emisor. Para los niveles: `logger.warning` para fallos esperados/recuperables (hay fallback), `logger.error`/`exception` (con stacktrace) para fallos que requieren atención.
**Ejemplo real del repo:** `app/services/llm_service.py:16` `logger = logging.getLogger(__name__)` + `app/main.py:78` `format="%(asctime)s [%(levelname)s] %(name)s: %(message)s"`. Warning recuperable en `llm_service.py:580`; error con stacktrace en `cron/runner.py:168` `logger.error(..., exc_info=True)`. Patrón en 40+ módulos.
**La regla para el próximo proyecto:** En cada módulo declará `logger = logging.getLogger(__name__)` a nivel de módulo y configurá el formato global con `%(name)s`. Elegí el nivel por consecuencia: warning si hay degradación con fallback, error/exception (con stacktrace) si algo quedó roto. No loguees todo como error ni todo como info.
**Señal de que lo estás usando bien:** `grep getLogger(__name__)` devuelve un match por módulo; los fallos con fallback aparecen como WARNING, los sin recuperación como ERROR/exception con traceback.
**Aplica a:** [x] Universal

### TORNILLO #31 — Fail-closed en firmas/tokens de webhook
**En una línea:** Si la firma/token no valida (o falta), se rechaza con 403 antes de procesar; nunca se asume válido.
**Lo que hace el Senior:** Valida la firma con `hmac.compare_digest` (comparación constant-time) y, ante firma ausente, formato inválido o mismatch, lanza `HTTPException(403)` con un `detail` descriptivo pero no sensible, cortando antes de tocar lógica de negocio.
**Ejemplo real del repo:** `app/services/mercadopago_service.py:136` `if not hmac.compare_digest(expected_signature, received_signature): raise HTTPException(status_code=403, detail="Firma de webhook de Mercado Pago inválida")`. Repetido en `webhooks.py:288,298` (Twilio) y token Meta.
**La regla para el próximo proyecto:** Validá firmas/tokens con `hmac.compare_digest` y rechazá con 403 ante ausencia, formato inválido o mismatch ANTES de ejecutar cualquier efecto; el camino por defecto es denegar, no permitir.
**Señal de que lo estás usando bien:** Toda rama que no logra confirmar autenticidad termina en `raise HTTPException(403)`; no existe un `else` implícito que deje pasar.
**Aplica a:** [x] Universal

### TORNILLO #32 — Proveedor externo caído no rompe el flujo (degradación)
**En una línea:** Fallos de LLM, envío de WhatsApp o transcripción se capturan y devuelven un fallback amable en vez de propagar.
**Lo que hace el Senior:** Envuelve cada llamada a un proveedor externo en try/except: ante excepción loguea con el `provider_label` y devuelve un valor seguro de degradación — el LLM devuelve `_fallback_intent()` o un mensaje de disculpa fijo, el envío Twilio devuelve `False`, el webhook ante fallo de transcripción responde un mensaje genérico. El usuario recibe respuesta usable; el detalle queda en el log.
**Ejemplo real del repo:** `app/services/llm_service.py:425` `except Exception as exc: logger.error("Error detectando intención con %s: %s", self.provider_label, exc); return self._fallback_intent()`. Repetido en `notification_service.py:141` (Twilio→False) y `webhooks.py:676` (transcripción→mensaje genérico).
**La regla para el próximo proyecto:** Para toda dependencia externa no determinista (LLM, mensajería, OCR, pagos), capturá la excepción, logueá con identificación del proveedor, y devolvé un fallback definido (mensaje de disculpa / `False` / `None`) en lugar de propagar; el endpoint siempre debe poder responder.
**Señal de que lo estás usando bien:** Apagar el proveedor externo no produce un 500: produce una respuesta de fallback y una línea de log con el error real.
**Aplica a:** [x] Universal

### TORNILLO #33 — Cron con try/rollback/finally que cuenta errores en vez de morir
**En una línea:** Cada job abre sesión, hace `rollback` en except y `close` en finally; los errores se cuentan y loguean, el scheduler sigue vivo.
**Lo que hace el Senior:** Estructura cada job con `db = SessionLocal(); try: ...; db.commit(); except Exception as e: db.rollback(); logger.error(..., exc_info=True); finally: db.close()`. El job devuelve un dict con contadores (encolados/enviados/errores) y nunca relanza, para que un fallo no detenga el scheduler ni las próximas corridas.
**Ejemplo real del repo:** `app/cron/runner.py:166` `except Exception as e: db.rollback(); logger.error(f"[CRON debt-notice ERROR] {e}", exc_info=True); finally: db.close()`. Patrón idéntico en `job_vencimiento:223`, `job_aprendizaje:293`.
**La regla para el próximo proyecto:** En jobs de cron/background gestioná la sesión con try/commit, except/rollback+log(exc_info=True), finally/close, devolvé contadores y NO relances; un fallo no debe matar el scheduler.
**Señal de que lo estás usando bien:** Tras un error, el log muestra el stacktrace, la sesión quedó cerrada sin transacción colgada, y el siguiente disparo corre normal.
**Aplica a:** [x] Universal

### TORNILLO #34 — Error boundary de render + interceptor 401 en el frontend
**En una línea:** Un `ErrorBoundary` captura crashes de render y el interceptor axios maneja el 401 en un solo lugar.
**Lo que hace el Senior:** Envuelve la app en un `AppErrorBoundary` (`componentDidCatch` loguea a consola, render muestra una pantalla amable con mensaje saneado y botón de recarga) y centraliza la expiración de sesión en el interceptor de respuesta: ante 401 hace `clearAuth()` y redirige a `/login`, en vez de repetir ese manejo en cada componente.
**Ejemplo real del repo:** `frontend/src/api/client.js:42` `if (error.response?.status === 401) { clearAuth(); window.location.href = "/login"; }` + `AppErrorBoundary.jsx:13` `componentDidCatch` que hace `console.error` y muestra card de error.
**La regla para el próximo proyecto:** En el frontend centralizá el manejo de errores transversales: un `ErrorBoundary` para crashes de render y un interceptor HTTP único para 401/sesión expirada; no dupliques ese manejo por componente.
**Señal de que lo estás usando bien:** Un 401 desde cualquier llamada termina siempre en logout+redirect sin código repetido; un error de render muestra la pantalla del boundary en vez de pantalla en blanco.
**Aplica a:** [x] Universal

---

## ÁREA 6 — LA LETRA DEL SENIOR

### TORNILLO #35 — Docstring de módulo en español al tope
**En una línea:** Cada archivo arranca con un docstring que dice qué es el módulo y su rol, antes de los imports.
**Lo que hace el Senior:** Primera línea de cada módulo: triple-quote con `'NombreModulo - descripción.'` y a veces un párrafo o lista de endpoints/reglas. Consistente en services y routes por igual.
**Ejemplo real del repo:** `app/routes/clientes.py:1` `"""Rutas - Clientes. CRUD completo scopeado por psicóloga."""`; `app/database.py:1` `"""Database - Configuración de conexión... Soporta exclusivamente PostgreSQL."""`.
**La regla para el próximo proyecto:** Abrí cada módulo Python con un docstring en español en formato `'NombreModulo - qué hace.'` antes de los imports; si expone endpoints o reglas, listalas.
**Señal de que lo estás usando bien:** El archivo no empieza con `import` sino con un docstring de 1-3 líneas que un humano lee para saber el rol del módulo.
**Aplica a:** [x] Solo este proyecto

### TORNILLO #36 — Helpers `_env_bool`/`_env_int` para leer entorno tipado
**En una línea:** Las variables de entorno se leen siempre vía helpers privados que normalizan, validan y caen al default con warning.
**Lo que hace el Senior:** En vez de `os.getenv` crudo disperso, define `_env_int(nombre, default, *, minimum)` (parsea, clampa, loguea warning ante valor inválido) y `_env_bool(nombre, default)` (normaliza a `{1,true,yes,on}`). El default se pasa como string y el helper lo castea. Lo repite por módulo de arranque en vez de importar uno común.
**Ejemplo real del repo:** `app/database.py:18` `def _env_int(nombre, default, *, minimum=1): valor = os.getenv(nombre, default).strip(); try: return max(minimum, int(valor)) except ValueError: logger.warning(...); return int(default)`; `:64` `_env_bool ... return valor in {"1","true","yes","on"}`. Re-declarado en `main.py:61`, `cron/runner.py:63`, `db_migration_runner.py:34`.
**La regla para el próximo proyecto:** No leas booleanos/enteros de env con `os.getenv` crudo: definí helpers `_env_bool`/`_env_int` que hagan strip, coerción y fallback con warning, y usalos en todos lados. El set truthy canónico es `{'1','true','yes','on'}`.
**Señal de que lo estás usando bien:** No hay `os.getenv(...) == 'true'` suelto; toda lectura de env pasa por un `_env_*` que devuelve el tipo final ya validado.
**Aplica a:** [x] Universal

### TORNILLO #37 — `typing.cast` para domar columnas SQLAlchemy
**En una línea:** Envuelve cada acceso a una columna ORM en `cast(TipoReal, obj.attr)` para que el type checker vea el tipo Python real, no `Column`.
**Lo que hace el Senior:** Como SQLAlchemy expone columnas con tipos genéricos que confunden a mypy/pyright, envuelve los accesos en `cast(str, usuario.id)`, `cast(bool, usuario.activo)`, `cast(datetime, obj.campo)` para mantener tipado estricto sin runtime overhead (`cast` es identidad en runtime). Combina con `getattr(obj, 'campo', default)` cuando la columna puede faltar.
**Ejemplo real del repo:** `app/services/jwt_service.py:188` `if not cast(bool, usuario.activo):`; `:219` `return cast(str, usuario.id)`; `reporting_service.py:238` `client_by_id = {cast(str, client.id): client for client in active_clients}`. `cast(` aparece ~960 veces en 40 archivos (verificado por `grep`).
**La regla para el próximo proyecto:** Al leer un atributo de una instancia SQLAlchemy que vas a pasar a lógica Python tipada, envolvelo en `typing.cast(TipoReal, instancia.columna)` en vez de `# type: ignore` global. Es no-op en runtime y mantiene la firma honesta. Para columnas opcionales combinalo con `getattr`.
**Señal de que lo estás usando bien:** El código pasa el type checker sin ignores dispersos; los accesos ORM en lógica de negocio aparecen siempre como `cast(...)`.
**Aplica a:** [x] Solo Python

### TORNILLO #38 — Triada `get_`/`require_`/`upsert_`/`serialize_`/`delete_` por entidad
**En una línea:** Para cada entidad de configuración hay un set canónico de funciones de service con nombres predecibles según el verbo de dominio.
**Lo que hace el Senior:** Por entidad define `get_X_for_psicologa` (devuelve `Optional`), `require_X_for_psicologa` (lanza `HTTPException(404)` si `None`), `upsert_X` (crea o actualiza vía `flush`), `serialize_X` (ORM→schema), `delete_X`. `require_` se construye SIEMPRE encima de `get_` + guard clause (nunca duplica la query); `delete_` encima de `require_`.
**Ejemplo real del repo:** `app/services/mercadopago_integration_service.py:48` `def require_mercadopago_config_for_psicologa(...): config = get_mercadopago_config_for_psicologa(...); if config is None: raise HTTPException(404, ...); return config`. Mismo patrón en `mobbex_integration_service.py:20-147`, `meta_integration_service.py`, `agent_profile_service.py:208-299`.
**La regla para el próximo proyecto:** Modelá las operaciones de cada entidad con esta nomenclatura: `get_` (Optional, sin error), `require_` (envuelve a `get_` y lanza 404), `upsert_` (crea o muta + `flush`, NO commit), `serialize_` (ORM→schema), `delete_`. `require_` nunca duplica la query: llama a `get_`.
**Señal de que lo estás usando bien:** Existe un par `get_`/`require_` por recurso donde `require_` tiene 3-4 líneas que solo invocan `get_` y validan `None`; cambiar el filtro de scope toca un solo lugar.
**Aplica a:** [x] Solo este proyecto

### TORNILLO #39 — Guard clauses con `HTTPException`, sin else anidado
**En una línea:** Valida y sale temprano lanzando `HTTPException`, en vez de envolver el happy path en if/else profundos.
**Lo que hace el Senior:** Chequea la condición de error primero (`if not X: raise HTTPException(...)`) y retorna el caso válido al final, plano. Tras `db.commit()` bajo try, captura `IntegrityError`, hace `rollback` y traduce a `HTTPException` 400/409.
**Ejemplo real del repo:** `app/services/jwt_service.py:184` `if not usuario: raise HTTPException(status_code=401, detail="Usuario no encontrado")`; `app/routes/clientes.py:148` `try: db.commit() except IntegrityError: db.rollback(); raise HTTPException(status_code=400, ...)`.
**La regla para el próximo proyecto:** Validá errores al principio con guard clauses que lanzan `HTTPException` (detail en español, status explícito) y dejá el camino feliz sin indentar. Rodeá `db.commit()` con try/except `IntegrityError` → `rollback` + `HTTPException` 400/409.
**Señal de que lo estás usando bien:** Las funciones no tienen pirámides de if/else; los `raise` están arriba y el `return` de éxito al ras del cuerpo. Ningún commit queda sin rollback ante `IntegrityError`.
**Aplica a:** [x] Universal

### TORNILLO #40 — Helpers privados `_` de normalización/parseo reutilizados
**En una línea:** Extrae micro-operaciones puras (normalizar texto, formatear plata, bucketizar fechas) en funciones privadas con prefijo `_` y las compone.
**Lo que hace el Senior:** Define funciones cortas con `_` al inicio (`_utcnow`, `_normalize_text`, `_money`, `_safe_pct`, `_bucket_key`, `_parse_expiration_hours`), cada una con una sola responsabilidad y type hints completos, reusadas en múltiples puntos del módulo.
**Ejemplo real del repo:** `app/services/reporting_service.py:36` `def _utcnow() -> datetime: return utc_now_naive()` y `:40` `def _money(value): return Decimal(str(value)).quantize(Decimal("0.01"))`. Bloque de 12+ helpers en `reporting_service.py:36-177`.
**La regla para el próximo proyecto:** Cuando una expresión (formateo de dinero, normalización, cálculo de %, parseo de env) aparece o aparecerá más de una vez, extraela a una función módulo-privada con prefijo `_` y type hints completos. Una sola responsabilidad, y componelas.
**Señal de que lo estás usando bien:** El módulo tiene un bloque de helpers `_` arriba; la lógica principal lee como composición de esos helpers y no repite `quantize`/`strip` inline.
**Aplica a:** [x] Solo Python

### TORNILLO #41 — Type hints completos + keyword-only para parámetros de dominio
**En una línea:** Toda función declara tipos de args y retorno, y usa `*` para forzar keyword en parámetros semánticos como `psicologa_id`/`days`.
**Lo que hace el Senior:** Firmas con anotaciones en todos los parámetros y `-> retorno`. Para funciones de service con varios ids/flags usa keyword-only tras `*`, evitando errores posicionales y haciendo la llamada autoexplicativa. Usa `Optional[...]` explícito y `dict[str, ...]`/`list[...]` modernos.
**Ejemplo real del repo:** `app/services/reporting_service.py:180` `def build_economic_report(db: Session, *, psicologa_id: str, days: int = 30) -> dict[str, object]:`; `mercadopago_integration_service.py:20` `def get_mercadopago_config_for_psicologa(db, psicologa_id, *, only_active: bool = False)`.
**La regla para el próximo proyecto:** Anotá todos los parámetros y el retorno de cada función. Cuando una función de service toma varios identificadores o flags, ponelos después de `*` como keyword-only para que la llamada sea autoexplicativa.
**Señal de que lo estás usando bien:** No hay funciones sin `-> retorno`; las llamadas a services pasan los ids por nombre y es imposible invertir argumentos por posición.
**Aplica a:** [x] Solo Python

### TORNILLO #42 — Raw SQL confinado y parametrizado (anti-patrón evitado)
**En una línea:** No hay SQL crudo ad hoc disperso; el único `text()` vive en la migración de arranque y usa parámetros bindeados.
**Lo que hace el Senior:** Toda la lectura/escritura normal usa el ORM (`db.query(...).filter(...)`). El único `text()` está en `database.py init_db` para migración de roles/columnas legacy, con valores pasados como parámetros bindeados (`{'legacy':..., 'nuevo':...}`). Ninguna route o service de negocio arma SQL por concatenación.
**Ejemplo real del repo:** `app/database.py:177` `connection.execute(text("UPDATE usuario SET rol = :nuevo WHERE rol = :legacy"), {"legacy": legacy, "nuevo": nuevo})`. Declarado en `.github/copilot-instructions.md:8` y `docs/CLAUDE.md:23`.
**La regla para el próximo proyecto:** Usá el ORM para todo acceso de negocio. Reservá `text()` solo para migraciones de bootstrap y SIEMPRE pasá valores como parámetros bindeados (`:nombre` + dict), nunca por f-string con datos de usuario.
**Señal de que lo estás usando bien:** `grep` de `text(`/`execute(` solo aparece en `database.py`; las queries de negocio no tienen f-strings con SQL ni concatenación.
**Aplica a:** [x] Universal

### TORNILLO #43 — Imports explícitos de helpers con guion bajo
**En una línea:** `import *` no trae nombres privados, así que los `_helpers` se importan por nombre explícito.
**Lo que hace el Senior:** En `operativo/`, `_core.py` hace `from ._shared import *` para lo público PERO además importa explícitamente los helpers privados que necesita. `_shared.py` lista un `__all__` gigante que a propósito NO contiene los nombres `_...`. Así se evita el bug silencioso de creer que un helper privado quedó disponible por el wildcard (causa #1 de `NameError` en producción según las convenciones del repo).
**Ejemplo real del repo:** `app/services/operativo/_core.py:10-15` `from ._shared import *  # noqa: F401, F403` seguido de `from ._shared import (_extract_cliente_query_for_ver_turnos, ...)`. El patrón `from ._...` explícito aparece 38 veces en 10 módulos. Declarado en `.github/copilot-instructions.md:9` y 3 archivos más.
**La regla para el próximo proyecto:** Nunca confíes en `from .module import *` para acceder a helpers que empiezan con `_`: importalos explícitamente por nombre. Ante un `NameError` sobre un `_...`, lo primero que revisás es si está importado.
**Señal de que lo estás usando bien:** Cada `_helper` que usás tiene una línea de import explícita; `grep 'from ._'` muestra el import del helper usado.
**Aplica a:** [x] Solo este proyecto

---

## ÁREA 7 — LAS LLAVES DEL LOCAL

### TORNILLO #44 — `.env.example` versionado como fuente de verdad, `.env` ignorado
**En una línea:** Un único template comentado por sección, commiteado, que documenta cada variable que la app espera; el `.env` real nunca entra a git.
**Lo que hace el Senior:** Mantiene `.env.example` en git con encabezado "safe to commit", secciones (Database, Redis, Notifications, LLM, Security, App) y placeholders explícitos `replace_with_*`. El `.gitignore` usa el patrón ignore + negación: ignora `.env` y `.env.*` y luego `!.env.example`. Es el contrato: cualquier var que el código lee aparece ahí.
**Ejemplo real del repo:** `.env.example:1-3` `# psic-loga - Environment template (safe to commit)`; `.gitignore:17-20` `.env / .env.* / !.env.example`.
**La regla para el próximo proyecto:** Versioná un `.env.example` con TODAS las variables que el código lee, agrupadas por sección y con placeholders evidentes. En `.gitignore` ignorá `.env`/`.env.*` y agregá la excepción `!.env.example`. Verificá con `git ls-files` que ningún `.env` real esté trackeado.
**Señal de que lo estás usando bien:** `git ls-files | grep .env` devuelve únicamente `.env.example`; cada `os.getenv` del código tiene su entrada documentada en el ejemplo.
**Aplica a:** [x] Universal

### TORNILLO #45 — Comandos para generar secretos embebidos en el template
**En una línea:** El `.env.example` incluye el comando exacto para generar cada clave fuerte, junto a la variable.
**Lo que hace el Senior:** Pone el one-liner de generación como comentario al lado del campo: `Fernet.generate_key()` para `ENCRYPTION_KEY` y `secrets.token_hex(64)` para `JWT_SECRET`. El mismo comando se repite en el mensaje de error de `get_cipher()`, así nadie inventa una clave débil.
**Ejemplo real del repo:** `.env.example:62-66` `# python -c "from cryptography.fernet import Fernet; print(Fernet.generate_key().decode())"` y `secrets.token_hex(64)`; `models.py:38` repite el comando Fernet en el `ValueError`.
**La regla para el próximo proyecto:** Al lado de cada secreto en el `.env.example`, escribí como comentario el comando exacto para generarlo de forma segura. Repetí ese comando en el mensaje de error cuando la clave falte.
**Señal de que lo estás usando bien:** Un dev nuevo genera todas las claves copiando comandos del template, sin buscar en docs.
**Aplica a:** [x] Universal

### TORNILLO #46 — Defaults distintos por ambiente (dev/test vs prod)
**En una línea:** El valor por defecto de una variable depende de `ENV`, y el default seguro es el de producción.
**Lo que hace el Senior:** Calcula el default según `ENV`: en dev/test crea el esquema automáticamente, en prod no (fuerza migraciones explícitas). Igual con `LOG_TO_FILE` (off en prod). El comportamiento seguro es el default productivo si `ENV` no está seteado.
**Ejemplo real del repo:** `app/database.py:60-61` `ENV = os.getenv("ENV", "development").strip().lower(); _AUTO_CREATE_DEFAULT = "true" if ENV in {"development","dev","test"} else "false"` luego `:69` `DB_AUTO_CREATE_SCHEMA = _env_bool("DB_AUTO_CREATE_SCHEMA", _AUTO_CREATE_DEFAULT)`.
**La regla para el próximo proyecto:** Cuando el valor seguro difiere entre dev y prod, derivá el default de `ENV` en vez de hardcodear uno. El default debe ser el comportamiento seguro de producción si `ENV` no está seteado o es prod.
**Señal de que lo estás usando bien:** Sin tocar ninguna var en prod, la app arranca en modo seguro (no autocrea schema, no loguea a archivo); en dev arranca cómoda.
**Aplica a:** [x] Universal

### TORNILLO #47 — Validación fail-fast de config al importar el módulo
**En una línea:** La config inválida revienta al arrancar el proceso, no a mitad de un request.
**Lo que hace el Senior:** Valida en el cuerpo del módulo (no dentro de un handler): `DATABASE_URL` obligatoria y con esquema postgresql, `JWT_ALGORITHM` en allowlist, `JWT_EXPIRATION_HOURS` acotado a 1..168, y en prod `JWT_SECRET` fuerte + issuer/audience. Lanza `RuntimeError` con mensaje accionable.
**Ejemplo real del repo:** `app/services/jwt_service.py:97` `_validar_configuracion_jwt()` (invocación top-level); `database.py:27` `if not _raw_database_url: raise RuntimeError("DATABASE_URL es obligatoria...")`; `_parse_expiration_hours` acota 1..168 al importar (`jwt_service.py:49-64`).
**La regla para el próximo proyecto:** Validá los secretos y config crítica al nivel de módulo (al importar), lanzando `RuntimeError` con mensaje accionable. Nunca dejes que una config inválida pase el arranque y falle recién en un request.
**Señal de que lo estás usando bien:** Con `ENV=production` y un secreto débil, el proceso no levanta: muere en el import con un mensaje claro, no devuelve un 500 en el primer login.
**Aplica a:** [x] Universal

### TORNILLO #48 — Secretos productivos inyectados por la plataforma, no en el repo
**En una línea:** `render.yaml` declara las vars pero deja los secretos como generados o `sync:false` (cargados en el dashboard).
**Lo que hace el Senior:** Usa `generateValue:true` para `JWT_SECRET` (la plataforma lo genera), `sync:false` para `ENCRYPTION_KEY`/`META_ACCESS_TOKEN`/`OPENAI_API_KEY` (se cargan a mano en el panel, nunca en git), y `fromDatabase`/`fromService` para `DATABASE_URL`/`REDIS_URL` (referencia al recurso, no el valor).
**Ejemplo real del repo:** `render.yaml:34-43` `- key: JWT_SECRET / generateValue: true` y `- key: ENCRYPTION_KEY / sync: false`; `:23-26` `DATABASE_URL ... fromDatabase: ... property: connectionString`. (Nota: `render.yaml:32-33` hardcodea `WEBHOOK_VERIFY_TOKEN` literal — debería ser `sync:false`.)
**La regla para el próximo proyecto:** En el IaC de deploy, declará nombres de vars de secretos con `generateValue`/`sync:false` y referenciá recursos con `fromDatabase`/`fromService`. Nunca pongas el valor del secreto en el yaml versionado.
**Señal de que lo estás usando bien:** El yaml de deploy no contiene ningún secreto real cargable; todos los sensibles son `generateValue`, `sync:false` o referencias a recursos.
**Aplica a:** [x] Universal

### TORNILLO #49 — Normalizar `DATABASE_URL` y validar el driver
**En una línea:** Acepta la URL del proveedor cloud, la normaliza a `postgresql://` y rechaza cualquier otro motor.
**Lo que hace el Senior:** Convierte el `postgres://` que dan algunos PaaS (Render) al `postgresql://` que espera SQLAlchemy, y si tras normalizar no empieza con `postgresql://`, lanza error explicando el formato. No hay fallback a SQLite: la app es deliberadamente mono-motor.
**Ejemplo real del repo:** `app/database.py:31-40` `DATABASE_URL = _raw_database_url.replace("postgres://", "postgresql://", 1) if ... else _raw_database_url; if not DATABASE_URL.startswith("postgresql://"): raise RuntimeError("DATABASE_URL inválida: ... solo soporta PostgreSQL")`.
**La regla para el próximo proyecto:** Al leer `DATABASE_URL`, normalizá el prefijo `postgres://`→`postgresql://` y validá el esquema esperado con un error claro. No asumas que el proveedor te da el formato exacto del ORM.
**Señal de que lo estás usando bien:** Pegar la connection string cruda de Render/Railway (`postgres://`) arranca sin tocar nada; una URL de otro motor falla al arrancar con mensaje del formato esperado.
**Aplica a:** [x] Solo Python

### TORNILLO #50 — Defaults de runtime en el script de arranque
**En una línea:** `start.sh` lee tuning de servidor con `${VAR:-default}`, corre migraciones una vez y desactiva la init per-worker.
**Lo que hace el Senior:** Centraliza el arranque: `start.sh` toma `WEB_CONCURRENCY`, `UVICORN_LIMIT_CONCURRENCY`, etc. con defaults bash, corre migraciones, y setea `INIT_DB_ON_STARTUP=false` para no reinicializar la BD en cada worker. La app respeta `PORT`/`HOST` con defaults.
**Ejemplo real del repo:** `start.sh:4-7` `WORKERS="${WEB_CONCURRENCY:-4}"; LIMIT_CONCURRENCY="${UVICORN_LIMIT_CONCURRENCY:-200}"`; `:13` `export INIT_DB_ON_STARTUP=false` tras migraciones; coherente con `main.py:102` `if _env_bool("INIT_DB_ON_STARTUP", "true"): init_db()`.
**La regla para el próximo proyecto:** Poné el tuning de servidor en el start script con `${VAR:-default}` y corré migraciones/init una sola vez antes de levantar workers, desactivando la init per-worker via env. La app debe tener defaults para `PORT`/`HOST`/concurrencia.
**Señal de que lo estás usando bien:** La app levanta sin setear ninguna var de tuning (usa defaults) y la BD se inicializa una vez, no una por worker.
**Aplica a:** [x] Universal

---

## ÁREA 8 — CUANDO ESTÁ TERMINADO

### TORNILLO #51 — El nombre del test ES la documentación del bug
**En una línea:** Cada test de regresión nombra el comportamiento de negocio que protege, no la función que llama.
**Lo que hace el Senior:** Nombres largos y descriptivos en español del dominio + docstring opcional con el root-cause del bug original. Un fallo dice qué regla se rompió sin abrir el archivo.
**Ejemplo real del repo:** `tests/test_financial_ledger_refactor.py:170` `def test_cancelar_turno_es_idempotente_y_crea_una_sola_multa(db_session)`; `tests/test_backend_regresiones.py:2838` docstring `"Bug root-cause: condicion.descripcion doesn't exist on the model."`. ~290 tests con nombres de dominio.
**La regla para el próximo proyecto:** Nombrá cada test por la regla de negocio o el bug que previene (`test_<comportamiento_esperado>`), no por la función. Si nace de un bug, documentá el root-cause en el docstring.
**Señal de que lo estás usando bien:** Al leer solo el nombre del test fallido sabés qué invariante de negocio se violó, sin abrir el cuerpo.
**Aplica a:** [x] Universal

### TORNILLO #52 — Fixture `db_session` efímera in-memory por test
**En una línea:** Cada test arranca con una DB SQLite vacía que se crea y destruye en el mismo test.
**Lo que hace el Senior:** Fixture local en cada suite que hace `create_all` en `sqlite:///:memory:`, yield de la session y en finally `close` + `drop_all`. Aislamiento total sin estado compartido.
**Ejemplo real del repo:** `tests/test_financial_ledger_refactor.py:58-70` `engine=create_engine('sqlite:///:memory:', future=True); Base.metadata.create_all(...); yield session; finally: session.close(); Base.metadata.drop_all(bind=engine)`. Patrón idéntico en `test_backend_regresiones.py:184`, `test_reporting_service.py:39`.
**La regla para el próximo proyecto:** Para tests de lógica que tocan ORM, usá una DB in-memory creada/destruida por test con `create_all`/`drop_all` en un fixture; nunca compartas estado de DB entre tests.
**Señal de que lo estás usando bien:** Los tests pasan en cualquier orden y aislados (`pytest -k uno`) sin depender de datos de otro test.
**Aplica a:** [x] Solo Python

### TORNILLO #53 — `monkeypatch` para cortar todo el IO externo
**En una línea:** WhatsApp, LLM, pagos y Redis se reemplazan por fakes en memoria que también actúan como espías.
**Lo que hace el Senior:** `monkeypatch.setattr` sobre el módulo donde se USA la dependencia (no donde se define), inyectando una función fake que registra sus llamadas en una lista local para luego afirmar exactamente qué se procesó/envió.
**Ejemplo real del repo:** `tests/test_backend_regresiones.py:441` `monkeypatch.setattr(webhooks_module, 'procesar_mensaje_agente', fake_procesar)`; `fake_procesar` acumula en `procesados[]` y luego `assert procesados == [...]` (`:481-484`). ~530 usos de monkeypatch.
**La regla para el próximo proyecto:** Aislá IO/proveedores con `monkeypatch.setattr` apuntando al módulo consumidor; hacé que el fake registre sus argumentos en una lista para poder afirmar el contrato exacto de la llamada.
**Señal de que lo estás usando bien:** Los tests corren sin red, sin claves reales y sin servicios externos; aun así verifican el payload exacto que se habría enviado.
**Aplica a:** [x] Solo Python

### TORNILLO #54 — Fixture global autouse que mata el cache no-determinista
**En una línea:** Un solo fixture en `conftest` desactiva el cache Redis de conversaciones en TODOS los tests.
**Lo que hace el Senior:** `conftest.py` mínimo con un único fixture `autouse=True` que parchea `cargar_conversacion` a `lambda: None`, eliminando una fuente global de flakiness (datos stale de Redis) sin repetirlo en cada test.
**Ejemplo real del repo:** `tests/conftest.py:4` `@pytest.fixture(autouse=True) def _disable_redis_conversation_cache(monkeypatch): monkeypatch.setattr('app.services.agent_context_service.cargar_conversacion', lambda *a, **k: None)`. El conftest contiene exactamente ese único fixture (10 líneas).
**La regla para el próximo proyecto:** Si existe UNA dependencia global no-determinista (cache, reloj, random) que afecta a toda la suite, neutralizala con un fixture `autouse` en `conftest.py` en vez de repetir el parche en cada test.
**Señal de que lo estás usando bien:** Ningún test individual necesita acordarse de desactivar el cache; la suite es estable sin importar el estado de Redis.
**Aplica a:** [x] Solo Python

### TORNILLO #55 — Idempotencia y reintentos como invariante testeada
**En una línea:** Toda operación financiera se invoca dos veces en el test para probar que no duplica.
**Lo que hace el Senior:** Los tests llaman la misma operación repetidamente (cancelar dos veces, encolar dos veces) y afirman que el segundo intento devuelve el mismo objeto o cuenta 0, y que solo existe un registro en la DB. Las reglas de negocio nombradas en el código (R1-R4, `idempotency_key`) tienen tests que verifican el monto exacto y la no-duplicación.
**Ejemplo real del repo:** `tests/test_financial_ledger_refactor.py:476` `assert queue_debt_notices(db_session, now=now) == 1; assert queue_debt_notices(db_session, now=now) == 0`; `:225` `assert multa_1.id == multa_2.id; assert len(multas) == 1`. Reglas R1-R4 en `financial_ledger_service.py:707,799`.
**La regla para el próximo proyecto:** Para cada operación con efecto secundario (cobro, multa, aviso, pago), escribí un test que la ejecute dos veces y afirme que no hay duplicación (mismo id / count==0 en el segundo intento). Dale nombre estable a cada regla de negocio y afirmá su efecto numérico exacto.
**Señal de que lo estás usando bien:** Existe un test `es_idempotente` (o que llama N veces) por cada operación mutante de dinero/estado.
**Aplica a:** [x] Universal

### TORNILLO #56 — `tests/manual` excluido del runner por diseño
**En una línea:** Scripts que requieren backend vivo y red se separan físicamente y se excluyen de pytest.
**Lo que hace el Senior:** Los verificadores operativos (golpean `localhost:8000`, ngrok, Postgres real, Meta) viven en `tests/manual/` y `pytest.ini` los excluye con `norecursedirs=manual`, así CI nunca intenta correrlos ni falla por falta de servicios.
**Ejemplo real del repo:** `pytest.ini:3` `norecursedirs = manual __pycache__`; `tests/manual/diagnostico_webhook.py:33` `requests.get('http://localhost:8000/health', timeout=3)`. 12 scripts en `tests/manual/`.
**La regla para el próximo proyecto:** Scripts de verificación que necesitan servicios vivos o credenciales reales van en una carpeta excluida del runner (`norecursedirs`); no los mezcles con los tests automatizados que deben correr en CI sin infraestructura.
**Señal de que lo estás usando bien:** `pytest` recoge cero tests de la carpeta manual; esos scripts se corren a mano con `python <archivo>`.
**Aplica a:** [x] Universal

### TORNILLO #57 — CI gatea por cobertura y prueba contra la DB real
**En una línea:** CI corre la suite en matriz de versiones contra Postgres real y exige cobertura mínima.
**Lo que hace el Senior:** El workflow levanta un servicio Postgres 16 con healthcheck, inyecta env de test (`ENCRYPTION_KEY` generada, `NOTIFICATION_PROVIDER=log`), corre pytest en 3.11 y 3.12, y falla el build si la cobertura baja de 53%. Lint es no-bloqueante (`|| true`).
**Ejemplo real del repo:** `.github/workflows/ci.yml:62` `python -m pytest tests/ --cov=app --cov=business_logic --cov-fail-under=53 -q`; `:55` `echo 'NOTIFICATION_PROVIDER=log'`; matriz 3.11/3.12 en `:33-34`.
**La regla para el próximo proyecto:** En CI corré la suite en matriz de versiones contra el motor de DB de producción (no solo el de los tests), con un gate de cobertura mínimo explícito y los proveedores de notificación en modo log/no-op.
**Señal de que lo estás usando bien:** El build se pone rojo si la cobertura cae bajo el umbral; los tests corren igual en cada versión soportada de Python.
**Aplica a:** [x] Universal

### TORNILLO #58 — Reloj y entorno deterministas en tests
**En una línea:** El `now()` se fija con monkeypatch y los secretos se autogeneran al importar el módulo de test.
**Lo que hace el Senior:** Los tests sensibles al tiempo parchean la función `_utcnow` del servicio a una fecha fija, y cada archivo hace `os.environ.setdefault('ENCRYPTION_KEY', Fernet.generate_key().decode())` al inicio para que el import no explote ni dependa de un `.env`.
**Ejemplo real del repo:** `tests/test_reporting_service.py:156` `monkeypatch.setattr('app.services.reporting_service._utcnow', lambda: now)` con `now=datetime(2026,3,23,...)`; `:17` `os.environ.setdefault('ENCRYPTION_KEY', Fernet.generate_key().decode())`.
**La regla para el próximo proyecto:** Centralizá la hora actual en una función (`_utcnow`) que los tests puedan parchear, y autogenerá secretos con `os.environ.setdefault` al tope del archivo de test para que la suite no dependa de un entorno externo.
**Señal de que lo estás usando bien:** Los tests de reportes/aging dan el mismo resultado hoy y dentro de un año; correr la suite no requiere configurar ningún `.env` previo.
**Aplica a:** [x] Solo Python

---

## ÁREA 9 — LO QUE PYTHON HACE MEJOR

### TORNILLO #59 — Generator como recurso con cierre garantizado (`get_db`)
**En una línea:** Una función generadora con `yield`/`finally` que FastAPI consume como dependency, cerrando la sesión siempre.
**Lo que hace el Senior:** `get_db()` abre `SessionLocal()`, hace `yield db` (el endpoint corre ahí) y garantiza `db.close()` en el `finally`. FastAPI reconoce el generator y ejecuta el código post-yield como teardown, incluso si el endpoint lanza excepción. El `sessionmaker` se configura con `autocommit=False, autoflush=False` para control explícito de la transacción.
**Ejemplo real del repo:** `app/database.py:82` `def get_db() -> Generator[Session, None, None]: db = SessionLocal(); try: yield db; finally: db.close()`; `:58` `SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)`.
**La regla para el próximo proyecto:** Para recursos que deben liberarse pase lo que pase (sesiones DB, conexiones), usá un generator con `try: yield recurso / finally: recurso.close()` y pasalo via `Depends`; nunca cierres manualmente en cada endpoint. Tipá el retorno como `Generator[Session, None, None]`.
**Señal de que lo estás usando bien:** Ningún endpoint cierra la sesión a mano; el cierre vive en un solo lugar y no hay `SessionLocal()` suelto.
**Aplica a:** [x] Solo Python

### TORNILLO #60 — Imports opcionales con degradación elegante
**En una línea:** `try/except ModuleNotFoundError` para que el sistema arranque aunque falte una dependencia, con fallback funcional o error diferido al punto de uso.
**Lo que hace el Senior:** Importa `jose` y `bcrypt` dentro de try/except; si faltan, deja sentinelas (`jwt = None`, `bcrypt = None`) y degrada: bcrypt cae a PBKDF2, y jose dispara un `RuntimeError` explícito solo cuando realmente se necesita JWT (`_require_jose`). Así los tests corren en entornos incompletos sin romper el import del módulo.
**Ejemplo real del repo:** `app/services/jwt_service.py:20` `try: from jose import JWTError, jwt except ModuleNotFoundError: JWTError = Exception; jwt = None`; fallback en `hash_password:104`.
**La regla para el próximo proyecto:** Para dependencias que pueden faltar en algunos entornos, importalas con `try/except ModuleNotFoundError`, dejá un sentinela `None`, y o bien provee un fallback equivalente o fallá con un mensaje claro solo en el punto de uso (no al importar).
**Señal de que lo estás usando bien:** El módulo importa sin la dependencia instalada; el error aparece solo al invocar la función que la requiere, con mensaje accionable.
**Aplica a:** [x] Solo Python

### TORNILLO #61 — Context managers de SQLAlchemy con `execution_options`
**En una línea:** `with engine.begin()/connect()` para transacciones/conexiones auto-cerradas, ajustando isolation por bloque.
**Lo que hace el Senior:** Usa `with engine.begin() as connection:` para transacciones que commitean/rollbackean solas, `with engine.connect()` para lecturas, y `.execution_options(isolation_level="AUTOCOMMIT")` para DDL que no puede correr en transacción (`ALTER TYPE` en Postgres). Cada bloque libera la conexión al salir.
**Ejemplo real del repo:** `app/database.py:158` `with engine.begin() as connection: ... connection.execute(text("UPDATE usuario SET rol = CAST(:nuevo AS rol) WHERE rol::text = :legacy"), {...})`; `:153` `with engine.connect().execution_options(isolation_level="AUTOCOMMIT") as connection:`.
**La regla para el próximo proyecto:** Para transacciones usá `with engine.begin()` (commit/rollback automático); para solo lectura `with engine.connect()`; cuando una sentencia DDL exige autocommit, encadená `.execution_options(isolation_level="AUTOCOMMIT")` en su propio with-block.
**Señal de que lo estás usando bien:** No hay `commit()`/`rollback()`/`close()` manuales sueltos; cada bloque de DB es un `with` que delimita la transacción.
**Aplica a:** [x] Solo Python

### TORNILLO #62 — Registry inmutable con dataclasses frozen (matcher→prepare→execute)
**En una línea:** Las acciones del agente son datos en una tupla de dataclasses frozen, cada una con callables `matcher`/`prepare`/`execute` de firma uniforme — no `if/else` dispersos.
**Lo que hace el Senior:** Define `@dataclass(frozen=True) OperationalActionDefinition` con campos que son funciones (tipadas con `Callable` y `TypedDict` para payloads) y arma `_ACTION_REGISTRY` como tupla inmutable de ~60 entradas. El dispatcher itera el registry corriendo `definition.matcher(texto)` hasta encontrar match; un `_ACTION_MAP = {d.action_id: d}` da lookup O(1) por id para el path LLM. Las tres funciones por acción tienen firma uniforme `(psicologa_id, db, payload)`. Agregar un comando = agregar una entrada + sus 3 funciones, sin tocar el dispatcher.
**Ejemplo real del repo:** `app/services/operativo/_action_registry.py:188` `@dataclass(frozen=True) class OperationalActionDefinition: action_id: str; ...; matcher: MatcherFn; prepare: PrepareFn; execute: ExecuteFn`; registry tuple en `:3615`; `_ACTION_MAP` en `:4510`; dispatch en `:4840`.
**La regla para el próximo proyecto:** Para un router de intents/comandos, modelá cada acción como una dataclass frozen que agrupa sus callables (`matcher`/`prepare`/`execute`) y juntá todas en una tupla-registry inmutable; despachá iterando o con un dict por id. Usá `TypedDict`/`Callable` para tipar payloads y funciones.
**Señal de que lo estás usando bien:** Agregar una acción no toca el motor de dispatch: solo se agrega una entrada a la tupla; las tres fases viven juntas y tipadas.
**Aplica a:** [x] Solo Python

### TORNILLO #63 — `@asynccontextmanager` para el lifespan de la app
**En una línea:** Un async generator decorado gestiona startup y shutdown de FastAPI en una sola función simétrica.
**Lo que hace el Senior:** Decora una corrutina con `@asynccontextmanager`; el código antes del `yield` es startup (init_db, init_scheduler) y el de después es shutdown. La pasa a `FastAPI(lifespan=lifespan)`, reemplazando los antiguos `on_event` con un único bloque legible.
**Ejemplo real del repo:** `app/main.py:97` `@asynccontextmanager async def lifespan(app: FastAPI): if _env_bool("INIT_DB_ON_STARTUP", "true"): init_db(); init_scheduler(); yield; shutdown_scheduler()` registrado en `:128` `lifespan=lifespan`.
**La regla para el próximo proyecto:** Gestioná startup/shutdown de FastAPI con una única función async decorada con `@asynccontextmanager` (setup antes del yield, teardown después) y pasala como `lifespan=` al construir la app; evitá `on_event` separados.
**Señal de que lo estás usando bien:** Hay una sola función `lifespan` con el yield al medio; el setup y su teardown correspondiente están visiblemente emparejados.
**Aplica a:** [x] Solo Python

---

## ÁREA 10 — CÓMO CRECE ESTO

### TORNILLO #64 — El orden sagrado de 8 capas para una feature nueva
**En una línea:** Toda feature nueva se construye en el mismo orden fijo, de la persistencia hacia la UI.
**Lo que hace el Senior:** Define primero el modelo en `models.py`, luego la migración idempotente, luego los schemas `Create`/`Update`/`Response`, luego el service de funciones libres, luego el router fino, lo registra en `main.py`, agrega las funciones de api client y el panel, y cierra con un test. Nunca salta capas ni mete lógica de negocio en la route.
**Ejemplo real del repo:** `app/main.py:158` `app.include_router(consultorios.router)`; el router `app/routes/consultorios.py:91` `items = obtener_consultorios(db, usuario.id, ...)` delega en `app/services/consultorio_service.py:19`. Mismo patrón en consultorios, ajuste_contable, comprobantes.
**La regla para el próximo proyecto:** Al agregar una entidad nueva, creá archivos en este orden exacto: (1) modelo, (2) migración `<fecha>_<n>_add_<feature>.py`, (3) schemas `Create`/`Update`/`Response`, (4) `<feature>_service.py`, (5) `routes/<feature>.py`, (6) `include_router` en `main.py`, (7) funciones en `api/client.js` + componente, (8) `test_<feature>.py`. No pongas reglas de negocio en la route.
**Señal de que lo estás usando bien:** El handler de la route es delgado (valida, llama al service, serializa) y toda consulta a DB vive en el service.
**Aplica a:** [x] Solo este proyecto

### TORNILLO #65 — Migración idempotente con inspector y guards
**En una línea:** Las migraciones se pueden correr dos veces sin romper: chequean existencia antes de crear/borrar.
**Lo que hace el Senior:** En vez de confiar en `op.create_table` a ciegas, define helpers `_has_table`/`_has_column`/`_has_index` sobre `sa.inspect(bind)`, y re-fetchea el inspector después de cada cambio estructural (su cache queda obsoleto). El downgrade hace lo inverso, también guardado por chequeos de existencia.
**Ejemplo real del repo:** `alembic/versions/20260404_01_add_consultorios_and_plantilla_horario.py:48` `if not _has_table(inspector, "consultorio"):` y `:79` `inspector = sa.inspect(bind)` re-fetchea antes del siguiente paso. `_has_table`/`_has_column` en 12 archivos de migración.
**La regla para el próximo proyecto:** Toda migración debe ser idempotente: copiá los helpers `_has_table`/`_has_column`/`_has_index`, guardá cada `op.create_*`/`add_column` tras un chequeo de no-existencia, re-instanciá el inspector después de cada DDL, y escribí el downgrade en orden inverso con los mismos chequeos.
**Señal de que lo estás usando bien:** Correr `alembic upgrade head` dos veces seguidas no lanza error de objeto duplicado.
**Aplica a:** [x] Solo este proyecto

### TORNILLO #66 — Enum Postgres idempotente (`DO $$ ... duplicate_object`)
**En una línea:** Crear un tipo enum de Postgres sin que falle si ya existe, sin usar el `sa.Enum` del autogenerate.
**Lo que hace el Senior:** No confía en `sa.Enum(...)` (intenta `CREATE TYPE` siempre y revienta en re-run). Emite un bloque PL/pgSQL `DO $$ BEGIN CREATE TYPE ... AS ENUM (...); EXCEPTION WHEN duplicate_object THEN NULL; END $$;` y luego crea la tabla con SQL puro referenciando el tipo, para que SQLAlchemy no re-emita el `CREATE TYPE`. Para reuso usa `postgresql.ENUM(..., create_type=False)`. El downgrade hace `DROP TYPE IF EXISTS`.
**Ejemplo real del repo:** `alembic/versions/20260404_02_add_ajuste_contable.py:22-29` `DO $$ BEGIN CREATE TYPE tipoajustecontable AS ENUM ('QUITA', 'CARGO_EXTRA'); EXCEPTION WHEN duplicate_object THEN NULL; END $$;`. Declarado en `.github/instructions/alembic.instructions.md:9-11`.
**La regla para el próximo proyecto:** Para enums Postgres en migraciones NO uses el `sa.Enum` autogenerado. Creá el tipo con el bloque `DO $$ BEGIN CREATE TYPE <nombre_lowercase> AS ENUM (...); EXCEPTION WHEN duplicate_object THEN NULL; END $$;` y creá la tabla con SQL puro (`op.execute`) que referencie el tipo ya existente. En downgrade: `DROP TYPE IF EXISTS`.
**Señal de que lo estás usando bien:** El nombre del tipo PG va en minúsculas (`tipoajustecontable`) aunque la clase Python sea `TipoAjusteContable`, y la migración re-corre sin error de tipo duplicado.
**Aplica a:** [x] Solo Python

### TORNILLO #67 — Migración encadenada por fecha+secuencia
**En una línea:** Las revisiones se nombran `<YYYYMMDD>_<NN>` y forman una cadena lineal explícita sin branches.
**Lo que hace el Senior:** Nombra el archivo y la revisión como `AAAAMMDD_NN` y setea `down_revision` a la revisión inmediatamente anterior, formando una cadena estrictamente lineal (`branch_labels=None`). Para revertir una feature, agrega una migración `remove_` nueva en vez de editar la vieja.
**Ejemplo real del repo:** `alembic/versions/20260404_01...py:13-15` `revision = "20260404_01"` / `down_revision = "20260324_01"`; cadena verificada `20260324_01 <- 20260404_01 <- 20260404_02`; par `add`/`remove` en `20260323_01` y `20260323_02`.
**La regla para el próximo proyecto:** Nombrá cada migración `<YYYYMMDD>_<NN>_<verbo>_<feature>.py` con `revision` = mismo id y `down_revision` = la cabeza actual. Mantené la cadena lineal (`branch_labels=None`). Para deshacer una feature ya mergeada, escribí una migración `remove_` nueva, no edites la existente.
**Señal de que lo estás usando bien:** `alembic heads` devuelve una sola cabeza y los archivos ordenados alfabéticamente reflejan el orden cronológico real.
**Aplica a:** [x] Solo Python

### TORNILLO #68 — Guardrails por nivel de riesgo + confirmación
**En una línea:** Cada acción declara un `risk_level` y las mutaciones (sobre todo financieras) exigen confirmar antes de ejecutar.
**Lo que hace el Senior:** Cada acción lleva `risk_level` (read/write/financial_sensitive/high_risk) y `requires_confirmation`. Las read se ejecutan directo; las write/sensibles construyen un `PendingOperationalAction` con prompt 'confirmar/cancelar', metadata de guardrail e interactive buttons, y solo ejecutan tras confirmación. Una sola función (`_build_guardrail_metadata`) mapea `risk_level` a política y fuerza confirmación en los niveles sensibles.
**Ejemplo real del repo:** `app/services/operativo/_action_registry.py:4399-4409` payment_register `requires_confirmation=True, risk_level="financial_sensitive"` vs `:4377-4386` patient_debt_get `requires_confirmation=False, risk_level="read"`; gating en `:4600` `if definition.requires_confirmation: ... pending = _build_pending_action(...)`.
**La regla para el próximo proyecto:** Etiquetá cada acción con un nivel de riesgo explícito; lecturas ejecutan directo, mutaciones (sobre todo financieras) deben generar un pending action con confirmación antes de ejecutar. Centralizá el mapeo riesgo→política en una sola función.
**Señal de que lo estás usando bien:** Ninguna acción que escribe dinero o estado se ejecuta sin pasar por el flujo pending/confirmar; el `risk_level` está declarado en la definición, no inferido ad hoc.
**Aplica a:** [x] Universal

### TORNILLO #69 — Integración paralela clonable (Meta/MercadoPago/Mobbex) + facade
**En una línea:** Cada proveedor externo es una copia idéntica del mismo molde de service, y un facade resuelve cuál usar por env o preferencia del tenant.
**Lo que hace el Senior:** Los tres proveedores tienen exactamente la misma forma: service con docstring estándar, schemas `PsicologaXConfigResponse`/`Upsert`, modelo `IntegracionX`, y el par `get_X_config_for_psicologa(db, psicologa_id, *, only_active=False)` y `has_active_X_config_for_psicologa`. Un `payment_provider_service.py` los abstrae detrás de un enum `ProveedorCobro` con resolución por orden de preferencia. El namespacing es estricto: `*_integration_service.py` para credenciales/config, `*_service.py` para la lógica del proveedor, `*_provider_service.py` para el facade. Donde el tipo concreto difiere usa `typing.cast`.
**Ejemplo real del repo:** `app/services/mobbex_integration_service.py:20` `def get_mobbex_config_for_psicologa(db, psicologa_id, *, only_active=False)` es byte-por-byte el mismo molde que `mercadopago_integration_service.py:20`; `payment_provider_service.py:96` `resolve_payment_provider_for_psicologa` con `_DEFAULT_PROVIDER_ORDER`.
**La regla para el próximo proyecto:** Cuando soportes >1 proveedor del mismo tipo, cloná el molde existente (modelo + schemas + `*_integration_service` con firma idéntica) y poné un único punto de resolución (facade `*_provider_service`) que decida por config/env. No metas `if proveedor==` disperso: centralizalo en el facade.
**Señal de que lo estás usando bien:** El service nuevo es prácticamente un diff de nombres del de Mobbex/MercadoPago, y `resolve_payment_provider_for_psicologa` lo reconoce sin tocar las routes.
**Aplica a:** [x] Universal

### TORNILLO #70 — Policy chain con precedencia y snapshot congelado
**En una línea:** La tarifa/penalidad de un turno se resuelve por una cascada DEFAULT→PACIENTE→TURNO y se congela como snapshot inmutable en el turno al crearlo.
**Lo que hace el Senior:** `resolve_turno_policy` parte de la política DEFAULT del agente, la sobreescribe con la condición especial del PACIENTE si está vigente, y finalmente con un override manual a nivel TURNO (que exige `motivo_excepcion_politica`). Cada nivel registra el `OrigenPoliticaTurno`. Luego `apply_policy_snapshot_to_turno` copia la política resuelta al turno, para que cambios futuros de config no muten turnos ya creados.
**Ejemplo real del repo:** `app/services/patient_policy_service.py:263` `origen = OrigenPoliticaTurno.DEFAULT; ... :289 origen = OrigenPoliticaTurno.PACIENTE; ... :301 origen = OrigenPoliticaTurno.TURNO`; consumido en `turno_booking_service.py:15,220`. Declarado en `.github/copilot-instructions.md:25` y `docs/CLAUDE.md:57-69` (R4).
**La regla para el próximo proyecto:** Para reglas con múltiples fuentes, resolvé en una sola función con precedencia explícita de menor a mayor especificidad, etiquetá el origen de la decisión, exigí justificación para overrides manuales, y persistí un snapshot inmutable en la entidad para que cambios de configuración no reescriban el histórico.
**Señal de que lo estás usando bien:** Existe un enum de Origen (DEFAULT/PACIENTE/TURNO), una función `resolve_*_policy` única, y una `apply_*_snapshot` que se invoca al crear la entidad; el override manual lanza error si falta motivo.
**Aplica a:** [x] Universal

---

## ÁREA 11 — AGENTES CONVERSACIONALES
### Los tornillos que gobiernan un agente de IA que habla con personas reales

### TORNILLO PY-16 — El bot se calla cuando hay un humano
**En una línea:** Antes de generar cualquier respuesta, el agente verifica si la conversación fue derivada a un operador humano; si es así, no dice nada.
**Lo que hace el Senior:** En `agent_service.py`, la primera compuerta de `procesar_mensaje_agente()` es `_tiene_control_humano()`. Si devuelve `True`, retorna `(True, "", None)` —respuesta vacía— sin tocar el LLM ni ejecutar ninguna acción. El silencio es explícito, no un olvido.
**Ejemplo real del repo:** `agent_service.py:711` `if _tiene_control_humano(contexto): return (True, "", None)`.
**La regla para el próximo proyecto:** En cualquier agente con posibilidad de escalada humana, la primera compuerta del flujo debe ser: ¿hay un humano manejando esto? Si la respuesta es sí, el agente devuelve silencio explícito y no ejecuta nada más. El humano no debe competir con el bot por la conversación.
**Señal de que lo estás usando bien:** Si un operador toma una conversación y el paciente escribe otro mensaje, el bot no responde. El operador no recibe una respuesta del bot que contradiga lo que acaba de escribir.
**Aplica a:** [x] Universal

---

### TORNILLO PY-17 — La confianza tiene umbral: debajo de 0.55 no actúes
**En una línea:** El LLM devuelve un número de confianza entre 0 y 1; si está por debajo del umbral, el agente no ejecuta la acción sino que escala o pide aclaración.
**Lo que hace el Senior:** En `agent_intents_service.py:1454`, después de llamar al LLM, evalúa `if intencion == "otro" or confianza < 0.55`. Si la confianza es baja, entra a una cascada de fallback: intenta el router de baja confianza, luego respuesta conversacional, luego información de la psicóloga. Si falla tres veces seguidas (contador de incomprensión), activa el handoff a humano con motivo `"incomprension_reiterada"`.
**Ejemplo real del repo:** `agent_intents_service.py:1454` `if intencion == "otro" or confianza < 0.55:` (cascada de fallback) y `if intentos >= 3: _activar_control_humano(..., "incomprension_reiterada")`.
**La regla para el próximo proyecto:** Nunca ejecutes una acción con impacto (reservar, cancelar, cobrar) si el LLM no está seguro. Pedile al LLM un score de confianza junto con la intención, definí un umbral explícito (entre 0.5 y 0.7 según el riesgo de la acción), y al tercer intento fallido derivá a humano con el historial de lo que no se entendió.
**Señal de que lo estás usando bien:** El agente nunca cancela un turno porque "entendió" un mensaje ambiguo. Si el usuario escribe algo confuso tres veces, un humano recibe la conversación con el resumen de los tres intentos fallidos.
**Aplica a:** [x] Universal

---

### TORNILLO PY-18 — Resolver antes de gastar tokens
**En una línea:** Un router determinístico (regex, keywords, reglas) intenta resolver el mensaje antes de llamar al LLM; el LLM es el último recurso, no el primero.
**Lo que hace el Senior:** `agent_message_router_service.py` implementa un router basado en patrones que intercepta intenciones operativas comunes (listar turnos, consultar deuda, FAQ) antes de que el mensaje llegue al LLM. Solo si el router no resuelve, se gasta la llamada a OpenAI/Anthropic. Además hay un sistema de aprendizaje: si una intención se resuelve bien repetidamente por el LLM, el patrón se agrega al router para la próxima vez.
**Ejemplo real del repo:** `agent_service.py` `resultado = _resolver_outcome_ruteo_temprano(mensaje, contexto); if resultado: return resultado` y recién después la llamada al LLM.
**La regla para el próximo proyecto:** En un agente conversacional, poné un router determinístico antes del LLM para las intenciones frecuentes y predecibles. Medí qué porcentaje de mensajes resuelve el router vs el LLM. Con el tiempo, los patrones del LLM deben migrar al router: el agente se vuelve más barato y más rápido sin perder capacidad.
**Señal de que lo estás usando bien:** El costo de tokens baja con el tiempo aunque el volumen de mensajes suba. Las consultas frecuentes (horarios, turnos, deuda) se resuelven en milisegundos sin llamada externa.
**Aplica a:** [x] Universal

---

### TORNILLO PY-19 — El handoff lleva contexto, no solo una notificación
**En una línea:** Cuando el agente deriva a un humano, arma un resumen estructurado con motivo, intención detectada y última frase del paciente, para que el operador entre con contexto y no a ciegas.
**Lo que hace el Senior:** `agent_handoff_service.py` implementa `build_handoff_summary()` que construye un objeto con: motivo del handoff (`solicitud_paciente` / `incomprension_reiterada` / `cliente_inactivo` / `sin_permiso`), la intención que el LLM detectó, la última frase del paciente, y el turno objetivo si había uno en juego. Este resumen se adjunta al contexto y se envía al operador en el mismo momento en que se activa el control humano.
**Ejemplo real del repo:** `agent_handoff_service.py` `def build_handoff_summary(motivo, intencion, ultimo_mensaje, turno_objetivo): return {"motivo": motivo, "intencion_detectada": intencion, "ultimo_mensaje_paciente": ultimo_mensaje, "turno_objetivo": turno_objetivo, "timestamp": now()}`.
**La regla para el próximo proyecto:** Cuando un agente escala a humano, nunca mandes solo "hay un usuario esperando". Armá un resumen estructurado con: por qué escaló (motivo), qué quería el usuario (intención), qué dijo exactamente (último mensaje), y qué estaba en juego (recurso/acción objetivo). El operador tiene que poder tomar el caso en 10 segundos sin leer todo el hilo.
**Señal de que lo estás usando bien:** Un operador puede responder al paciente correctamente sin leer el historial completo de la conversación. El tiempo promedio de primera respuesta del operador baja porque ya sabe de qué se trata.
**Aplica a:** [x] Universal

---

### TORNILLO PY-20 — Historial acotado para el LLM
**En una línea:** El agente no le pasa toda la conversación al LLM; define ventanas distintas según la tarea (10 mensajes para responder, 6 para clasificar) y las respeta siempre.
**Lo que hace el Senior:** En `llm_service.py`, `generar_respuesta()` toma `mensajes[-10:]` y los mapea a roles user/assistant. El clasificador de intención usa `mensajes[-6:]` recortados a 120 caracteres cada uno. Estas ventanas son constantes deliberadas, no defaults olvidados: controlan el costo de tokens y evitan que el LLM se pierda en conversaciones largas.
**Ejemplo real del repo:** `llm_service.py` `contexto_reciente = mensajes[-10:]` (para responder) y `snippet = [m[:120] for m in mensajes[-6:]]` (para clasificar intención).
**La regla para el próximo proyecto:** Definí explícitamente cuántos mensajes del historial le pasás al LLM según la tarea: más contexto para generar respuestas, menos para clasificar intención. Nunca pases el historial completo sin techo: el costo escala linealmente y la calidad no mejora después de cierto punto. Documentá las ventanas como constantes con nombre, no como números mágicos.
**Señal de que lo estás usando bien:** El costo por conversación no escala con la duración de la conversación después de los primeros mensajes. Podés calcular el costo máximo por turno antes de que el usuario escriba nada.
**Aplica a:** [x] Universal

---

## COMPARACIÓN CON EL TORNILLERO TS

Tengo un tornillero previo de TypeScript con universales numerados (#2, #5, #8, #9, #10, #11, #13, #14, #18, #19, #20, #23, #24, #26, #28, #29, #32, #33, #34, #37, #38, #39, #40, #41, #46, #52, #53, #54, #55, #57, #59, #60), pero **no tengo el texto de esos números, solo los números**. Por lo tanto, todos los mapeos de "Confirma #N" abajo son **inferenciales y honestamente declarados como tales** (no pude verificar contra el texto TS). Solo se listan acá los tornillos Python marcados Universal.

| Tornillo Python | Relación | Comentario |
|---|---|---|
| #1 Composition root | Confirma #N (inferido, texto TS no provisto) | Registro central de rutas/módulos es patrón cross-lenguaje (NestJS/Express tienen su `AppModule`/montaje central). Probable universal del TS. |
| #2 Route flaca, service gordo | Confirma #N (inferido) | Separación controller/service es canónica en TS (NestJS). Casi seguro ya existe como universal TS. |
| #3 Fachada `__init__` con `__all__` | Confirma #N (inferido) | Equivale al `index.ts` barrel que re-exporta API pública. Patrón gemelo. |
| #5 404 vs 403 multi-tenant | Confirma #N (inferido) | Decisión de seguridad agnóstica al lenguaje; muy probablemente ya esté en el TS. |
| #6 Scoping por tenant desde el token | Confirma #N (inferido) | Tenant derivado del sujeto autenticado, no del request; universal de seguridad. |
| #7 Fail-closed en auth (revalidar contra DB) | Confirma #N (inferido) | Revocación efectiva sin blacklist; agnóstico. Probable universal TS. |
| #8 Validar config crítica al boot | Confirma #N (inferido) | Fail-fast de config es transversal; encaja con los típicos universales de "config validada al arrancar". |
| #9 Detección de secretos débiles por marcadores | Nuevo (o refinamiento) | Es una especialización concreta de #8; si el TS no lo separa, vale como nuevo. |
| #11 Guard de ownership previo en escrituras | Confirma #N (inferido) | Validar ownership antes de mutar es agnóstico; probable que el TS lo tenga. |
| #13 401 global en el cliente | Confirma #N (inferido) | Interceptor HTTP de 401→logout es idéntico en axios/TS; casi seguro ya está. |
| #14 Schema entrada ≠ salida | Confirma #N (inferido) | DTO de request vs response (class-validator/Zod). Patrón gemelo fortísimo. |
| #17 Error de negocio → status por contexto | Confirma #N (inferido) | Mapear excepción de dominio a 4xx en el borde HTTP es transversal. |
| #20 Cliente normaliza y parsea el error | Confirma #N (inferido) | Normalización de payload y parseo de error en el front; agnóstico. |
| #22 Dinero con decimal, nunca float | Confirma #N (inferido) | En TS sería evitar `number` para plata (usar `Decimal`/centavos/string). Misma regla, distinto vehículo. |
| #23 created_at/updated_at con onupdate | Confirma #N (inferido) | Timestamps automáticos (TypeORM `@CreateDateColumn`/`@UpdateDateColumn`). Gemelo. |
| #24 Enums tipados para estados | Confirma #N (inferido) | Union types/enums de TS para estados en vez de strings mágicos. Patrón equivalente. |
| #25 Índices/unicidad tenant-first | Confirma #N (inferido) | Estrategia de indexado multi-tenant; agnóstico al ORM. |
| #27 Pool de conexiones endurecido | Confirma #N (inferido) | `pre_ping`/`recycle`/pool size aplican igual a `pg`/TypeORM en Node. Gemelo. |
| #29 Stacktrace adentro, genérico afuera | Confirma #N (inferido) | No filtrar internals al cliente es seguridad transversal; muy probable universal TS. |
| #30 Logger por módulo + niveles | Confirma #N (inferido) | Logger con namespace por módulo (pino/winston child loggers). Patrón gemelo. |
| #31 Fail-closed en firmas de webhook | Confirma #N (inferido) | Verificación de firma con compare constant-time; agnóstico. |
| #32 Proveedor externo caído no rompe (degradación) | Confirma #N (inferido) | Graceful degradation de dependencias externas; transversal. |
| #33 Cron try/rollback/finally que cuenta errores | Confirma #N (inferido) | Job resiliente que no mata el scheduler; agnóstico (BullMQ/node-cron). |
| #34 Error boundary + interceptor 401 | Confirma #N (inferido) | React idéntico en TS; casi seguro ya está como universal de front. |
| #36 `_env_bool`/`_env_int` tipados | Confirma #N (inferido) | Lectura de env validada/coercionada (en TS sería Zod sobre `process.env`). Gemelo. |
| #39 Guard clauses sin else anidado | Confirma #N (inferido) | Early-return es estilístico transversal; probable universal TS. |
| #42 Raw SQL confinado y parametrizado | Confirma #N (inferido) | Prohibir SQL concatenado, usar binds; seguridad agnóstica. |
| #44 `.env.example` versionado, `.env` ignorado | Confirma #N (inferido) | Gemelo exacto en cualquier stack. |
| #45 Comandos de generación de secretos en el template | Nuevo (o refinamiento) | Detalle de DX que el TS puede no haber aislado como tornillo propio. |
| #46 Defaults por ambiente | Confirma #N (inferido) | Default seguro derivado de `NODE_ENV`; gemelo. |
| #47 Fail-fast de config al importar | Confirma #N (inferido) | Validar config al cargar el módulo; agnóstico. Probable solape con #8. |
| #48 Secretos por la plataforma, no en el repo | Confirma #N (inferido) | IaC con secret refs; gemelo. |
| #50 Defaults de runtime en el start script | Confirma #N (inferido) | `${VAR:-default}` + migrar una sola vez; agnóstico al runtime. |
| #51 El nombre del test ES la doc del bug | Confirma #N (inferido) | Naming de tests por comportamiento (Jest/Vitest `describe/it`); gemelo. |
| #55 Idempotencia testeada (llamar 2x) | Confirma #N (inferido) | Invariante de idempotencia en tests; agnóstico. |
| #56 `tests/manual` fuera del runner | Confirma #N (inferido) | Separar smoke manual del CI; gemelo (`testPathIgnorePatterns`). |
| #57 CI con gate de cobertura + DB real | Confirma #N (inferido) | Matriz de versiones + servicio DB + threshold; gemelo. |
| #68 Guardrails por nivel de riesgo + confirmación | Nuevo | Patrón de dominio (agente conversacional con confirmación para mutaciones sensibles). Improbable en el TS genérico. |
| #69 Integración paralela clonable + facade | Confirma #N (inferido) parcial | El facade/strategy de proveedores es agnóstico; el "molde clonable estricto" es más específico de este repo. |
| #70 Policy chain con precedencia + snapshot | Nuevo | Resolución por cascada con snapshot inmutable; patrón de dominio que el TS genérico probablemente no tenga. |
| #71 Ruteo de tenant fail-closed por identificador entrante | Confirma #N (inferido) | Resolver el tenant desde el canal entrante y fallar cerrado es seguridad multi-tenant agnóstica; probable variante/extensión de #5/#6 en el TS. |
| #72 Proveedores intercambiables (ABC + Strategy + factory) | Confirma #N (inferido) | Strategy + factory por config es patrón GoF clásico, idéntico en TS con `interface` + factory. Patrón gemelo casi seguro. |

**Posibles contradicciones:** No detecté contradicciones fuertes. La única tensión teórica: el TS podría favorecer un único schema con grupos de validación en lugar de DTOs request/response separados (#14). Si así fuera, **el patrón Python es más robusto**: separar entrada de salida elimina por construcción la fuga de campos derivados (`tarifa_base`, `estado`) hacia el cliente y la posibilidad de mass-assignment, a costa de un poco más de boilerplate. La separación explícita gana en superficie de seguridad.

---

## CANDIDATOS A UNIVERSAL CONFIRMADO

Estos tornillos marcados Universal trascienden claramente el lenguaje y merecen subir a `szoluciones_tornillos_universales.md`:

- **#5 — 404 vs 403 para datos de otro tenant:** decisión de seguridad pura; no filtrar existencia ajena vale en cualquier stack multi-tenant.
- **#6 — Tenant derivado del token, no del request:** elimina por diseño la clase entera de bugs de cross-tenant via parámetro; agnóstico.
- **#7 — Fail-closed revalidando contra DB:** revocación efectiva sin blacklist de tokens; principio de auth universal.
- **#8 — Validar config de seguridad al boot:** fail-fast de configuración crítica; aplica a todo proceso desplegable.
- **#14 — Schema de entrada distinto del de salida:** previene mass-assignment y fuga de campos derivados en cualquier framework con DTOs.
- **#17 — Error de negocio → status HTTP por contexto:** el dominio no conoce HTTP; el borde traduce. Capa transversal.
- **#22 — Dinero con decimal, nunca float:** corrección numérica financiera independiente del lenguaje.
- **#25 — Índices/unicidad tenant-first:** estrategia de performance y correctitud multi-tenant agnóstica al ORM.
- **#29 — Stacktrace adentro, mensaje genérico afuera:** no filtrar internals al cliente; seguridad universal.
- **#31 — Fail-closed en firmas de webhook (constant-time):** verificación de autenticidad de webhooks transversal.
- **#32 — Degradación ante proveedor externo caído:** resiliencia de dependencias no deterministas; aplica a todo sistema con integraciones.
- **#42 — Raw SQL confinado y parametrizado:** prevención de inyección SQL universal.
- **#44/#46/#47/#48 — Gestión de config y secretos (`.env.example`, defaults por ambiente, fail-fast, secrets por plataforma):** higiene de configuración válida en cualquier stack.
- **#51/#55/#57 — Tests por comportamiento, idempotencia testeada, CI con gate de cobertura:** disciplina de calidad agnóstica al runner.
- **#68/#70 — Guardrails por riesgo y policy chain con snapshot:** patrones de dominio (mutaciones confirmadas, resolución por precedencia con histórico inmutable) que cualquier sistema con dinero/acciones sensibles necesita, independientemente del lenguaje.
- **#71 — Ruteo de tenant fail-closed por identificador entrante:** resolver el destinatario desde el canal compartido (webhook/número/dominio) y denegar ante ambigüedad es seguridad multi-tenant pura; vale en cualquier stack con canales compartidos.
- **#72 — Proveedores intercambiables (Strategy + factory por config):** desacoplar al consumidor del proveedor concreto detrás de una interfaz seleccionada por env es patrón GoF clásico, agnóstico al lenguaje (ABC es solo el vehículo Python).

---

## LO QUE PYTHON APORTA QUE TYPESCRIPT NO TIENE

Esta es la parte donde el lenguaje deja huella. Los siguientes tornillos ("Solo Python") no son simplemente "lo mismo en otra sintaxis": son ergonomías que Python hace de forma idiomática y que en TypeScript se replican con más fricción o no existen con la misma naturalidad.

- **#59 — Generator como recurso con cierre garantizado (`get_db` con `yield`/`finally`).** El truco de un generador que cede el recurso en el `yield` y lo libera en el `finally`, consumido directo por el sistema de dependencias, es puro Python. En TS no hay un equivalente tan limpio de "dependency que se autolimpia post-handler"; se termina con middlewares o `try/finally` manuales. Acá el cierre vive en un solo lugar y FastAPI ejecuta el teardown aunque el endpoint explote.

- **#10 — Dependency factory con closure (`require_rol(*roles)`).** Una función que cierra sobre sus parámetros y devuelve la dependency lista para `Depends`, más shortcuts pre-aplicados. TS lo hace con decoradores y guards (NestJS), pero el closure + first-class function de Python lo vuelve trivial y sin metadata reflection.

- **#60 — Imports opcionales con `try/except ModuleNotFoundError`.** Que un módulo importe aunque falte `jose` o `bcrypt`, dejando un sentinela `None` y degradando o difiriendo el error al punto de uso, es idiomático de Python (imports en runtime, no estáticos). En TS los imports son estáticos: replicar esto requiere `import()` dinámico y bastante ceremonia.

- **#61 — Context managers de SQLAlchemy (`with engine.begin()` / `execution_options(AUTOCOMMIT)`).** El protocolo `with` que delimita transacción/conexión y la libera al salir, ajustando isolation por bloque, es muy ergonómico. TS no tiene `with`; se emula con callbacks (`db.transaction(async tx => ...)`), que funciona pero es menos visualmente delimitado.

- **#62 — Registry inmutable con dataclasses frozen (matcher→prepare→execute).** `@dataclass(frozen=True)` para acciones que agrupan callables, juntadas en una `tuple` inmutable, con `TypedDict` para payloads. TS tiene objetos y tipos, pero la combinación dataclass frozen + tuple + funciones de primera clase con firma uniforme da un registry declarativo muy compacto.

- **#63 — `@asynccontextmanager` para el lifespan.** Una sola función async donde el `yield` parte startup y shutdown, decorada y pasada como `lifespan=`. Es el mismo protocolo de context manager elevado a async; muy legible. En TS el setup/teardown del servidor se hace con hooks o eventos separados.

- **#26 — Encriptación transparente vía `hybrid_property`.** El `@hybrid_property` de SQLAlchemy permite que un atributo se vea como texto plano en Python pero esté cifrado en la columna, e incluso exponer la columna cruda para queries SQL vía `.expression`. Es una capa de magia controlada que el ORM de Python ofrece y que en TypeORM se resuelve con transformers, menos integrados con el query layer.

- **#37 — `typing.cast` para domar columnas ORM.** `cast(str, usuario.id)` es no-op en runtime y solo informa al type checker, reconciliando los `Column[...]` genéricos con los tipos Python reales. Es una herramienta específica del gradual typing de Python; TS, al ser tipado de raíz, no necesita este truco (su problema es otro: los tipos del ORM ya vienen más resueltos).

- **#66 — Enum Postgres idempotente (`DO $$ ... duplicate_object`) y #15 (commit en handler / flush en service), #16 (validators sin Session), #18 (`with_for_update`), #19 (`response_model` + `from_attributes`), #40 (helpers `_`), #41 (keyword-only tras `*`), #49 (normalización `DATABASE_URL`), #52/#53/#54/#58 (fixtures, `monkeypatch`, autouse, reloj determinista).** Todo este conjunto se apoya en piezas que son marca registrada del ecosistema Python: el patrón flush/commit de la Session de SQLAlchemy, los `model_validator`/`field_validator` de Pydantic v2, los parámetros keyword-only con `*`, y el potentísimo `monkeypatch` de pytest que parchea por atributo de módulo sin tooling extra. En TS estas mismas ideas existen, pero distribuidas entre Zod, Prisma/TypeORM, Jest mocks y tipos estáticos; en Python conviven con una densidad y una uniformidad de "una sola forma idiomática de hacerlo" que es justamente lo que hace al tornillero coherente.

En resumen: TypeScript te da seguridad de tipos de fábrica y un front-end de primera; **Python te da `with`, generators con teardown, closures como dependencies, imports diferidos, `monkeypatch` quirúrgico y un ORM con properties híbridas** — y este Senior los explota todos. Esa es la ferretería extra que el lado Python suma al tornillero común.
