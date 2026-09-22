WHITEPAPER

by Lerry Alexander Elizondo Villalobos, alias LAEV; Inspired by the work done in the past by Mierca Popescu with GPG technology. 

Architecture for Requests, Commitments, and Cryptographic Authorizations Without Mandatory Private-Key Custody

Version: 0.1 — Concept Document
Edition Date: September 20, 2026
Nature: Architectural Proposal / Reference Protocol

---

Abstract

I, byLAEV, proposes an institutional trust architecture in which the primary unit of control ceases to be the account and becomes the verifiable request.

The system separates five capabilities that are often tightly coupled in many architectures: identity, request, authorization, execution, and evidence. An institution can receive, validate, and execute operations according to its policies without needing to custody the private key through which the holder demonstrates control over their authorization.

The proposal does not introduce a new cryptographic primitive. It combines existing components — digital signatures, hashing/commitments, access control, policies, auditable records, and, for critical operations, multipartite or threshold schemes — within an architecture based on separated responsibilities.

OpenPGP is one possible cryptographic layer: RFC 9580 defines formats for encryption, digital signatures, compression, and key management, but explicitly does not intend to provide a complete recipe for storage or application design. Therefore, GPG/OpenPGP can implement part of the protocol, but it does not replace its governance model.

The central hypothesis is:

«An institution should be able to administer an authorization without automatically becoming the custodian of the private capability that allows the holder to produce it.»

---

1. Problem Being Addressed

Institutional systems tend to concentrate identity, authentication, authorization, execution, and auditing within the same trust domain. This is operationally convenient, but it creates a compromise surface in which administrative access can become, directly or indirectly, the ability to act.

The architectural problem is not merely “protecting keys.” It is determining which capability belongs to each actor and how to subsequently demonstrate that an operation occurred under the correct conditions.

Trilema Records proposes separating:

who requests → what is requested → who may authorize → who may execute → what evidence remains.

The institution remains necessary. What changes is that it ceases to be the sole source of cryptographic authority.

---

2. Thesis

The architecture is based on six theses:

2.1 Request ≠ Authorization

Creating a request does not mean that the operation has been approved.

2.2 Signature ≠ Execution

A signature demonstrates control of a key or authenticity within a cryptographic scheme; by itself, it does not mean that an institution must execute the operation.

2.3 Authorization ≠ Custody

An institution can verify an authorization without possessing the holder's private key.

2.4 Record ≠ Sufficient Evidence

An institutional database records states, but independent cryptographic evidence can make it possible to verify the integrity and continuity of certain objects.

2.5 Privacy ≠ Absence of Auditing

Verifiable evidence can be preserved without revealing all operational data to every participant.

2.6 Decentralization ≠ Elimination of Institutions

The objective is to distribute capabilities and trust points, not to completely replace institutional governance.

---

3. Architecture

Trilema Records is organized into eight layers.

Layer 1 — Identity

Associates an operational identity with one or more verifiable credentials.

Depending on the context, this may include:

- holder identifier;
- public key or certificate;
- validity status;
- revocation;
- recovery mechanism;
- attributes required by the policy.

Layer 2 — Request

The request is a structured object:

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

The request must have a canonical representation to prevent ambiguity when signing or calculating commitments.

Layer 3 — Commitment

The system generates a cryptographic commitment to the requested object:

COMMITMENT =
SHA-256(
  canonical_request
)

For cases requiring greater separation between data and commitment, a nonce-based construction may be used:

COMMITMENT =
SHA-256(
  canonical_request || nonce
)

The commitment makes subsequent changes to the committed object detectable.

Layer 4 — Authorization

The policy engine determines whether the request may proceed.

Examples of conditions include:

- valid identity;
- amount limit;
- operation type;
- dual approval;
- time window;
- relationship between requester and resource;
- regulatory controls;
- absence of revocation.

Layer 5 — Execution

The instruction reaches the system that actually materializes the action: institutional core, treasury, custody system, asset registry, or corresponding application.

Layer 6 — Evidence

Each relevant state transition produces verifiable evidence:

EVENT_ID
REQUEST_ID
PREVIOUS_STATE
NEW_STATE
ACTOR
TIMESTAMP
POLICY_VERSION
EVIDENCE_HASH
SIGNATURE

Layer 7 — Audit

A third party can reconstruct the sequence of events without receiving private keys.

Layer 8 — Optional Anchoring

The fingerprint of records may periodically be anchored to an external network or blockchain. This is optional: the protocol does not depend on a blockchain to function.

