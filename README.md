# Confidentiality Notice

> This repository contains an anonymized portfolio case study based on a real-world project.
> Company names, client information, URLs, credentials, proprietary business logic, and sensitive implementation details have been removed or modified to protect confidentiality.
> Any code snippets included are examples only and do not represent any protected production codebase.

---

# 3PLICITY — E2E Test Automation Framework

**B2B Logistics SaaS | Cypress | JavaScript | Azure DevOps | CI/CD | Mochawesome | JUnit XML**

## Project Overview

3PLICITY is a B2B logistics platform that connects third-party logistics (3PL) companies with merchants, supporting workflows such as purchase orders, sales orders, inventory management, shipping, and payments.

The goal of this project was to automate critical end-to-end workflows with Cypress and integrate regression testing into Azure DevOps CI/CD to improve execution speed, release confidence, and test-result visibility.

## QA Strategy

The automation focuses on validating complete business workflows across Company and Merchant roles, including their interactions with inventory, shipping, and payment functionality.

- **End-to-end validation:** Covers key business flows rather than isolated UI interactions.
- **Business logic:** Validates order lifecycles, inventory changes, role-based workflows, and relevant negative scenarios.
- **Regression efficiency:** Replaces a lengthy manual regression process with repeatable automated execution.
- **Continuous validation:** Runs Cypress against the application build before deployment in the delivery pipeline.

## Automation Scope

| Functional area | Coverage |
|---|---|
| Company authentication and onboarding | Automated |
| Merchant authentication | Automated |
| Merchant invitation validation | Automated validation, subject to workflow and lifecycle constraints |
| Purchase Orders | Automated |
| Sales Orders | Automated |
| Inventory adjustments | Automated |
| Shipping label scanning | Automated |
| Payment workflow with Stripe Elements | Automated |

The scope prioritizes critical user journeys and business workflows. Integrations and lifecycle scenarios that require additional setup or reliable external-state control remain outside the current automated regression scope.

## Technical Implementation & Helpers

### Cypress E2E Architecture

The framework uses Cypress with JavaScript and organizes tests by functional area and business workflow, separating Company and Merchant functionality from broader end-to-end scenarios.

Key implementation details include:

- Reusable Cypress commands and helpers for common actions and API availability checks.
- Environment-based configuration for application URLs, API endpoints, test accounts, and payment data.
- Automated cleanup of test-generated Purchase Orders to reduce interference between executions.
- A reusable onboarding helper that completes the six-step Company setup workflow.
- Cypress keyboard input to simulate a HID shipping label scanner.
- A command-line helper for entering sensitive payment data during production-environment debugging.
- Exclusion of draft test specifications from normal execution.

### Reusable Login Helpers and Environment Variables

Reusable `companyLogin()` and `merchantLogin()` commands centralize authentication for the two application roles. They retrieve credentials through Cypress's `cy.env()` API and use a shared sign-in helper to complete login and validate navigation.

```javascript
Cypress.Commands.add("companyLogin", () => {
  cy.env(["3PL"]).then((env) => {
    const threePL = env["3PL"];
    cy.get("#email-3pl").type(threePL);
  });

  cy.env(["PASSWORD"]).then(({ PASSWORD }) => {
    cy.get("#password-3pl").type(PASSWORD);
  });

  signInAndLandOnDashboard(
    "[data-cy='3pl-sign-in']",
    "/dashboard",
    30000,
  );
});
```

The framework uses the newer `cy.env(["KEY"])` syntax to retrieve environment values asynchronously, replacing direct access patterns used in older Cypress implementations. This approach is also used for merchant credentials, payment details, and environment-specific configuration.

### Purchase Order Cleanup Helper

The `purchaseOrderCleanUp()` custom command removes accumulated test data by repeatedly locating Purchase Orders identified by the `CYPRESS ORDER` marker and cancelling them through the application UI.

The helper logs in as a Merchant, opens the in-progress Purchase Orders list, checks whether a matching order exists, cancels the first match, and repeats until no matching orders remain. It then logs out and clears cookies and local storage.

```javascript
function cancelNextCypressOrder() {
  cy.get("body").then(($body) => {
    if ($body.text().includes("CYPRESS ORDER")) {
      cy.contains("td", "CYPRESS ORDER").first().dblclick();

      cy.contains("Order Information").should("be.visible");
      cy.get("[data-cy='cancelPurchase']").click();
      cy.get("[data-cy='confirmCancel']").click();

      cy.wait(2000);
      cy.contains("In progress").should("be.visible").click();

      cancelNextCypressOrder();
    } else {
      cy.log("No Cypress orders found. Continuing...");
      cy.get("[data-cy='user-icon']").click();
      cy.contains("Log out").click();
      cy.clearCookies();
      cy.clearLocalStorage();
    }
  });
}
```

