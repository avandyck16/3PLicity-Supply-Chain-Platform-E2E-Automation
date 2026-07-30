# Confidentiality Notice

> This repository contains an anonymized portfolio case study based on a real-world project.
> Company names, client information, URLs, credentials, proprietary business logic, and sensitive implementation details have been removed or modified to protect confidentiality.
> Any code snippets included are examples only and do not represent any protected production codebase.

---

# 3PLicity – E2E Automation Framework

Framework de automatización E2E para plataforma logística B2B | Cypress | JavaScript

---

# Proyecto

Automatización E2E para una plataforma SaaS de gestión logística enfocada en operaciones entre compañías 3PL y merchants.

La plataforma permite gestionar procesos comerciales y operativos como registro de compañías, incorporación de merchants, creación de órdenes de compra, órdenes de venta, procesamiento logístico, actualización de inventario y seguimiento del ciclo completo de pedidos.

---

# Objetivo

Implementar una suite de automatización E2E orientada a validar flujos críticos de negocio, reduciendo esfuerzo manual de regresión y proporcionando mayor confianza durante los ciclos de entrega.

La estrategia de automatización fue diseñada para cubrir escenarios de alto valor funcional, incluyendo workflows completos entre diferentes roles del sistema.

---

# Descripción de la Solución

Diseñé e implementé un framework de automatización utilizando Cypress y JavaScript para validar procesos críticos dentro de la plataforma 3PLicity.

La solución fue estructurada mediante suites independientes por dominio de negocio:

- Company workflows
- Merchant workflows
- Purchase Order lifecycle
- Sales Order lifecycle
- Inventory validation

La arquitectura permite ejecutar validaciones sobre distintos ambientes mediante variables de entorno, reutilización de comandos personalizados y configuración dinámica de datos.

Los escenarios fueron diseñados para representar flujos reales de usuario, incluyendo interacción entre diferentes roles, validación de estados del sistema y procesos operativos completos.

---

# Stack

- Cypress
- JavaScript
- Mochawesome Reporter
- Azure DevOps
- GitHub
- Stripe Elements
- CI/CD Pipelines

---

# Mi participación

→ Diseño de estrategia de pruebas E2E para flujos críticos de negocio.

→ Desarrollo y mantenimiento de specs utilizando Cypress y JavaScript.

→ Organización de suites por módulos funcionales y ciclos operativos.

→ Implementación de comandos personalizados para acciones reutilizables.

→ Gestión de datos sensibles para la ejecución en ambientes productivos.

→ Integración de validaciones dentro del flujo de desarrollo.

→ Configuración de ejecución multi-entorno mediante variables de entorno.

→ Implementación de reportes automáticos mediante Mochawesome.

→ Investigación y resolución de problemas técnicos relacionados con iframes, integraciones externas y dispositivos de entrada.

→ Definición de alcance de automatización considerando dependencia, estabilidad y costo de mantenimiento.

---

# Logros y Resultados

➜ Reducción del tiempo de validación de ~1h15 min manuales a ~2min 42seg automatizados (~96% de reducción).

➜ Automatización de flujos críticos de operación logística.

➜ Cobertura E2E de procesos completos entre Company y Merchant.

➜ Validación automatizada del ciclo completo de Purchase Orders.

➜ Validación automatizada del ciclo completo de Sales Orders.

➜ Integración de reportes HTML para análisis de ejecuciones.

➜ Implementación de limpieza automática de datos generados por Cypress.

➜ Manejo seguro de información sensible durante ejecuciones productivas.

➜ Resolución de desafíos técnicos relacionados con:

- Stripe iframe integration.
- Datos dinámicos.
- Flujos dependientes entre roles.
- Simulación de escaneo mediante dispositivos HID.
- Estabilidad de pruebas E2E.

---

# Alcance de la Solución

| Suite                    | Pruebas Cubiertas                                        |
| ------------------------ | -------------------------------------------------------- |
| Company                  | Login validation                                         |
| Company                  | Registration flow                                        |
| Company                  | Onboarding validation                                    |
| Company                  | Merchant invitation validation                           |
| Merchant                 | Login validation                                         |
| Merchant                 | Purchase Order creation                                  |
| Merchant                 | Sales Order creation                                     |
| Purchase Order Lifecycle | Creation, approval and inventory validation              |
| Sales Order Lifecycle    | Approval, carrier assignment, fulfillment and completion |
| Inventory                | Stock update validation                                  |

---

# Automation Decisions