---

4. State Model

The reference state machine is:

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

Alternative states:

REJECTED
CANCELLED
EXPIRED
REVOKED
FAILED
DISPUTED

The fundamental rule is that a state transition must not conceptually erase the previous state. It should generate new evidence that allows the sequence to be reconstructed.

---

5. Authorization Protocol

Step 1 — Creation

The holder creates a request.

Step 2 — Canonicalization

The client transforms the request into an exact and stable format.

Step 3 — Commitment

The hash of the object is calculated.

Step 4 — Signature

The holder signs the object or commitment using their private capability.

The institution receives the signature and verifies it using the corresponding public information.

Step 5 — Institutional Validation

The institution applies identity, operational rules, limits, and regulatory controls.

Step 6 — Authorization

The request enters "AUTHORIZED" only if it satisfies the policy.

Step 7 — Execution

The operational system executes the action.

Step 8 — Evidence

A closing event is generated and linked to the request, authorization, and result.

---

6. Key Property: No Custody of the Holder's Private Key

The principle can be expressed formally:

PRIVATE_KEY_USER ∉ REQUIRED_INSTITUTIONAL_STATE

That is: system operation must not depend on the institution possessing the user's private key.

This does not prevent the institution from having its own private keys. It may require institutional keys to:

- sign events;
- authenticate services;
- protect communications;
- issue certificates;
- sign evidence;
- operate infrastructure components.

The important separation is between:

the holder's private capability

and

the institution's capability to process the request.

---

7. High-Risk Operations

Critical operations may incorporate multipartite authorization.

Examples:

2 of 3
3 of 5
4 of 7

The selection depends on risk, availability, and the governance model.

Threshold cryptography allows a cryptographic capability to be distributed among multiple participants, so that a minimum number can collaborate to produce a valid operation without requiring a single party to possess the complete secret. NIST published a specific call for threshold multipartite schemes in January 2026, confirming that this area continues to receive active standardization and technical evaluation.

Trilema Records may use these techniques when the risk justifies distributing approval or execution capabilities.

---

8. Threat Model

The model considers the following primary threats:

8.1 Database Compromise

An attacker modifies states or records.

Mitigation: hashes, event signatures, append-only records, and reconciliation.

8.2 Malicious Administrator

An operator attempts to authorize an operation outside their assigned authority.

Mitigation: separation of duties and multipartite policies.

8.3 Application Server Compromise

A server receives a legitimate session but attempts to alter the authorized object.

Mitigation: signing of the canonical object and independent validation.

8.4 Request Replay

An attacker attempts to reuse an old authorization.

Mitigation: nonce, expiration, unique identifier, and replay protection.

8.5 Private-Key Loss

The user loses their signing capability.

Mitigation: previously designed recovery, revocation, and multipartite recovery schemes.

8.6 Collusion

Multiple actors attempt to produce a fraudulent authorization.

Mitigation: a sufficient number of independent participants, organizational separation, and monitoring.

Cryptography alone does not solve collusion, human error, or regulatory problems.

---

9. Protocol Invariants

A reference implementation should maintain at least the following properties:

Invariant I

An executed request must correspond to an identifiable request.

Invariant II

An authorization must refer to the object that was validated.

Invariant III

A modification to substantive parameters must produce a cryptographically distinct object.

Invariant IV

An expired or revoked authorization must not be reusable without a new valid authorization.

Invariant V

A critical operation must not depend on a single credential when the policy requires multipartite approval.

Invariant VI

Subsequent evidence must allow the system to distinguish between:

REQUESTED
AUTHORIZED
EXECUTED

---

10. Difference from the Traditional Paradigm

The conventional paradigm can be summarized as:

USER
   ↓
ACCOUNT
   ↓
INSTITUTION
   ↓
SYSTEM
   ↓
OPERATION

The proposed architecture introduces an explicit intermediate unit:

IDENTITY
   ↓
REQUEST
   ↓
COMMITMENT
   ↓
SIGNATURE
   ↓
POLICY
   ↓
AUTHORIZATION
   ↓
EXECUTION
   ↓
EVIDENCE

The difference is not that the second sequence simply uses “more cryptography.” It is that it converts operational intent and its authorization into separate and verifiable objects.

---

11. Why It May Provide Advantages

The expected advantages are architectural, not universal.

The model may be particularly useful where the system requires:

