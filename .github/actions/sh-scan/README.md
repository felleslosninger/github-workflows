# GitHub Action: Shai-Hulud scan

Author: **Digdir Platform Team**

## Description

Scans for Indicators of Compromise (IOCs) related to the Shai-Hulud supply chain attack in NPM. The action installs dependencies with `--ignore-scripts` to prevent execution of potentially malicious install scripts, then computes SHA256 hashes of known targeted filenames in `node_modules` and compares them against a list of known malicious hashes.

If no `package.json` is found in the application path, the scan is skipped gracefully.

## Prerequisites

- The repository must be checked out before running this action.
- Node.js and the relevant package manager (`npm`, `yarn`, or `pnpm`) must be available on the runner.

## Inputs

| Input | Description | Required | Default |
| :---- | :---------- | :------- | :------ |
| `install_method` | Package manager to use: `npm`, `yarn`, `ci`, or `pnpm` | Yes | - |
| `application-path` | Path to the application directory containing `package.json` | No | `./` |

## Example usage

```yaml
- name: Run Shai-Hulud scan
  uses: felleslosninger/github-workflows/.github/actions/sh-scan@main
  with:
    install_method: npm
    application-path: ./apps/frontend
```

## How it works

1. Changes directory to `application-path`.
2. Checks for `package.json` — skips if not found (non-JS project).
3. Installs dependencies using the specified package manager with `--ignore-scripts`.
4. Searches `node_modules` for files matching known IOC filenames ( eks. `bundle.js`, `setup.mjs`, `Math_Symbol.js`, `setup_bun.js`, `bun_environment.js`,..).
5. Computes SHA256 hashes of any matches and compares against known malicious hashes.
6. Fails the step if any hash matches an IOC.
