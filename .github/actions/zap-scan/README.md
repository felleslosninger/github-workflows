# GitHub Action: ZAP scan (full scan / API scan)

MORE DETAILS COMING.  
Author: **Digdir Platform Team**

## Description

ZAP (OWASP Zed Attack Proxy) is a Dynamic Application Security Testing (DAST) tool. These scans perform active testing and will send payloads designed to identify and exploit potential vulnerabilities in your application.

This action wraps two official ZAP GitHub Actions:

- **Full Scan**: Crawls and actively scans a web application URL.
- **API Scan**: Scans an API based on an OpenAPI/SOAP definition (file or URL), or scans a GraphQL endpoint URL.

These scans perform active testing, and will send payloads that try to exploit your application.

## Limitations

- We have only tested this action against public applications so far. To scan a application that is not reachable from the internet you will have to setup a self-hosted runner with network access to you application.

- Authentication is currently limited to authentication headers. Applications that rely on more complex authentication mechanisms (for example interactive login flows or advanced SSO) may not be supported. Support for script-based authentication may be added in the future if there is sufficient demand.

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

## Enviroment variables (Only needed if you have to authenticate to use the application you want to scan)

| Env | Description | Required |
| :--- | :--- | :--- |
| `auth_header_name` | If this is defined then its value will be used as the header name - if it is not defined then the standard Authorization header will be used. | false |
| `auth_header_value` | If this is defined then its value will be added as a header to all of the requests. | false |
| `auth_header_site` | Sites where we want to send authenticated requests. If not set, zap is allowed to send requests which include the auth_header_name and auth_header_value to any site. | false |

## Example Usage

In the repo of the application you want to test, create a new zap-workflow.yml in the .github/workflows/ folder, example: <https://github.com/felleslosninger/plattform-test-app/blob/main/.github/workflows/call-zap-devops-digdir-poc.yml>. Then configure the zap parameters and environment variables for the scan, see examples below. You can then call the workflow manually from the actions tab, or configure the workflow to run as a cron schedule. A report showing the results of the scan will be created as an artifact when the scan finishes.

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
      - uses: felleslosninger/github-workflows/.github/actions/zap-scan@main 
        with:
          target: "https://example-url.no"
          scan_api: "false"
          cmd_options: "-a"
          allow_issue_writing: "true"
          issue_title: "Zap-fullscan-report"
          artifact_name: "Zap-fullscan-summary"
        env:
          ZAP_AUTH_HEADER: Authorization
          ZAP_AUTH_HEADER_VALUE: "Bearer ${{ secrets.ZAP_TOKEN }}"
          ZAP_AUTH_HEADER_SITE: "example-url.no"

```

### Api scan

```yaml
name: ZAP api scan <appname>

on:
  workflow_dispatch:
  schedule:
    - cron: "0 2 15 * *" # Example of running the workflow every month on the 15th at 02:00 UTC. 

jobs:
  zap:
    runs-on: ubuntu-latest
    timeout-minutes: 20 # Arbitrary value, consider increasing it if needed. 
    steps:
      - uses: felleslosninger/github-workflows/.github/actions/zap-scan@main 
        with:
          target: "https://example-url.no/openapi.json"
          scan_api: "true"
          format: "openapi"
          cmd_options: "-a"
          allow_issue_writing: "true"
          issue_title: "Zap-apiscan-report"
          artifact_name: "Zap-apiscan-summary"
        env:
          ZAP_AUTH_HEADER: Authorization
          ZAP_AUTH_HEADER_VALUE: "Bearer ${{ secrets.ZAP_TOKEN }}"
          ZAP_AUTH_HEADER_SITE: "example-url.no"

```

## Documentation

- **Zap fullscan github action**: <https://github.com/marketplace/actions/zap-full-scan>
- **ZAP fullscan doc**: <https://www.zaproxy.org/docs/docker/full-scan/>
- **Zap api scan github action**: <https://github.com/marketplace/actions/zap-api-scan>
- **ZAP api scan doc**: <https://www.zaproxy.org/docs/docker/api-scan/>
