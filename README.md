WHITEPAPER

by Lerry Alexander Elizondo Villalobos alias LAEV 

Arquitectura de Solicitudes, Compromisos y Autorizaciones Criptográficas sin Custodia Obligatoria de Claves Privadas

Versión: 0.1 — Documento de concepto
Fecha de edición: 20 de septiembre de 2026
Naturaleza: Propuesta arquitectónica / protocolo de referencia

---

Resumen

Trilema Records propone una arquitectura de confianza institucional en la que la unidad principal de control deja de ser la cuenta y pasa a ser la solicitud verificable.

El sistema separa cinco capacidades que en muchas arquitecturas aparecen fuertemente acopladas: identidad, solicitud, autorización, ejecución y evidencia. La institución puede recibir, validar y ejecutar operaciones de acuerdo con sus políticas sin necesitar custodiar la clave privada con la que el titular demuestra control sobre su autorización.

La propuesta no inventa una nueva primitiva criptográfica. Combina componentes existentes —firmas digitales, hash/compromisos, control de acceso, políticas, registros auditables y, para operaciones críticas, esquemas multipartitos o threshold— dentro de una arquitectura de responsabilidades separadas.

OpenPGP es una posible capa criptográfica: RFC 9580 define formatos para cifrado, firmas digitales, compresión y gestión de claves, pero explícitamente no pretende ser una receta completa para almacenamiento o diseño de aplicaciones. Por tanto, GPG/OpenPGP puede implementar una parte del protocolo, no sustituir su modelo de gobernanza.

La hipótesis central es:

«Una institución debe poder administrar una autorización sin convertirse automáticamente en custodio de la capacidad privada que permite al titular producirla.»

---

1. Problema que resuelve

Los sistemas institucionales tienden a concentrar identidad, autenticación, autorización, ejecución y auditoría en un mismo dominio de confianza. Esto es operacionalmente conveniente, pero crea una superficie de compromiso donde el acceso administrativo puede convertirse, directa o indirectamente, en capacidad de actuar.

El problema arquitectónico no es únicamente “proteger las claves”. Es determinar qué capacidad pertenece a cada actor y cómo demostrar posteriormente que una operación ocurrió bajo las condiciones correctas.

Trilema Records plantea separar:

quién solicita → qué se solicita → quién puede autorizar → quién puede ejecutar → qué evidencia queda.

La institución continúa siendo necesaria. Lo que cambia es que deja de ser la única fuente de autoridad criptográfica.

---

2. Tesis

La arquitectura se apoya en seis tesis:

2.1 Solicitud ≠ autorización

Crear una solicitud no significa que la operación esté aprobada.

2.2 Firma ≠ ejecución

Una firma demuestra control de una clave o autenticidad dentro de un esquema criptográfico; no implica, por sí sola, que una institución deba ejecutar la operación.

2.3 Autorización ≠ custodia

Una institución puede verificar una autorización sin poseer la clave privada del titular.

2.4 Registro ≠ evidencia suficiente

La base de datos institucional registra estados, pero una evidencia criptográfica independiente puede permitir comprobar integridad y continuidad de determinados objetos.

2.5 Privacidad ≠ ausencia de auditoría

Puede conservarse evidencia verificable sin revelar todos los datos de una operación a todos los participantes.

2.6 Descentralización ≠ eliminación de instituciones

La meta es distribuir capacidades y puntos de confianza, no sustituir por completo la gobernanza institucional.

---

3. Arquitectura

Trilema Records se organiza en ocho capas.

Capa 1 — Identidad

Asocia una identidad operativa con una o más credenciales verificables.

Incluye, según el contexto:

- identificador del titular;
- clave pública o certificado;
- estado de vigencia;
- revocación;
- método de recuperación;
- atributos necesarios para la política.

Capa 2 — Solicitud

La solicitud es un objeto estructurado:

REQUEST_ID
ACTOR_ID
ACTION
RESOURCE
PARAMETERS
POLICY_ID
NONCE
CREATED_AT
EXPIRES_AT
VERSION

La solicitud debe tener una representación canónica para impedir ambigüedad al firmar o calcular compromisos.

Capa 3 — Compromiso

El sistema genera un compromiso criptográfico sobre el objeto solicitado:

COMMITMENT =
SHA-256(
  canonical_request
)

Para casos que requieran mayor separación entre datos y compromiso puede utilizarse una construcción con nonce:

COMMITMENT =
SHA-256(
  canonical_request || nonce
)

El compromiso permite detectar cambios posteriores del objeto comprometido.

Capa 4 — Autorización

El motor de políticas determina si la solicitud puede avanzar.

Ejemplos de condiciones:

