# WB Playwright ID-First Recorder

This repository is a modified internal distribution of Microsoft Playwright.

The purpose of this fork is to customize Playwright Codegen locator generation for internal W&B test automation workflows.

## Modification

The selector generation priority has been changed so that stable HTML `id` attributes are preferred when generating locators.

Example:

Original Playwright Codegen may generate:

```ts
page.getByRole('button', { name: 'Finalize' })
```

This fork prefers:

```ts
page.locator('#passenger-finalize')
```

when a suitable stable `id` exists.

If no suitable `id` exists, Playwright's standard locator generation logic is used.

## Locator Priority

The customized Codegen prefers locators in the following order:

1. Stable HTML `id`
2. Test ID attributes such as `data-testid`
3. Role and accessible name
4. Placeholder / label / alt text
5. Text-based locators
6. Other CSS-based fallback locators

Dynamic or GUID-like IDs continue to be ignored by Playwright's existing selector generation logic.

## Installation

Install project dependencies:

```powershell
npm ci
```

Install Chromium:

```powershell
node packages/playwright/cli.js install chromium
```

## Usage

Start Playwright Codegen:

```powershell
node packages/playwright/cli.js codegen
```

Start Codegen with a specific URL:

```powershell
node packages/playwright/cli.js codegen https://example.internal
```

Example generated output:

```ts
await page.locator('#username').fill('user');
await page.locator('#password').fill('password');
await page.locator('#btn_login').click();
```

If an element does not have a suitable stable `id`, standard Playwright locator generation is used instead.

Example:

```ts
await page.getByRole('button', { name: 'Confirm' }).click();
```

## Internal Modification

The main customization is located in:

```text
packages/injected/src/selectorGenerator.ts
```

The CSS ID selector score was changed so that stable IDs are preferred during selector generation.

The upstream implementation uses locator scoring where lower scores have higher priority.

This internal fork changes the ID selector priority while retaining the rest of Playwright's selector generation behavior.

## Upstream Project

This project is based on Microsoft Playwright:

https://github.com/microsoft/playwright

Playwright is licensed under the Apache License, Version 2.0.

This repository contains modifications to the upstream Playwright source code and is not an official Microsoft Playwright distribution.

The original copyright notices, license headers, and applicable license files from the upstream project are retained.

See the `LICENSE` file for the applicable license terms.

If a `NOTICE` file is included in the upstream distribution, it must also be retained.

## Internal Use

This repository is maintained for internal test automation purposes.

The upstream Playwright project, Microsoft name, Playwright name, logos, and related trademarks remain the property of their respective owners.

This project does not imply endorsement by or affiliation with Microsoft beyond use and modification of the Playwright open-source project under its applicable license.

## Updating From Upstream

The official Playwright repository can be maintained as an upstream Git remote.

Example:

```powershell
git remote add upstream https://github.com/microsoft/playwright.git
```

Fetch upstream changes:

```powershell
git fetch upstream
```

When updating Playwright versions, the internal ID-first selector modification should be reviewed and reapplied if necessary.

## Notes

Do not commit generated dependencies or local development artifacts such as:

```text
node_modules/
.idea/
.vscode/
test-results/
playwright-report/
blob-report/
*.log
```

Use `npm ci` to install dependencies from the committed lock file.
