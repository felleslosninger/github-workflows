# GitHub Action: Execute Maven command

Author: **Digdir Platform Team**

## Description

This composite action safely evaluates and executes dynamic Maven arguments. It
handles the complex logic required for building mono-repos, scoping builds to
specific modules within a reactor, applying profiles, and injecting system
properties without duplicating verbose Bash scripts across workflows.

## Inputs

| Input | Description | Required | Default |
| :---- | :---------- | :------- | :------ |
| `goals` | The Maven lifecycle phases or plugins to execute (e.g., `clean install`, `deploy`, `spring-boot:build-image`). | `true` | |
| `application-path` | Path to the reactor root `pom.xml`. Used as the `--file` target. | false | `"./"` |
| `module-name` | Specific module directory to build (`-pl <module>`) for multi-module projects. | false | `""` |
| `build-dependencies` | Whether to build upstream reactor dependencies (`-am`). Automatically applied if a module is specified. | false | `"true"` |
| `maven-profile` | Specific profile to activate (`-P`). | false | `""` |
| `update-snapshots` | Force update of SNAPSHOT dependencies (`-U`). | false | `"false"` |
| `extra-args` | Additional raw arguments to append (e.g., `-DskipTests -DcustomProperty=value`). | false | `""` |

## Outputs

None.

## Example usage

```yaml
steps:
  - name: Build Spring Boot Image
    uses: felleslosninger/github-workflows/.github/actions/maven-run@main
    with:
      goals: install spring-boot:build-image
      application-path: my-app/
      module-name: api
      extra-args: >-
        -DskipTests=true
        -Dspring-boot.build-image.imageName="my-registry/my-app:latest"
```

## How it works

The action maps all inputs to environment variables and uses a Bash script to
iteratively construct a safe array of arguments (`ARGS=("-B" "--file"
"pom.xml")`).

It natively understands multi-module execution: if `module-name` is provided, it
automatically appends `-pl "$MODULE"` and (if enabled) `-am` so upstream reactor
dependencies are built in the correct order. Finally, it uses `eval` to cleanly
resolve and expand any complex string properties provided in `extra-args` before
executing the `mvn` command.