- identidad válida;
- límite de monto;
- tipo de operación;
- doble aprobación;
- ventana temporal;
- relación entre solicitante y recurso;
- controles regulatorios;
- ausencia de revocación.

Capa 5 — Ejecución

La orden llega al sistema que materializa la acción: core institucional, tesorería, custodia, registro de activos o aplicación correspondiente.

Capa 6 — Evidencia

Cada transición relevante produce evidencia verificable:

EVENT_ID
REQUEST_ID
PREVIOUS_STATE
NEW_STATE
ACTOR
TIMESTAMP
POLICY_VERSION
EVIDENCE_HASH
SIGNATURE

Capa 7 — Auditoría

Un tercero puede reconstruir la secuencia de acontecimientos sin recibir claves privadas.

Capa 8 — Anclaje opcional

La huella de los registros puede anclarse periódicamente en una red externa o blockchain. Esto es opcional: el protocolo no depende de una blockchain para funcionar.

---

4. Modelo de estados

La máquina de estados de referencia es:

CREATED
   ↓
SUBMITTED
   ↓
VERIFIED
   ↓
COMMITTED
   ↓
PENDING_APPROVAL
   ↓
AUTHORIZED
   ↓
EXECUTED
   ↓
CLOSED

Estados alternativos:

REJECTED
CANCELLED
EXPIRED
REVOKED
FAILED
DISPUTED

La regla fundamental es que una transición de estado no debe borrar conceptualmente el estado anterior. Debe generar una nueva evidencia que permita reconstruir la secuencia.

---

5. Protocolo de autorización

Paso 1 — Creación

El titular crea una solicitud.

Paso 2 — Canonicalización

El cliente transforma la solicitud a un formato exacto y estable.

Paso 3 — Compromiso

Se calcula el hash del objeto.

Paso 4 — Firma

El titular firma el objeto o el compromiso utilizando su capacidad privada.

La institución recibe la firma y la verifica con la información pública correspondiente.

Paso 5 — Validación institucional

La institución aplica identidad, reglas operativas, límites y controles regulatorios.

Paso 6 — Autorización

La solicitud pasa a "AUTHORIZED" sólo si satisface la política.

Paso 7 — Ejecución

El sistema operacional ejecuta la acción.

Paso 8 — Evidencia

Se genera un evento de cierre y se enlaza con la solicitud, la autorización y el resultado.

---

6. Propiedad clave: no custodiar la clave privada del titular

El principio puede expresarse formalmente:

PRIVATE_KEY_USER ∉ REQUIRED_INSTITUTIONAL_STATE

Es decir: el funcionamiento del sistema no debe depender de que la institución posea la clave privada del usuario.

Esto no impide que la institución tenga sus propias claves privadas. Puede necesitar claves institucionales para:

- firmar eventos;
- autenticar servicios;
- proteger comunicaciones;
- producir certificados;
- firmar evidencias;
- operar componentes de infraestructura.

La separación importante es entre:

capacidad privada del titular

y

capacidad institucional de procesar la solicitud.

---

7. Operaciones de alto riesgo

Las operaciones críticas pueden incorporar autorización multipartita.

Ejemplos:

2 de 3
3 de 5
4 de 7

La selección depende del riesgo, disponibilidad y modelo de gobernanza.

La criptografía threshold permite distribuir una capacidad criptográfica entre múltiples participantes, de modo que una cantidad mínima pueda colaborar para producir una operación válida sin requerir que una sola parte tenga el secreto completo. NIST publicó en enero de 2026 un llamado específico para esquemas multipartitos threshold, lo que confirma que este espacio continúa siendo objeto de estandarización y evaluación técnica.

Trilema Records puede utilizar estas técnicas cuando el riesgo justifique distribuir la capacidad de aprobación o ejecución.

---

8. Threat model

El modelo considera como amenazas principales:

8.1 Compromiso de la base de datos

Un atacante modifica estados o registros.

Mitigación: hashes, firmas de eventos, registros append-only y reconciliación.

8.2 Administrador malicioso

Un operador intenta autorizar una operación fuera de su atribución.

Mitigación: segregación de funciones y políticas multipartitas.

8.3 Compromiso de un servidor de aplicación

Un servidor recibe una sesión legítima pero intenta alterar el objeto autorizado.

Mitigación: firma sobre el objeto canónico y validación independiente.

8.4 Reutilización de una solicitud

Un atacante intenta reproducir una autorización antigua.

Mitigación: nonce, expiración, identificador único y control de replay.

8.5 Pérdida de una clave privada

El usuario pierde su capacidad de firma.

