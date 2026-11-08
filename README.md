# Aur Cartera — Presentación y Documentación del Ecosistema

**Aur Cartera** (`Aur-Cartera-Full-Project`) es una infraestructura integral de finanzas descentralizadas y ahorro colectivo ("natilleras digitales en USDC") diseñada para impulsar la inclusión financiera en América Latina. Conecta la red blockchain **Stellar** con la interfaz conversacional de **WhatsApp**, una aplicación móvil no custodia en **React Native**, y una plataforma web de aterrizaje (**Landing Page**).

---

## 🎬 Presentación y Recursos Multimedia

### 🌐 Portal Web & Landing Page
* **Landing Page Oficial**: Explora la plataforma y la propuesta de valor en el submódulo [`aur-landing`](./aur-landing).

### 📺 Demos en Video y Documental de Origen

| Recurso | Descripción | Enlace Directo |
| :--- | :--- | :--- |
| ⚡ **Video Presentación & Demo Corto** | Demostración en acción de la billetera y el flujo interactivo de natilleras por WhatsApp. | [▶️ Ver Short en YouTube](https://www.youtube.com/shorts/KYbSKk6fQJo) |
| 📖 **La Historia Detrás de AUR (Capítulo 1)** | Documental de la startup desde cero: origen del proyecto y visión fundacional. | [▶️ Ver Documental en YouTube](https://www.youtube.com/watch?v=ae50cbaWbJ8) |

---

## 💡 El Problema y Nuestra Propuesta de Valor

En América Latina, más del 30% de la población adulta no bancarizada depende de esquemas informales de ahorro como las "natilleras" o "cadenas". Estos métodos sufren de opacidad, riesgo de pérdida de fondos e incapacidad de proteger el capital contra la inflación. Por otro lado, las aplicaciones DeFi tradicionales imponen una alta barrera técnica al exigir la gestión manual de frases semilla, compra de criptomonedas en exchanges y copiado de direcciones públicas.

**Aur Cartera resuelve esta fricción mediante tres pilares:**
1. **Auto-Custodia Real sin Complicaciones:** Las claves privadas Ed25519 se generan y cifran exclusivamente en el dispositivo móvil del usuario (`self-wallet`).
2. **Interfaz Habitual por WhatsApp:** Creación de grupos, consultas de saldo en tiempo real con conversión a tasa TRM local (USD/COP) e interacciones por WhatsApp.
3. **Seguridad Multifirma Consensual:** Reglas de aprobación obligatoria entre múltiples guardianes para mover fondos grupales.

---

## 1. Arquitectura General del Ecosistema

El monorepositorio **`Aur-Cartera-Full-Project`** orquesta tres componentes principales totalmente desacoplados y conectados a la red Stellar:

```
                      +------------------------------------------+
                      |         Landing Page (aur-landing)       |
                      |        Portal Web Oficial de Aur         |
                      +--------------------+---------------------+
                                           |
                                           v
+----------------------------------+               +----------------------------------+
|   WhatsApp Graph API / Flows     |               |    App Móvil (aur-accesscontrol-app)|
|   (Interfaz Conversacional)      |               |     React Native / Self-Wallet   |
+----------------+-----------------+               +----------------+-----------------+
                 |                                                  |
                 v                                                  v
+-------------------------------------------------------------------------------------+
|                              Backend API (aur-api)                                  |
|         Node.js + Express + Google Gemini AI + PostgreSQL + Stellar SDK              |
+----------------------------------------+--------------------------------------------+
                                         |
                                         v
+-------------------------------------------------------------------------------------+
|                                Red Blockchain Stellar                               |
|            Horizon Server (Sponsorship, USDC Trustlines, Multisig, XDRs)            |
+-------------------------------------------------------------------------------------+
```

### Componentes del Repositorio

* **`aur-landing` (Portal Web Landing Page)**: Interfaz web pública orientada a la educación del usuario, presentación de la solución y redirección hacia WhatsApp.
* **`aur-api` (Orquestador Backend y Bot de WhatsApp)**: Servidor Node.js/Express que gestiona la API Graph de WhatsApp (v18.0), procesa intenciones conversacionales con inteligencia artificial (**Google Gemini**), mantiene la persistencia de grupos en **PostgreSQL** y construye sobres de transacción XDR.
* **`aur-app` / `aur-accesscontrol-app` (Cliente Móvil y Self-Wallet)**: Aplicación React Native que implementa el módulo no custodio `self-wallet/wallet.js` para la generación criptográfica de claves Ed25519, cifrado simétrico local de semillas y aprobación visual de solicitudes XDR (`Solicitudes.jsx`).
* **Red Blockchain Stellar**: Infraestructura descentralizada que procesa y liquida operaciones en USDC y XLM, aplica patrocinio de reservas (*sponsorship*) y ejecuta reglas multifirma.

---

## 2. Desglose Tecnológico por Submódulo

### 2.1 Portal Web Landing Page (`aur-landing`)

Alojado en el submódulo `aur-landing`, constituye el punto de entrada principal para el público general y aliados estratégicos:

* **Diagnóstico de Inclusión Financiera**: Documenta las barreras económicas de la región (comisiones elevadas en remesas, falta de transparencia en ahorros comunitarios e inflación).
* **Demostración Interactiva**: Enlaces directos a los videos demostrativos ([Demo Short](https://www.youtube.com/shorts/KYbSKk6fQJo) y [Documental](https://www.youtube.com/watch?v=ae50cbaWbJ8)).
* **Onboarding en 3 Pasos**: Botón de llamada a la acción (CTA) que redirige al usuario directamente a WhatsApp para iniciar su cuenta o natillera digital en segundos.

### 2.2 Orquestador Conversacional y Backend API (`aur-api`)

El backend Node.js centraliza la lógica de negocio y la integración con WhatsApp:

* **Flujos Conversacionales Interactivos**: Menús dinámicos (`sendMenu`) como `ONBOARDING`, `GROUPS_HOME`, `MY_MONEY`, `CREATE_GROUP` y `SOLICITUD`, permitiendo consultar saldos, crear natilleras e invitar participantes mediante enlaces directos.
* **Conversión de Divisas e Indicadores TRM**: Integración con APIs de tasa de cambio en tiempo real (USD/COP) a través de funciones como `getUsdToCop`, `getTrm` y `formatWithCop` para calcular el valor equivalente de USDC en pesos colombianos.
* **Procesamiento de IA (Google Gemini)**: Servicio de IA conversacional (`@google/generative-ai`) y tratamiento de archivos multimedia (`fluent-ffmpeg` / `ffmpeg-static`).
* **Firma de Administración y XDR**: Endpoint `/api/sign-transaction` para firmar operaciones administrativas de patrocinio de cuentas y coordinación de transacciones a través de `/transactions/:pubkey`.

### 2.3 Módulo Criptográfico de Autocustodia (`aur-accesscontrol-app/self-wallet/wallet.js`)

Gestión no custodia del ciclo de vida de claves criptográficas en el dispositivo móvil:

* **Generación y Derivación Bip39**: Generación de mnemónicos de 12 palabras (`@scure/bip39`) y derivación de semillas Ed25519 de 32 bytes para claves Stellar con `@stellar/stellar-sdk`.
* **Cifrado Simétrico Local**: Cifrado simétrico `nacl.secretbox` (`tweetnacl`) mediante hashes SHA-512 derivados de la contraseña del usuario antes de la persistencia local.
* **Patrocinio de Cuentas (Sponsorship)**: Funciones `beginSponsoringFutureReserves` y `endSponsoringFutureReserves` para asumir las reservas mínimas de XLM requeridas por nuevos usuarios.
* **Líneas de Confianza y Multifirma**: Ejecución de `Add_AdmSign_Trustline`, `ChangeSigner` y `AddSigner` para habilitar el activo USDC y ajustar ponderaciones de firma en natilleras grupales.

### 2.4 Interfaz de Aprobación de Transacciones (`aur-accesscontrol-app/components/Solicitudes.jsx`)

Componente React Native para visualizar y autorizar solicitudes XDR en el móvil:

* **Gestión por Pestañas**: Categorización de solicitudes en Pendientes (`PENDING`), Aprobadas (`SUCCESS`) y Rechazadas (`REJECTED`).
* **Decodificación XDR**: Interpretación de operaciones Stellar mediante `parseXdrToSolicitudes` (`CHANGE_TRUST`, `PAYMENT`, `INVOKE_CONTRACT`).
* **Accesibilidad y Modo Invitado**: Función `solicitudSpokenSummary` para resúmenes hablados y modo de previsualización para usuarios sin registro completo.

---

## 3. Matriz de Integración con el Protocolo Stellar

| Operación / Estándar Stellar | Ubicación en el Código | Propósito e Impacto Técnico |
| :--- | :--- | :--- |
| **Cuenta Patrocinada (Sponsorship)** | `wallet.js` (`createAccountTobeSigner`) | Asume la reserva inicial de XLM para nuevos usuarios sin requerir compra previa de criptomonedas. |
| **Líneas de Confianza (Trustlines)** | `wallet.js` (`Add_AdmSign_Trustline`) | Habilita la recepción y custodia del activo estable USDC. |
| **Control Multifirma (Multisig)** | `wallet.js` (`ChangeSigner`, `AddSigner`) | Asigna pesos diferenciados a firmantes para exigir consenso obligatorio en natilleras grupales. |
| **Construcción de Sobres XDR** | `aur-api/index.js` & `Solicitudes.jsx` | Permite construir transacciones en el backend y enviarlas en formato XDR para su firma aislada en la app. |
| **Pasarela On/Off Ramp (SEP-6 / SEP-24)** | `aur-api` & WhatsApp Flows | Permite depósitos y retiros de moneda local directamente en WhatsApp Flows en el sprint Instawards. |

---

## 4. Modelo de Seguridad y Privacidad

1. **Custodia Exclusiva en el Cliente**: La clave privada jamás abandona el dispositivo del usuario ni se transmite a los servidores de `aur-api`.
2. **Cifrado de Semilla en Reposo**: La semilla Ed25519 de 32 bytes se cifra localmente con `nacl.secretbox` y un nonce aleatorio de 24 bytes.
3. **Firmado Aislado de Guardianes**: En esquemas multifirma, se generan transacciones XDR independientes (`configTx` y `configTxRemove`) para ser enviadas y firmadas por la aplicación del guardián.

---

## 5. Hoja de Ruta de Innovación (Sprint Instawards 30 Días)

En el marco del sprint respaldado por el programa **Instawards** de Stellar Starmaker LATAM (BAF), se implementan dos innovaciones principales:

1. **Módulo de Recuperación Multisig con Zero-Knowledge Proofs (ZKP)**: Protocolo no custodio en `self-wallet` que permite la rotación de claves y restablecimiento de guardianes mediante pruebas de conocimiento cero, eliminando el riesgo de congelamiento por pérdida de claves.
2. **Pasarela On/Off Ramp en WhatsApp Flows (SEP-6 / SEP-24)**: Integración de los estándares de anclaje de Stellar dentro de los formularios interactivos de WhatsApp Flows en `aur-api`, facilitando depósitos y retiros en moneda local sin salir de la conversación ni copiar direcciones públicas.

---

## 6. Configuración y Despliegue del Proyecto Completo

### Requisitos Previos
* **Node.js**: Versión 18.0 o superior.
* **Expo CLI / React Native**: Entorno para desarrollo móvil.
* **PostgreSQL**: Base de datos relacional para el estado de natilleras y sesiones en `aur-api`.
* **Git**: Con soporte para submódulos.

### Instrucciones de Despliegue

1. **Clonar el Repositorio Completo con Submódulos**:
   ```bash
   git clone --recursive https://github.com/batublockdev/Aur-Cartera-Full-Project
   cd Aur-Cartera-Full-Project
   ```

2. **Ejecutar el Backend API (`aur-api`)**:
   ```bash
   cd aur-api
   npm install
   npm start
   ```

3. **Ejecutar la App Móvil (`aur-app` / `aur-accesscontrol-app`)**:
   ```bash
   cd ../aur-app
   npm install
   npx expo start
   ```

4. **Desplegar / Ejecutar la Landing Page (`aur-landing`)**:
   ```bash
   cd ../aur-landing
   npm install
   npm run dev
   ```
