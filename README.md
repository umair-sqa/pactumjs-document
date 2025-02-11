# PactumJS Setup Guide

## Project Structure

Organize your project as follows:

```bash
  project-root/
  ├── e2e/
  │   ├── Specs/
  │   │   ├── health.spec.ts
  │   ├── Suites/
  │   │   ├── health.test.ts
  │   ├── Reporting/
  │   │   ├── pactum-html-appender.ts
  │   ├── pactum-setup.ts
  │   ├── jest.config.ts
  │   ├── package.json
  │   ├── tsconfig.json
```

## Required Packages

To set up PactumJS, install the following packages:

```json
  "pactum": "^3.6.9",
  "pactum-json-reporter": "^1.0.2",
  "jest": "^29.7.0",
  "jest-html-reporters": "^3.1.4"
```

You can install them using:

```sh
  npm install pactum pactum-json-reporter jest jest-html-reporters --save-dev
```

## Jest Configuration

Create a Jest configuration file (e.g., `jest.config.ts`) and specify it using the Jest command when running tests.
By default, Jest looks for `jest.config.ts`.

### Jest Configuration File (`jest.config.ts`)

```ts
  module.exports = {
    preset: 'ts-jest',
    setupFiles: ['<path-to-global-config-file>'],
    setupFilesAfterEnv: ['<path-to-post-setup-file>'],
    reporters: [
      "default",
      ["jest-html-reporters", {
        "publicPath": "./html-report",
        "filename": "report.html",
        "expand": true
      }]
    ]
  };
```

- `preset: 'ts-jest'` – Enables Jest to work with TypeScript.
- `setupFiles` – Define the file for the global config (default URLs, setup configurations).
- `setupFilesAfterEnv` – Define the file executed after the global setup.
- `reporters` – Configure reporters, including `jest-html-reporters`.

## PactumJS Setup

Create a Pactum setup file to configure global settings and integrate the reporter.

### Pactum Setup File (`pactum-setup.ts`)

```ts
  import { reporter, request, settings } from 'pactum';
  import { PactumJestHtmlReporterAppender } from './reporting/pactum-html-appender';

  global.beforeAll(() => {
    reporter.add(PactumJestHtmlReporterAppender);
    settings.setLogLevel('SILENT');
    request.setDefaultTimeout(15000);
    jest.retryTimes(1);
  });
```

## Specs & API Objects

Use **Spec Handlers** to manage API specs efficiently. Utilize a single object to control API endpoints and prevent magic string issues.

### Example Spec Controller (`health.spec.ts`)

```ts
  import { handler } from 'pactum';
  const { addSpecHandler } = handler;

  const getHealthStatus = 'getHealthStatus';

  export const specController = {
    getHealthStatus
  };

  // Handler for GET /health
  addSpecHandler(getHealthStatus, ({ spec, data }) => {
    const { status } = data;
    spec.get('/health');
    spec.expectStatus(status);
  });
```

## Suites & Tests

Import the spec controller and use it in your tests.

### Example Test File (`health.test.ts`)

```ts
  import { spec } from 'pactum';
  import { specController } from '../Specs/health.spec';

  describe('Health check', () => {
    it('should check app health', async () => {
        await spec(specController.getHealthStatus, { status: 200 });
      });
  });
```

## Custom Reporter

Create a custom reporter folder inside `e2e/` to enhance test reports.

### Pactum HTML Appender (`pactum-html-appender.ts`)

```ts
  import { addMsg } from 'jest-html-reporters/helper';

  export const PactumJestHtmlReporterAppender = {
    name: 'PactumJestHtmlReporterAppender',

    afterSpec(spec) {
      // Append each request made by Pactum to the test that is running
      addMsg({ message: JSON.stringify(spec, null, 2) });
    },

    end() {},

    reset() {}
  };
```
## Running the Tests

To execute your tests, run the following command:

```sh
  jest --config=e2e/jest.config.ts
```

This will execute all tests in the `e2e/Suites` directory and generate an HTML report in `./html-report/report.html`.

