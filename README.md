# Confidentiality Notice

> This repository contains an anonymized portfolio case study based on a real-world project.
> Company names, client information, URLs, credentials, proprietary business logic, and sensitive implementation details have been removed or modified to protect confidentiality.
> Any code snippets included are examples only and do not represent any protected production codebase.

---

# 3PLicity — E2E Automation Framework

**B2B Logistics SaaS | Cypress | JavaScript | Azure DevOps | CI/CD**

E2E automation framework designed to validate critical business workflows across a B2B logistics platform connecting 3PL companies and merchants.

---

## Quick Scope

| Area             | Details                       |
| ---------------- | ----------------------------- |
| **Role**         | QA Engineer                   |
| **Testing**      | E2E · Regression · Functional |
| **Automation**   | Cypress · JavaScript          |
| **Reporting**    | Mochawesome                   |
| **CI/CD**        | Azure DevOps Pipelines        |
| **Integrations** | Stripe Elements · HID Scanner |
| **Environments** | Multi-environment execution   |
| **Focus**        | Critical business workflows   |

### Key Results

* **~96% reduction** in manual regression execution time: ~1h 15m → ~2m 42s.
* Automated critical workflows across Company and Merchant roles.
* Automated complete Purchase Order and Sales Order lifecycles.
* Integrated E2E validation into the CI/CD delivery process.
* Implemented automated reporting and test-data cleanup.
* Addressed technical challenges involving external integrations, sensitive data, dynamic data, and physical input devices.

---

## Case Study Index