Mitigación: recuperación previamente diseñada, revocación y esquemas de recuperación multipartitos.

8.6 Colusión

Varios actores intentan producir una autorización fraudulenta.

Mitigación: número suficiente de participantes independientes, separación organizacional y monitoreo.

La criptografía no resuelve por sí sola la colusión, los errores humanos ni los problemas regulatorios.

---

9. Invariantes del protocolo

Una implementación de referencia debe mantener al menos estas propiedades:

Invariante I

Una solicitud ejecutada debe corresponder a una solicitud identificable.

Invariante II

Una autorización debe referirse al objeto que fue validado.

Invariante III

Una modificación de los parámetros sustantivos debe producir un objeto criptográficamente distinto.

Invariante IV

Una autorización expirada o revocada no debe ser reutilizable sin una nueva autorización válida.

Invariante V

Una operación crítica no debe depender de una sola credencial cuando la política exija aprobación multipartita.

Invariante VI

La evidencia posterior debe permitir distinguir entre:

SOLICITADA
AUTORIZADA
EJECUTADA

---

10. Diferencia frente al paradigma tradicional

El paradigma convencional puede resumirse como:

USUARIO
   ↓
CUENTA
   ↓
INSTITUCIÓN
   ↓
SISTEMA
   ↓
OPERACIÓN

La arquitectura propuesta introduce una unidad explícita intermedia:

IDENTIDAD
   ↓
SOLICITUD
   ↓
COMPROMISO
   ↓
FIRMA
   ↓
POLÍTICA
   ↓
AUTORIZACIÓN
   ↓
EJECUCIÓN
   ↓
EVIDENCIA

La diferencia no reside en que la segunda secuencia utilice “más criptografía”, sino en que convierte la intención operativa y su autorización en objetos separados y verificables.

---

11. Por qué puede aportar ventajas

Las ventajas esperadas son arquitectónicas, no universales.

El modelo puede ser especialmente útil cuando se requiere:

- separación fuerte de funciones;
- trazabilidad de operaciones;
- múltiples autorizadores;
- auditoría independiente;
- reducción de custodia de secretos;
- límites programables;
- recuperación distribuida;
- integración de sistemas heterogéneos.

El costo es mayor complejidad de diseño y operación.

Por tanto, la propuesta no afirma que este modelo deba sustituir toda arquitectura existente. Propone utilizarlo cuando el beneficio de separar capacidades supera ese costo.

---

12. Relación con GPG/OpenPGP

OpenPGP debe considerarse una capa criptográfica, no una arquitectura de negocio.

RFC 9580 especifica formatos de mensajes, firmas digitales, cifrado, compresión y manejo de claves, y señala expresamente que cuestiones de almacenamiento e implementación quedan fuera de su alcance.

En este whitepaper:

OpenPGP/GPG
      ↓
CRIPTOGRAFÍA
      ↓
Trilema Records
      ↓
GOBERNANZA + POLÍTICAS + ESTADOS + EVIDENCIA
      ↓
SISTEMA INSTITUCIONAL

Esto permite usar GPG donde resulte conveniente sin obligar a que el sistema completo sea diseñado alrededor de GPG.

---

13. Blockchain: componente opcional

La propuesta puede funcionar sin blockchain.

Una blockchain puede utilizarse para:

- anclar hashes;
- proporcionar una referencia temporal externa;
- permitir verificación pública;
- distribuir parte del registro.

Pero introducir blockchain no corrige automáticamente:

- políticas mal diseñadas;
- claves robadas;
- identidad incorrecta;
- autorizaciones fraudulentas;
- datos incorrectos;
- errores de ejecución.

El valor de la arquitectura debe existir incluso si el anclaje externo desaparece.

---

14. Modelo de gobernanza

Cada actor recibe una capacidad definida.

Titular

Puede:

- crear solicitudes;
- firmar;
- revisar;
- cancelar cuando proceda;
- revocar sus credenciales.

Institución

Puede:

- validar;
- aplicar políticas;
- aprobar o rechazar;
- ejecutar;
- registrar evidencias.

Auditor

Puede:

- verificar firmas;
- reconstruir estados;
- comprobar hashes;
- revisar políticas aplicadas.

Operadores críticos

Pueden ejecutar funciones limitadas según autorización y separación de responsabilidades.

---

15. Privacidad

La arquitectura debe evitar que “auditable” signifique “público”.

Una implementación puede separar:

DATOS OPERATIVOS
DATOS CONFIDENCIALES
METADATOS
COMPROMISOS
EVIDENCIA

Los auditores pueden recibir evidencia suficiente para verificar una propiedad sin recibir todos los datos personales de la operación.

