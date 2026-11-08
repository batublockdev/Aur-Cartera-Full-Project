# Aur Cartera — Decentralized Savings Groups & Self-Custodial Stellar Infrastructure

Aur Cartera es una infraestructura de finanzas descentralizadas y ahorro grupal ("natilleras digitales") diseñada para impulsar la inclusión financiera de comunidades en América Latina. El sistema conecta la red blockchain Stellar con interfaces conversacionales de WhatsApp y una aplicación móvil no custodia desarrollada en React Native, permitiendo a los usuarios administrar ahorros colectivos, realizar transferencias en activos estables y autorizar transacciones multifirma sin exponer sus claves privadas a servidores intermedios.

---

## 1. Arquitectura General del Sistema

El ecosistema de Aur Cartera opera mediante una arquitectura desacoplada que separa la captura de comandos del usuario, la orquestación de servicios backend y la custodia criptográfica en el cliente.

### Componentes Principales
* **Backend API (`aur-api`)**: Servidor Node.js que gestiona la interacción con la API Graph de WhatsApp (v18.0), procesa intenciones conversacionales mediante inteligencia artificial (Google Gemini), mantiene el estado de los grupos de ahorro en PostgreSQL y construye sobres de transacción XDR sin firmar.
* **Aplicación Móvil (`aur-accesscontrol-app`)**: Cliente React Native que implementa el módulo local `self-wallet` para la generación de mnemónicos, cifrado de semillas Ed25519 y firmado de transacciones en el dispositivo móvil.
* **Red Blockchain Stellar**: Red descentralizada que procesa y liquida operaciones de activos (XLM y USDC), reservas patrocinadas (*sponsorship*) y autorizaciones multifirma mediante el servidor Horizon (`horizon.stellar.org`).

---

## 2. Desglose Tecnológico por Repositorio

### 2.1 Orquestador Conversacional y Backend API (`aur-api`)

El servicio `aur-api` centraliza la lógica de negocio y la comunicación con la plataforma de mensajería WhatsApp.

* **Flujos Conversacionales Interactivos**: Implementa menús interactivos de WhatsApp (`sendMenu`) como `ONBOARDING`, `GROUPS_HOME`, `MY_MONEY` y `CREATE_GROUP`, permitiendo a los usuarios consultar saldos, crear natilleras e invitar miembros mediante enlaces directos (`wa.me`).
* **Conversión de Divisas e Indicadores TRM**: Integra APIs de tasa de cambio en tiempo real (USD/COP) mediante funciones como `getUsdToCop` y `formatWithCop`, ofreciendo una visualización clara del saldo en dólares USDC y su equivalente aproximado en pesos colombianos.
* **Servicio de Firmado de Administración**: Expone el endpoint `/api/sign-transaction` para firmar operaciones administrativas de patrocinio de cuentas y coordinar solicitudes de firma a través del endpoint `/transactions/:pubkey`.

### 2.2 Módulo Criptográfico de Autocustodia (`aur-accesscontrol-app/self-wallet/wallet.js`)

El módulo `wallet.js` gestiona el ciclo de vida criptográfico del usuario en el cliente móvil.

* **Generación y Derivación de Semillas**: Utiliza `@scure/bip39` para generar mnemónicos de 12 palabras y derivar semillas raw de 32 bytes para crear pares de claves Ed25519 con `@stellar/stellar-sdk`.
* **Cifrado Simétrico Local**: Emplea `nacl.secretbox` (`tweetnacl`) junto con hashes SHA-512 derivados de la contraseña del usuario para cifrar la semilla antes de persistirla en el dispositivo.
* **Patrocinio de Cuentas (Sponsorship)**: Configura las operaciones `beginSponsoringFutureReserves` y `endSponsoringFutureReserves` para permitir que una cuenta administradora patrocine las reservas mínimas de XLM requeridas por nuevas cuentas.
* **Gestión de Firmantes y Trustlines**: Ejecuta funciones como `ChangeSigner`, `AddSigner` y `Add_AdmSign_Trustline` para establecer líneas de confianza para el activo USDC y ajustar los pesos de firma (*masterWeight* y firmantes secundarios).

### 2.3 Interfaz de Firma de Transacciones (`aur-accesscontrol-app/components/Solicitudes.jsx`)

El componente `Solicitudes.jsx` brinda una interfaz gráfica para la gestión y aprobación de peticiones de firma.