- strong separation of duties;
- operational traceability;
- multiple authorizers;
- independent auditing;
- reduced custody of secrets;
- programmable limits;
- distributed recovery;
- integration of heterogeneous systems.

The cost is greater design and operational complexity.

Therefore, the proposal does not claim that this model should replace every existing architecture. It proposes using it when the benefit of separating capabilities outweighs that cost.

---

12. Relationship with GPG/OpenPGP

OpenPGP should be considered a cryptographic layer, not a business architecture.

RFC 9580 specifies message formats, digital signatures, encryption, compression, and key management, and expressly notes that storage and implementation questions are outside its scope.

Within this whitepaper:

OpenPGP/GPG
      ↓
CRYPTOGRAPHY
      ↓
Trilema Records
      ↓
GOVERNANCE + POLICIES + STATES + EVIDENCE
      ↓
INSTITUTIONAL SYSTEM

This makes it possible to use GPG where appropriate without requiring the entire system to be designed around GPG.

---

13. Blockchain: Optional Component

The proposal can operate without blockchain.

A blockchain may be used to:

- anchor hashes;
- provide an external temporal reference;
- enable public verification;
- distribute part of the record.

However, introducing blockchain does not automatically correct:

- poorly designed policies;
- stolen keys;
- incorrect identity;
- fraudulent authorizations;
- incorrect data;
- execution errors.

The value of the architecture should remain even if the external anchoring layer disappears.

---

14. Governance Model

Each actor receives a defined capability.

Holder

May:

- create requests;
- sign;
- review;
- cancel where applicable;
- revoke their credentials.

Institution

May:

- validate;
- apply policies;
- approve or reject;
- execute;
- record evidence.

Auditor

May:

- verify signatures;
- reconstruct states;
- verify hashes;
- review applied policies.

Critical Operators

May execute limited functions according to authorization and separation-of-duty rules.

---

15. Privacy

The architecture must prevent “auditable” from becoming synonymous with “public.”

An implementation may separate:

OPERATIONAL DATA
CONFIDENTIAL DATA
METADATA
COMMITMENTS
EVIDENCE

Auditors may receive sufficient evidence to verify a property without receiving all personal data associated with the operation.

Hash commitments are a possible tool, not an automatic privacy guarantee: the design must consider inference, domain sizes, metadata, and access controls.

---

16. Recovery and Continuity

Recovery is part of the protocol, not a feature added afterward.

The system must define:

- credential loss;
- signer replacement;
- key rotation;
- revocation;
- death or incapacity;
- institutional departure;
- loss of a threshold participant;
- disaster recovery.

The architecture must prevent “not holding the private key” from producing an unrecoverable system.

---

17. Institutional Integration Model

The recommended integration is layered:

CLIENT
  │
  ▼
TRILEMA RECORDS
  │
  ├── Identity
  ├── Requests
  ├── Cryptography
  ├── Policies
  ├── Evidence
  │
  ▼
EXISTING SYSTEM
  │
  ├── Banking Core
  ├── Accounting
  ├── KYC/AML
  ├── Treasury
  └── Regulatory Systems

The proposal can therefore operate as a control and evidence layer over existing infrastructure rather than requiring immediate replacement.

---

18. Use Cases

Cooperatives

- disbursement authorizations;
- sensitive changes;
- credit approvals;
- extraordinary operations;
- multipartite administration.

Banking

- high-value authorizations;
- exceptional operations;
- corporate signatures;
- committee approvals;
- custody workflows.

Institutional Treasury

- fund releases;
- extraordinary payments;
- beneficiary changes;
- reserve operations.

Digital Ecosystems

- tokenized assets;
- distributed organizations;
- data vaults;
- verifiable mandates.

---

19. Reference Architecture

A minimal implementation may consist of:

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

Optional components:

Threshold Service
External Timestamp
Blockchain Anchor
Independent Auditor Node
Recovery Service
Key Transparency Log

---

20. Request Object Schema

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

The actual implementation must establish a normative canonical format before signatures are permitted.

---

21. Cryptographic Security

The final selection of algorithms should not be frozen merely for the convenience of a particular tool.

The protocol must define:

- permitted algorithms;
- key sizes;
- signature formats;
- rotation policy;
- revocation;
- nonce protection;
- hashing;
- canonical serialization;
- key storage;
- response to compromise.

The selection should be reviewed as standards and adversarial capabilities evolve.

---

22. Design Rules

Rule 1

Never sign an ambiguous structure.

Rule 2

Never execute an operation merely because a signature exists.

Rule 3