| Flow                 | Decision    | Coverage                           |
| -------------------- | ----------- | ---------------------------------- |
| Company Login        | ✅ Automate | Smoke + Regression                 |
| Company Registration | ✅ Automate | Full E2E flow                      |
| Company Onboarding   | ✅ Automate | Critical business flow             |
| Merchant Login       | ✅ Automate | Smoke + Regression                 |
| Merchant Invitation  | ⚠️ Deferred | Depends on external email flow     |
| Merchant Onboarding  | ❌ Deferred | Depends on invitation lifecycle    |
| Purchase Orders      | ✅ Automate | Regression coverage                |
| Sales Orders         | ✅ Automate | Core workflow                      |
| Inventory Adjustment | ✅ Automate | Functional validation              |
| Zapier Integration   | ❌ Deferred | External dependency                |
| Zapier Notifications | ❌ Deferred | Third-party integration complexity |

---

# Estructura del Proyecto

```txt
automated-tests
└── cypress
    └── e2e
        ├── company
        │   ├── login.cy.js
        │   ├── registration.cy.js
        │   ├── onboarding.cy.js
        │   └── merchant-invitation.cy.js
        │
        ├── merchant
        │   ├── login.cy.js
        │   ├── create-purchase-order.cy.js
        │   └── create-sales-order.cy.js
        │
        └── workflows
            ├── purchase-orders
                ├── purchase-order-lifecycle.cy.js
                └── inventory-adjustments.cy.js
            │
            └── sales-order
                ├── sales-order-approval.cy.js
                ├── sales-order-processing.cy.js
                └── sales-order-completion.cy.js
```

---

# Desafíos Técnicos y Soluciones Implementadas

### Problema 1:

**Interacción con Stripe Elements mediante Iframes** <br> Los campos de pago de Stripe utilizan elementos embebidos dentro de iframes cross-origin, lo que impide la interacción directa mediante Cypress sin configuración adicional.

### Solución

Se configuró Cypress para permitir la interacción con contenido embebido y se incorporó la dependencia iframe de Stripe para manejo de renderizado de iframes.

### Resultado

→ Validación automatizada de flujos de pago.

→ Integración estable con componentes externos.

→ Reducción de validaciones manuales en escenarios de checkout.

---

#### Problema 2:

**Manejo seguro de información sensible de pagos** <br> Las pruebas productivas requerían información sensible de pago que no podía permanecer almacenada dentro del repositorio ni archivos de configuración.

### Solución

Se desarrolló un script de ejecución que solicita los valores temporalmente durante el proceso y los asigna mediante variables de entorno de Cypress.

El flujo:

- Solicita información mediante consola.
- Asigna valores temporalmente.
- Ejecuta pruebas.
- Evita persistencia de información sensible.

Snippet:

```javascript
const readline = require("readline");
const { spawn } = require("child_process");

const rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout,
});

function ask(question) {
  return new Promise((resolve) => {
    rl.question(question, resolve);
  });
}

(async () => {
  const card = await ask("Card Number: ");
  const expire = await ask("Expiration (MMYY): ");
  const cvc = await ask("CVC: ");

  rl.close();

  process.env.CYPRESS_BASE_URL = "https://app.[redactedForPrivacy]";
  process.env.CYPRESS_3PL = "[redactedForPrivacy]";
  process.env.CYPRESS_MERCHANT = "[redactedForPrivacy]";
  process.env.CYPRESS_PASSWORD = "[redactedForPrivacy]";
  process.env.CYPRESS_CUPON = "[redactedForPrivacy]";

  process.env.CYPRESS_CARD = card;
  process.env.CYPRESS_EXPIRE = expire;
  process.env.CYPRESS_CVC = cvc;

  spawn("npx", ["cypress", "open"], {
    stdio: "inherit",
    shell: true,
    env: process.env,
  });
})();
```

### Resultado

→ Ejecuciones productivas sin exposición de credenciales.

→ Separación entre código de automatización y datos sensibles.

---

### Problema 3:

**Automatización de escaneo de Shipping Labels.** <br> Imitar el escaneo con un dispositivo en las pruebas sin interacción humana.

- El dispositivo operaba como un dispositivo HID (Human Interface Device), enviando la información como pulsaciones de teclado.

- El comportamiento esperado requería recibir el código completo de manera prácticamente instantánea.

### Solución

Se investigó el comportamiento real del dispositivo, descubriendo que envía datos no secuenciales o pulsaciones y se simuló mediante Cypress utilizando escritura inmediata del código de barras.

Ejemplo:

```javascript
cy.get("td[data-index='0'][aria-colindex='3']", {
  timeout: 2000,
}).dblclick();
cy.contains(
  "Scan the shipping label barcode (tracking number) to mark this order as ready to send.",
  { timeout: 1200 },
).should("be.visible");
cy.get("body").type("7501000153763{enter}", { delay: 0 });
cy.contains("Shipping label verified", { timeout: 1200 }).should("be.visible");
```

### Resultado

→ Automatización completa del proceso de escaneo.

→ Validación del flujo real utilizado por operadores.

→ Eliminación de dependencia física del dispositivo durante pruebas automatizadas.

---

# Problema 4:

**Limpieza automática de datos de prueba**. <br> Las ejecuciones repetidas generaban Purchase Orders acumuladas, provocando interferencias entre escenarios.

### Solución

Se implementó un comando personalizado para identificar órdenes generadas por Cypress mediante un identificador específico y realizar limpieza antes de ejecutar nuevos escenarios.

Snippet:

```javascript
function cancelNextCypressOrder() {
  cy.log("Checking for Cypress orders");
  cy.wait(1500);
  cy.get("body").then(($body) => {
    cy.log($body.text());
    if ($body.text().includes("CYPRESS ORDER")) {
      cy.log("Cypress order found. Cancelling...");

      cy.contains("td", "CYPRESS ORDER").first().dblclick();

      cy.contains("Order Information").should("be.visible");

      cy.get("[data-cy='cancelPurchase']").click();

      cy.get("[data-cy='confirmCancel']").click();

      cy.wait(2000);

      cy.get("body").then(($body) => {
        if (!$body.text().includes("In progress")) {
          cy.contains("Purchase Orders").click();
        }
      });

      cy.contains("In progress").should("be.visible").click();

      cancelNextCypressOrder();
    } else {
      cy.log("No Cypress orders found. Continuing...");
      cy.wait(1500);
      cy.get("[data-cy='user-icon']").should("be.visible").click();
      cy.contains("Log out").should("be.visible").click();
      cy.contains("Welcome back").should("be.visible");
      cy.clearCookies();
      cy.clearLocalStorage();
    }
  });
}
cancelNextCypressOrder();
```

### Resultado

→ Ejecuciones independientes.

→ Menor contaminación de datos.

→ Mayor estabilidad durante regresiones.

---

# Problema 5:

**Dependencia de datos existentes del ambiente QA.** <br> Algunos flujos dependían de información previamente configurada como productos, warehouses y cuentas.

### Solución

Se documentaron precondiciones necesarias y se establecieron validaciones considerando datos existentes del ambiente.

### Resultado

→ Ejecuciones más predecibles.

→ Reducción de fallos causados por configuración incompleta.

---

# Known Issues

- Algunos escenarios requieren esperas controladas debido a la ausencia de estados confiables expuestos por UI o API.
- La ejecución depende de datos base configurados previamente en el ambiente QA.
- La validación de inventario confirma cambios posteriores al flujo, pero no valida incrementos exactos esperados.
- La configuración de iframe handling debe mantenerse para continuar validando integraciones Stripe.

---

# Pipeline

La suite está diseñada para integrarse dentro de procesos CI/CD:

```text
Build → Test → Deploy
```

Ejemplos de ejecución:

```text
Build Development → Deploy Development → Execute Tests

Build Production → Deploy Production → Execute Validation Tests
```

---

# Reportes

Los resultados de ejecución son generados mediante Mochawesome Reporter.

Los reportes permiten:

- Visualización de resultados.
- Análisis de fallos.
- Seguimiento histórico de ejecuciones.
- Integración como artifacts dentro del pipeline.

---

# Conclusión

La implementación de este framework permitió automatizar procesos críticos dentro de una plataforma logística SaaS, cubriendo flujos completos entre diferentes roles y operaciones de negocio.

La solución redujo dependencia de validaciones manuales, mejoró la confiabilidad de regresiones y permitió detectar problemas tempranos dentro del ciclo de entrega.

Además, el proyecto involucró la resolución de retos técnicos relacionados con integraciones externas, manejo seguro de información sensible, automatización de dispositivos físicos y diseño de estrategias mantenibles para pruebas E2E.

La arquitectura fue diseñada para escalar conforme la plataforma incorpora nuevos flujos y funcionalidades.

---

# Project Metadata

> Author: Axel Van Dyck
> Keywords: 3PLicity, Cypress, QA Automation, E2E Testing
> License: ISC
> Module Type: CommonJS

> This project is maintained and authored by Axel Van Dyck as part of QA automation efforts for a SaaS logistics platform.