Los compromisos hash son una herramienta posible, no una garantía automática de privacidad: el diseño debe considerar inferencia, tamaños de dominio, metadatos y controles de acceso.

---

16. Recuperación y continuidad

La recuperación es parte del protocolo, no una función añadida después.

Debe definirse:

- pérdida de credenciales;
- sustitución de firmantes;
- rotación;
- revocación;
- fallecimiento o incapacidad;
- salida de una institución;
- pérdida de un participante threshold;
- recuperación ante desastre.

La arquitectura debe evitar que “no custodiar la clave privada” produzca un sistema irrecuperable.

---

17. Modelo de integración institucional

La integración recomendada es por capas:

CLIENTE
  │
  ▼
TRILEMA RECORDS
  │
  ├── Identidad
  ├── Solicitudes
  ├── Criptografía
  ├── Políticas
  ├── Evidencia
  │
  ▼
SISTEMA EXISTENTE
  │
  ├── Core bancario
  ├── Contabilidad
  ├── KYC/AML
  ├── Tesorería
  └── Sistemas regulatorios

La propuesta, por tanto, puede actuar como una capa de control y evidencia sobre infraestructura existente en vez de exigir una sustitución inmediata.

---

18. Casos de uso

Cooperativas

- autorizaciones de desembolso;
- cambios sensibles;
- aprobaciones de crédito;
- operaciones extraordinarias;
- administración multipartita.

Banca

- autorizaciones de alto valor;
- operaciones excepcionales;
- firmas corporativas;
- aprobación por comité;
- flujos de custodia.

Tesorería institucional

- liberación de fondos;
- pagos extraordinarios;
- cambios de beneficiarios;
- operaciones de reserva.

Ecosistemas digitales

- activos tokenizados;
- organizaciones distribuidas;
- bóvedas de datos;
- mandatos verificables.

---

19. Arquitectura de referencia

Una implementación mínima puede componerse de:

CLIENT
  |
  +-- Canonicalizer
  |
  +-- Signer
  |
  v
REQUEST API
  |
  +-- Validator
  +-- Policy Engine
  +-- Evidence Log
  +-- State Machine
  |
  v
EXECUTION ADAPTER
  |
  v
INSTITUTIONAL SYSTEM

Componentes opcionales:

Threshold Service
External Timestamp
Blockchain Anchor
Independent Auditor Node
Recovery Service
Key Transparency Log

---

20. Esquema de un objeto de solicitud

{
  "request_id": "TRX-000001",
  "actor_id": "ACTOR-123",
  "action": "RELEASE",
  "resource": "ASSET-456",
  "parameters": {
    "amount": "100.00",
    "currency": "CRC"
  },
  "policy_id": "POLICY-7",
  "nonce": "random-value",
  "created_at": "2026-09-20T20:00:00-06:00",
  "expires_at": "2026-09-20T20:15:00-06:00",
  "version": "1"
}

La implementación real debe establecer un formato canónico normativo antes de permitir firmas.

---

21. Seguridad criptográfica

La selección final de algoritmos no debe congelarse únicamente por conveniencia de una herramienta.

El protocolo debe definir:

- algoritmos permitidos;
- tamaños de clave;
- formatos de firma;
- política de rotación;
- revocación;
- protección de nonce;
- hashing;
- serialización canónica;
- almacenamiento de claves;
- respuesta ante compromiso.

La selección deberá revisarse conforme evolucionen los estándares y las capacidades de los adversarios.

---

22. Reglas de diseño

Regla 1

Nunca firmar una estructura ambigua.

Regla 2

Nunca ejecutar una operación únicamente porque existe una firma.

Regla 3

Nunca interpretar una autorización sin conocer la política que la gobierna.

Regla 4

Nunca depender de una única copia de una evidencia crítica.

Regla 5

Nunca convertir la recuperación en una puerta trasera secreta.

Regla 6

Toda capacidad crítica debe poder revocarse.

---

23. Limitaciones

La propuesta no resuelve automáticamente:

- validez jurídica en todas las jurisdicciones;
- KYC/AML;
- fraude fuera del sistema;
- compromiso del dispositivo del usuario;
- errores de identidad;
- corrupción del sistema que ejecuta la operación;
- colusión de participantes suficientes para superar un umbral;
- pérdida total de las credenciales y del mecanismo de recuperación.

Tampoco debe presentarse como “trustless”. La confianza siempre existe; el objetivo es reducir la concentración y hacer sus condiciones explícitas y verificables.

---

24. Plan de implementación

Fase 0 — Especificación

Definir:

- modelos;
- estados;
- políticas;
- formato canónico;
- threat model;
- criterios de auditoría.