Never interpret an authorization without knowing the policy governing it.

Rule 4

Never depend on a single copy of critical evidence.

Rule 5

Never turn recovery into a secret backdoor.

Rule 6

Every critical capability must be revocable.

---

23. Limitations

The proposal does not automatically solve:

- legal validity in every jurisdiction;
- KYC/AML;
- fraud outside the system;
- compromise of the user's device;
- identity errors;
- corruption of the system executing the operation;
- collusion by enough participants to exceed a threshold;
- total loss of credentials and recovery mechanisms.

Nor should it be presented as “trustless.” Trust always exists; the objective is to reduce concentration and make its conditions explicit and verifiable.

---

24. Implementation Roadmap

Phase 0 — Specification

Define:

- models;
- states;
- policies;
- canonical format;
- threat model;
- audit criteria.

Phase 1 — Prototype

Implement:

- request;
- hash;
- signature;
- verification;
- states;
- evidence.

No real money.

Phase 2 — Multipartite Authorization

Add:

- multisignature;
- threshold;
- recovery;
- revocation.

Phase 3 — Integration

Connect with:

- identity;
- institutional core;
- policy engine;
- auditing.

Phase 4 — Controlled Operation

Begin with reversible operations and strict limits.

Phase 5 — External Audit

Attempt to break the model through testing of:

- manipulation;
- replay;
- privilege escalation;
- key loss;
- collusion;
- record modification.

---

25. Success Criteria

The protocol will be technically successful if it can reproducibly demonstrate:

1. that a specific request was created;
2. that its exact content can be verified;
3. that the signature corresponds to the expected credential;
4. that the request was governed by an identifiable policy;
5. that authorization was issued under that policy;
6. that execution corresponds to the authorization;
7. that subsequent evidence does not contradict the sequence;
8. that the institution can operate without custodizing the holder's private key, where that is the selected policy.

---

26. Conceptual Contribution

The proposal can be summarized as a modification of the system's fundamental object.

Traditional Model

Account → Permission → Execution

Trilema Model

Identity → Request → Commitment → Signature → Policy → Authorization → Execution → Evidence

The central object is no longer merely “the account that has access.”

It becomes:

«an operation with verifiable identity, conditions, authority, and evidence.»

This makes it possible to conceptualize institutions as systems for managing verifiable commitments, rather than merely repositories of accounts and permissions.

---

27. Protocol Declaration

Trilema Records proposes the following foundational principle:

«No institution should need to custody a private secret that it does not require to perform the function assigned to it.

Every critical authorization should be distinguishable from the request that originated it and from the execution that ultimately occurred.

Every critical piece of evidence should be independently verifiable from the component that performed the operation.»

---

28. Conclusion

Trilema Records is a distributed-trust architecture intended for institutions that need to combine security, traceability, separation of duties, and auditability.

Its proposed innovation does not lie in replacing OpenPGP, GPG, blockchain, or existing institutional systems. Its potential value lies in orchestrating them around an explicit operational unit: the committed, signed, authorized, executed, and evidenced request.

OpenPGP provides a standardized cryptographic and message-exchange layer; threshold schemes enable the study of distributed critical capabilities; databases and institutional systems continue to provide operations and compliance. RFC 9580 clearly defines the role of OpenPGP, while recent NIST work on threshold schemes confirms the continuing relevance of multipartite cryptography as a technical field.

The final thesis is not that institutions should disappear.

It is that an institution can cease to be the only place where all secrets, all authorizations, and all evidence simultaneously reside.

---

Appendix I — Commitment Formula

C = SHA-256(CANONICAL_REQUEST || NONCE || VERSION)

Appendix II — Chained Evidence

E_n =
SHA-256(
  EVENT_n ||
  E_(n-1)
)

This mechanism produces an evidence chain in which each event incorporates a cryptographic reference to the preceding event.

Appendix III — Operational Principle

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

Appendix IV — Technology Position

GPG/OpenPGP: cryptography and interoperable formats.

SHA-256 or another approved hash: integrity and commitment.

Threshold/multipartite: distribution of critical capabilities.

Policy Engine: governance.

State Machine: lifecycle control.

Evidence Log: auditing.

Institutional Core: execution.

Blockchain/external anchoring: optional component.

---

Document Status: Conceptual proposal. This document does not constitute a security certification, legal advice, or final normative specification. Before real-world implementation, independent cryptographic review, formal threat modeling, security testing, legal definition of signatures and authorizations, and interoperability testing must be completed.

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