* **Categorización de Solicitudes**: Organiza las solicitudes devueltas por la API en tres estados: Pendientes (`PENDING`), Aprobadas (`SUCCESS`) y Rechazadas (`REJECTED`).
* **Decodificación de Sobres XDR**: Emplea el utilitario `parseXdrToSolicitudes` para interpretar operaciones Stellar como adición de activos (`CHANGE_TRUST`), transferencias de fondos (`PAYMENT`) o ejecuciones de contratos inteligentes (`INVOKE_CONTRACT`).
* **Accesibilidad y Previsualización**: Incorpora `solicitudSpokenSummary` para generar resúmenes audibles de las solicitudes y un modo invitado (*guest mode*) que permite previsualizar la experiencia de firma antes del registro definitivo.

---

## 3. Integración con el Protocolo Stellar

La siguiente tabla resume los componentes del protocolo Stellar integrados en Aur Cartera y su propósito técnico dentro de la aplicación.

| Operación / Estándar Stellar | Ubicación en el Código | Propósito e Impacto Técnico |
| :--- | :--- | :--- |
| **Cuenta Patrocinada (Sponsorship)** | `wallet.js` (`createAccountTobeSigner`) | Asume la reserva inicial de XLM para nuevos usuarios sin requerir que adquieran criptomonedas previamente. |
| **Líneas de Confianza (Trustlines)** | `wallet.js` (`Add_AdmSign_Trustline`) | Habilita la recepción y custodia del activo estable USDC dentro de la billetera. |
| **Control Multifirma (Multisig)** | `wallet.js` (`ChangeSigner`, `AddSigner`) | Asigna pesos diferenciados a firmantes para exigir consenso en la administración de ahorros grupales. |
| **Construcción de Sobres XDR** | `aur-api/index.js` & `Solicitudes.jsx` | Permite construir transacciones en el servidor y transferirlas en formato XDR para su firma aislada en el cliente. |

---

## 4. Modelo de Seguridad y Privacidad

El diseño de seguridad de Aur Cartera cumple con los estándares de autocustodia y minimización de datos sensibles:

1. **Custodia Exclusiva en el Cliente**: La clave privada jamás abandona el dispositivo del usuario ni se transmite a los servidores de `aur-api`.
2. **Cifrado de Semilla en Reposo**: La semilla Ed25519 de 32 bytes se cifra localmente mediante algoritmos simétricos de clave secreta (`nacl.secretbox`) y sal/nonce aleatorio de 24 bytes.
3. **Flujo de Firma de Guardianes**: En esquemas multifirma que requieren aprobación adicional, se generan transacciones XDR separadas (`configTx` y `configTxRemove`) para ser enviadas a la aplicación del guardián.

---

## 5. Hoja de Ruta de Innovación (Sprint Instawards 30 Días)

Como parte del sprint de desarrollo respaldado por el programa Instawards de Stellar Starmaker LATAM (BAF), el equipo se encuentra implementando dos mejoras técnicas fundamentales:

* **Módulo de Recuperación Multisig con Zero-Knowledge Proofs (ZKP)**: Protocolo integrado en `self-wallet` que permite la rotación de claves y restablecimiento de guardianes en cuentas multifirma mediante pruebas de conocimiento cero, eliminando el riesgo de fondos congelados por pérdida de claves.
* **Pasarela On/Off Ramp en WhatsApp Flows (SEP-6 / SEP-24)**: Integración de los estándares de anclaje de Stellar directamente en los formularios interactivos de WhatsApp Flows en `aur-api`, facilitando depósitos y retiros en moneda local sin requerir descargas de aplicaciones externas ni copiado manual de direcciones públicas.

---

## 6. Configuración e Instalación

### Requisitos Previos
* **Node.js**: Versión 18.0 o superior.
* **Expo CLI / React Native**: Entorno de desarrollo para aplicaciones móviles.
* **PostgreSQL**: Instancia de base de datos para la persistencia de estados de `aur-api`.

### Pasos de Despliegue

1. **Obtener los Repositorios**:
   * Repositorio Backend: `aur-api`
   * Repositorio Frontend: `aur-accesscontrol-app`

2. **Ejecutar el Backend (`aur-api`)**:
   ```bash
   cd aur-api
   npm install
   npm start
   ```

3. **Ejecutar la App Móvil (`aur-accesscontrol-app`)**:
   ```bash
   cd aur-accesscontrol-app
   npm install
   npx expo start
   ```
