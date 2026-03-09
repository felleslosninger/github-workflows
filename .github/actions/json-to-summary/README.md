# GitHub Action: Write JSON to step summary

Author: **Digdir Platform Team**

## Description

This GitHub Action is designed to write JSON content to the step summary. It
enables you to present structured data in a clear and readable format in the
GitHub Actions UI. The action accepts a JSON input and an optional title. It
also includes a best-effort check to avoid showing secret input values in the
table.

## Prerequisites

None

## Inputs

| Input | Description | Required | Default |
| :--- | :--- | :--- |
| `title` | The Markdown heading for the step summary table | false | `Inputs` |
| `json-payload` | JSON payload to parse and write to the step summary | true | N/A |

## Example usage

```yaml
name: Some workflow where you want to log inputs

jobs:
  some job:
    runs-on: ubuntu-latest

    steps:
    - name: Write inputs to summary
      uses: felleslosninger/github-workflows/.github/actions/json-to-summary@main
      with:
        json-payload: ${{ toJson(inputs) }}
```

## How it works

This action uses a composite run to execute a Bash script. The script
dynamically constructs a Markdown-table in the step summary, including the
optional title and JSON content.
