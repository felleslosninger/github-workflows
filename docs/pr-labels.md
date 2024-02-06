# How to run workflow when PR is labeled

Create folder `.github/workflows/` folder in your application repo if not already created.

Add workflow file `.github/workflows/on-pr-label.yml` as described.

## Run workflow when PR is labeled
```
name: On PR label

on:
  pull_request:
    types: [labeled]

jobs:
  on-pr-label:
    uses: felleslosninger/github-workflows/.github/workflows/on-pr-label.yml@main
    
```

For more details see description in workflow [on-pr-label.yml](../.github/workflows/on-pr-label.yml).