Fase 1 — Prototipo

Implementar:

- solicitud;
- hash;
- firma;
- verificación;
- estados;
- evidencia.

Sin dinero real.

Fase 2 — Autorización multipartita

Añadir:

- multifirma;
- threshold;
- recuperación;
- revocación.

Fase 3 — Integración

Conectar con:

- identidad;
- core institucional;
- motor de políticas;
- auditoría.

Fase 4 — Operación controlada

Comenzar con operaciones reversibles y límites estrictos.

Fase 5 — Auditoría externa

Intentar romper el modelo mediante pruebas de:

- manipulación;
- replay;
- escalamiento de privilegios;
- pérdida de claves;
- colusión;
- modificación de registros.

---

25. Criterios de éxito

El protocolo será técnicamente exitoso si puede demostrar, de manera reproducible:

1. que una solicitud específica fue creada;
2. que su contenido exacto puede verificarse;
3. que la firma corresponde a la credencial esperada;
4. que la solicitud estaba sujeta a una política identificable;
5. que la autorización fue emitida bajo esa política;
6. que la ejecución corresponde a la autorización;
7. que las evidencias posteriores no contradicen la secuencia;
8. que la institución puede operar sin custodiar la clave privada del titular, cuando esa sea la política elegida.

---

26. Aporte conceptual

La propuesta puede resumirse en una modificación del objeto fundamental del sistema.

Modelo tradicional

Cuenta → permiso → ejecución

Modelo Trilema

Identidad → solicitud → compromiso → firma → política → autorización → ejecución → evidencia

El objeto central deja de ser únicamente “la cuenta que tiene acceso”.

Pasa a ser:

«una operación con identidad, condiciones, autoridad y evidencia verificables.»

Esto permite pensar instituciones como sistemas de gestión de compromisos verificables, no solamente como repositorios de cuentas y permisos.

---

27. Declaración del protocolo

Trilema Records propone el siguiente principio fundacional:

«Ninguna institución debería necesitar custodiar un secreto privado que no requiere para ejecutar la función que le corresponde.

Toda autorización crítica debería poder distinguirse de la solicitud que la originó y de la ejecución que finalmente ocurrió.

Toda evidencia crítica debería poder verificarse independientemente del componente que realizó la operación.»

---

28. Conclusión

Trilema Records es una arquitectura de confianza distribuida orientada a instituciones que necesitan combinar seguridad, trazabilidad, separación de funciones y capacidad de auditoría.

Su innovación propuesta no reside en reemplazar OpenPGP, GPG, blockchain o los sistemas institucionales existentes. Su valor potencial está en orquestarlos alrededor de una unidad de operación explícita: la solicitud comprometida, firmada, autorizada, ejecutada y evidenciada.

OpenPGP aporta una capa estandarizada de criptografía e intercambio de mensajes; los esquemas threshold permiten estudiar la distribución de capacidades críticas; las bases de datos y sistemas institucionales continúan proporcionando operación y cumplimiento. RFC 9580 delimita claramente el papel de OpenPGP, mientras que el trabajo reciente de NIST sobre esquemas threshold confirma la relevancia actual de la criptografía multipartita como área técnica.

La tesis final no es que la institución deba desaparecer.

Es que la institución puede dejar de ser el único lugar donde residen simultáneamente todos los secretos, todas las autorizaciones y toda la evidencia.

---

Anexo I — Fórmula de compromiso

C = SHA-256(CANONICAL_REQUEST || NONCE || VERSION)

Anexo II — Evidencia encadenada

E_n =
SHA-256(
  EVENT_n ||
  E_(n-1)
)

El mecanismo produce una cadena de evidencia donde cada evento incorpora una referencia criptográfica al precedente.

Anexo III — Principio operativo

REQUEST
   +
VALIDATION
   +
SIGNATURE
   +
POLICY
   +
AUTHORIZATION
   +
EXECUTION
   =
EVIDENCE

Anexo IV — Posición tecnológica

GPG/OpenPGP: criptografía y formatos interoperables.

SHA-256 u otro hash aprobado: integridad y compromiso.

Threshold/multipartite: distribución de capacidades críticas.

Policy Engine: gobernanza.

State Machine: control del ciclo de vida.

Evidence Log: auditoría.

Core institucional: ejecución.

Blockchain/anclaje externo: componente opcional.

---

Estado del documento: Propuesta conceptual. No constituye certificación de seguridad, asesoría jurídica ni especificación normativa final. Antes de una implementación real deberán completarse revisión criptográfica independiente, threat modeling formal, pruebas de seguridad, definición jurídica de firmas y autorizaciones, y pruebas de interoperabilidad.
