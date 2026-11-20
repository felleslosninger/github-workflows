# GitHub Action: Setup Maven environment

Author: **Digdir Platform Team**

## Description

This composite action standardizes the Java and Maven environment for workflows.
It sets up the JDK, configures Maven dependency caching, dynamically injects
credentials for private artifact resolution (like GitHub Packages).

## Inputs

| Input | Description | Required | Default |
| :---- | :---------- | :------- | :------ |
| `java-version` | Java version to set up. | false | `"25"` |
| `java-distribution` | Java distribution to use. | false | `"liberica"` |
| `cache-path` | `pom.xml` file(s) used to compute the Maven cache key. The cache is invalidated when these change. The cache itself is stored in `~/.m2/repository`. | false | `"**/pom.xml"` |
| `server-username` | Username for the Maven server (typically GitHub Packages read access). | `true` | |
| `server-password` | Password/PAT for the Maven server. | `true` | |

## Outputs

None.

## Example usage

```yaml
steps:
  - name: Setup Environment
    uses: felleslosninger/github-workflows/.github/actions/setup-maven-env@main
    with:
      java-version: "25"
      server-username: ${{ secrets.GH_PACKAGES_READ_USER }}
      server-password: ${{ secrets.GH_PACKAGES_READ_PAT }}
```

## How it works

The action delegates JDK installation to `actions/setup-java`. Crucially, it
configures `setup-java` to write placeholder environment variables
(`${env.MAVEN_SERVER_USERNAME}`) into `~/.m2/settings.xml` instead of hardcoding
credentials.

It then exports the provided `server-username` and `server-password` to the
global `$GITHUB_ENV`. This makes the read-only credentials the default for all
subsequent `mvn` steps in the job, while safely allowing deployment steps to
temporarily override them with write credentials (e.g., `github.token`) using
step-level `env` blocks.

## Server ids

Maven picks credentials by matching a pom's `<repository><id>` against a
`<server><id>` in `settings.xml`. The action writes one server per GitHub org,
all using the same credentials:

| Server id | Use it for |
| :-------- | :--------- |
| `github` | Packages in the `felleslosninger` org |
| `github-eudi-wallet-no` | Packages in the `eudi-wallet-no` org |

```xml
<repository>
  <id>github-eudi-wallet-no</id>
  <url>https://maven.pkg.github.com/eudi-wallet-no/eudiw-bevisporten</url>
</repository>
```

To support another org, add a `github-<org>` line to `mvn-server-credentials`
in `action.yml`. Servers no pom references are ignored by Maven.

## Caching

Maven dependencies are cached in `~/.m2/repository` using the built-in cache in
[`actions/setup-java`](https://github.com/actions/setup-java#caching).

`cache-path` does **not** control where the cache is stored. It points to the
`pom.xml` file(s) used to compute the cache key (passed to
`cache-dependency-path`). When any of these files change, a new cache is
created; otherwise the existing cache is reused.
