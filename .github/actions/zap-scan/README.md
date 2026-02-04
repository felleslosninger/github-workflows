# GitHub Action: ZAP scan (full scan / API scan)

MORE DETAILS COMING.  
Author: **Digdir Platform Team**

## Description

ZAP (OWASP Zed Attack Proxy) is a Dynamic Application Security Testing (DAST) tool. The scans perform active testing, and will send payloads that try to exploit your application.

This action wraps two official ZAP GitHub Actions:

- **Full Scan**: Crawls and actively scans a web application URL.
- **API Scan**: Scans an API based on an OpenAPI/SOAP definition (file or URL), or scans a GraphQL endpoint URL.

These scans perform active testing, and will send payloads that try to exploit your application.

## Inputs

| Input | Description | Required |
| :--- | :--- | :--- |
| `target` | **Full scan:** URL of the web application to test. **API scan:** OpenAPI/SOAP/GraphQL definition (file path or URL). | true |
| `scan_api` | `true` to run API scan, `false` to run full scan | false |
| `format` | API spec format: `openapi`, `soap`, or `graphql` (only used when `scan_api=true`) | false |
| `cmd_options` | Additional command-line options passed to the underlying scan script | false |
| `rules_file_name` | Relative path to a rules file (e.g. `.zap/rules.tsv`) to ignore selected alerts. Requires `actions/checkout` if the file is in the repo. | false |
| `allow_issue_writing` | Set `true` to create/update a GitHub issue with results. | false |
| `issue_title` | Title of the issue when `allow_issue_writing=true`. | false |
| `token` | Token used for issue writing (use `secrets.GITHUB_TOKEN`), if not set then allow_issue_writing will not work. | false |
| `artifact_name` | Name of the workflow artifact containing the ZAP reports. | false |

## Example Usage

### Full scan (web application)

```yaml
name: ZAP full scan <appname>

on:
  workflow_dispatch:

jobs:
  zap:
    runs-on: ubuntu-latest
    timeout-minutes: 20 # Arbitrary value, consider increasing it if needed. 
    steps:
      - uses: felleslosninger/github-workflows/.github/actions/zap-scan@zap-scan
        with:
          target: "https://example-url.no"
          scan_api: "false"
          cmd_options: "-a"
          allow_issue_writing: "true"
          issue_title: "Zap-fullscan-report"
          artifact_name: "Zap-fullscan-summary"

```

### Api scan

```yaml
name: ZAP api scan <appname>

on:
  workflow_dispatch:

jobs:
  zap:
    runs-on: ubuntu-latest
    timeout-minutes: 20 # Arbitrary value, consider increasing it if needed. 
    steps:
      - uses: felleslosninger/github-workflows/.github/actions/zap-scan@zap-scan
        with:
          target: "https://example-url.no"
          scan_api: "true"
          cmd_options: "-a"
          allow_issue_writing: "true"
          issue_title: "Zap-apiscan-report"
          artifact_name: "Zap-apiscan-summary"

```

## Documentation

- **Zap fullscan github action**: <https://github.com/marketplace/actions/zap-full-scan>
- **ZAP fullscan doc**: <https://www.zaproxy.org/docs/docker/full-scan/>
- **Zap api scan github action**: <https://github.com/marketplace/actions/zap-api-scan>
- **ZAP api scan doc**: <https://www.zaproxy.org/docs/docker/api-scan/>
