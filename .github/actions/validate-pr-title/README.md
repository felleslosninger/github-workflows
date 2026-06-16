# GitHub Action: Validate PR Title

Author: **Digdir Platform Team / Digdir Development Teams**

## Description

This composite action provides lightweight, bash-based validation for Pull
Request titles to enforce naming conventions and standards across repositories.
By enforcing some standards, it makes tracking work much easier (e.g. via Jira).

It supports:

- Automatic PR title detection directly from the GitHub event context
- Required prefix validation against a standard list (e.g., `ID-`, `Bump`,
  `MP-`)
- Minimum and maximum title length enforcement
- Optional case-sensitive or case-insensitive prefix matching
- Seamless fallback to golden-path defaults, requiring zero configuration for
  standard applications

## Inputs

| Input | Description | Required | Default |
| :---- | :---------- | :------- | :------ |
| `pull-request-title` | The title of the PR. Defaults to the actual PR title if omitted. | false | `""` |
| `allowed-prefixes` | Override allowed prefixes. Leave empty to use standard defaults (`Bump, ID-, MIN-, PBLEID-, MP-, KRR-, PF-, AOS-, SP-, EUW-, MOVE-`). | false | `""` |
| `min-length-title` | Minimum length of the title. Defaults to `10`. | false | `""` |
| `max-length-title` | Maximum length of the title (`-1` to disable). Defaults to `100`. | false | `""` |
| `case-sensitive-prefix` | Whether prefix validation is case-sensitive (`true` or `false`). Defaults to `false`. | false | `""` |

## Outputs

This action does not generate explicit workflow outputs. Instead, it natively
fails the workflow step (`exit 1`) and prints a standard GitHub Actions
`::error::` annotation if the PR title fails any validation checks.

## Example usage

This action is normally used via reusable workflows, and because of how it's
built, should not require any overrides to use sane defaults. In a reusable
workflow it is normally implemented as a separate job that runs in parallell
with other jobs

```yaml
  validate-pr-title:
    if: |
      inputs.enable-pr-title-verify == true &&
      github.event_name == 'pull_request'
    runs-on: ubuntu-latest
    steps:
      - name: Validate PR title
        uses: felleslosninger/github-workflows/.github/actions/validate-pr-title@main
        with:
          pull-request-title: ${{ inputs.pull-request-title }}
          allowed-prefixes: ${{ inputs.pull-request-allowed-prefixes }}
          min-length-title: ${{ inputs.pull-request-min-length-title }}
          max-length-title: ${{ inputs.pull-request-max-length-title }}
          case-sensitive-prefix: ${{ inputs.pull-request-case-sensitive-prefix }}
```

## How it works

The action relies on a fast, dependency-free bash script. It first checks if
specific input variables were provided. If inputs are left empty, it gracefully
falls back to defaults within the bash script to avoid duplicating inputs in
reusable workflows that uses the action. This means that the only place to
handle defaults are within the script itself.