This recursive cleanup strategy helps prevent test-generated orders from accumulating and interfering with subsequent regression runs. The helper depends on the application UI, the order naming convention, and the availability of the expected navigation elements.

### Company Onboarding Helper

The `completeOnboarding(registermail)` custom command automates the six-step Company onboarding process, including profile configuration, shipping setup, billing, merchant invitation, integrations, and the final summary.

The helper covers multiple setup dependencies in one reusable workflow:

- Enters company contact and address information and uploads the company logo.
- Configures and verifies ShipEngine shipping settings.
- Configures billing and payment details through Stripe Elements.
- Creates a merchant invitation using a generated merchant name and email.
- Skips the deferred integration setup when appropriate.
- Validates the onboarding summary and completes registration.
- Logs in with the new Company credentials and verifies that the dashboard loads.

For example, the helper validates the shipping configuration before proceeding:

```javascript
cy.get("#shipEngineApiKey").type(shipEngineKey);

cy.get("[data-cy='verifySEbtn']")
  .should("be.enabled")
  .click();

cy.contains("Verified").should("be.visible");
cy.get("#se-1037799").click();
cy.contains("Next Step").click();
```

Payment fields are populated using values retrieved through `cy.env()`, keeping the onboarding workflow reusable across test configurations.

### Shipping Label Scanner Simulation

The shipping workflow requires a shipping label to be associated with a Sales Order and then scanned to mark the order as ready to send.

Instead of relying on a physical HID scanner, Cypress simulates scanner input by typing the tracking number directly into the page body and appending the Enter key.

```javascript
cy.get("#merchant-tracking").type("7501000153763");
cy.get("#labelUrl").type(
  "https://jsonplaceholder.typicode.com/",
);

cy.get("[data-cy='saveShippingLabel']")
  .should("be.visible")
  .click();

cy.contains(
  "Scan the shipping label barcode (tracking number) to mark this order as ready to send.",
).should("be.visible");

cy.get("body").type("7501000153763{enter}", {
  delay: 0,
});

cy.contains("Shipping label verified", {
  timeout: 8000,
}).should("be.visible");
```

This validates the application's barcode-input handling and shipping-label verification response without requiring physical scanner hardware during automated execution.

### Production Debugging and Sensitive Payment Data

I created a standalone script to provide sensitive payment details when running the suite against the production environment for controlled debugging sessions.

This separate command-line script supports debugging payment-related flows in the production environment. It prompts the tester to enter the required real payment details through the command-line interface and passes those values to the test execution through environment variables.

This avoids hardcoding the entered card details in the Cypress test source and allows the same payment workflow to consume runtime-provided values.

The script is intended for controlled debugging rather than as a substitute for a dedicated test-payment environment. Sensitive values must not be committed to the repository or printed in logs.