* [Project Overview](#project-overview)
* [QA Strategy](#qa-strategy)
* [Automation Scope & Decisions](#automation-scope--decisions)
* [Technical Implementation](#technical-implementation)
* [Test Architecture](#test-architecture)
* [CI/CD & Reporting](#cicd--reporting)
* [Results](#results)
* [Limitations & Known Issues](#limitations--known-issues)

---

# Project Overview

## Context

3PLicity is a SaaS logistics management platform focused on operations between 3PL companies and merchants.

The platform supports business and operational processes including company registration, merchant onboarding, Purchase Orders, Sales Orders, logistics processing, inventory updates, and order lifecycle management.

## Objective

Design and implement an E2E automation suite capable of validating high-value business workflows, reducing manual regression effort, and increasing confidence during software delivery cycles.

The automation strategy prioritized workflows with high functional impact that could be executed reliably and maintained within the test framework.

## My Role

* Designed the E2E testing strategy for critical business workflows.
* Developed and maintained Cypress/JavaScript test specifications.
* Organized test suites by functional domain and operational lifecycle.
* Implemented reusable custom commands.
* Managed sensitive test data for productive environment execution.
* Configured multi-environment execution through environment variables.
* Integrated automated reporting using Mochawesome.
* Investigated and resolved technical challenges involving iframes, external integrations, dynamic data, and HID devices.
* Defined automation scope based on dependency, stability, business value, and maintenance cost.
* Integrated automated validation into the software delivery workflow.

## Technology Stack

* Cypress
* JavaScript
* Azure DevOps
* YAML Pipelines
* GitHub
* Mochawesome Reporter
* Stripe Elements
* CI/CD

---

# QA Strategy

The automation strategy focused on validating complete business workflows rather than isolated UI interactions.

The suite was organized around the main operational domains of the platform:

* Company workflows
* Merchant workflows
* Purchase Order lifecycle
* Sales Order lifecycle
* Inventory validation

Scenarios were designed to represent real user workflows across different system roles, including state transitions, dependent processes, and operational validations.

Automation scope was determined by:

* Business criticality
* Workflow stability
* Dependency complexity
* Test repeatability
* Maintenance cost
* Regression value

---

# Automation Scope & Decisions

| Flow                 | Decision    | Coverage / Reason                  |
| -------------------- | ----------- | ---------------------------------- |
| Company Login        | ✅ Automate  | Smoke + Regression                 |
| Company Registration | ✅ Automate  | Full E2E flow                      |
| Company Onboarding   | ✅ Automate  | Critical business flow             |
| Merchant Login       | ✅ Automate  | Smoke + Regression                 |
| Merchant Invitation  | ⚠️ Deferred | Depends on external email flow     |
| Merchant Onboarding  | ❌ Deferred  | Depends on invitation lifecycle    |
| Purchase Orders      | ✅ Automate  | Regression coverage                |
| Sales Orders         | ✅ Automate  | Core workflow                      |
| Inventory Adjustment | ✅ Automate  | Functional validation              |
| Zapier Integration   | ❌ Deferred  | External dependency                |
| Zapier Notifications | ❌ Deferred  | Third-party integration complexity |

### Automation Principle

Not every functional flow was considered a candidate for immediate automation.

Flows with strong external dependencies or unstable prerequisites were deferred when their maintenance cost or execution reliability outweighed the immediate value of automation.

---

# Technical Implementation

## Framework Architecture

The framework uses Cypress and JavaScript with independent suites organized around business domains and operational workflows.

The architecture supports:

* Reusable custom commands.
* Environment-specific configuration.
* Dynamic test data.
* Multi-role workflows.
* Automated cleanup.
* External integration handling.
* Automated reporting.

## Key Technical Challenges

### Stripe Elements — Cross-Origin Iframes

Stripe payment fields use cross-origin iframes, requiring additional Cypress configuration.

**Implementation:** Integrated `cypress-iframe` and configured `chromeWebSecurity: false`.

**Result:** Stable automated validation of Stripe payment workflows.

---

### Secure Handling of Payment Data

Productive test executions required sensitive payment information that could not be stored in the repository or configuration files.

**Implementation:** Created a Node.js execution script that requests payment values temporarily and passes them to Cypress through environment variables for the duration of the test run.

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

**Result:** Sensitive payment data remained outside the repository while still allowing productive test execution.

---

### Shipping Label Scanner Automation

Shipping label validation required simulating the behavior of a physical scanner operating as a **HID (Human Interface Device)**.

**Implementation:** Investigated the device behavior and reproduced its keyboard-input pattern through Cypress using immediate barcode entry.

```javascript
cy.get("td[data-index='0'][aria-colindex='3']", {
  timeout: 2000,
}).dblclick();

cy.contains(
  "Scan the shipping label barcode (tracking number) to mark this order as ready to send.",
  { timeout: 1200 },
).should("be.visible");

cy.get("body").type("7501000153763{enter}", { delay: 0 });

cy.contains("Shipping label verified", {
  timeout: 1200,
}).should("be.visible");
```

**Result:** Automated the complete scanning workflow without requiring the physical device during test execution.

---

### Automated Test Data Cleanup

Repeated executions generated Purchase Orders that could interfere with subsequent scenarios.

**Implementation:** Created a reusable Cypress command that identifies Cypress-generated orders and removes them before continuing with new scenarios.

```javascript
function cancelNextCypressOrder() {
  cy.log("Checking for Cypress orders");

  cy.get("body").then(($body) => {
    if ($body.text().includes("CYPRESS ORDER")) {
      cy.log("Cypress order found. Cancelling...");

      cy.contains("td", "CYPRESS ORDER").first().dblclick();

      cy.contains("Order Information").should("be.visible");

      cy.get("[data-cy='cancelPurchase']").click();
      cy.get("[data-cy='confirmCancel']").click();

      cancelNextCypressOrder();
    } else {
      cy.log("No Cypress orders found. Continuing...");
    }
  });
}

cancelNextCypressOrder();
```

**Result:** Reduced test-data contamination and improved stability across repeated regression executions.

---

### Existing Environment Data Dependencies

Some workflows depended on preconfigured products, warehouses, accounts, and other baseline data.

**Implementation:** Documented required preconditions and incorporated environment data requirements into the test execution strategy.

**Result:** More predictable executions and fewer failures caused by incomplete environment configuration.

---

# Test Architecture

The framework is organized by functional domain and operational workflow.

```text
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
            │   ├── purchase-order-lifecycle.cy.js
            │   └── inventory-adjustments.cy.js
            │
            └── sales-order
                ├── sales-order-approval.cy.js
                ├── sales-order-processing.cy.js
                └── sales-order-completion.cy.js
```

This separation allows suites to be executed independently according to the required validation scope.

---

# CI/CD & Reporting

## Pipeline Strategy

The E2E suite is integrated into the Azure DevOps CI/CD workflow as a validation gate before deployment.

The pipeline builds the application according to the **target branch**, deploys the resulting build to the corresponding test environment, executes Cypress against that deployed build, and proceeds with the final deployment only when the tests pass.

### Development Flow

```text
Merge → dev
   ↓
Build DEV
   ↓
Deploy to Test Environment
   ↓
Execute Cypress Tests
   ↓
Tests Passed?
   ├── No  → Pipeline Stops
   └── Yes → Deploy to DEV
```

### Production Flow

```text
Merge → main
   ↓
Build PROD
   ↓
Deploy to Test Environment
   ↓
Execute Cypress Tests
   ↓
Tests Passed?
   ├── No  → Pipeline Stops
   └── Yes → Deploy to PROD
```

### Pipeline Principle

The target branch determines which application build is generated and validated.

This ensures that the **same build configuration intended for the target environment is tested before the final deployment**.

A failed Cypress execution prevents the corresponding deployment stage from continuing.

---

## Automated Reporting

Mochawesome Reporter generates execution reports containing:

* Test results.
* Failure details.
* Execution evidence.
* Historical execution information.
* Pipeline artifacts for post-run analysis.

---

# Results

## Automation Impact

| Metric                      |   Result |
| --------------------------- | -------: |
| Manual regression execution |  ~1h 15m |
| Automated execution         |  ~2m 42s |
| Time reduction              | **~96%** |

## Functional Coverage

* Company authentication and registration workflows.
* Company onboarding.
* Merchant authentication.
* Merchant invitation validation.
* Purchase Order lifecycle.
* Sales Order lifecycle.
* Inventory adjustment validation.
* Shipping-label scanning.
* Payment workflow validation.

## Additional Improvements

* Automated regression execution.
* Automated HTML reporting.
* Automated test-data cleanup.
* Multi-environment execution.
* Secure handling of sensitive execution data.
* CI/CD integration with deployment gating.
* Coverage of complete multi-role business workflows.

---

# Limitations & Known Issues

* Some scenarios require controlled waits because reliable state indicators are not consistently exposed through the UI or API.
* Execution depends on baseline data configured in the target environment.
* Inventory validation confirms the resulting state but does not validate the exact expected quantity increment.
* Stripe iframe handling must remain configured for payment workflow validation.
* Some workflows remain deferred because of external dependencies or lifecycle constraints.

These limitations are documented as part of the current automation scope rather than treated as uncovered defects in the framework.

---

<div style="text-align:center; color:#888; font-size:10px;">
    QA Documentation | Axel Van Dyck | QA Engineer 
</div>
