# GitHub Action: Write JSON to step summary

Author: **Digdir Platform Team**

## Description

This GitHub Action is designed to write JSON content to the step summary. It
enables you to present structured data in a clear and readable format in the
GitHub Actions UI. The action accepts a JSON input and an optional title.

## Prerequisites

None

## Inputs

| Input | Description | Required |
| :--- | :--- | :--- |
| `TBA` | TBA | true |

## Example usage

```yaml
TBA
```

## How it works

This action uses a composite run to execute a Bash script. The script
dynamically constructs a Markdown-formatted section in the step summary,
including the specified title and JSON content.