After collecting the card number, expiration date, and CVC, it assigns them to environment variables, closes the shell with its provided information, and launches Cypress using `child_process.spawn()`.


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

  process.env.CYPRESS_BASE_URL = "https://app.3plicity.com/";
  process.env.CYPRESS_CARD = card;
  process.env.CYPRESS_EXPIRE = expire;
  process.env.CYPRESS_CVC = cvc;

  spawn("npx", ["cypress", "run", "--browser", "chrome"], {
    stdio: "inherit",
    shell: true,
    env: process.env,
  });
})();
```

**Execution flow:**

1. Prompts the QA engineer for payment details through the command line.
2. Sets the production application URL and payment data as environment variables.
3. Starts Cypress in Chrome and passes the environment to the child process.
4. Runs the existing E2E suite using the production configuration.

The script runs independently of the delivery pipeline because the current pipeline design does not require interactive payment-data entry.

---


### Key Technical Challenges

**1. Stripe Elements and cross-origin interactions**

The payment workflow interacts with Stripe Elements embedded in an iframe. The Cypress configuration uses `chromeWebSecurity: false` to support the required cross-origin test interactions.

**2. Sensitive payment test data**

Payment data is supplied through environment variables rather than embedded in individual test cases. The command-line debugging helper supports runtime entry of payment details for production debugging. Sensitive values should be managed carefully and excluded from public documentation.

**3. HID shipping label scanner**

The shipping workflow requires scanner-like input. Cypress non-delayed keyboard interactions simulate the input needed to validate the scanning flow without depending on a physical scanner during automated execution.

**4. Test-data cleanup**

A recursive cleanup helper cancels test-generated Purchase Orders identified by the `CYPRESS ORDER` naming convention. This helps control accumulated test data and reduces interference from previous runs.

## CI/CD Integration | Azure DevOps

Cypress E2E regression testing is integrated into Azure DevOps using YAML pipelines.

The primary delivery workflow follows this sequence:

`Build → Cypress E2E Tests → Deploy`

- **Build:** Installs dependencies and generates the Vite application build for the target environment.
- **Artifact publication:** Publishes the built application as a pipeline artifact.
- **E2E validation:** Downloads and serves the built application locally on the pipeline agent, then runs Cypress against it.
- **Deployment gating:** Deployment stages depend on successful completion of the test stage.
- **Artifact consistency:** The deployment stages consume the same application build artifact that was tested, rather than rebuilding the application after validation.

The workflow selects the DEV or PROD build and test configuration based on the source branch (`dev` or `main`).

The project also includes a second, separate pipeline for manually triggered Cypress regression execution. The separation of test execution from the delivery pipeline is documented as a distinct CI/CD architecture project.

## Automated Reporting | Cypress Multi Reporters | JUnit XML

The reporting configuration was extended from a single-reporter approach to **`cypress-multi-reporters`**, enabling Cypress to generate both human-readable reports and structured test results from the same execution.

### Multiple report formats

The configuration is maintained in a dedicated `reporter-config.json` file.

```javascript
// cypress.config.js
reporter: "cypress-multi-reporters",
reporterOptions: {
  configFile: "reporter-config.json",
},
```

The reporter configuration enables Mochawesome and JUnit XML:

```json
{
  "reporterEnabled": "mochawesome, mocha-junit-reporter",
  "mochawesomeReporterOptions": {
    "reportDir": "cypress/reports",
    "reportFilename": "Report",
    "timestamp": "mmddyyyy_HHMMss",
    "overwrite": true,
    "html": true,
    "json": true
  },
  "mochaJunitReporterReporterOptions": {
    "mochaFile": "results/junit-[hash].xml",
    "toConsole": false
  }
}
```

This provides two complementary reporting outputs:

- **Mochawesome HTML/JSON:** Detailed reports for reviewing execution results and investigating failures.
- **JUnit XML:** Structured test results that Azure DevOps can ingest and display in the pipeline run.

### Azure DevOps Tests tab

The YAML template uses `PublishTestResults@2` with the JUnit format to publish Cypress results directly to the pipeline run.

```yaml
- task: PublishTestResults@2
  displayName: "Publish Cypress results (Tests tab) - Dev"
  condition: and(always(), eq('${{ parameters.environment }}', 'dev'))
  inputs:
    testResultsFormat: "JUnit"
    testResultsFiles: "automated-tests/results/junit-*.xml"
    searchFolder: "$(System.DefaultWorkingDirectory)"
    mergeTestResults: true
    testRunTitle: "Cypress E2E (dev)"
    failTaskOnFailedTests: true
```

The PROD configuration uses the same publishing mechanism with its own environment condition and test-run title.

Publishing JUnit results makes individual test outcomes available in Azure DevOps' native **Tests** tab, while the Mochawesome reports provide a separate, detailed view of the execution.

### Report and screenshot artifacts

The pipeline publishes the Cypress report directory and screenshots as separate artifacts. These publishing steps use `succeededOrFailed()` so the artifacts can still be collected when test execution fails, supporting post-failure investigation.

## Results

- **Manual regression:** Approximately 1 hour and 15 minutes.
- **Automated regression:** Approximately 2 minutes and 42 seconds.
- **Execution-time reduction:** Approximately 96%.

The automated workflow reduces repetitive regression effort and provides repeatable validation of critical business processes. CI/CD integration adds pre-deployment checks, while multi-format reporting improves visibility into individual test outcomes and failure investigation.

## Limitations & Known Issues

- Some scenarios use controlled waits where the application does not expose sufficiently reliable state indicators.
- Inventory validation checks the resulting state but does not always assert the exact expected inventory increment.
- Stripe payment tests depend on the current iframe configuration and browser-security settings.


## Technical Takeaway

This project combines business-focused E2E automation with CI/CD integration and multi-format reporting. Beyond automating critical logistics workflows, it demonstrates how Cypress results can be connected to Azure DevOps so that teams can validate releases, inspect individual test outcomes, and investigate failures using both native test results and detailed HTML reports.

---

<div style="text-align:center; color:#888; font-size:10px;">
    QA Documentation | Axel Van Dyck | QA Engineer 
</div>
